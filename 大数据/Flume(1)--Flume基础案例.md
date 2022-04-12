# 一、概述

Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的==海量日志采集==、聚合和传
输的系统。 Flume 基于流式架构，灵活简单。  

# 二、Flume架构

![image-20201206191004385](C:\Users\VSUS\Desktop\笔记\大数据\img\34.png)

上图是 [Flume1.7官方文档](http://flume.apache.org/releases/content/1.7.0/FlumeUserGuide.html#hdfs-sink)中的架构图，可以看到Flume由3个组件组成：Source、Channel、Sink，由Agent这个JVM进程进行管管理。

外部数据源以特定格式向 Flume 发送 events (事件)，当 source 接收到 events 时，它将其存储到
一个或多个 channel ， channe 会一直保存 events 直到它被 sink 所消费。 sink 的主要功能从
channel 中读取 events ，并将其存入外部存储系统或转发到下一个 source ，成功后再从 channel
中移除 。

- Agent

Agent 是一个 JVM 进程，它以事件的形式将数据从源头送至目的。Agent 主要有 3 个部分组成， Source、 Channel、 Sink。  

- Source

Source 是负责接收数据到 Flume Agent 的组件。 Source 组件可以处理各种类型、各种
格式的日志数据，包括 ==avro==、 thrift、==exec==、 jms、 ==spooling directory==、 netcat、 sequence
generator、 syslog、 http、 legacy。  

- Sink

Sink 不断地轮询 Channel 中的事件且批量地移除它们，并将这些事件批量写入到存储
或索引系统、或者被发送到另一个 Flume Agent。
Sink 组件目的地包括 ==hdfs==、 ==logger==、 ==avro==、 thrift、 ipc、 ==file==、 ==HBase==、 solr、自定
义。  

- Channel

Channel 是位于 Source 和 Sink 之间的缓冲区。因此， Channel 允许 Source 和 Sink 运
作在不同的速率上。 Channel 是线程安全的，可以同时处理几个 Source 的写入操作和几个
Sink 的读取操作。
        Flume 自带两种 Channel： Memory Channel 和 File Channel 以及 Kafka Channel。

1. Memory Channel 是内存中的队列。 Memory Channel 在不需要关心数据丢失的情景下适
   用。如果需要关心数据丢失，那么 Memory Channel 就不应该使用，因为程序死亡、机器宕
   机或者重启都会导致数据丢失。
   
2. File Channel 将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数
   据  

- Event

传输单元， Flume 数据传输的基本单元，以 Event 的形式将数据从源头送至目的地。
Event 由 Header 和 Body 两部分组成， Header 用来存放该 event 的一些属性，为 K-V 结构，
Body 用来存放该条数据，形式为字节数组 





# 三、Flume安装

下载地址Flume1.7 http://archive.apache.org/dist/flume/1.7.0/

安装Flume比较简单，将tar.gz文件下载好之后，解压在安装目录后，只需要配置一个地方，flume目录下的conf目录下的下的 flume-env.sh.template 文件修改为 flume-env.sh，并配置 flume-env.sh 文件  

### 3.1 解压

```shell
[root@hadoop102 software]# tar -zxvf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
```

### 3.2 修改配置文件

```shell
[root@hadoop102 software]# cd /opt/module/apache-flume-1.7.0-bin/
[root@hadoop102 apache-flume-1.7.0-bin]# cd conf/
[root@hadoop102 conf]# cp flume-env.sh.template flume-env.sh
[root@hadoop102 conf]# vim flume-env.sh
```

![image-20201210195152487](C:\Users\VSUS\Desktop\笔记\大数据\img\53.png)

# 四、Flume案例



### 4.1 监控端口数据

##### 4.1.1 需求

使用 Flume 监听一个端口， 收集该端口数据，并打印到控制台  

##### 4.1.2 分析

1. 客户端通过netcat工具向6666端口发送数据
2. Flume监控本机的6666端口，通过Flume的source段读取数据
3. Flume将获取的数据通过Sink写出到控制台

##### 4.1.3 创建配置文件

创建配置文件flume-netcat-logger.conf

这个配置文件也是官方文档上的

```properties
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1		#r1:表示a1的Source名称
a1.sinks = k1		#k1:表示a1的Sink名称
a1.channels = c1	#c1:表示a1的Channer名称

# Describe/configure the source
a1.sources.r1.type = netcat			#示a1的输入类型是netcat端口类型
a1.sources.r1.bind = localhost		#表示a1的监听的主机
a1.sources.r1.port = 6666			#表示a1监听的端口号

# Describe the sink
a1.sinks.k1.type = logger			#表示a1的输出目的是控制台logger类型

# Use a channel which buffers events in memory
a1.channels.c1.type = memory		#a1的Channel类型是memory类型
a1.channels.c1.capacity = 1000		#Channel总容量1000个event
a1.channels.c1.transactionCapacity = 100	#Channel传输收集到了100条Event以后再提交事务

# Bind the source and sink to the channel
a1.sources.r1.channels = c1		#表示将r1和c1连接起来
a1.sinks.k1.channel = c1		#表示k1和c1连接起来，注意这里是channel
```

可以发现配置文件有五部分的内容：

- 配置名称，相当于命名
- 配置Source
- 配置Sink
- 配置Channel
- 配置如何Channel与Source、Sink连接

##### 4.1.4 测试后

- 第一种写法

```shell
bin/flume-ng agent --conf conf/ --name a1 --conf-file job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

- 第二种更简单写法

```shell
bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

- --conf/-c：表示配置文件存储在 conf/目录
- --name/-n： 表示给 agent 起名为 a1
- --conf-file/-f： flume 本次启动读取的配置文件是在 job 文件夹下的 flume-telnet.conf
  文件。
- -Dflume.root.logger=INFO,console ： -D 表示 flume 运行时动态修改 flume.root.logger
  参数属性值，并将控制台日志打印级别设置为 INFO 级别。日志级别包括:log、 info、 warn、
  error  

### 4.2 实时监控单个追加文件

##### 4.2.1 需求

实时监控Hive日志信息，并且上传到HDFS目录下，hive日志信息的配置是在hive-log4j2.properties配置文件下配置的

```properties
/opt/module/apache-hive-2.1.1-bin/conf/hive-log4j2.properties

property.hive.log.dir = /opt/module/apache-hive-2.1.1-bin/logs
```

##### 4.2.2 分析

1. Hive实时更新日志到/opt/module/apache-hive-2.1.1-bin/logs/hive.log
2. Exec Source监控这个文件，将HDFS Sink将文件数据写入到HDFS上

##### 4.2.3 创建配置文件

这个案例通过分析要使用Exec Source和HDFS Sink，可以通过查看官方文档来配置

![image-20201206201208081](C:\Users\VSUS\Desktop\笔记\大数据\img\35.png)

可以看到上面3个参数是必选的，在文档下方还有使用的例子

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1
```

==同样的==，HDFS Sink在文档上都有详细的描述

jobs/flume-file-hdfs.conf

```properties

# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/apache-hive-2.1.1-bin/logs/hive.log

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop101:8020/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k2.hdfs.batchSize = 1000
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件，单位s
a2.sinks.k2.hdfs.rollInterval = 30
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a2.sinks.k2.hdfs.rollCount = 0
# Use a channel which buffers events in memory

a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

> - tail -f
>
> 等同于--follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止
>
> - tail -F
>
> 等同于--follow=name --retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪。
>
> 因为Hiv日志会每天进行更新，并且按时间命名，所以这里使用tail -F

##### 4.2.4 上传必要的jar包

```
commons-configuration-1.6.jar、
hadoop-auth-2.7.2.jar、
hadoop-common-2.7.2.jar、
hadoop-hdfs-2.7.2.jar、
commons-io-2.4.jar、
htrace-core-3.1.0-incubating.jar
```

拷贝到/opt/module/flume/lib 文件夹下。  

##### 4.2.5 测试

开启flume

```
bin/flume-ng agent -c conf/ -n a2 -f job/flume-file-hdfs.conf
```

查看结果

1. 在配置文件中配置了按照时间来滚动文件夹，所以在hdfs上按照时间创建文件夹，这样对之后Hive进行分区操作很友好，这里==因为要使用时间，所以必须要配置使用本地时间戳==

```properties
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\36.png)

2. 设置了文件30秒滚动一次，这个时间只是为了好测试，实际开发中不设置这么小

```properties
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件，单位s
a2.sinks.k2.hdfs.rollInterval = 30
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
```

### 4.3 实时监控目录下的多个文件

##### 4.3.1 需求 

1. 将待上传的文件上传至/opt/module/apache-flume-1.7.0-bin/flumeupload
2. 使用Spooldir监控目录，将符合条件的文件数据写入Channel
3. HDFS Sink将文件数据写到HDFS上
4. 查看/opt/module/apache-flume-1.7.0-bin/flumeupload目录中上传的文件是否已经标记为.COMPLETED结尾，.tmp后缀文件没有上传

##### 4.3.2 创建配置文件

```properties
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = spooldir
a2.sources.r2.spoolDir = /opt/module/apache-flume-1.7.0-bin/flumeupload
a2.sources.r2.fileSuffix = .COMPLETED
a2.sources.r2.fileHeader = true
#忽略所有以.tmp 结尾的文件，不上传
a2.sources.r2.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop101:8020/flume/upload/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
##多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
##重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
##是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
##积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k2.hdfs.batchSize = 1000
##设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
##多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 30
##设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
##文件的滚动与 Event 数量无关
a2.sinks.k2.hdfs.rollCount = 0
## Use a channel which buffers events in memory

a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

##### 4.3.3 测试

上传文件至 /opt/module/apache-flume-1.7.0-bin/flumeupload目录下，可以看到监控的目录下的文件的后缀名已经变为.COMPLETED，这是在配置文件中配置的

![image-20201207202505475](C:\Users\VSUS\Desktop\笔记\大数据\img\image-20201207202505475.png)

如果上传以.tmp结尾的文件，因为在配置文件中配置了a2.sources.r2.ignorePatter，所以自动忽略了以.tmp结尾的文件

![image-20201207202758834](C:\Users\VSUS\Desktop\笔记\大数据\img\38.png)

![](C:\Users\VSUS\Desktop\笔记\大数据\img\40.png)

### 4.4 实时监控目录下的多个追加文件

Exec source 适用于监控一个实时追加的文件， 但不能保证数据不丢失； Spooldir
Source 能够保证数据不丢失，且能够实现断点续传， 但延迟较高，不能实时监控；而 Taildir
Source 既能够实现断点续传，又可以保证数据不丢失，还能够进行实时监控。  

##### 4.4.1 需求

使用Flume监控整个目录的实时追加文件，并上传是HDFS	

##### 4.4.2分析

1. 被监控的目录：/opt/module/apache-flume-1.7.0-bin/flumeupload3
2. 向监控文件中追加内容echo>> file1.txt
3. Taildir Source监控目录，将数据写入Channel，最后将数据上传至HDFS

##### 4.4.3 配置文件

再次向官网查看Taildir Source 是如何配置的

![](C:\Users\VSUS\Desktop\笔记\大数据\img\41.png)

```properties
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = TAILDIR
a2.sources.r2.positionFile = /opt/module/apache-flume-1.7.0-bin/tail_dir.json
a2.sources.r2.filegroups = f1
a2.sources.r2.filegroups.f1 = /opt/module/apache-flume-1.7.0-bin/flumeupload3/file.*

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop101:8020/flume/upload3/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
##多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
##重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
##是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
##积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k2.hdfs.batchSize = 1000
##设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
##多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 30
##设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
##文件的滚动与 Event 数量无关
a2.sinks.k2.hdfs.rollCount = 0
## Use a channel which buffers events in memory

a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

