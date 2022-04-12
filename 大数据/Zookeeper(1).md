# 一、Zookeeper简介

是一个分布式服务框架，是Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。



# 二、Zookeeper应用场景

### 2.1 维护配置信息  

java编程经常会遇到配置项， 比如数据库的url、 schema、 user和password等。 通常这些配置项我们会放置在配置文件中， 再将配置文件放置在服务器上当需要更改配置项时， 需要去服务器上修改对应的配置文件。 但是随着分布式系统的兴起， 由于许多服务都需要使用到该配置文件， 因此有必须保证该配置服务的高可用性（highavailability） 和各台服务器上配置数据的一致性。 通常会将配置文件部署在一个集群上，然而一个集群动辄上千台服务器， 此时如果再一台台服务器逐个修改配置文件那将是非常繁琐且危险的的操作， 因此就需要一种服务， 能够高效快速且可靠地完成配置项的更改等操作， 并能够保证各配置项在每台服务器上的数据一致性。
 zookeeper就可以提供这样一种服务， 其使用Zab这种一致性协议来保证一致性。 现在有很多开源项目使用zookeeper来维护配置， 比如在hbase中， 客户端就是连接一个zookeeper， 获得必要的hbase集群的配置信息， 然后才可以进一步操作。 还有在开源的消息队列kafka中， 也使用zookeeper来维护broker的信息。 在alibaba开源的soa框架dubbo中也广泛的使用zookeeper管理一些配置来实现服务治理。  

### 2.2 分布式锁服务  

一个集群是一个分布式系统， 由多台服务器组成。 为了提高并发度和可靠性，多台服务器上运行着同一种服务。 当多个服务在运行时就需要协调各服务的进度， 有时候需要保证当某个服务在进行某个操作时， 其他的服务都不能进行该操作， 即对该操作进行加锁， 如果当前机器挂掉后， 释放锁并fail over 到其他的机器继续执行该服务。  

### 2.3 集群管理  

一个集群有时会因为各种软硬件故障或者网络故障， 出现某些服务器挂掉而被移除集群， 而某些服务器加入到集群中的情况， zookeeper会将这些服务器加入/移出的情况通知给集群中的其他正常工作的服务器， 以及时调整存储和计算等任务的分配和执行等。 此外zookeeper还会对故障的服务器做出诊断并尝试修复。  

### 2.4 生成分布式唯一ID  

在过去的单库单表型系统中， 通常可以使用数据库字段自带的auto_increment属性来自动为每条记录生成一个唯一的ID。 但是分库分表后， 就无法在依靠数据库的auto_increment属性来唯一标识一条记录了。 此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。 做法如下： 每次要生成一个新Id时， 创建一个持久顺序节点， 创建操作返回的节点序号， 即为新Id， 然后把比自己节点小的删除即可  

# 三、Zookeper的数据模型

### 3.1 Znode

zookeeper的数据节点可以视为树状结构（或者目录） ， 树中的各节点被称为znode（即zookeeper node） ， 一个znode可以有多个子节点。 zookeeper节点在结构上表现为树状； 使用路径path来定位某个znode。

 znode， 兼具文件和目录两种特点。 既像文件一样维护着数据、 元信息、 ACL、 时间戳等数据结构， 又像目录一样可以作为路径标识的一部分  

![image-20201216192138964](Zookeeper(1).assets/image-20201216192138964.png)

一个Znode大体上有3个部分

- 节点的数据： 即znode data(节点path, 节点data)的关系就像是java map中(key,value)的关系  
- 节点的子节点children  
- 节点的状态stat： 用来描述当前节点的创建、 修改记录， 包括cZxid、 ctime等  

### 3.2 节点类型

zookeeper中的节点有两种， 分别为临时节点和永久节点。 节点的类型在创建时即被确定， 并且不能改变。

- 临时节点： 该节点的生命周期依赖于创建它们的会话。 一旦会话(Session)结束， 临
  时节点将被自动删除， 当然可以也可以手动删除。 虽然每个临时的Znode都会绑定到
  一个客户端会话， 但他们对所有的客户端还是可见的。 另外， ZooKeeper的临时节
  点不允许拥有子节点。
- 持久化节点： 该节点的生命周期不依赖于会话， 并且只有在客户端显示执行删除操作
  的时候， 他们才能被删除  

# 四、集群安装Zookeeper

### 4.1下载

