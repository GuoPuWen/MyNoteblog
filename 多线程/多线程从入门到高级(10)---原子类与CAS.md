# 一、原子操作类

Java从JDK 1.5开始提供了java.util.concurrent.atomic包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。因为变量的类型有很多种，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）。

### 1.1 原子更新基本类型

- AtomicBoolean：原子更新布尔类型。
- AtomicInteger：原子更新整型。
- AtomicLong：原子更新长整型。 

| 方法                                            | 作用                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| int addAndGet（int delta）                      | 以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果。 |
| boolean compareAndSet（int expect，int update） | 如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。 |
| int getAndIncrement()                           | 以原子方式将当前值加1，注意，这里返回的是自增前的值          |
| int getAndSet（int newValue）                   | 以原子方式设置为newValue的值，并返回旧值。                   |

### 1.2 原子更新数组

通过原子的方式更新数组里的某个元素，Atomic包提供了以下4个类。

- AtomicIntegerArray：原子更新整型数组里的元素。
- AtomicLongArray：原子更新长整型数组里的元素。
- AtomicReferenceArray：原子更新引用类型数组里的元素  

| 方法                                                   | 作用                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| int addAndGet（int i，int delta）                      | 以原子方式将输入值与数组中索引i的元素相加                    |
| boolean compareAndSet（int i，int expect，int update） | 如果当前值等于预期值，则以原子<br/>方式将数组位置i的元素设置成update值 |

### 1.3 原子更新引用

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类。

- AtomicReference：原子更新引用类型。
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
- AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference（V initialRef，boolean initialMark）

### 1.4 原子更新字段

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新。

- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题  

# 二、CAS

### 2.1 概述

CAS（Compare-And-Swap）看单词意思，比较和替换。比较和替换是使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与我们期望的值相等，就使用一个新值替换当前变量的值。java.util.concurrent.atomic包下都是利用了CAS的思想。

它是CPU的并发原语。原语属于操作系统的范畴，是由若干条指令组成，用于完成某个功能的一个过程。原语的执行必须是连续的，在执行过程中不允许被中断。也就是说CAS不会造成数据不一致的问题，CAS是线程安全的

==CAS有3个操作数，内存值V，旧的预期值A，要修改的更新值B，当且仅当预期值和内存值V相同时，将内存值修改为B，否则什么都不做==。

### 2.2 CAS的使用

```java
AtomicInteger atomicInteger = new AtomicInteger(10);
System.out.println(atomicInteger.compareAndSet(10, 2020) + "\t当前的值" + atomicInteger.get());
System.out.println(atomicInteger.compareAndSet(10, 2019) + "\t当前的值" + atomicInteger.get());
```

