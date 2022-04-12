# redo log

### **redo log基本介绍**

redo log是Innodb存储引擎独有的日志，工作在引擎层（binlog是在serve层产生的，无论是各种存储引擎都会产生）的日志。redo log的作用是保证数据的完整性，当数据库由于某种原因宕机了，Innodb存储引擎可以使用redo log恢复到宕机之前的时刻，所以redo log对Innodb存储引擎有着非常大的作用

redo log采用了MySQL的WAL机制，也就是说一个事务发生之后，日志优先磁盘执行，Innodb会先将数据页的变动写入redo log中，而不是实际的数据文件中，一旦 `redolog` 写入完成，就认为这个事务的操作记录完成了。之后mysql会有一套更新机制，定期的将 `redolog` 中的内容写入到数据文件中。

> WAL机制：WAL机制是数据库系统提供原子性和持久性的一些列技术，在使用WAL的系统中，所有的修改都先被写入到日志中，然后再被应用到系统状态中。通常包含redo和undo两部分信息。

在MySQL中，redo log一般用来做持久性操作，无论mysql有没有发生异常，重新启动的时候，mysql都会通过 `redolog` 恢复，确保数据没有问题。

### redo log的写入机制

对于redo log来说，生成的事务时先需要写在redo log buffer，然后在写在redo log里面的，这样做的目的是为了减少磁盘IO，正常来说，redo log的写入流程为下图

<img src="http://cdn.noteblogs.cn/image-20210801161208046.png" alt="image-20210801161208046" style="zoom: 50%;" />

- 首先写入redo log buffer
- redo log buffer属于用户空间的缓存，无法直接写入磁盘上，因此需要写入os buffer中，操作系统内核空间
- 持久化到磁盘上

在速度上，可以说从写入到redo log buffer，write到os buffer两者的速度差距是不大的，只是多了系统调用，但是持久化磁盘中的速度就比较慢了

也就是说对应着三个过程，redo log一共存在在三个地方redo log buffer、os buffer和磁盘上的文件中，为了控制redo log的写入过程，Innodb提供了一个参数`innodb_flush_log_at_trx_commit	`来控制写入过程：

- 设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
- 设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘
- 设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 os buffer

同时，Innodb有一个后台线程，每隔1秒就会将redo log buffer中的日志，写入到os buffer中同时持久化到磁盘中。

这里可以抛出一个疑问，是不是每次写入到redo log buffer中的数据都要持久化到磁盘呢？

并不是的，如果在MySQL宕机期间执行了一个事务，这个事务也写入到了redo log buffer中，但是由于事务没有提交所以这部分的数据丢失没有关系

但是也会存在事务在没有提交的时候，redo log buffer中的部分日志已经持久化到磁盘里了，例如：

- 当redo log buffer中的数据大小即将占用`innodb_log_buffer_log`的一半的时候，后台线程会主动写盘，将数据持久化到磁盘里面
- 当存在并行事务的时候，一个A事务执行到了一半，并且将数据写入到了redo log buffer中，这时有另外一个并行事务提交，如果将`innodb_flush_log_at_trx_commit	`设置为1的情况下，B事务的提交也会将A事务存在于redo log buffer中的数据一并持久化到磁盘中

这个问题也涉及到redo log和binlog之间的两阶段提交，这在后面讲MySQL如何保证数据的一致性的时候会提到。

上面的一个写入过程是一个整体的流程，就是redo log首先要先写入redo log buffer -> os buffer -> ib_logfileN，可以注意到的是 ib_logfileN，也就说redo log持久化不仅只写入到一个文件 内，而是写入到多个文件内，如图

<img src="http://cdn.noteblogs.cn/image-20210801164450382.png" alt="image-20210801164450382" style="zoom:50%;" />

Innodb采用循环写入的方式，redo log有一个重做日志文件组，组里面有多个文件，使用了两个指针write pos、check point代表着两个指针之间是可以写入的，通过移动两个指针来进行写入，这里不展开介绍

对于redo log的存储格式可以参考 [MySQL · 源码分析 · Innodb 引擎Redo日志存储格式简介](http://mysql.taobao.org/monthly/2017/09/07/)

# binlog

### binlog基本介绍

binlog是在MySQL serve层的日志，记录所有数据库表结构变更以及表修改的二进制日志，是以事件形式记录，还包含了语句所执行的消耗时间，binlog日志有以下两个非常重要的场景：

- 主从复制：在主库中开启binlog功能，这样主库就可以把binlog传递给从库，从库拿到binlog后实现数据恢复达到主从数据一致性
- 数据恢复：可以通过mysqlbinlog工具来恢复数据

binlog文件记录模式一般有三种：STATMENT、ROW和MIXERD：

- ROW：仅保存记录被修改细节，不记录sql语句上下文相关的信息。优点是：能清楚的知道每一行数据的修改细节，能完全实现主从数据同步和数据的恢复，缺点是：批量操作会产生大量的日志
- STATMENT：每一条被修改数据的SQL都会记录到master的binlog中，优点是日志量少，减少磁盘IO，提升存储和恢复速度，缺点是在某些情况下导致主从数据不一致，在使用了一些系统函数的时候不能准确复制或不能复制
- MIXED：以上两种模式的混合使用，一般会使用STATMENT保存binlog，在遇到STATMENT模式无法复制的操作再使用ROW模式保存binlog

在生产环境下，一般建议使用row的格式，使用statment格式会造成数据丢失，使用mixed混合模式在某些情况下仍然会造成存储数据不一致的问题，具体可以看 [为什么MySQL binlog_format一定要设置成row](https://hzkeung.com/2017/05/28/why-mysql-binlog_format-use-row)

### binlog的写入机制

binlog的写入机制与redo log的很相似，在事务执行的过程中，先把日志写入到binlog cache，事务提交的时候再把binlog 写入到binlog文件中，一共两个过程：

- write：将日志写入文件系统的os buffer中，这个阶段没有将数据写入磁盘中，只是系统调用将数据用用户缓冲区写入内核缓存区
- fsync：将数据持久化到磁盘的操作

sync_binlog参数控制着write和fsync的时机：

- sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
- sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
- sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

所以，为了提高IO性能的瓶颈，可以将sync_binlog设置为一个比较大的值，可以提升性能，但是对应的风险就是如果主机发生异常重启，会丢失最近N个事务的binlog日志

