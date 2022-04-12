[TOC]



# 一、HDFS命令行操作

`ls` 

~~~shell
格式：  hdfs dfs -ls  URI
作用：类似于Linux的ls命令，显示文件列表
~~~

~~~shell
 hdfs  dfs   -ls  /
~~~

`lsr`  

~~~shell
格式  :   hdfs  dfs -lsr URI
作用  : 在整个目录下递归执行ls, 与UNIX中的ls-R类似
~~~

~~~shell
 hdfs  dfs   -lsr  /
~~~

`mkdir` 

~~~shell
格式 ： hdfs  dfs [-p] -mkdir <paths>
作用 :   以<paths>中的URI作为参数，创建目录。使用-p参数可以递归创建目录
~~~

`put`  

~~~shell
格式   ： hdfs dfs -put <localsrc >  ... <dst>
作用 ：  将单个的源文件src或者多个源文件srcs从本地文件系统拷贝到目标文件系统中（<dst>对应的路径）。也可以从标准输入中读取输入，写入目标文件系统中
~~~

~~~shell
hdfs dfs -put  /rooot/a.txt  /dir1
~~~

`moveFromLocal`  

~~~shell
格式： hdfs  dfs -moveFromLocal  <localsrc>   <dst>
作用:   和put命令类似，但是源文件localsrc拷贝之后自身被删除
~~~

~~~shell
hdfs  dfs -moveFromLocal  /root/install.log  /
~~~

`moveToLocal`  

~~~shell
未实现
~~~

`get` 

~~~shell
格式   hdfs dfs  -get [-ignorecrc ]  [-crc]  <src> <localdst>

作用：将文件拷贝到本地文件系统。 CRC 校验失败的文件通过-ignorecrc选项拷贝。 文件和CRC校验和可以通过-CRC选项拷贝
~~~

```shell
hdfs dfs  -get   /install.log  /export/servers
```

`mv`

~~~shell
格式： hdfs  dfs -mv URI   <dest>
作用： 将hdfs上的文件从原路径移动到目标路径（移动之后文件删除），该命令不能夸文件系统
~~~

~~~shell
hdfs  dfs  -mv  /dir1/a.txt   /dir2
~~~

`rm`

~~~she
格式： hdfs dfs -rm [-r] 【-skipTrash】 URI 【URI 。。。】
作用： 删除参数指定的文件，参数可以有多个。   此命令只删除文件和非空目录。
如果指定-skipTrash选项，那么在回收站可用的情况下，该选项将跳过回收站而直接删除文件；
否则，在回收站可用时，在HDFS Shell 中执行此命令，会将文件暂时放到回收站中。
~~~

~~~shell
hdfs dfs -rm -r  /dir1
~~~

`cp`

~~~shell
格式: hdfs  dfs  -cp URI [URI ...] <dest>
作用：将文件拷贝到目标路径中。如果<dest>  为目录的话，可以将多个文件拷贝到该目录下。
-f
选项将覆盖目标，如果它已经存在。
-p
选项将保留文件属性（时间戳、所有权、许可、ACL、XAttr）。
~~~

~~~shell
hdfs dfs -cp /dir1/a.txt  /dir2/b.txt
~~~

`cat`  

~~~shell
hdfs dfs  -cat  URI [uri  ...]
作用：将参数所指示的文件内容输出到stdout
~~~

~~~shell
hdfs dfs  -cat /install.log
~~~

`chmod`  

~~~shell
格式:      hdfs   dfs  -chmod  [-R]  URI[URI  ...]
作用：    改变文件权限。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
~~~

~~~shell
hdfs dfs -chmod -R 777 /install.log
~~~

`chown`    

~~~shell
格式:      hdfs   dfs  -chmod  [-R]  URI[URI  ...]
作用：    改变文件的所属用户和用户组。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
~~~

~~~shell
hdfs  dfs  -chown  -R hadoop:hadoop  /install.log
~~~

`appendToFile`

~~~shell
格式: hdfs dfs -appendToFile <localsrc> ... <dst>
作用: 追加一个或者多个文件到hdfs指定文件中.也可以从命令行读取输入.
~~~

