# 五、Flume事务

Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。

![image-20201208202524212](C:\Users\VSUS\Desktop\笔记\大数据\img\42.png)

Flume事务分为Put事务，推送事件，负责处理从Source到Channel中的数据；Take事务，拉取事件负责处理从Channel到Sink中的事务处理

- Put事务流程
  - doPut：将批数据先写入到临时缓冲区putList
  - doCommit：检查Channel内存队列是否足够合并
  - doRollback：内存队列空间不足，回滚队列

- Take事务流程
  - doTake：将数据取到临时缓冲区takeList，并将数据发送到HDFS，或者其他组件
  - doCommit：如果数据全部发送成功，则清除临时缓冲区takeList
  - doRollBack：数据发送过程中如果出现异常，rollback将临时缓冲区takeList中的数据归还给Channel内存队列

 Flume的事务机制，总的来说，保证了source产生的每个事件都会传送到sink中。但是值得一说的是，实际上Flume作为高容量并行采集系统采用的是At-least-once（传统的企业系统采用的是exactly-once机制）提交方式，这样就造成每个source产生的事件至少到达sink一次，换句话说就是同一事件有可能重复到达。这样虽然看上去是一个缺陷，但是相比为了保证Flume能够可靠地将事件从source,channel传递到sink,这也是一个可以接受的权衡。如上博客中spooldir的使用，Flume会对已经处理完的数据进行标记。

# 六、Agent的内部原理

![image-20201208202439146](C:\Users\VSUS\Desktop\笔记\大数据\img\43.png)

1. Source采集数据，将数据封装成Event时间，然后交给Channel Processor

2. Channel Processor将Event事件传递给拦截器链(可以定义多个)，进行简单的数据清洗，然后再将数据返回给Channel Processor
3. Channel Processor将拦截过滤之后的Event事件传递给Channel选择器(Channel Selector)，主要是因为一个Source可以对应对个Channel，那么要确定发送到哪一个Channel就需要Channel Selector来确定，然后Channel Selector返回给Channel Processor写入event事件的Channel列表
4. Channel Processor根据Channel选择器的选择结果，将Event事件写入对应的Channel
5. 然后SinkProcessor启动Sink，将Event写入到对应的Sink中

- ==Channel Selector==
  - ReplicatingSelector  ：会将同一个 Event 发往所有的 Channel  
  - Multiplexing  ：会根据相应的原则，将不同的 Event 发往不同的 Channel。  

ChannelSelector 的作用就是选出 Event 将要被发往哪个 Channel。其共有两种类型，
分别是 Replicating（复制）和 Multiplexing（多路复用）。  

- ==SinkProcessor==
  - DefaultSinkProcessor  ：对应单个的Sink，也是默认的Sink
  - DefaultSinkProcessor  ：对应Sink Group，可以实现负载均衡的功能  
  - FailoverSinkProcessor ：对应Sink Group，可以实现故障转移，容灾恢复的功能  

# 七、Flume拓扑结构

### 7.1 简单串联

![image-20201208205528358](C:\Users\VSUS\Desktop\笔记\大数据\img\44.png)

这种模式是将多个 flume 顺序连接起来了，从最初的 source 开始到最终 sink 传送的目的存储系统。此模式不建议桥接过多的 flume 数量， flume 数量过多不仅会影响传输速率，而且一旦传输过程中某个节点 flume 宕机，会影响整个传输系统。  

### 7.2 复制和多路复用

![image-20201208205658953](C:\Users\VSUS\Desktop\笔记\大数据\img\45.png)

Flume 支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个Channel 中，或者将不同数据分发到不同的 channel 中， sink 可以选择传送到不同的目的地  

### 7.3 负载均衡和故障转移

![image-20201208205827929](C:\Users\VSUS\Desktop\笔记\大数据\img\46.png)

Flume支持使用将多个sink逻辑上分到一个sink组， sink组配合不同的SinkProcessor可以实现负载均衡和错误恢复的功能。  

### 7.4 聚合

![image-20201208210010362](C:\Users\VSUS\Desktop\笔记\大数据\img\47.png)

这种模式是我们最常见的，也非常实用，日常 web 应用通常分布在上百个服务器，大者甚至上千个、上万个服务器。产生的日志，处理起来也非常麻烦。用 flume 的这种组合方式能很好的解决这一问题， 每台服务器部署一个 flume 采集日志，传送到一个集中收集日志的flume，再由此 flume 上传到 hdfs、 hive、 hbase 等，进行日志分析  

# 八、企业开发案例

### 8.1 复制和多路复用