[zookeeper下载地址](https://zookeeper.apache.org/releases.html#download)

### 4.2配置

|    服务器IP    |  主机名   | myid的值 |
| :------------: | :-------: | :------: |
| 192.168.18.101 | hadoop101 |    1     |
| 192.168.18.102 | hadoop102 |    2     |
| 192.168.18.103 | hadoop103 |    3     |

因为zookeeper的安装是需要jdk的环境的，这里不说jdk的安装。配置zookeeper非常简单，只需要更改一下存储zookeeper中数据的内存快照、 及事物日志文件  

```shell
cd /opt/module/zookeeper-3.4.9/conf
cp zoo_sample.cfg zoo.cf
vim zoo.cfg

# 存储zookeeper中数据的内存快照、 及事物日志文件  
dataDir=/opt/module/zookeeper-3.4.9/zkdatas
# 保留多少个快照
autopurge.snapRetainCount=3
# 日志多少小时清理一次
autopurge.purgeInterval=1
# 集群中服务器地址
server.1=hadoop101:2888:3888
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
```

### 4.3 添加myid配置

myid相当于每一台机器的标识，后面会详细讲到

在/opt/module/zookeeper-3.4.9/zkdatas/下创建文件名为myid，内容为1

```shell
echo 1 > /opt/module/zookeeper-3.4.9/zkdatas/myid
```

然后分发到各个机器上，注意xsync是自己写的脚本，这个在hadoop那提到过，最后修改每台机器上的myid文件内容

### 4.3 启动和停止

```shell
cd /opt/module/zookeeper-3.4.9/bin
#启动
./zkServer.sh start

#停止
./zkServer.sh stop
#查看状态
./zkServer.sh status
```

注意：zookeeper不像hadoop一样，zookeeper是不能群起的，所以对每台机器都必须启动一下

### 4.4 远程登录zookeeper

注意端口号一般为2181

```shell
./zkCli.sh -server ip
```

### 4.5 启动脚本

因为集群启动需要一个一个机器启动，所以自己写了一个脚本

```shell
#!/bin/sh
params=$1
if [ "$params" = "start" ]
then
        for (( i=1 ; i <= 3 ; i = $i + 1 )) ;
         do
                echo ============= hadoop10$i $params =============
                 ssh root@192.168.18.10$i "/opt/module/zookeeper-3.4.9/bin/zkServer.sh start"
        done
fi
if [ "$params" = "stop" ]
then
        for (( i=1 ; i <= 3 ; i = $i + 1 )) ;
         do
                echo ============= hadoop10$i $params =============
                 ssh root@192.168.18.10$i "/opt/module/zookeeper-3.4.9/bin/zkServer.sh stop"
        done
fi
if [ "$params" = "status" ]
then
        for (( i=1 ; i <= 3 ; i = $i + 1 )) ;
         do
                echo ============= hadoop10$i $params =============
                 ssh root@192.168.18.10$i "/opt/module/zookeeper-3.4.9/bin/zkServer.sh status"
        done
fi
```

将该脚本放到/usr/local/bin/zk.sh就可以使用，一键启动脚本



# 五、常用shell操作

### 5.1 新增节点

```shell
create [-s] [-e] path data #其中-s 为有序节点， -e 临时节点
```

### 5.2 更新节点

```shell
set /hadoop "345"
```

### 5.3 删除节点

```shell
delete path
```

想删除某个节点及其所有后代节点， 可以使用递归删除， 命令为 rmr path  

### 5.4 查看节点stat数据结构

```shell
get path
```

![image-20201216203119816](Zookeeper(1).assets/image-20201216203119816.png)

节点各个属性如下表。 其中一个重要的概念是 Zxid(ZooKeeper Transaction Id)， ZooKeeper 节点的每一次更改都具有唯一的 Zxid， 如果 Zxid1 小于 Zxid2， 则Zxid1 的更改发生在 Zxid2 更改之前。  

|    状态属性    |                             说明                             |
| :------------: | :----------------------------------------------------------: |
|     cZxid      |                   数据节点创建时的事务 ID                    |
|     ctime      |                     数据节点创建时的时间                     |
|     mZxid      |               数据节点最后一次更新时的事务 ID                |
|     mtime      |                 数据节点最后一次更新时的时间                 |
|     pZxid      |          数据节点的子节点最后一次被修改时的事务 ID           |
|    cversion    |                       子节点的更改次数                       |
|  dataVersion   |                      节点数据的更改次数                      |
|   aclVersion   |                    节点的 ACL 的更改次数                     |
| ephemeralOwner | 如果节点是临时节点， 则表示创建该节点的会话的 SessionID； 如果节点是持久节点， 则该属性值为 0 |
|   dataLength   |                        数据内容的长度                        |
|  numChildren   |                   数据节点当前的子节点个数                   |

### 5.5 查看节点列表

![image-20201216203412038](Zookeeper(1).assets/image-20201216203412038.png)

```shell
ls /
```

### 5.6 监听器

使用 get path [watch] 注册的监听器能够在节点内容发生改变的时候， 向客户端发出通知。 需要注意的是 zookeeper 的触发器是一次性的 (One-time trigger)， 即触发一次后就会立即失效 。

```shell
get /hadoop watch
```

当使用另外一个客户端修改/hadoop的值得时候，watch将发出通知

![image-20201216204006985](Zookeeper(1).assets/image-20201216204006985.png)



# 六、Zookeper权限控制ACL

zookeeper的文件系统类似于linux，client 可以创建节点、 更新节点、 删除节点， 那么如何做到节点的权限的控制呢？ zookeeper的access control list 访问控制列表可以做到这一点。  

zookeeper中权限控制是通过访问控制列表来实现的，访问控制列表格式为：

```
权限模式(schemem):授权对象(id):权限(permission)
```

- zooKeeper的权限控制是基于每个znode节点的， 需要对每个节点设置权限
- 每个znode支持设置多种权限控制方案和多个权限
- 子节点不会继承父节点的权限， 客户端无权访问某节点， 但可能可以访问它的子节点  

### 6.1 schemem

权限模式：采用何种方式授权  

|  方案  |                          描述                          |
| :----: | :----------------------------------------------------: |
| world  | 只有一个用户： anyone， 代表登录zokeeper所有人（默认） |
|   ip   |                 对客户端使用IP地址认证                 |
|  auth  |                使用已添加认证的用户认证                |
| digest |               使用“用户名:密码”方式认证                |

### 6.2 id

授权对象ID是指， 权限赋予的实体， 例如： IP 地址或用户  

### 6.3 permission

授予什么权限  

|  权限  | ACL简写 |               描述               |
| :----: | :-----: | :------------------------------: |
| create |    c    |          可以创建子节点          |
| delete |    d    |  可以删除子节点（仅下一级节点）  |
|  read  |    r    | 可以读取节点数据及显示子节点列表 |
| write  |    w    |         可以设置节点数据         |
| admin  |    a    |   可以设置节点访问控制列表权限   |

### 6.4 授权的命令

|  命令   | 使用方式 |     描述     |
| :-----: | :------: | :----------: |
| getAcl  |  getAcl  | 读取ACL权限  |
| setAcl  |  setAcl  | 设置ACL权限  |
| addauth | addauth  | 添加认证用户 |

### 6.5 具体操作

##### 6.5.1 world 模式

```shell
[zk: localhost:2181(CONNECTED) 5] create /node1 "node"
#赋予/node1节点不能创建子节点的权限
[zk: localhost:2181(CONNECTED) 8] setAcl /node1 world:anyone:drwa
#试图创建子节点
[zk: localhost:2181(CONNECTED) 11] create /node1/node11 "node11"
Authentication is not valid : /node1/node11

[zk: localhost:2181(CONNECTED) 12] getAcl /node1
'world,'anyone
: drwa
```

##### 6.5.2 ip模式

```shell
[zk: localhost:2181(CONNECTED) 13] create /hive "hive"
#设置只有ip为192.168.18.102的主机才能访问/hive节点
[zk: localhost:2181(CONNECTED) 14] setAcl /hive ip:192.168.18.102:cdrwa
#当前节点已经不能访问
[zk: localhost:2181(CONNECTED) 15] get /hive
Authentication is not valid : /hive
[zk: localhost:2181(CONNECTED) 16] getAcl /hive
'ip,'192.168.18.102
: cdrwa
```

##### 6.5.3 Auth模式

```shell
[zk: localhost:2181(CONNECTED) 0] addauth digest  zookeeper1:12345
[zk: localhost:2181(CONNECTED) 1] create /node3 "node1"
Created /node3
[zk: localhost:2181(CONNECTED) 2] setAcl /node3 auth:zookeeper1:cdrwa
```

##### 6.5.4 digest模式

```shell
setAcl <path> digest:<user>:<password>:<acl>
```

这里的密码是通过了SHA1及BASE64处理的密文

```shell
#先计算出密文
[root@hadoop101 ~]# echo -n user1:12345 | openssl dgst -binary -sha1 | openssl base64
+owfoSBn/am19roBPzR1/MfCblE=
```

digest授权

```shell
#创建节点
[zk: localhost:2181(CONNECTED) 16] create /node5 "node5"
Created /node5
#设置权限使用digest
[zk: localhost:2181(CONNECTED) 17] setAcl /node5 digest:user1:+owfoSBn/am19roBPzR1/MfCblE=:crdwa
#尝试获取节点
[zk: localhost:2181(CONNECTED) 0] get /node5
Authentication is not valid : /node5
#添加认证用户
[zk: localhost:2181(CONNECTED) 1] addauth digest user1:12345
#获取节点成功
[zk: localhost:2181(CONNECTED) 2] get /node5
node5
cZxid = 0x40000001e
ctime = Thu Dec 17 20:26:31 CST 2020
mZxid = 0x40000001e
mtime = Thu Dec 17 20:26:31 CST 2020
pZxid = 0x40000001e
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

##### 6.5.6 super模式  

zookeeper的权限管理模式有一种叫做super， 该模式提供一个超管可以方便的访问
任何权限的节点。

假设这个超管是： user2:admin， 需要先为超管生成密码的密文  











# 六、Zookeeper的Java Api操作

- 连接到zookeeper服务器。 zookeeper服务器为客户端分配会话ID。
- 定期向服务器发送心跳。 否则， zookeeper服务器将过期会话ID， 客户端需要重新连
  接。
- 只要会话ID处于活动状态， 就可以获取/设置znode。
- 所有任务完成后， 断开与zookeeper服务器的连接。 如果客户端长时间不活动， 则
  zookeeper服务器将自动断开客户端。  



### 6.1 maven坐标

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>4.13.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.16</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.6.1</version>
</dependency>
```



### 6.2 连接Zookeeper

```java
public class ZookeeperConnection {
    public static void main(String[] args) throws IOException {
        final CountDownLatch countDownLatch = new CountDownLatch(1);
        String ip = "192.168.18.101:2181";
        ZooKeeper zooKeeper = new ZooKeeper(ip, 5000, new Watcher() {
            public void process(WatchedEvent watchedEvent) {
                if(watchedEvent.getState() == Event.KeeperState.SyncConnected){
                    System.out.println("链接创建成功！");
                    countDownLatch.countDown();
                }
            }
        });
        try {
            countDownLatch.await();
            System.out.println(zooKeeper.getSessionId());
            zooKeeper.close();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

![image-20201217184328776](Zookeeper(1).assets/image-20201217184328776.png)

ZooKeeper构造函数的参数：

- connectionString - zookeeper主机(注意端口2181)
- sessionTimeout - 会话超时（以毫秒为单位)
- watcher - 实现“监视器”对象。 zookeeper集合通过监视器对象返回连接状态。  

### 6.3 新增节点

```java
// 同步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode)
// 异步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode，
AsyncCallback.StringCallback callBack,Object ctx)  
```

- path：znode路径。 例如， /node1 /node1/node11
- data：要存储在指定znode路径中的数据
- acl：要创建的节点的访问控制列表。 zookeeper API提供了一个静态接口：ZooDefs.Ids 来获取一些基本的acl列表。 例如， ZooDefs.Ids.OPEN_ACL_UNSAFE返回打开znode的acl列表。
- createMode：节点的类型,这是一个枚举。
- callBack：异步回调接口
- ctx：传递上下文参数  

### 6.4 更新节点

```java
// 同步方式
setData(String path, byte[] data, int version)
// 异步方式
setData(String path, byte[] data, int version， AsyncCallback.StatCallback
callBack， Object ctx)
```

- path：znode路径
- data：要存储在指定znode路径中的数据。
- version：znode的当前版本。 每当数据更改时， ZooKeeper会更新znode的版本号。-1代表版本号不作为修改条件
- callBack：异步回调接口
- ctx：传递上下文参数  

### 6.4 删除节点

```java
// 同步方式
delete(String path, int version)
// 异步方式
delete(String path, int version, AsyncCallback.VoidCallback callBack,
Object ctx)
```

- path ：znode路径。
- version：znode的当前版本
- callBack：异步回调接口
- ctx：传递上下文参数  

### 6.5 查看节点

```java
// 同步方式
getData(String path, boolean b, Stat stat)
// 异步方式
getData(String path, boolean b， AsyncCallback.DataCallback callBack，
Object ctx)
```

- path：znode路径。
- b：是否使用连接对象中注册的监视器。
- stat ：返回znode的元数据。
- callBack：异步回调接口
- ctx：传递上下文参数  