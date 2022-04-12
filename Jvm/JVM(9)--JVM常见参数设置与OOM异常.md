# 一、JVM参数类型

JVM一共有3种参数类型：

- 标准参数
- X参数
- XX参数

### 1.1 标准参数

标准参数，在各个版本的JVM里面中，基本保持不变。相对比较稳定，例如：

- -help
- -server -client
- -version -showversion
- -cp -classpath

### 1.2 X参数

- Xint 解释执行
- -Xcomp：第一次使用就编译成本地代码
- -Xmixed：混合模式，JVM自己来决定是否编译成本地代码

### 1.3 XX参数

xx参数是主要我们调优需要设置的参数，XX参数分为两大类Boolean类型和kv设值类型

- Boolean类型：格式：-XX:[+-]<name>表示启用或者禁用name属性。其中+号表示启用该参数，-号表示禁用该参数。

- KV键值对类型参数

格式：-XX:<name>=<value>表示name属性的值是value，主要是以key，value形式存在。

# 二、常用参数

### 2.1 查看默认值

##### 2.1.1 jinfo

jinfo 的作用是实时地查看和调整虚拟机的各项参数。例如：需要查看jvm中元空间大小

```shell
jinfo -flag MetaspaceSize 15264(进程号)
```

![image-20201225192147441](http://cdn.noteblogs.cn/image-20201225192147441.png)

##### 2.1.2 使用命令

- java -XX:+PrintFlagsInitial 查看初始默认值

![image-20201225192335854](http://cdn.noteblogs.cn/image-20201225192335854.png)

- java -XX:+PrintFlagsInitial 查看修改更新

![image-20201225192816621](http://cdn.noteblogs.cn/image-20201225192816621.png)

查看输出的结果会发现，有一些是:=，这些是指修改过的值

* = 没有修改过
* := 修改过

### 2.2 JVM常用参数

##### 2.2.1 -Xms

-Xms等价于-XX:InitialHeapSize，初始化堆空间的大小

![image-20201225195339698](http://cdn.noteblogs.cn/image-20201225195339698.png)

默认的堆空间大小是电脑内存的1/64，我的电脑是20g的内存，所以初始化堆空间大小是320mb

使用举例：-Xms10m

![image-20201225195831573](http://cdn.noteblogs.cn/image-20201225195831573.png)

##### 2.2.2 -Xmx

-Xmx等价于-XX:MaxHeapSize，最大堆空间大小

![image-20201225200241874](http://cdn.noteblogs.cn/image-20201225200241874.png)

默认的最大堆空间大小是电脑内存的1/4，所以最大堆空间大小是5120mb

使用举例：-Xmx10m

![image-20201225200607096](http://cdn.noteblogs.cn/image-20201225200607096.png)

##### 2.2.3 -Xss

设置堆内存大小，等价于-XX:ThreadStackSize

默认值一般为512k~1024k

![image-20201226094317425](http://cdn.noteblogs.cn/image-20201226094317425.png)

这里-XX:ThreadStackSize=0为0不是说虚拟机栈的内存大小为0，而是jvm的默认值

使用举例：-Xss1024k

![image-20201226094544778](http://cdn.noteblogs.cn/image-20201226094544778.png)

##### 2.3.4 -Xmn

设置年轻代的大小，年轻代的大小一般为堆的1/3，这个参数一般不改动

##### 2.3.5 -XX:MetaspaceSize

 -XX:MetaspaceSize是设置元空间的大小，元空间并不在虚拟机上，而是使用本地内存，所以默认情况下，元空间的大小受本地内存的限制

查看元空间的初始值：

![image-20201226095643692](http://cdn.noteblogs.cn/image-20201226095643692.png)

所以，要把MetaspaceSize调大一些

##### 2.3.6 -XX:+PrintGCDetails

打印GC的信息，主要关注的点是打印的日志信息

![-XX_+PrintGCDetails打印信息](http://cdn.noteblogs.cn/-XX_+PrintGCDetails%E6%89%93%E5%8D%B0%E4%BF%A1%E6%81%AF.jpg)![image-20201227123936029](http://cdn.noteblogs.cn/image-20201227123936029.png)

GC打印之后的日志信息主要是以->分割，前是GC前，后面是GC后，括号里面是总大小

##### 2.3.7 -XX:SurvivorRatio

设置新生代中Eden区域和Survivor区域（From幸存区或To幸存区）的比例，默认值为8

![image-20201227130739072](http://cdn.noteblogs.cn/image-20201227130739072.png)

使用XX:SurvivorRatio可以设置这个比例大小

##### 2.3.8 -XX:NewRatio

设置新生代与老年代的比例，默认值为2

![image-20201227131113791](http://cdn.noteblogs.cn/image-20201227131113791.png)

使用 -XX:NewRatio可以设置这个比例大小

##### 2.3.9 -XX:MaxTenuringThreshold

控制新生代需要经历多少次GC晋升到老年代中的最大阈值，该值默认为15

![image-20201227131253986](http://cdn.noteblogs.cn/image-20201227131253986.png)

注意设置这个值，必须在0-15内，下图为我想设置为20，然后报出的错误，这个原因是因为控制这个对象晋升老年代的最大值在对象头里面，而且是用了2个bit所以最大值为15

![image-20201227131403346](http://cdn.noteblogs.cn/image-20201227131403346.png)

##### 2.3.10 总结

以一张图来总结上述参数

![JVM常用参数](http://cdn.noteblogs.cn/JVM%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0.png)



# 三、常见的异常

OOM是OutOfMemoyError ，首先明确OOM是一种错误，为jvm内存不足的一种错误

![image-20201229203844943](http://cdn.noteblogs.cn/image-20201229203844943.png)

在Java虚拟机规范中，明确规定了除了程序计数器之外的所由运行时数据区都有可能会发出OOM的异常(当然，我们口头上习惯称之为异常)

### 3.1 StackOverflowError

StackOverflowError为栈溢出异常，线程请求的栈的深度大于虚拟机所允许的深度，那么就会抛出StackOverflowError，例如：

```java
public static void main(String[] args) {
    test();
}
public static void test(){
    test();	//不断递归调用自己
}
```

![image-20201229204642034](http://cdn.noteblogs.cn/image-20201229204642034.png)

### 3.2 OOM:Java heap space

```java
/**
     * -Xms5m -Xmx5m
     * @param args
     */
public static void main(String[] args) {
    Byte[] bytes = new Byte[10 * 1024 * 1024];
}
```

![image-20201229204947624](http://cdn.noteblogs.cn/image-20201229204947624.png)

解决方案是调整堆的大小

### 3.3 OOM:GC overhead limit exceeded

当 Java 进程花费 98% 以上的时间执行 GC，但只恢复了不到 2% 的内存，且该动作连续重复了 5 次，就会抛出 `java.lang.OutOfMemoryError:GC overhead limit exceeded` 错误。简单地说，就是应用程序已经基本耗尽了所有可用内存， GC 也无法回收。这是应该要抛出异常，如果不抛出异常那么将会导致GC清理的2%的内存又马上被填满，导致GC再次执行，然后又填满，导致恶性循环

```java
/**
     * -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
     * @param args
     */
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    int i = 0;
    try {
        while(true){
            list.add(String.valueOf(i++).intern());
        }
    }catch (Throwable e){
        e.printStackTrace();
    }finally {
        System.out.println(i);
    }

}
```

![image-20201229210310101](http://cdn.noteblogs.cn/image-20201229210310101.png)

### 3.4 OOM:Direct buffer memory

Java 允许应用程序通过 Direct ByteBuffer 直接访问堆外内存，许多高性能程序通过 Direct ByteBuffer 结合内存映射文件（Memory Mapped File）实现高速 IO。Javad的NIO程序中使用ByteBuffer来读取或者写入数据，ByteBuffer有两种方式：

- allocate：分配JVM堆内存，属于GC的回收范围，但是由于要从本地内存中拷贝所以速度较慢
- allocateDirect：直接分配堆外内存，也就是本地内存，传输速度较快

```java
/**
     * -XX:MaxDirectMemorySize=5m
     * @param args
     */
public static void main(String[] args) {
    ByteBuffer.allocateDirect(6 * 1024 * 1024);
}
```

![image-20201229211855036](http://cdn.noteblogs.cn/image-20201229211855036.png)

### 3.5 OOM:Unable to create new native thread

JVM 向操作系统请求创建 native 线程失败，就会抛出 `Unable to create new nativethread`，常见的原因包括以下几类：

1、线程数超过操作系统最大线程数 ulimit 限制；

2、线程数超过 kernel.pid_max（只能重启）；

3、native 内存不足；

```java
public class UnableCreateNewThreadDemo {
    public static void main(String[] args) {
        int i = 0;
        try {
            for (;;i++) {
                new Thread(() -> {
                    //防止进程醒来
                    try { TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);} catch (InterruptedException e) {e.printStackTrace();}
                }).start();
            }
        }catch (Throwable e){
            e.printStackTrace();
        }finally {
            System.out.println(i);
        }
    }
}
```

在Centos7上运行上面java程序，注意：运行此程序需要使用普通用户，使用root可能会导致一直创建进程导致卡机

![image-20201230212644363](http://cdn.noteblogs.cn/image-20201230212644363.png)

发现创建了4085个进程，便报出了Unable to create new native thread异常

解决上面问题有两种方案：

- 增大操作系统所允许的最大进程数，以Centos7为例

```shell
[test@hadoop101 ~]$ ulimit -u	#查看当前用户所能创建的最大进程数
```

![image-20201230213004120](http://cdn.noteblogs.cn/image-20201230213004120.png)

可以发现，其实和上面的4085有些出入，java程序在运行时还会有一些守护进程

通过修改配置文件/etc/security/limits.d/20-nproc.conf可以增大普通用户可以创建的最大进程数

![image-20201230214510271](http://cdn.noteblogs.cn/image-20201230214510271.png)

由上面也可以看到普通用户所具有的最大进程数也就是4096个，那么可以修改这个值来增加进程数

- 第二种方法，减少不必要的进程

### 3.6 OOM:Metaspace

元空间存放的是类的结构信息，元空间并不在JVM中，而是在本地内存中，但是元空间的默认大小才接近21mb左右

![image-20201230220036713](http://cdn.noteblogs.cn/image-20201230220036713.png)

所以，有可能出现OOM的情况，看下面代码：

```java
public class MetaspaceDemo extends ClassLoader{
    /**
     *jdk1.8 环境配置如下参数
     * -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
     * @param args
     */
    public static void main(String[] args) {
        int j = 0;
        try {
            MetaspaceDemo test = new MetaspaceDemo();
            for (int i = 0; i < 10000; i++) {
                //创建ClassWriter对象，用于生成类的二进制字节码
                ClassWriter classWriter = new ClassWriter(0);
                //指明版本号，修饰符，类名，包名，父类，接口
                classWriter.visit(Opcodes.V1_6, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                //返回byte[]
                byte[] code = classWriter.toByteArray();
                //类的加载
                test.defineClass("Class" + i, code, 0, code.length);//Class对象
                j++;
            }
        } finally {
            System.out.println(j);
        }
    }
}
```

![image-20201230220221569](http://cdn.noteblogs.cn/image-20201230220221569.png)