##### 8.1.1 需求分析

使用Flume-1 监控文件，Flume-1 将变动内容传递给 Flume-2， Flume-2 负责存储到 HDFS。同时 Flume-1 将变动内容传递给 Flume-3， Flume-3 负责输出到 Local FileSystem  

![image-20201209194640598](C:\Users\VSUS\Desktop\笔记\大数据\img\48.png)

1. **Flume-1**Source部分使用Exec Source，也可以使用Taildir Source，Channel部分配置两个Memory Channel，Sink部分使用Avro Sink，因为要使用多个agent
2. **Flume-2** Source部分使用Avro Source，Channel部分使用Memory Channel，Sink使用HDFS  Sink
3. **Flume-3** Source部分使用Acro Source，Channel部分使用Memory Channel，Sink使用File_roll Sink

##### 8.1.2 配置文件

flume-1.conf

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1 c2

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/apache-hive-2.1.1-bin/logs/hive.log

#配置sink1
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop101
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop101
a1.sinks.k2.port = 4142


a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100


# 将数据流复制给所有 channel
a1.sources.r1.selector.type = replicating


a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

flume-2.conf

```properties
# Name the components on this agent
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# source 端的 avro 是一个数据接收服务
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop101
a2.sources.r1.port = 4141


a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop101:8020/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k1.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k1.hdfs.batchSize = 1000
#设置文件类型，可支持压缩
a2.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件，单位s
a2.sinks.k1.hdfs.rollInterval = 30
#设置每个文件的滚动大小
a2.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a2.sinks.k1.hdfs.rollCount = 0



# Describe the channel
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1

```

flume-3.conf

```properties
# Name the components on this agent
a3.sources = r1
a3.sinks = k1
a3.channels = c2

a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop101
a3.sources.r1.port = 4142

# Describe the sink
a3.sinks.k1.type = file_roll
a3.sinks.k1.sink.directory = /opt/module/data/flume

# Describe the channel
a3.channels.c2.type = memory
a3.channels.c2.capacity = 1000
a3.channels.c2.transactionCapacity = 100


# Bind the source and sink to the channel
a3.sources.r1.channels = c2
a3.sinks.k1.channel = c2
```

##### 8.1.3 测试

注意：hdfs sink中指定的路径如果不存在会自己创建，但是file_roll sink的指定目录如果不存在会报错

![image-20201209203725585](C:\Users\VSUS\Desktop\笔记\大数据\img\49.png)

### 8.2 负载均衡和故障转移

##### 8.2.1 需求分析

   使用 Flume1 监控一个端口，其 sink 组中的 sink 分别对接 Flume2 和 Flume3，采用
FailoverSinkProcessor，实现故障转移的功能  

![image-20201210183529477](C:\Users\VSUS\Desktop\笔记\大数据\img\50.png)

 

1. Flume-1： 
   - Source：netcat Source
   - Channel：Memory
   - Sink1：Avro Sink
   - Sink2：Avro Sink
   - 采用FailoverSinkProcessor故障转移Sink Group

2. Flume-2：
   - Source：Avro Source
   - Channel：Memory
   - Sink：logger Sink

3. Flume-3：
   - Source：Avro Source
   - Channel：Memory
   - Sink：logger Sink

##### 8.2.2 配置文件

Flume-1.conf

```properties
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1
a1.sinkgroups = g1

a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

#sink组的配置
a1.sinkgroups.g1.processor.type = failover
#定义优先级
a1.sinkgroups.g1.processor.priority.k1 = 5
a1.sinkgroups.g1.processor.priority.k2 = 10
a1.sinkgroups.g1.processor.maxpenalty = 10000

#定义avro sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop101
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop101
a1.sinks.k2.port = 4142

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sources.r1.channels = c1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c1
```

Flume-2.conf

```properties
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# source 端的 avro 是一个数据接收服务
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop101
a2.sources.r1.port = 4141

a2.sinks.k1.type = logger

# Describe the channel
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
```

Flume-3.conf

```properties
a3.sources = r1
a3.sinks = k1
a3.channels = c1

# source 端的 avro 是一个数据接收服务
a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop101
a3.sources.r1.port = 4142

a3.sinks.k1.type = logger

# Describe the channel
a3.channels.c1.type = memory
a3.channels.c1.capacity = 1000
a3.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r1.channels = c1
a3.sinks.k1.channel = c1                     
```

##### 8.2.3 测试

1. 使用netcat工具向4444端口发送数据

2. 检测a3所在的agent收到信息，因为在配置文件中配置了优先级

