## 一、Synchronized的特性

在高并发编程中，线程安全是需要重点关注的话题，而造成线程安全的方面有两点：

- 需要有共享资源或者叫临界资源
- 多个线程同时操作

满足了上面两个条件，就有可能会有线程安全的问题，解决的办法很简单，就是让每一时刻操作这个共享变量的线程控制在一个即可，也就是互斥锁。Synchronized就是一种互斥锁，Synchronized可以保证在同一时刻只有一个线程进入到被锁住的临界资源中，Synchronized也具有以下特点：

- 原子性：这是Synchronized能保证线程安全的重要前提，原子性的操作要么一起成功，或者完全失败，也就是中间操作是不可被分割的，例如i++这一代码，其实底层字节码分为三个步骤读取、计数、赋值，如果加入了sync修饰的话，这三个操作就是一体的是不可被分割的。`volatile和sync的重要区别之一就是sync具有原子性和volatile不具有原子性`

- 可见性：可见性就是说sync锁住的代码中对任意共享变量的操作都是对其他线程可见的，这里也涉及到Java的线程模型，JMM(java线程模型)规定所有的共享变量都必须存在主存中，而每一个线程都具有一个工作区，线程对共享变量的操作都必须在工作区中进行，也就是将主存中的共享变量复制一份到自己的工作区中，修改之后又放回工作区中，每个线程之间的互通数据必须通过主存来完成，不同工作区之间也无法传递数据
- 有序性：Java为了提高程序的运行效率会对指令程序进行重新排序，但是在高并发下这种的重新排序很有可能会改变程序的运行结果，所以sync保证了有序性也就是不让JVM进行指令重排序
- 可重入性：就是一个线程拥有了当前的锁，还可以重复申请获取当前锁，`synchronized和ReentrantLock都是可重入锁`

Synchronized一直被称为重量级锁，但是在jdk1.6之后为了减少锁和释放锁带来的性能消耗，而引入了偏向锁和轻量级锁

以及锁的存储结构和升级过程

## 二、Synchronized锁同步原理

### 1.锁住同步代码块

```java
public class SynchronizedDemo {
    Object lock = new Object();
    public void test1(){
        synchronized (lock){};
    }
}
```

看JVM底层是如何实现的，最直接的方法是使用javap指令查看字节码文件

```java
  public void test1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #3                  // Field lock:Ljava/lang/Object;
         4: dup
         5: astore_1	
         6: monitorenter		// !!! 关注这里
         7: aload_1
         8: monitorexit			// !!! 关注这里
         9: goto          17
        12: astore_2
        13: aload_1
        14: monitorexit
        15: aload_2
        16: athrow
        17: return
      Exception table:
         from    to  target type
             7     9    12   any
            12    15    12   any
      LineNumberTable:
        line 6: 0
        line 7: 17
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ class cn/java/SynchronizedDemo, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
```

从反编译的结果来看，在同步方法的前后分别有monitorenter和monitorexit，看这两个单词就可以大致知道就是一个进入monitor和退出monitor的两个操作，至于monitor是什么后续会继续讲到，现在可以理解为就是一个锁，而且这个锁是每个对象一被new出来就带着的

`monitorenter的过程：`

- 如果一个对象的monitor进入数为0，那么该线程进入monitor，并且将monitor进入数加1，则该线程即为monitor的使用者
- 如果一个线程已经持有monitor，只是想重新获取，那么monitor进入数再次加1，这就是可重入性的实现原理
- 如果一个线程发现monitor的进入数不为0，即monitor已经被其他线程占用，那么该线程将会堵塞直到monitor的进入数为0，再次获取monitor的持有权

`monitorexit的过程：`

- 执行monitorexit的线程必须是monitor的持有者，指令执行时，monitor的进入数减1，如果monitor的数为0，那么说明该线程将退出monitor，不在持有monitor的使用权，此时其他线程可以获取monitor的使用权

### 2.锁住同步方法

```java
public class SynchronizedDemo {
    synchronized public void test2(){}
}
```

同样的使用javap反编译代码

```java

  public synchronized void test2();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 11: 0
```

从上面的反编译的结果分析没有使用到monitorenter和monitorexit但是在常量池中多了一个标记ACC_SYNCHRONIZED，直接找到官方java虚拟机规范文档中是这么描述的

> A `synchronized` method is not normally implemented using *monitorenter* and *monitorexit*. Rather, it is simply distinguished in the run-time constant pool by the `ACC_SYNCHRONIZED` flag, which is checked by the method invocation instructions ([§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10)).
>
> 同步方法通常不是使用monitorenter和monitorexit实现的。相反，它只是在运行时常量池中通过ACC_SYNCHRONIZED标志来区分，该标志由方法调用指令检查



可以在接着看2.11.10章节，官方文档描述的很详细。方法级同步是隐式执行的。当调用了设置ACC_SYNCHRONIZED标记的方法时，执行线程将要获取monitor，获取成功之后才能执行方法体，方法执行完毕之后释放monitoir。

也就是说在本质上，Synchronized锁住的同步方法和同步代码块都是靠monior来实现的，只不过同步方法只是一种隐式的执行

### 3.java对象头

不特别说明，以下说的都是Hostop的虚拟机。对象在虚拟机内存的布局可以分为三块：

- 对象头
- 实例数据
- 填充数据

