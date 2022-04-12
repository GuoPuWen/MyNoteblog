# 一、概述

Kafka 是一个分布式的基于发布/订阅模式的消息队列（Message Queue） ， 主要应用于大数据实时处理领域。  

### 1.1 使用消息队列的好处

- 解耦

允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。  

- 可恢复性

系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所
以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理  

- 缓冲

有助于控制和优化数据流经过系统的速度， 解决生产消息和消费消息的处理速度不一致
的情况

- 灵活性 & 峰值处理能力  

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。
如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列
能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃  

- 异步通信  

很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户
把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要
的时候再去处理它们。  

### 1.2 消息队列的两种模式

- 点对点模式

一对一，消费者主动拉取数据，消息收到后消息清除。消息生产者生产消息发送到Queue中， 然后==消息消费者从Queue中取出并且消费消息==。消息被消费以后， queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue 支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。  

![image-20201221202620902](kafka(1).assets/image-20201221202620902.png)

- 发布/订阅模式  

一对多，消费者消费数据之后不会清除消息 。消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到 topic 的消息会】可以被所有订阅者消费  

![image-20201221202737917](kafka(1).assets/image-20201221202737917.png)

### 1.3 使用kafka的好处

- 支持消息的发布和订阅，类似于 RabbtMQ、ActiveMQ 等消息队列；
- 支持数据实时处理；
- 能保证消息的可靠性投递；
- 支持消息的持久化存储，并通过多副本分布式的存储方案来保证消息的容错；
- 高吞吐率，单 Broker(值的是kafka集群中的机器)可以轻松处理数千个分区以及每秒百万级的消息量  

# 二、kafka架构

![image-20201221205922420](kafka(1).assets/image-20201221205922420.png)

- Producer：消息生产者，就是向 kafka broker 发消息的客户端
- Consumer ：消息消费者，向 kafka broker 取消息的客户端
- Consumer Group （CG） ：消费者组，由多个 consumer 组成。 **消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内成员消费者消费；消费者组之间互不影响**。 所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。  

例如：上图中ConsumerA和CondumerB是一个消费者组，TopicA-Partition 0 已经被消费者A消费了，那么TopicA-Partition 0就不能被消费者B消费

- Broker：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker可以容纳多个 topic
- Partition：为了实现扩展性，就是为了提高效率，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列

- Replica：副本，为保证集群中的某个节点发生故障时， 该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作， kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower

- leader： 每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader

- follower： 每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。 leader 发生故障时，某个 follower 会成为新的 follower 

# 三、kafka集群搭建

### 3.1 集群规划

| hadoop101 | hadoop102 | hadoop103 |
| --------- | --------- | --------- |
| zookeeper | zookeeper | zookeeper |
| kafka     | kafka     | kafka     |

### 3.2 下载解压安装

[kafka各个版本下载地址](http://kafka.apache.org/downloads.html)

```shell
tar -zxvf kafka-eagle-bin-1.3.7.tar.gz -C /opt/software/
```

### 3.3 修改配置文件

kafka的配置文件都在/config目录下

![image-20201221211708535](kafka(1).assets/image-20201221211708535.png)

配置集群，要修改server.properties

主要修改的有

```properties
#broker 的全局唯一编号，不能重复
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0
#删除 topic 功能使能
# Switch to enable topic deletion or not, default value is false
delete.topic.enable=true

#kafka 运行日志存放的路径
# A comma seperated list of directories under which to store log files
log.dirs=/opt/module/kafka_2.11-0.11.0.0/data

#配置连接 Zookeeper 集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181
```

### 3.4 分发安装包

使用xsync脚本将安装包分发到各个机器上

### 3.5 修改各个机器上的brokerid

注意：brokerid是不能重复的

### 3.6 集群启动与关闭

kafka集群和fluem集群启动一样都是不能群起，所以要一个一个机器启动

```shell
#启动集群
[root@hadoop101 kafka_2.11-0.11.0.0]# bin/kafka-server-start -daemon config/server.properties
#关闭集群
[root@hadoop101 kafka_2.11-0.11.0.0]# kafka-server-stop.sh
```

-daemon：守护进程

### 3.7 集群脚本

```shell

#!/bin/sh
params=$1
if [ "$params" = "start" ]
then
        for (( i=1 ; i <= 3 ; i = $i + 1 )) ;
         do
                echo ============= hadoop10$i $params =============
                 ssh root@192.168.18.10$i "/opt/module/kafka_2.11-0.11.0.0/bin/kafka-server-start.sh  -daemon /opt/module/kafka_2.11-0.11.0.0/config/server.properties"
        done
fi
if [ "$params" = "stop" ]
then
        for (( i=1 ; i <= 3 ; i = $i + 1 )) ;
         do
                echo ============= hadoop10$i $params =============
                 ssh root@192.168.18.10$i "/opt/module/kafka_2.11-0.11.0.0/bin/kafka-server-stop.sh"
        done
fi
```

将脚本复制到/usr/local/bin/即可

# 四、kafka命令行

### 4.1 topic操作

kafka的操作都是基于一个一个topic主题的

1. 查看当前服务器中的所有 topic  

```shell
[root@hadoop101 bin]# kafka-topics.sh --zookeeper hadoop102:2181 --list
```

2. 创建topic

```shell
[root@hadoop101 bin]# kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partitions 1 --topic first
```

- --topic 定义 topic 名  
- --replication-factor 定义副本数
- --partitions 定义分区数  

3. 删除topic

```shell
[root@hadoop101 bin]# bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
```

需要 server.properties 中设置 delete.topic.enable=true 否则只是标记删除  