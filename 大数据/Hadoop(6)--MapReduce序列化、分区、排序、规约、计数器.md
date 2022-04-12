# 一、MapReduce分区Partitioner

![](C:\Users\VSUS\Desktop\笔记\大数据\img\12.png)

在wordcount案例中，没有进行分区，也就是shuffer阶段的分区，mapreduce 默认的分区方式是`hashPartition`，在这种分区方式下，KV对根据key的hashcode值与reduceTask个数进行取模，决定该键值对该要访问哪个ReduceTask

```java
public int getPartition(K2 key, V2 value, int numReduceTasks) {
    return (key.hashCode() & 2147483647) % numReduceTasks;
}
```

![image-20201118193824231](C:\Users\VSUS\Desktop\笔记\大数据\img\14.png)

在 MapReduce 中, 通过我们指定分区, 会将同一个分区的数据发送到同一个 Reduce 当中进行处理

例如: 为了数据的统计, 可以把一批类似的数据发送到同一个 Reduce 当中, 在同一个 Reduce 当中统计相同类型的数据, 就可以实现类似的数据分区和统计等，其实就是相同类型的数据, 有共性的数据, 送到一起去处理

`Reduce 当中默认的分区只有一个`

下面通过一个案例来理解MapReduce分区的作用

##### 1.1 案例分析

![image-20201118194119621](C:\Users\VSUS\Desktop\笔记\大数据\img\15.png)

有一个partition.csv文件，文件部分内容如上，要将该文件中红框中的数字大于15在一个文件，小于15在一个文件

要完成这个案例，可以使用分区技术，MapReduce自定义逻辑的核心，便是要明白个各个阶段的K和V

- Map阶段：<k1,v1>：<行偏移量,每行的数据>

  ​					<K2,V2>：<每行的数据,null>

- Shuffer阶段：将k2进行分区
- Reduce阶段：<K3,V3>：<每行的数据,null>

##### 1.2 Map阶段

```java

public class Mapper extends org.apache.hadoop.mapreduce.Mapper<LongWritable,Text,Text, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        context.write(value,NullWritable.get());
    }
}
```

##### 1.3 Parttition分区

```java
public class MyPartitioner extends Partitioner<Text, NullWritable> {
    @Override
    public int getPartition(Text text, NullWritable nullWritable, int i) {
        String[] split = text.toString().split("\t");
        String num = split[5];
        if(Integer.parseInt(num) > 15){
            return 1;
        }else {
            return 0;
        }
    }
}
```

##### 1.4 Reduce阶段

```java

public class Reducer extends org.apache.hadoop.mapreduce.Reducer<Text, NullWritable,Text,NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        context.write(key, NullWritable.get());
    }
}
```

##### 1.5 主类

```java
public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] strings) throws Exception {
        //获取job
        Job job = Job.getInstance(super.getConf(), "Partition_Job");
        //设置输入格式
        job.setInputFormatClass(TextInputFormat.class);
        //TextInputFormat.addInputPath(job, new Path(("file:///D:\\input")));
        TextInputFormat.addInputPath(job, new Path(("hdfs://hadoop101:8020/input")));
        //设置mapper类
        job.setMapperClass(Mapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);
        //设置分区

        /**
         * 设置分区类，以及reduceTask的个数，注意reduceTask的个数一定要与
         * 分区数保持一致
         */
        job.setPartitionerClass(MyPartitioner.class);
        job.setNumReduceTasks(2);

        //排序，规约，分组默认
        //设置reduce类
        job.setReducerClass(Reducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        //设置输出格式
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, new Path("hdfs://hadoop101:8020/out"));
        //TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out"));

        //等待进程结束
        boolean b = job.waitForCompletion(true);
        return b ? 1 : 0;

    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        ToolRunner.run(conf, new JobMain(), args);
    }
}
```

# 二、MapReduce排序和序列化

