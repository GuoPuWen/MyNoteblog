## TDSQL架构

架构图

![image-20210611170627865](/Users/guopuwen/Library/Application Support/typora-user-images/image-20210611170627865.png)

- LVS：负载均衡模块，负责处理用户的请求
- proxy：网关模块，可以将SQL请求路由到后端set节点，同时也支持读写分离，也可以对SQL语句进行分析
- mysql：存储节点
- agent：可以检测mysql实例的存活情况，将情况上报至zk集群，可以监控是否需要扩容，以及容灾切换，主备复制
- scheduler：调度模块，管理set集群，所有的DDL操作统一下发和调度
- collection：数据采集，采集日志数据到前台页面
- 前台：web界面，可以友好的去管理整个TDSQL的后台
- OSS模块：提供一个统一的接口，用来接收赤兔管理台的请求，例如在赤兔上发送一个https请求，通过OSS模块提交到zk上，然后执行请求如果请求完成，再通过zk相应回前端，有了OSS模块，可以将前台web页面和后端TDSQL整个绑定在一起
- hadoop：用来做一些数据分析或者冷备，对一些重要的日志

## 分布式事务问题

TDSQL分布式事务是基于XA的两阶段提交的方式，两阶段提交步骤如下：

- 第一阶段：事务管理器向所有本地资源管理器发起请求，询问是否是ready状态
- 第二阶段：事务管理器根据本地资源管理器的反馈，通知所有本地资源管理器，并且步调一致的进行回滚或者提交

TDSQL分布式事务的架构图

![image-20210609111826660](/Users/guopuwen/Library/Application Support/typora-user-images/image-20210609111826660.png)

- GW：网关。网关可以有两种模式工作：noshard：此模式下网关基本不处理SQL语句，直接将用户请求的SQL语句发送给后断的DB；group shard：网关会解析SQL语句，并且把拆分后的SQL语句发送给后端的DB
- set：后端的mysql集群一般是一主两备的配置，称为一个set

在group shard模式下，用户在创建表的时候，需要指定一个列做shardkey，用于后续做set映射，对于多行插入，如果网关解析SQL语句时发现该多行插入SQL语句分别对应不同的set，则网关会重新组装sql语句，然后分别发送到对应的set；

对于update/delete语句如果指定了shard key，则将该语句发送到对应的set，否则发送到所有set；

对于select语句，同样的根据shard key判断是否要发送到对应的set，否则做将该语句广播到所有set，同时网关需要收集每个set的结果到客户端



TDSQL的底采用的是mysql数据库，mysql数据库已经是支持分布式事务：

```sql
XA START 'any_unique_id'; // 'any_unique_id' 是用户给的，全局唯一在一台mysql中开启一个XA事务
XA END 'any_unique_id '; //标识XA事务的操作结束
XA PREPARE 'any_unique_id'; //告知mysql 准备提交这个xa事务
XA COMMIT 'any_unique_id'; //告知mysql提交这个xa事务
XA ROLLBACK 'any_unique_id'; //告知mysql回滚这个xa事务
XA RECOVER;//查看本机mysql目前有哪些xa事务处于prepare状态
```

所以我觉得对于分布式事务处理的重点在于网关上，网关作为TM事务协调器，协调器要做的工作是维护好每一个全局事务的状态。对于网关来说需要判断当前是一阶段提交（shard 模式下可以指定shard key，那么就可以发送到指定的set，所以这个时候就没有分布式事务这一说，因为只有一个set），还是两阶段提交。所以网关发送给后端的事务启动语句总是以xa start开始，对于一阶段提交不发送xa commit而是发送xa commit one phase，只有在网关写入了多个set的时候，才进行两阶段提交，这是对只进行一阶段提交的优化，对于只读set的提交也是只进行一阶段提交

TDSQL的死锁处理：

- 单一MySQL死锁，MySQL内部具备死锁检测，有MySQL返回死锁错误，网关发送rollback GT
- 全局死锁：当两个全局事务分别在set1和set2上更新R1和R2造成死锁等待，解决办法是，网关定时从每个后端set上查询innodb_trx和innode_lock_waits表，得到环路图接着kill掉一个db连接，返回错误

## 强同步

`MySql的主从复制模式`：Master端存在一个IO线程，将binlog日志发送发从Slave端，Slave端存在两个线程，I/O线程请求Master库的binlog日志，同时将得到的binlog日志写入到本地的relay log日志文件中，SQL线程会读取relay log文件中的日志，解析成SQL语句逐一执行。这种模式是属于异步模式，即Master不管Slave的备份进度，如果Slave库进度落后，且Master库宕机，就会数据丢失



`半同步复制`：大体过程和主从复制相同，不过在Master写数据到binlog日志的时候会进行等待至少有一个从库接收到并写入realy log日志中，才进行整个事务的提交并返回给客户端。存在问题：1⃣️有一定的延迟，至少是一个数据包的往返时间RTT

下面是半同步复制的流程图

![image-20210610101629498](/Users/guopuwen/Library/Application Support/typora-user-images/image-20210610101629498.png)

2⃣️ 如果Master在等待Slave确认的时候，宕机了，客户端收到事务提交失败，但是在原本的主库上已经commit了事务，当这个宕机的机器重新启动后，以从库的身份，就会提交两次



`Loss-Less无损半同步复制`：将上述流程图中步骤“Waiting Slave dump”调整到“Storage Commit”之前



`TDSQL强同步`：在无损半同步复制的基础上，增加线程池+业务线程异步，在TDSQL上会开启两个业务线程，一是接受ack应答线程，二是唤醒hang住的客户端连接线程。当连续多个请求到达时，上一个请求commit写入binlog之后，就此打住在内存中存储本次会话的信息，接着处理其他请求，当ack应答线程收到ack确认之后去主动唤醒保存的会话，接着才对客户端返回commit信息