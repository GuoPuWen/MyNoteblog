# HDFS的工作机制

## 一、HDFS文件写入过程

hdfs写入文件可以直接通过命令行的方式：

```shell
格式 ： hdfs dfs -put <localsrc >  ... <dst>
作用 ： 将单个的源文件src或者多个源文件srcs从本地文件系统拷贝到目标文件系统中（<dst>对应的路径）。也可以从标准输入中读取输入，写入目标文件系统中
```

下面剖析一下HDFS写入文件的步骤

![image-20201211205849528](C:\Users\VSUS\Desktop\笔记\大数据\img\60.png)

1. Client发起文件上传请求，通过RPC与NameNode建立通讯，NameNode检查一些信息，比如说·目标文件是否存在，目录是否存在，父目录是否存在等等
2. NameNode放回是否可以上传
3. Client请求上传第一个Block，请求NaneNode返回我要上传到那个DataNode上
4. NameNode根据配置文件中配置的副本数量以及机架感知原理进行分配，返回可用的DataNode地址，例如为dn1，dn2，dn3
5. Client请求往dn1上传数据，dn1收到请求后调用dn2，然后dn2调用dn3，直到将这个管道建立完成
6. dn1，dn2，dn3逐级应答客户端
7. Client开始向dn1上传第一个block，以packet为单位(默认大小为64k)，dn1收到一个packet后会传给dn2，dn2传给dn3，dn1每次传入一个packet会放入一个应答队列等待回答。这样数据被分割成一个个packet数据包在管道上依次传输，在管道流的反方向上逐个发送ack正确应答
8. 当一个block传输完成之后，Client再次请求NameNode上传第二个block，知道传输完成

## 二、HDFS读取数据流程

![](C:\Users\VSUS\Desktop\笔记\大数据\img\61.png)

1. Client向NameNode发起RPC请求，NameNode通过查询元数据，找到文件块所在的DataNode地址
2. NameNode放回目标文件的元数据
3. Client根据元数据信息找到DataNode，请求读取数据
4. DataNode开始传输数据给客户端，从磁盘中读取数据输入流，以packet为单位
5. Client以packet为单位接收，先在本地缓存，然后写入目标文件

