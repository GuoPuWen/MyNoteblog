参考《Hadoop核心技术》

# HDFS

### 一、HDFS概述

- 分布式文件系统：在现代的企业环境中，单机容量往往无法存储大量数据，需要跨机器存储。统一管理分布在集群上的文件系统称为分布式文件系统
-    HDFS（Hadoop  Distributed  File  System）是 Apache Hadoop 项目的一个子项目. Hadoop 非常适于存储大型数据 (比如 TB 和 PB), 其就是使用 HDFS 作为存储系统. HDFS 使用多台计算机存储文件, 并且提供统一的访问接口, 像是访问一个普通文件系统一样使用分布式文件系统. 

### 二、HDFS的适用场景

##### 2.1 适合的场景

- 存储非常大的文件：这里非常大指的是几百M、G、或者TB级别，需要==高吞吐量==，对==延时没有要求==。
- 采用流式的数据访问方式: 即==一次写入、多次读取==，数据集经常从数据源生成或者拷贝一次，然后在其上做很多分析工作 。
- 运行于商业硬件上: Hadoop不需要特别贵的机器，可运行于普通廉价机器，可以处==节约成本==
- 需要高==容错性==

* 为数据存储提供所需的==扩展能力==

##### 2.2 不适合的场景

-  低延时的数据访问 
    对延时要求在毫秒级别的应用，不适合采用HDFS。HDFS是为高吞吐数据传输设计的,因此可能牺牲延时
- 大量小文件 
  文件的元数据保存在==NameNode的内存中==， 整个文件系统的文件数量会受限于NameNode的内存大小。 经验而言，一个文件/目录/文件块一般占有150字节的元数据内存空间。如果有100万个文件，每个文件占用1个文件块，则需要大约300M的内存。因此十亿级别的文件数量在现有商用机器上难以支持。
- 多方读写，需要任意的文件修改 
  HDFS采用追加（append-only）的方式写入数据。不支持文件任意offset的修改。不支持多个写入器（writer）

### 三、HDFS的架构

 HDFS是一个`主/从（Mater/Slave）体系结构`，HDFS由4个部分组成：

- Client
- NameNode
- DataNode
- Secondary NameNode

![](C:\Users\VSUS\Desktop\笔记\大数据\img\8.jpg)

##### 3.1 Client

- 文件切分。文件上传 HDFS 的时候，Client 将文件切分成一个一个的Block，然后进行存储。
- 与 NameNode 交互，获取文件的位置信息。
- 与 DataNode 交互，读取或者写入数据。
- Client 提供一些命令来管理 和访问HDFS，比如启动或者关闭HDFS。

##### 3.2 NameNode

**NameNode：就是 master，它是一个主管、管理者。**

- 管理 HDFS 的名称空间
- 管理数据块（Block）映射信息
- 配置副本策略
- 处理客户端读写请求。

##### 3.2 DataNode

DataNode：就是Slave。NameNode 下达命令，DataNode 执行实际的操作。

- 存储实际的数据块。
- 执行数据块的读/写操作。

##### 4. Secondary NameNode

Secondary NameNode：并非 NameNode 的热备。当NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务。

- 辅助 NameNode，分担其工作量。
- 定期合并 fsimage和fsedits，并推送给NameNode。
- 在紧急情况下，可辅助恢复 NameNode。-

### 四、NameNode和DataNode

![](C:\Users\VSUS\Desktop\笔记\大数据\img\9.png)

##### 4.1 NameNode作用

* NameNode在内存中保存着整个文件系统的==名称空间==和文件数据块的==地址映射==
* 整个HDFS可存储的文件数受限于NameNode的==内存==大小 

 `1、NameNode元数据信息` 

- 文件名，文件目录结构，文件属性(生成时间，副本数，权限)每个文件的块列表。 
  以及列表中的块与块所在的DataNode之间的地址映射关系 
- 在内存中加载文件系统中每个文件和每个数据块的引用关系(文件、block、datanode之间的映射信息) 
- 数据会定期保存到本地磁盘（fsImage文件和edits文件）

`2、NameNode文件操作` 

- NameNode负责文件元数据的操作 
- DataNode负责处理文件内容的读写请求，数据流不经过NameNode，会询问它跟那个DataNode联系

`3、NameNode副本` 
文件数据块到底存放到哪些DataNode上，是由NameNode决定的，NameNode根据全局情况做出放置副本的决定 

`4、NameNode心跳机制`

- 全权管理数据块的复制，周期性的接受心跳和块的状态报告信息（包含该DataNode上所有数据块的列表） 
- 若接受到心跳信息，NameNode认为DataNode工作正常，如果在10分钟后还接受到不到DN的心跳，那么NameNode认为DataNode已经宕机 ,这时候NN准备要把DN上的数据块进行重新的复制。 块的状态报告包含了一个DN上所有数据块的列表，blocks report 每个1小时发送一次.

##### 4.2  DataNode作用 

提供真实文件数据的存储服务。 

- Data Node以数据块的形式存储HDFS文件
- Data Node 响应HDFS 客户端读写请求
- Data Node 周期性向NameNode汇报心跳信息
- Data Node 周期性向NameNode汇报数据块信息
- Data Node 周期性向NameNode汇报缓存数据块信息

### 五、HDFS的副本机制和机架感知

##### 5.1 副本机制

所有的文件都是以 block 块的方式存放在 HDFS 文件系统当中,作用如下

1. 一个文件有可能大于集群中任意一个磁盘，引入块机制,可以很好的解决这个问题
2. 使用块作为文件存储的逻辑单位可以简化存储子系统
3. 块非常适合用于数据备份进而提供数据容错能力

在 Hadoop1 当中, 文件的 block 块默认大小是 64M, hadoop2 当中, 文件的 block 块大小默认是 128M, block 块的大小可以通过 hdfs-site.xml 当中的配置文件进行指定

可以在官方文档的hdfs-size.xml文档中查看dfs.blocksize的值为134217728(hadoop2以上)

```xml
<property>
    <name>dfs.block.size</name>
    <value>块大小 以字节为单位</value>
</property>
```

##### 5.2 机架感知

HDFS分布式文件系统的内部有一个副本存放策略：以默认的副本数=3为例：

1、第一个副本块存本机

2、第二个副本块存跟本机同机架内的其他服务器节点

3、第三个副本块存不同机架的一个服务器节点上

