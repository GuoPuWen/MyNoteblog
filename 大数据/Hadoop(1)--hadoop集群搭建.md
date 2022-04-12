### 一、hadoop介绍

1. Hadoop最早起源于Nutch。Nutch的设计目标是构建一个大型的全网搜索引擎，包括网页抓取、索引、查询等功能，但随着抓取网页数量的增加，遇到了严重的可扩展性问题——如何解决数十亿网页的存储和索引问题。
2. 2003年、2004年谷歌发表的两篇论文为该问题提供了可行的解决方案。

——分布式文件系统（GFS），可用于处理海量网页的存储

——分布式计算框架MAPREDUCE，可用于处理海量网页的索引计算问题。

3. Nutch的开发人员完成了相应的开源实现HDFS和MAPREDUCE，并从Nutch中剥离成为独立项目HADOOP，到2008年1月，HADOOP成为Apache顶级项目.

狭义上来说，hadoop就是单独指代hadoop这个软件，

* HDFS    ：分布式文件系统
* MapReduce : 分布式计算系统
* Yarn：分布式样集群资源管理 

广义上来说，hadoop指代大数据的一个生态圈，包括很多其他的软件

![1558225014064](C:\Users\VSUS\Desktop\笔记\大数据\img\1.png) 

### 二、hadoop架构

这里主要介绍hadoop2.x版本的结构模型

第一种：NameNode与ResourceManager单节点架构模型

![1558232924095](C:\Users\VSUS\Desktop\笔记\大数据\img\2.png)	

文件系统核心模块：

- NameNode：集群当中的主节点，主要用于管理集群当中的各种数据

- secondaryNameNode：主要能用于hadoop当中元数据信息的辅助管理

- DataNode：集群当中的从节点，主要用于存储集群当中的各种数据

数据计算核心模块：

- ResourceManager：接收用户的计算请求任务，并负责集群的资源分配

- NodeManager：负责执行主节点APPmaster分配的任务

第二种：NameNode单节点与ResourceManager高可用架构模型

![1558232966712](C:\Users\VSUS\Desktop\笔记\大数据\img\5.png)	

文件系统核心模块：

- NameNode：集群当中的主节点，主要用于管理集群当中的各种数据

- secondaryNameNode：主要能用于hadoop当中元数据信息的辅助管理

- DataNode：集群当中的从节点，主要用于存储集群当中的各种数据

数据计算核心模块：

- ResourceManager：接收用户的计算请求任务，并负责集群的资源分配，以及计算任务的划分，通过zookeeper实现ResourceManager的高可用

- NodeManager：负责执行主节点ResourceManager分配的任务

第三种：NameNode高可用与ResourceManager单节点架构模型

![1558232980575](C:\Users\VSUS\Desktop\笔记\大数据\img\6.png)	

文件系统核心模块：

- NameNode：集群当中的主节点，主要用于管理集群当中的各种数据，其中nameNode可以有两个，形成高可用状态

- DataNode：集群当中的从节点，主要用于存储集群当中的各种数据

- JournalNode：文件系统元数据信息管理

数据计算核心模块：

- ResourceManager：接收用户的计算请求任务，并负责集群的资源分配，以及计算任务的划分

- NodeManager：负责执行主节点ResourceManager分配的任务

 

第四种：NameNode与ResourceManager高可用架构模型

![1558232995675](C:\Users\VSUS\Desktop\笔记\大数据\img\7.png)	

文件系统核心模块：

- NameNode：集群当中的主节点，主要用于管理集群当中的各种数据，一般都是使用两个，实现HA高可用

- JournalNode：元数据信息管理进程，一般都是奇数个

- DataNode：从节点，用于数据的存储

数据计算核心模块：

- ResourceManager：Yarn平台的主节点，主要用于接收各种任务，通过两个，构建成高可用

- NodeManager：Yarn平台的从节点，主要用于处理ResourceManager分配的任务

### 三、分发脚本xsync

```shell
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for((host=102; host<104; host++)); do
        echo ------------------- hadoop$host --------------
        rsync -rvl $pdir/$fname root@hadoop$host:$pdir
done

```

分发脚本关键使用rsync传输命令，可见该脚本放在/usr/loacl/bin/目录下，以便直接使用

### 四、搭建hadoop集群



集群规划

