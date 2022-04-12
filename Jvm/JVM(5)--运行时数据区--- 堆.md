参考：《深入理解Java虚拟机》

​			《宋红康JVM教程》

### 一、概述

在介绍堆之前，简单的回顾一下运行时数据区的各个部分组成：
![](http://cdn.noteblogs.cn/7.PNG)

堆是Java虚拟机所管理的最大的内存区域，由上图也可见，堆是线程共享的一个区域，在虚拟机启动时创建，==堆的唯一作用便是存放对象实例==

==几乎所有的对象都在堆上分配内存===，这里的描述是“几乎”，因为随着栈上分配，标量替换这些优化技术，所有对象的分配在堆上不是那么“绝对”。

堆，是垃圾收集器管理的主要区域，虚拟机栈上不存在垃圾回收。

堆是运行时数据区的线程共享区域，但是在这里还可以划分线程私有的缓冲区（TLAB:Thread Local Allocation Buffer）

### 二、分配对象的过程

##### 2.1 堆的内部结构框架

堆的内部结构在jdk7之后发生了比较的变化。

==在jdk7以前：新生区+养老区+永久区==

- Young Generation Space：又被分为Eden区和Survior区  ==Young/New==
- Tenure generation Space
- Permanent Space

![](http://cdn.noteblogs.cn/27.png)

==在jdk7之后：新生区+养老区+元空间：==

- Young Generation Space：又被分为Eden区和Survior区  ==Young/New==
- Tenure generation Space：        
- Meta Space：  

![](http://cdn.noteblogs.cn/28.png)

##### 2.2 对象分配过程

由上图可以知道，==堆区进一步细分可以划分为新生区、老年区，元空间其实是用来存储class的信息的，是方法区的实现，新生区又可以划分为Eden（伊甸园区）、Survivor0和Survivor1区（也可以叫做from、to区）。==

通过下面的程序来解释对象在内存的具体分配过程：

```java
	/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/26 20:35
 */
public class HeapInstanceTest {
    //创建一个数组 0-200k之间
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) {
        //创建一个list集合
        ArrayList<HeapInstanceTest> list = new ArrayList<HeapInstanceTest>();
        while (true) {
            //list集合里面一直new HeapInstanceTest
            list.add(new HeapInstanceTest());
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

以上程序执行，很显然会报一个异常OOM(这里说的异常时广义上的异常)：

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.java.HeapInstanceTest.<init>(HeapInstanceTest.java:12)
	at com.java.HeapInstanceTest.main(HeapInstanceTest.java:17)
```

堆空间分配对象的过程：

1. 首先，new的对象被放在新生区的Eden区，Eden区有一个空间限制，具体的大小看后面章节
2. 当Eden区的空间被对象填满时，JVM的垃圾回收机制会触发Minor GC（也叫YGC），将Eden区的不需要使用的垃圾对象进行回收，具体的垃圾回收算法在后面章节。
3. Minor GC会将Eden区中不需要被使用的对象进行回收处理，而需要被使用的对象进行上升，也就是提升到Survivor0区，同时维护一个标记（年龄计数器），将这些对象标记为1。==也就是说，经过一个Minor GC后的Eden区一定是空的==
4. 此时，Eden区已经为空，可以再次分配对象在Eden区上，当Eden区又再次被对象填满时再次触发Minor GC，此时Minor GC有两项工作：①判断Survivor0区中是否有垃圾需要进行回收，有垃圾那么进行回收，然后将剩下的对象==提升到Survivor1区，同时将该标记++（年龄计数器，此时为2）==，那么经过这个过程，Survivor0区为空；②判断Eden区中是否有垃圾进行回收，将剩余的对象分配到Survivor1区中，==注意此时不能分配到Survivor0区==，同时将年龄计数器设置为1
5. 经过一阵时间后，Survivor0区或者Survivor1区出现了年龄计数器为==15==的老对象，那么恭喜，这些对象将会再次晋升，提升到养老区（老年区），可以通过设置参数：-XX:MaxTenuringThreshold=进行设置年龄计数器的大小
6. 在养老区，相对悠闲。当老年区内存不足时，再次触发GC：Major GC，进行养老区的内存清理。
7. 若养老区执行了Major GC之后发现依然无法进行对象的保存，就会产生OOM异常。

##### 2.3 特殊对象分配

上面7个过程是一般的对象分配过程，其中可能会因为对象的大小不同会有一些其他的问题，例如：如果对象太大，Eden区一开始就放不下呢？如果出现Minor GC之后，将Eden区的剩余对象放入Survivor区时，Survivor区放不下？等等这些会通过下面流程图来详细解答

![](http://cdn.noteblogs.cn/29.png)



新对象申请时，首先判断Eden是否放得下，如果放得下那么该对象分配至Eden中，如果放不下需要进行Minor GC，Minor GC会将Eden中的对象进行清理，将不用的对象回收，需要使用的对象防止在Survivor区中，如果进行Minor GC后，Eden放得下，那么分配对象，如果还放不下，那么判断老年代是否放的下，如果放得下，那么分配对象，如果放不下，进行FGC，如果在FGC之后还是放不下，则报出OOM

在YGC的过程中，需要将Eden区中的可用对象放到Survivor区，如果放不下，则将这些对象直接晋升老年代，不需要年龄计数器达到15，如果放得下，那么检查该对象的年龄计数器，如果大于165，将该对象晋升老年代。

### 三、内存分配策略

##### 3.1 查看堆的内存空间

在默认情况下，堆的初始空间大小是：物理内存大小/64 ，最大内存大小是：物理内存大小/4，

```java
/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/16 16:21
 */
public class HeapSpaceInitial {
    public static void main(String[] args) {

        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() /1024 /1024 ;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        System.out.println("-Xms : " + initialMemory + "M");//-Xms :  123M
        System.out.println("-Xmx : " + maxMemory + "M");//-Xmx : 1796M

        System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");//系统内存大小为：7.6875G
        System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");//系统内存大小为：7.015625G

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


```

通过上面程序计算出来的系统空间相差比较大，原因是：通过分配对象的过程可以知道，Survivor有两个区域，但是每次只能使用其中一个区域，所以最大可以使用空间是没有将另外一个Survivor计算上去的。

> 可以在cmd窗口上使用jstat -gc 进程id 查看各个区域的内存大小

![](http://cdn.noteblogs.cn/30.png)

后面有个U的代表以及使用，那么123m是如下计算出来的：（S0C(或者S1C) + EC + OC)/1024 = 123MB

这里也证实了S区中只能使用其中一个区域 

##### 3.2 设置堆空间大小

参数：

- -Xms 设置堆的起始内存
- -Xmx 设置堆的最大内存

注意：一般会将-Xms和-Xmx两个参数配置相同的值，其目的就是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能

##### 3.3 堆内部结构空间大小划分

堆内部有新生区，养老区；新生区内有Eden区，S0，S1区，那么这些结构分别占堆整个空间大小的多少呢？

1. **新生区与老年区占比**

   新生区与老年区的大小占比，可以使用参数：==-XX:NewRatio==，查看官方文档：

   https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html可知，

> -XX:NewRatio=*ratio*
>
> Sets the ratio between young and old generation sizes. By default, this option is set to 2. The following example shows how to set the young/old ratio to 1:
>
> -XX:NewRatio=1

-XX:NewRatio参数的默认值是2，也就是新生区 ： 老年区 = 2，可以使用该参数来改变这个占比值

2. **新生区内结构占比**

   参数： -XX:SurvivorRatio 

   默认值是8，参考官方文档

> -XX:SurvivorRatio=*ratio*
>
> Sets the ratio between eden space size and survivor space size. By default, this option is set to 8. The following example shows how to set the eden/survivor space ratio to 4:
>
> -XX:SurvivorRatio=4

也就是Eden ： S0 ： S1 = 8 ： 1 ：1

例如：

```java
/**
 * -Xms30m -Xmx30m
 */
public class HeapDemo1 {
    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("end...");
    }
}

```

使用VisualVM工具查看各个空间占比

![](http://cdn.noteblogs.cn/31.png)

就是和上面两个参数的默认值一致

##### 3.4 堆空间分代思想

为什么要把Java堆分代？不分代就不能正常工作了么

* 经研究，不同对象的生命周期不同。70%-99%的对象都是临时对象。
  * 新生代：有Eden、Survivor构成（s0,s1 又称为from to），to总为空
  * 老年代：存放新生代中经历多次依然存活的对象
* 其实不分代完全可以，分代的唯一理由就是优化GC性能。如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。GC的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描，而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。

### 四、堆不是分配对象的唯一选择

“几乎”所有的对象都在堆上分配，但是不是所有的对象都在堆上分配，如果一个对象经过逃逸分析后发现这个对象没有逃逸出方法的话，很有可能这个对象会被优化为“栈上分配”，那么对象所占用的空间就会随栈帧的出栈而销毁。

##### 4.1 逃逸分析

当一个对象在方法中定义之后，只能在方法内部使用，则认为没有发生逃逸；否则认为是发生了逃逸。

```java
public void method(){
    V v = new V();
    //use V
    //......
    v = null;
}
```

上述例子中，对象V只能在方法内部使用，则认为对象V是没有发生逃逸的

* 在JDK 6u23版本之后，HotSpot中默认就已经开启了逃逸分析
* 如果使用了较早的版本，开发人员可以通过
  * -XX:DoEscapeAnalysis 显式开启逃逸分析
  * -XX:+PrintEscapeAnalysis查看逃逸分析的筛选结果

##### 4.2 代码优化

通过逃逸分析，编译器可以做以下优化：

- 栈上分配：将堆分配转化为栈分配。如果一个对象在子线程中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配

- 同步省略：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步

- 分离对象或标量替换：有的对象可能不需要作为一个连续的内存结构存在也可以北方问道，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

下面只说明栈上分配，例如如下代码：

```java
/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/23 21:54
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();//未发生逃逸
    }

    static class User {

    }
}

```

由于使用的是jdk1.8版本，那么默认开启逃逸分析，运行代码

![](http://cdn.noteblogs.cn/32.png)

显式关闭逃逸分析，使用-XX:-DoEscapeAnalysis参数，则代码运行的结果为

![](http://cdn.noteblogs.cn/33.png)

由此可见，通过逃逸分析后栈上分配，代码运行速度提高不小！