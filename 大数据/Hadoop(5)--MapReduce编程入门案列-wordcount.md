# 一、MapReduce 介绍

- Map负责“分”，即把复杂的任务分解为若干个“简单的任务”来并行处理。可以进行拆分的
  前提是这些小任务可以并行计算，彼此间几乎没有依赖关系。
- Reduce负责“合”，即对map阶段的结果进行全局汇总。

MapReduce是一个分布式运算程序的编程框架，核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在Hadoop集群上。

MapReduce设计并提供了统一的计算框架，为程序员隐藏了绝大多数系统层面的处理细节。为程序员提供一个抽象和高层的编程接口和框架。程序员仅需要关心其应用层的具体计算问题，仅需编写少量的处理应用本身计算问题的程序代码。如何具体完成这个并行计算任务所相关的诸多系统层细节被隐藏起来,交给计算框架去处理：

Map和Reduce为程序员提供了一个清晰的操作接口抽象描述。MapReduce中定义了如下的Map和Reduce两个抽象的编程接口，由用户去编程实现，Map和Reduce，MapReduce处理的数据类型是<key,value>键值对。

* Map: `(k1,v1) → [(k2,v2)]`

* Reduce: `(k2,[v2]) → [(k3,v3)]`

一个完整的mapreduce程序在分布式运行时有三类实例进程：

1. `MRAppMaster` 负责整个程序的过程调度及状态协调
2. `MapTask` 负责map阶段的整个数据处理流程
3. `ReduceTask` 负责reduce阶段的整个数据处理流程

# 二、MapReduce 编程规范

> MapReduce 的开发一共有八个步骤, 其中 Map 阶段分为 2 个步骤，Shuffle 阶段 4 个步骤，Reduce 阶段分为 2 个步骤

#####  Map 阶段 2 个步骤

1. 设置 InputFormat 类, 将数据切分为 Key-Value**(K1和V1)** 对, 输入到第二步
2. 自定义 Map 逻辑, 将第一步的结果转换成另外的 Key-Value（**K2和V2**） 对, 输出结果

##### Shuffle 阶段 4 个步骤

3. 对输出的 Key-Value 对进行**分区**
4. 对不同分区的数据按照相同的 Key **排序**
5. (可选) 对分组过的数据初步**规约**, 降低数据的网络拷贝
6. 对数据进行**分组**, 相同 Key 的 Value 放入一个集合中

##### Reduce 阶段 2 个步骤

7. 对多个 Map 任务的结果进行排序以及合并, 编写 Reduce 函数实现自己的逻辑, 对输入的 Key-Value 进行处理, 转为新的 Key-Value（**K3和V3**）输出
8. 设置 OutputFormat 处理并保存 Reduce 输出的 Key-Value 数据

# 三、wordcount案例

##### 1.分析需求

```shell
hello,world,hadoop
hive,sqoop
flume,hello
kitty,tom,jerry,world
hadoop
......
```

要将上面的word.txt(大小为300mb)统计出现的单词个数，变成如下：

```java
hello  2
word  2
hadoop 2
hive 1
sqoop 1
.....
```

##### 2.准备输入文件

##### 3.map阶段

![image-20201118183726706](C:\Users\VSUS\Desktop\笔记\大数据\img\11.png)

Map阶段：word.txt存在HDFS上，被切分为3个blk，通过TextInputFormat输入，生成了键值对<K1,V1>，分别代表<行偏移量,每一行的文本>，例如上图。进入map阶段后执行自定义逻辑，将<K1,V1>转为<K2,V2>

```java
public class WordCountMap extends Mapper<LongWritable, Text, Text, LongWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        Text key2 = new Text();
        LongWritable val2 = new LongWritable();
        String[] split = value.toString().split(",");
        for(String word : split){
            key2.set(word);
            val2.set(1);
            context.write(key2, val2);
        }
    }
}
```

##### 4.shuffer阶段

wordcount案例通过分析输入与输出，在shuffer阶段使用默认即可，

![image-20201118184658494](C:\Users\VSUS\Desktop\笔记\大数据\img\12.png)

##### 5. Reducer阶段

通过shuffle阶段，生成新的<K2,V2>，不同分区的文件被派发到不同的ReduceTask，然执行自定义的Reduce逻辑

![image-20201118184852631](C:\Users\VSUS\Desktop\笔记\大数据\img\13.png)

```java
public class WordCountReduce extends Reducer<Text, LongWritable, Text, LongWritable> {
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        Long count = 0L;
        for (LongWritable value : values) {
            count += value.get();
        }
        context.write(key, new LongWritable(count));
    }
}
```

##### 6.定义主类, 描述 Job 并提交 Job

```java
public class JobMain extends Configured implements Tool {


    @Override
    public int run(String[] strings) throws Exception {
        Job job = Job.getInstance(super.getConf());
        job.setJarByClass(JobMain.class);
        //第一步：读取输入文件解析成key value对
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("hdfs://hadoop101:8020/wordcount"));

        //第二步：设置mapper类
        job.setMapperClass(WordCountMap.class);
        //设置我们map阶段完成之后的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        //第三步，第四步，第五步，第六步，省略

        //第七步：设置reduce类
        job.setReducerClass(WordCountReduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        //第八步：设置输出类以及输出路径
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, new Path("hdfs://hadoop101:8020/wordcount/out"));

        boolean b = job.waitForCompletion(true);

        return b ? 1 : 0;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

# 四、MapReduce运行模式

##### 4.1 本地运行模式

1. MapReduce 程序是被提交给 LocalJobRunner 在本地以单进程的形式运行
2. 处理的数据及输出结果可以在本地文件系统, 也可以在hdfs上
3. 怎样实现本地运行? 写一个程序, 不要带集群的配置文件, 本质是程序的 `conf` 中是否有 `mapreduce.framework.name=local` 以及 `yarn.resourcemanager.hostname=local` 参数
4. 本地模式非常便于进行业务逻辑的 `Debug`, 只要在 `idea` 中打断点即可

```java
configuration.set("mapreduce.framework.name","local");
configuration.set(" yarn.resourcemanager.hostname","local");
TextInputFormat.addInputPath(job,new Path("file:///输入的文件路径"));
TextOutputFormat.setOutputPath(job,new Path("file:///输出的文件路径"));
```

##### 4.2 集群运行模式

1. 将 MapReduce 程序提交给 Yarn 集群, 分发到很多的节点上并发执行
2. 处理的数据和输出结果应该位于 HDFS 文件系统
3. 提交集群的实现步骤: 将程序打成JAR包，然后在集群的任意一个节点上用hadoop命令启动

```shell
 hadoop jar original-WordCount-1.0-SNAPSHOT.jar com.hadoop.JobMain
```

注意：输出路径一定不能存在！

输出结果为：

```
flume	1
hadoop	2
hello	2
hive	1
jerry	1
kitty	1
sqoop	1
tom	1
world	2
```