~~~shell
 hdfs dfs -appendToFile  a.xml b.xml  /big.xml
~~~

# 二、HDFS的高级使用命令

在多人共用HDFS的环境下，配置设置非常重要。特别是在Hadoop处理大量资料的环境，如果没有配额管理，很容易把所有的空间用完造成别人无法存取。Hdfs的配额设定是针对目录而不是针对账号，可以 让每个账号仅操作某一个目录，然后对目录设置配置。 

​    hdfs文件的限额配置允许我们以文件个数，或者文件大小来限制我们在某个目录下上传的文件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大的文件的量。

### 2.1 查看配额

```shell
 hdfs dfs -count -q -h /user/root/dir1  #查看配额信息
 文件数限额  可用文件数  空间限额 可用空间 目录数  文件数  总大小 文件/目录名
```



### 2.2 配置数量限额

```shell
hdfs dfsadmin -setQuota 2  /dir   # 给该文件夹下面设置最多上传两个文件，发现只能上传一个文件
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\10.png)

设置数量限额为n个文化，则实际可以上传的文件个数为n-1，原因是它把当前文件夹也当成一个文件

```shell
hdfs dfsadmin -clrQuota  /dir  # 清除文件数量限制
```



### 2.2 配置空间大小限额

默认情况下 在设置空间配额时，设置的空间至少是block_size * 3大小，例如下代码

```shell
hdfs dfsadmin -setSpaceQuota 4k /dir4 # 限制空间大小4KB
hdfs dfs -put test1.txt /dir4	#尝试上传一个少于4k的文件
```

发现报出一下错误

```java
20/11/17 22:03:11 WARN hdfs.DFSClient: DataStreamer Exception
org.apache.hadoop.hdfs.protocol.DSQuotaExceededException: The DiskSpace quota of /dir4 is exceeded: quota = 4096 B = 4 KB but diskspace consumed = 402653184 B = 384 MB
	at org.apache.hadoop.hdfs.server.namenode.DirectoryWithQuotaFeature.verifyStoragespaceQuota(DirectoryWithQuotaFeature.java:211)
```

报错信息以及说的清楚，设置的空间至少384mb

`清除空间配额限制`

```shell
hdfs dfsadmin -clrSpaceQuota /dir4
```

`生成任意大小文件的命令:`

```shell
dd if=/dev/zero of=1.txt  bs=1M count=2     #生成2M的文件
```

### 2.3 HDFS的安全模式

安全模式是hadoop的一种保护机制，用于保证集群中的数据块的安全性。当集群启动的时候，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。

假设我们设置的副本数（即参数dfs.replication）是3，那么在datanode上就应该有3个副本存在，假设只存在2个副本，那么比例就是2/3=0.666。hdfs默认的副本率0.999。我们的副本率0.666明显小于0.999，因此系统会自动的复制副本到其他dataNode，使得副本率不小于0.999。如果系统中有5个副本，超过我们设定的3个副本，那么系统也会删除多于的2个副本。 

在安全模式状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。在，当整个系统达到安全标准时，HDFS自动离开安全模式。

```shell
hdfs  dfsadmin  -safemode  get #查看安全模式状态
hdfs  dfsadmin  -safemode  enter #进入安全模式
hdfs  dfsadmin  -safemode  leave #离开安全模式
```





# 三、HDFS的API操作

### 3.1 配置环境变量

步骤:

第一步：将hadoop2.7.5文件夹拷贝到一个没有中文没有空格的路径下面

第二步：在windows上面配置hadoop的环境变量： HADOOP_HOME，并将%HADOOP_HOME%\bin添加到path中

第三步：把hadoop2.7.5文件夹中bin目录下的hadoop.dll文件放到系统盘:  C:\Windows\System32 目录

第四步：关闭windows重启

在windows系统需要配置hadoop运行环境，否则直接运行代码会出现以下问题:

`缺少winutils.exe`

~~~shell
Could not locate executable null \bin\winutils.exe in the hadoop binaries 
~~~

`缺少hadoop.dll`

```shell
Unable to load native-hadoop library for your platform… using builtin-Java classes where applicable
```







### 3.2 导入依赖

```xml
<packaging>jar</packaging>
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>RELEASE</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <!--    <verbal>true</verbal>-->
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.4.3</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <minimizeJar>true</minimizeJar>
                    </configuration>
                </execution>
            </executions>
        </plugin>

    </plugins>