![image-20201218112322010](http://cdn.noteblogs.cn/image-20201218112322010.png)

CAS的作用是比较当前工作内存中的值和主物理内存中的值，如果相同则更新为我们规定的值，否则继续比较直到工作内存和主物理内存的值一致。我们通常称它为：比较并交换

CAS中有两个参数，第一个是预期值，第二个是更改后的值。

### 2.3 CAS的底层原理

在AtomicInteger中有一个方法：getAndIncrement()，这个方法不加synchronized照样可以保证安全性，前面说volatile的不保证原子性的解决办法上已经提到过。那么通过这个方法来看看CAS的底层原理

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

`CAS的底层使用了unsafe类，由于java需要通过本地(Native)方法才能访问底层系统，unsafe就是这个后门，通过unsafe可以直接操作特定的内存数据。看源码也可以知道unsafe有一堆的本地方法。`

unsafe的getAndAddInt方法需要传入三个参数：

- var1：当前对象
- var2(valueOffset)：表示该变量在内存中的偏移地址
- var4：要修改的值

在getAndAddInt方法中有一个循环判断方法compareAndSwapInt，这是一个本地方法。传入4个参数

- var1：AtomicInteger对象本身
- var2：对象值的引用地址，也就是偏移量
- var4：要修改的值
- var5：用当前对象的值和var5进行比较，如果相同，更新为var5+var4并返回true，如果不同，继续取值比较，直到更新完成。

下面用图来展示一下AtomicInteger为什么不加synchronized也可以实现线程安全

![aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa](http://cdn.noteblogs.cn/image-20201219092044055.png)

1. AtomicInteger里面的value原始值为0，即主内存中AtomicInteger的value为0，根据JMM模型，A，B线程各持有一份value=0的副本
2. 线程A通过获取到CPU资源将value++，然后通过getIntVolatile方法获取到主内存中value的值var5=0，执行compareAndSwapInt比较并交换，线程A成功修改主内存中的Value值
3. 线程B也将value++，并通过通过getIntVolatile方法获取到主内存中value的值var5=1，行compareAndSwapInt比较并交换发现不一样，那么返回false，进入下一次循环，知道发现自己拿的值和主内存中的值一样时，才进行值得更新，这样就避了写的覆盖问题，也就是保住了原子操作

### 2.4 CAS的缺点

##### 2.4.1长时间的自旋，消耗资源

自旋就是cas的一个操作周期，如果一个线程特别倒霉，每次获取的值都被其他线程的修改了，那么它就会一直进行自旋比较，直到成功为止，在这个过程中cpu的开销十分的大，所以要尽量避免。

##### 2.4.2 只能保证一个共享变量的原子操作

##### 2.4.3 ABA问题

CAS算法的重要前提是需要取出内存中某时刻的数据并在当下时刻==比较并替换==，那么这个时间差会导致数据的变化，例如：**1号线程从主内存中取出A，然后2号线程也从主内存中取出A，2号线程经过一些操作后将值变成了B，然后2号线程又将值变为了A，这个时候1号线程进行CAS操作的时候发现内存中的还是A，然后1号线程操作成功**

也就是说2号线程对1号线程是不可见的，这个时候数据没有问题，但是中间的过程有些问题

### 2.5 使用AtomicStampedReference解决ABA问题

ABA问题就是说如果有一个线程拿到一个数据一开始是A，最后一次修改的值也是A，中间的过程不知道，然后另外一个线程也对中间过程不可见，最后操作成功。

要解决ABA问题，可以加一个版本号，也就是说每一个线程每一次进行修改都会改变版本号，然后比较的时候还需要比较版本号，那么就解决了ABA问题

原子操作类中提供了AtomicStampedReference类用于解决ABA问题，看下面代码：

首先模拟ABA问题的产生：

```java
private static AtomicReference<Character> atomicReference = new AtomicReference<>('A');
public static void main(String[] args) {
    new Thread(() -> {
        atomicReference.compareAndSet('A', 'B');
        atomicReference.compareAndSet('B','A');
    },"t1").start();

    new Thread(() -> {
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        System.out.println(atomicReference.compareAndSet('A', 'C') + "\t" + atomicReference.get());
    },"t2").start();
}
```

![image-20201219095136231](http://cdn.noteblogs.cn/image-20201219095136231.png)

t2线程成功修改，将A的值变为C，下面看使用AtomicStampedReference操作类解决ABA问题

```java
private static AtomicStampedReference<Character> atomicStampedReference = new AtomicStampedReference<>('A', 1);

public static void main(String[] args) {

    new Thread(() -> {
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName() + "第1次版本号" + stamp);
        //睡眠1秒保证线程t1拿到当前的版本号
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        atomicStampedReference.compareAndSet('A', 'B', atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(Thread.currentThread().getName() + "第2次版本号" + atomicStampedReference.getStamp());
        atomicStampedReference.compareAndSet('B', 'A', atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(Thread.currentThread().getName() + "第3次版本号" + atomicStampedReference.getStamp());
    },"t3").start();


    new Thread(() -> {
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName() + "第1次版本号" + stamp);
        //睡眠3秒保证线程t3执行完毕
        try { TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}
        boolean res = atomicStampedReference.compareAndSet('A', 'C', stamp, stamp + 1);
        System.out.println(Thread.currentThread().getName() + "\t修改成功与否" + res);
        System.out.println(Thread.currentThread().getName() + "\t当前实际版本号" + atomicStampedReference.getStamp());
        System.out.println(Thread.currentThread().getName() + "\t当前实际最新值" + atomicStampedReference.getReference());

    },"t4").start();
}
```

执行结果：

![image-20201219100332061](http://cdn.noteblogs.cn/image-20201219100332061.png)