与Synchronized相关的是对象头里面的数据，填充数据不是必须部分，只是要求对象起始地址必须是8字节的整数倍，填充数据仅仅是为了使字节对齐

对象头的主要结构是由`Mark Word(标记字段)`、`Class Metadata Address(类型指针)`，Class Metadata Address存储到对象类型数据的指针，虚拟机便是通过这个指针来确定是哪个对象的实例，如果对象是数组的话，还会额外存储一个`Array length`数组长度的值

| 长度     | 内容                   | 说明                       |
| -------- | ---------------------- | -------------------------- |
| 32/64bit | Mark Word              | 存储对象的hashcode或锁信息 |
| 32/64bit | Class Metadata Address | 存储对象类型数据的指针     |
| 32/32bit | Array length           | 数组的长度                 |

在64位虚拟机下，Mark Word是64bit的，其存储结构如下

![img](http://cdn.noteblogs.cn/2062729-5f6d3993ba018942.png)

JVM根据锁标志位的不同，Mark Word中存储的数据也会随之变化，例如：

![img](http://cdn.noteblogs.cn/2062729-c63ff6c2d337ad5f.png)

其中的偏向锁、轻量级锁、重量级锁后面讲到锁的优化的时候会涉及到，目前只是介绍Mark Word这个对象头中可以存放什么数据

### 4.monitor

在Synchronized锁住代码块或者方法时，都涉及到一个monitor，根据Java万物皆对象，monitor可以理解为一个对象，也可以理解为一种工具这种工具可以保证Synchronized锁的同步机制。所有的Java对象都自带了monitor对象

可以在虚拟机源码中找到monitor的源码：

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

主要是要关注上面的两个容器waitSet和EntryList用来保存当前由于某种原因正在等待的线程，_owner指向了持有当前monitor的线程。当有多个线程同时想要获取锁时，处理过程如下：

1. 线程进入_EntryList集合里面进行等待，当线程获取到对象的monitor'时，把monitor中的owner变量设置为当前线程，并将count变量++
2. 当线程调用wait()方法，释放当前的monitor的持有权，并将owner恢复为null，count减1，同时该线程进入waitSet集合中等待被唤醒
3. 当当前使用权的线程执行代码完毕，释放当前锁，owner恢复为null吗，count减1，以便让其他线程可以获取



## 三、锁的优化

释放锁和获取锁设涉及到操作系统底层的内核态和用户态之间的切换，而这种切换是非常消耗资源的，Jdk1.6为了减少获得锁和释放锁而带来的性能消耗，引入了"偏向锁"和”轻量级锁"这些锁。这些锁其实就是使用Synchronized在不同的应用场景下的不同状态，在Mark Word中的锁标志位也保存了当前处于何种状态的锁

![image-20210314161134325](http://cdn.noteblogs.cn/image-20210314161134325.png)

锁的升级过程为上面那副图所示，需要注意的是这种升级过程是不可逆的，也就是只能按照箭头方向进行升级，大致的流程为

![image-20210314162037441](http://cdn.noteblogs.cn/image-20210314162037441.png)

### 1.偏向锁

对象头是由`Mark Word(标记字段)`、`Class Metadata Address(类型指针)`组成的，当一个线程持有了这个对象的monitor，那么就将Mark Word的标记字段中的锁标记为改为01，锁进入到偏向模式

![image-20210314162914542](http://cdn.noteblogs.cn/image-20210314162914542.png)

这种模式只是让同一线程获得锁的代价更低，也就是说在单线程模式下使用，如果在高并发的环境下，偏向锁关闭，多个线程开始竞争向锁，锁升级为轻量级锁

### 2.轻量级锁

在竞争锁之前，JVM会在当前线程的栈帧中创建用于存储锁记录的空间(Lock Record)，并将对象头中的Mark Word复制到锁记录中。当线程尝试获取锁时，使用CAS将对象头中的Mark Word替换为指向锁记录的指针，如果成功则获取锁，如果失败表示其他资源竞争锁，当前线程便尝试使用自旋来获取锁

![image-20210314164719709](http://cdn.noteblogs.cn/image-20210314164719709.png)

### 3.自旋锁

用户态和内核态之间的切换资源消耗比较大，而JVM为了减少这种消耗引入了自旋锁，一旦没有获取到资源就一直自旋等待，而不是直接挂起堵塞，虽然这样会消耗CPU资源，但是这种消耗比用户态和内核态之间的切换要小，但是如果线程一直获取不到锁那么也不能一直自旋下去，自旋需要有一个次数的限制，当自旋次数超过10次后，锁升级为重量锁 

![img](http://cdn.noteblogs.cn/007S8ZIlgy1gevjnq4j3mj306z0bt0sy.jpg)

### 4.总结

所谓偏向锁，就是偏袒的意思，我偏袒第一个获得锁的线程，这种锁适用于只有一个线程的同步场景，因为在单线程环境下，不会有线程来抢夺锁，那虚拟机也没有必要进行加锁和解锁操作；

当有另外一个线程来获取锁时，偏向锁宣告结束，但是此时也不是直接调用系统内核态的语句进行加锁，而是在CPU代码层面上等待希望这样能获取到锁，这就是自旋锁；

当自旋了10次以上还没有获取到锁，那么不能再一直空等下去浪费CPU资源了，而将锁升级为重量锁，将想要争夺锁的线程暂时挂起。