3. 手动kill掉a3所在的agent，发现a2所在的agent替代了a2的工作，故障转移

   查看flume的日志，可以追踪这一个过程

![image-20201210192711717](C:\Users\VSUS\Desktop\笔记\大数据\img\51.png)

### 8.3聚合

##### 8.3.1 需求分析

1. hadoop101 上的 Flume-1 监控文件/opt/module/data/group.log，
2. hadoop102 上的 Flume-2 监控某一个端口44444的数据流，
3. Flume-1 与 Flume-2 将数据发送给 hadoop103 上的 Flume-3， Flume-3 将最终数据打印到控
   制台。  

![image-20201210193622947](C:\Users\VSUS\Desktop\笔记\大数据\img\52.png)

1. Flume-1： 在机器hadoop101上
   - Source：Exec Source
   - Channel：Memory
   - Sink1：Avro Sink

2. Flume-2：在机器hadoop102上
   - Source：Netcat Source
   - Channel：Memory
   - Sink：Avro Sink

3. Flume-3：在机器hadoop103上
   - Source1：Avro Source
   - Source2：Avro Source
   - Channel：Memory
   - Sink：logger Sink

前提：各个机器上都安装好了flume

![image-20201210202857477](C:\Users\VSUS\Desktop\笔记\大数据\img\54.png)

##### 8.3.2 配置文件

hadoop101上的flume-file-avro.conf

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1


a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/group.log

a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop103
a1.sinks.k1.port = 4141


a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100


a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```

hadoop102上的flume-netcat-avro.conf

```properties
a2.sources = r1
a2.sinks = k1
a2.channels = c1

a2.sources.r1.type = netcat
a2.sources.r1.bind = hadoop102
a2.sources.r1.port = 44444

a2.sinks.k1.type = avro
a2.sinks.k1.hostname = hadoop103
a2.sinks.k1.port = 4141

a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
```

hadoop103上的flume-avro-logger.conf 

```properties
a3.sources = r1
a3.sinks = k1
a3.channels = c1

a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop103
a3.sources.r1.port = 4141


a3.sinks.k1.type = logger

a3.channels.c1.type = memory
a3.channels.c1.capacity = 1000
a3.channels.c1.transactionCapacity = 100

a3.sources.r1.channels = c1
a3.sinks.k1.channel = c1
```

##### 8.3.3 测试

1. 向hadoop101中的目标文件写入数据，hadoop103中打印到控制台

![](C:\Users\VSUS\Desktop\笔记\大数据\img\55.png)

![](C:\Users\VSUS\Desktop\笔记\大数据\img\56.png)

2. 向hadoop102中目标端口写入数据，hadoop103中打印到控制台

![](C:\Users\VSUS\Desktop\笔记\大数据\img\57.png)

![](C:\Users\VSUS\Desktop\笔记\大数据\img\58.png)

# 九、题目

### 9.1 Flume 的 Source， Sink， Channel 的作用？ 

-  Source 组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，
  包括 avro、 thrift、 exec、 jms、 spooling directory、 netcat、 sequence generator、 syslog、
  http、 legacy
- Channel 组件对采集到的数据进行缓存，可以存放在 Memory 或 File 中。
- Sink 组件是用于把数据发送到目的地的组件，目的地包括 HDFS、 Logger、 avro、
  thrift、 ipc、 file、 Hbase、 solr、自定义  

### 9.2 Flume 的 Channel Selectors  

 Channel Selectors  一共有两种

- ReplicatingSelector  ：会将同一个 Event 发往所有的 Channel  
- Multiplexing  ：会根据相应的原则，将不同的 Event 发往不同的 Channel。  

ChannelSelector 的作用就是选出 Event 将要被发往哪个 Channel。其共有两种类型，
分别是 Replicating（复制）和 Multiplexing（多路复用）。  

### 9.3 Flume 的事务机制  

### 9.4 Flume 采集数据会丢失吗?  

根据 Flume 的架构原理， Flume 是不可能丢失数据的，其内部有完善的事务机制，Source 到 Channel 是事务性的， Channel 到 Sink 是事务性的，因此这两个环节不会出现数据的丢失，唯一可能丢失数据的情况是 Channel 采用 memoryChannel， agent 宕机导致数据丢失，或者 Channel 存储数据已满，导致 Source 不再写入，未写入的数据丢失。

Flume 不会丢失数据，但是有可能造成数据的重复，例如数据已经成功由 Sink 发出，
但是没有接收到响应， Sink 会再次发送数据，此时可能会导致数据的重复。  