| 服务器IP          | 192.168.18.101 | 192.168.18.102 | 192.168.18.103 |
| ----------------- | -------------- | -------------- | -------------- |
| 主机名            | hadoop01       | hadoop02       | hadoop03       |
| NameNode          | 是             | 否             | 否             |
| SecondaryNameNode | 是             | 否             | 否             |
| dataNode          | 是             | 是             | 是             |
| ResourceManager   | 是             | 否             | 否             |
| NodeManager       | 是             | 是             |                |



##### 1.上传hadoop安装包并解压



我使用的winscp工具进行文件的传输

![](C:\Users\VSUS\Desktop\笔记\大数据\img\3.png)

并且解压到/opt/module目录下

```shell
tar -zxvf hadoop-2.7.5.tar.gz -C /opt/module/
```

最后使用集群分发脚本xsync分发到其他服务器

```shell
xsync /opt/application/hadoop-2.7.3/
```

现在，每个服务器都有hadoop安装包，目录为/opt/module，另外hadoop运行需要java环境。

##### 2.配置环境变量

```shell
[root@hadoop101 hadoop-2.7.5]# pwd
/opt/application/hadoop-2.7.3
[root@hadoop101 hadoop-2.7.5]# vim /etc/profile
export HADOOP_HOME=/opt/application/hadoop-2.7.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
[root@hadoop101 hadoop-2.7.5]# source /etc/profile

```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\4.png)

到这，第一台机器的hadoop已经安装完毕，然后使用集群分发脚本将/etc/profile文件分发至各台机器，不要忘记重新加载配置文件

##### 3.修改配置文件

配置文件都在etc/hadoop中

###### 3.1 core-site.xml

```xml
<property>
    <name>fs.default.name</name>
    <value>hdfs://hadoop01:8020</value>
</property>

<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/application/hadoop-2.7.3/hadoopDatas/tempDatas</value>
</property>

<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->

<property>
    <name>io.file.buffer.size</name>
    <value>4096</value>
</property>

<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
<property>
    <name>fs.trash.interval</name>
    <value>10080</value>
</property>

```

###### 3.2 hdfs-site.xml

```xml
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop101:50090</value>
</property>

<property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop101:50070</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
</property>

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///opt/application/hadoop-2.7.3/hadoopDatas/datanodeDatas,file:///opt/application/hadoop-2.7.3/hadoopDatas/datanodeDatas2</value>
</property>

<property>
    <name>dfs.namenode.edits.dir</name>
    <value>file:///opt/application/hadoop-2.7.3/hadoopDatas/nn/edits</value>
</property>

<property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file:///opt/application/hadoop-2.7.3/hadoopDatas/snn/name</value>
</property>

<property>
    <name>dfs.namenode.checkpoint.edits.dir</name>
    <value>file:///opt/application/hadoop-2.7.3/hadoopDatas/dfs/snn/edits</value>
</property>

<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>

```

###### 3.3 hadoop-env.sh

```shell
export JAVA_HOME=/opt/application/jdk1.8.0_131
```

###### 3.4 mapred-site.xml

```xml
<property>
    <name>mapreduce.job.ubertask.enable</name>
    <value>true</value>
</property>

<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop101:10020</value>
</property>

<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop101:19888</value>
</property>

```

###### 3.5 yarn-site.xml

```xml
<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop101</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
	<property>    
		<name>yarn.nodemanager.resource.memory-mb</name>    
		<value>20480</value>
	</property>
	<property>  
        	 <name>yarn.scheduler.minimum-allocation-mb</name>
         	<value>2048</value>
	</property>
	<property>
		<name>yarn.nodemanager.vmem-pmem-ratio</name>
		<value>2.1</value>
	</property>
```

###### 3.6 mapred-env.sh

```shell
export JAVA_HOME=/opt/application/jdk1.8.0_131
```

###### 3.7 修改slaves

```xml
hadoop101
hadoop102
hadoop103      
```

###### 3.8 创建上面配置文件出现的一些文件

```shell
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/tempDatas
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/namenodeDatas
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/namenodeDatas2
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/datanodeDatas
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/datanodeDatas2
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/snn/name
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/nn/edits
mkdir -p /opt/application/hadoop-2.7.3/hadoopDatas/dfs/snn/edits
```

###### 3.9 分发配置文件到其他机器

### 五、启动集群

```java
bin/hdfs namenode -format
sbin/start-dfs.sh
sbin/start-yarn.sh
```