- 序列化 (Serialization) 是指把结构化对象转化为字节流
- 反序列化 (Deserialization) 是序列化的逆过程. 把字节流转为结构化对象. 当要在进程间传递对象或持久化对象的时候, 就需要序列化对象成字节流, 反之当要将接收到或从磁盘读取的字节流转换为对象, 就要进行反序列化
- Java 的序列化 (Serializable) 是一个重量级序列化框架, 一个对象被序列化后, 会附带很多额外的信息 (各种校验信息, header, 继承体系等）, 不便于在网络中高效传输. 所以, Hadoop 自己开发了一套序列化机制(Writable), 精简高效. 不用像 Java 对象类一样传输多层的父子关系, 需要哪个属性就传输哪个属性值, 大大的减少网络传输的开销
- Writable 是 Hadoop 的序列化格式, Hadoop 定义了这样一个 Writable 接口. 一个类要支持可序列化只需实现这个接口即可
- 另外 Writable 有一个子接口是 WritableComparable, WritableComparable 是既可实现序列化, 也可以对key进行比较, 我们这里可以通过自定义 Key 实现 WritableComparable 来实现我们的排序功能

##### 2.1 案例分析

数据格式如下：

```
a	1
a	9
b	3
a	7
b	8
b	10
a	5
```

需求：

- 第一列按照字典顺序进行排列
- 第一列相同的时候, 第二列按照升序进行排列

思路：将V1封装为对象，实现WritableComparable接口，重写里面的compareTo方法，Map阶段<SortBean,Null> ，Reduce阶段<SortBean,Null> 

##### 2.2  Map阶段

```java
public class Mapper extends org.apache.hadoop.mapreduce.Mapper<LongWritable, Text, SortBean, NullWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] split = value.toString().split("\t");
        SortBean sortBean = new SortBean();
        sortBean.setNum(Integer.parseInt(split[1]));
        sortBean.setWord(split[0]);
        context.write(sortBean,NullWritable.get());
    }
}

```

##### 2.3 自定义类型和比较器

```java
public class SortBean implements WritableComparable<SortBean> {
    private String word;
    private Integer num;

    @Override
    public String toString() {
        return "SortBean{" +
                "word='" + word + '\'' +
                ", num=" + num +
                '}';
    }

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(word);
        out.writeInt(num);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.word = in.readUTF();
        this.num = in.readInt();
    }

    @Override
    public int compareTo(SortBean o) {
        int res = this.word.compareTo(o.getWord());
        return res == 0 ? this.num - o.getNum() : res;
    }
}
```

##### 2.4 Reduce阶段

```java
public class Reduce  extends Reducer<SortBean, NullWritable,SortBean,NullWritable> {
    @Override
    protected void reduce(SortBean key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        context.write(key, NullWritable.get());
    }
}
```

##### 2.5 主类

```java
public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] strings) throws Exception {
        //获取job
        Job job = Job.getInstance(super.getConf(), "Partition_Job");
        //设置输入格式
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path(("file:///D:\\input")));
        //TextInputFormat.addInputPath(job, new Path(("hdfs://hadoop101:8020/input")));
        //设置mapper类
        job.setMapperClass(Mapper.class);
        job.setMapOutputKeyClass(SortBean.class);
        job.setMapOutputValueClass(NullWritable.class);
        //分区排序，规约，分组默认
        //设置reduce类
        job.setReducerClass(Reduce.class);
        job.setOutputKeyClass(SortBean.class);
        job.setOutputValueClass(NullWritable.class);

        //设置输出格式
        job.setOutputFormatClass(TextOutputFormat.class);
        //TextOutputFormat.setOutputPath(job, new Path("hdfs://hadoop101:8020/out"));
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out"));

        //等待进程结束
        boolean b = job.waitForCompletion(true);
        return b ? 1 : 0;

    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        ToolRunner.run(conf, new JobMain(), args);
    }
}
```

运行结果：这里结果与源文件不一致，主要是toString方法

```jav
SortBean{word='a', num=1}
SortBean{word='a', num=5}
SortBean{word='a', num=7}
SortBean{word='a', num=9}
SortBean{word='b', num=3}
SortBean{word='b', num=8}
SortBean{word='b', num=10}
```

# 三、combiner规约

每一个 map 都可能会产生大量的本地输出，Combiner 的作用就是对 map 端的输出先做一次合并，以减少在 map 和 reduce 节点之间的数据传输量，以提高网络IO 性能，是 MapReduce 的一种优化手段之一

编程步骤：

1. 自定义一个 combiner 继承 Reducer，重写 reduce 方法
2. 在 job 中设置 `job.setCombinerClass(CustomCombiner.class)`

combiner 能够应用的前提是不能影响最终的业务逻辑，而且，combiner 的输出 kv 应该跟 reducer 的输入 kv 类型要对应起来