</build>
```

### 3.3 **使用**文件系统方式访问数据

##### 3.3.1 涉及的主要类

在 Java 中操作 HDFS, 主要涉及以下 Class:：

- `Configuration`：该类的对象封转了客户端或者服务器的配置

- `FileSystem`

该类的对象是一个文件系统对象, 可以用该对象的一些方法来对文件进行操作, 通过 FileSystem 的静态方法 get 获得该对象

```java
FileSystem fs = FileSystem.get(conf)
```

* `get` 方法从 `conf` 中的一个参数 `fs.defaultFS` 的配置值判断具体是什么类型的文件系统
* 如果我们的代码中没有指定 `fs.defaultFS`, 并且工程 ClassPath 下也没有给定相应的配置, `conf` 中的默认值就来自于 Hadoop 的 Jar 包中的 `core-default.xml`
* 默认值为 `file:/// `, 则获取的不是一个 DistributedFileSystem 的实例, 而是一个本地文件系统的客户端对象

##### 3.3.2  获取 FileSystem 的几种方式

```java
    /**
     * 第一种方式
     * @throws IOException
     */
    @Test
    public void getFileSystem1() throws IOException {
        Configuration configuration = new Configuration();
        //指定我们使用的文件系统类型:
        configuration.set("fs.defaultFS", "hdfs://hadoop101:8020/");

        //获取指定的文件系统
        FileSystem fileSystem = FileSystem.get(configuration);
        System.out.println(fileSystem.toString());
    }

    /**
     * 第二种方式
     * @throws Exception
     */
    @Test
    public void getFileSystem2() throws  Exception{
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop101:8020"), new Configuration(),"root");
        System.out.println("fileSystem:"+fileSystem);
    }

    /**
     * 第三种方式
     * @throws Exception
     */
    @Test
    public void getFileSystem3() throws  Exception{
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://hadoop101:8020");
        FileSystem fileSystem = FileSystem.newInstance(configuration);
        System.out.println(fileSystem.toString());
    }

    /**
     * 第四种方式
     * @throws Exception
     */
    @Test
    public void getFileSystem4() throws  Exception{
        FileSystem fileSystem = FileSystem.newInstance(new URI("hdfs://hadoop101:8020") ,new Configuration());
        System.out.println(fileSystem.toString());
    }
```

##### 3.3.3 遍历 HDFS 中所有文件

```java
/**
         * 测试遍历所有文件
         * @throws Exception
         */
@Test
public void listMyFiles() throws Exception {
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop101:8020"), new Configuration());
    RemoteIterator<LocatedFileStatus> iterator = fileSystem.listFiles(new Path("/"), true);
    while(iterator.hasNext()){
        LocatedFileStatus fileStatus = iterator.next();
        System.out.println(fileStatus.getPath().toString());
    }
    fileSystem.close();
}
```

##### 3.3.5 HDFS 上创建文件夹

```java
/**
     * 测试添加文件夹
     * @throws Exception
     */
@Test
public void mkdirs() throws Exception{
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop101:8020"), new Configuration());
    fileSystem.mkdirs(new Path("/dir2"));
    fileSystem.close();
}
```

##### 3.3.6 HDFS 文件上传

```jav
    @Test
    public void putFile() throws Exception{
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop101:8020"), new Configuration());
        fileSystem.copyFromLocalFile(new Path("file:///D://test2.txt"), new Path("/dir2"));
    }
```

# 四、一些报错

```java
Could not locate executable null \bin\winutils.exe in the hadoop binaries 
```

原因：缺少winutils.exe，复制一个winutils.exe到hadoop的bin安装目录下即可

![image-20201119191048758](C:\Users\VSUS\Desktop\笔记\大数据\img\16.png)

解决办法：在main下创建类：org.apache.hadoop.io.nativeio.NativeIO;将下面代码复制

```java

```

