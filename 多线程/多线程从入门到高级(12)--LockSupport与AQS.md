

# 一、LockSupport

### 1.1 简介

LockSupport是用来创建锁和其他同步类的基本线程堵塞原语，LockSupport为JUC并发包下的各种同步组件的底层实现提供了基础。

LockSupport可以用来堵塞线程和唤醒线程，也就是说LockSupport的出现是为了改进原有的wait/notify或者await/signal的不足的。



### 1.2 wait/notify的不足

看一个小例子

```java
public class WaitNotifyDemo {
    private static  Object objectLock = new Object();
    public static void main(String[] args) {
        new Thread( () -> {
            synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "\t被唤醒");
            }

        },"A").start();
        //确保A线程先获取锁
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        new Thread( () -> {
            synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                objectLock.notify();
                System.out.println(Thread.currentThread().getName() + "\t通知");
            }

        },"B").start();
    }
}
```

![image-20201221111308014](http://cdn.noteblogs.cn/image-20201221111308014.png)

以上代码实现了A线程被堵塞，然后需要B线程进行获取，这也是传统的通知唤醒机制。上述代码有两个限制：

1. **Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行**
2. **必须要先wait然后notify，否则无法唤醒**

下面通过代码证明：

```java
public class WaitNotifyDemo {
    private static  Object objectLock = new Object();
    public static void main(String[] args) {
        new Thread( () -> {
            //注释掉同步代码块
            //synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "\t被唤醒");
           // }

        },"A").start();
        
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        new Thread( () -> {
            //注释掉同步代码块
           // synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                objectLock.notify();
                System.out.println(Thread.currentThread().getName() + "\t通知");
           // }

        },"B").start();
    }
}

```

结果：编译通过，idea并没有报错，但是直接抛出IllegalMonitorStateException异常

![image-20201221111458877](http://cdn.noteblogs.cn/image-20201221111458877.png)



上述代码证明了，第一条结论，Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行，也就是必须要有synchronized锁。

接着证明第二条：

```java
public class WaitNotifyDemo {
    private static  Object objectLock = new Object();
    public static void main(String[] args) {
        new Thread( () -> {
            //休眠，让B线程先获取锁
            try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
            synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "\t被唤醒");
            }

        },"A").start();

        
        new Thread( () -> {
            synchronized (objectLock){
                System.out.println(Thread.currentThread().getName() + "\t来了");
                objectLock.notify();
                System.out.println(Thread.currentThread().getName() + "\t通知");
            }

        },"B").start();
    }
}
```

![image-20201221111713442](http://cdn.noteblogs.cn/image-20201221111713442.png)

代码一直卡主住，因为A线程等着其他线程将它唤醒，但是B线程已经结束了，所以如果先进行notify后wait，那么线程将一直堵塞



### 1.3 await/signal的不足

await/signal和wait/notify一样，不足之处也是这两点：

1. **Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在Lock内部执行**
2. **必须要先wait然后notify，否则无法唤醒**

```java
public class AwaitSignalDemo {
    private static Lock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();
    public static void main(String[] args) {
        new Thread(() -> {
            //注释掉lock锁
            //lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "\t来了");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "\t被唤醒");
            }finally {
                //lock.unlock();
            }
        },"A").start();

        //确保A线程先获取锁
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        new Thread(() -> {
             //注释掉lock锁
            //lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "\t来了");

                condition.signal();

                System.out.println(Thread.currentThread().getName() + "\t通知");
            }finally {
                //lock.unlock();
            }
        },"B").start();
    }
}
```

![image-20201221112804512](http://cdn.noteblogs.cn/image-20201221112804512.png)

```java
public class AwaitSignalDemo {
    private static Lock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();
    public static void main(String[] args) {
        new Thread(() -> {
            //确保B先获得锁
            try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "\t来了");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "\t被唤醒");
            }finally {
                lock.unlock();
            }
        },"A").start();

        //确保A线程先获取锁
        new Thread(() -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "\t来了");

                condition.signal();

                System.out.println(Thread.currentThread().getName() + "\t通知");
            }finally {
                lock.unlock();
            }
        },"B").start();
    }
}
```

![image-20201221112956060](http://cdn.noteblogs.cn/image-20201221112956060.png)

线程也一直堵塞在这里

而，LockSupport的出现就是为了解决上面两个问题的，下面看看基本使用

### 1.4 LockSupport的基本使用

LockSupport下的方法不多

![image-20201221113151394](http://cdn.noteblogs.cn/image-20201221113151394.png)

- park()/park(Object blocker)	堵塞当前线程/堵塞传入的具体线程
- unpark(Thead thread)    唤醒处于堵塞状态的指定线程

```java
public class LockSupportDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t来了");
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t被唤醒");
        },"t1");
        t1.start();
        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        Thread t2 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t来了");
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName() + "\t通知");;
        },"t2");
        t2.start();
    }
}
```

LockSupport都是静态方法，可以直接调用，而且上述代码时没有加任何锁的，可见解决了第一个问题，那么能否解决要先通知然后唤醒的限制呢？

```java
public class LockSupportDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            //让t1线程先获取
            try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
            System.out.println(Thread.currentThread().getName() + "\t来了");
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t被唤醒");
        },"t1");
        t1.start();

        Thread t2 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t来了");
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName() + "\t通知");;
        },"t2");
        t2.start();
    }
}
```



![image-20201221124920336](http://cdn.noteblogs.cn/image-20201221124920336.png)

线程并没有堵塞！所以LockSupport解决了传统的通知/唤醒机制的不足！

### 1.5 LockSupport原理

LockSuport的底层使用了Unsafe类，前面也说过了，unsafe是java层面用来操作底层操作系统的API。LockSuport提供了park()和unpark()方法用于堵塞线程和解除线程堵塞

LockSuport和每个使用它的线程都有一个许可证，也就是java api中的permit。permit默认为0，当调用一个unpark方法时就加1变成1，调用一次pack会消费permit，将1变成0，同时park立即返回。每个线程都有一个相关的permit，permit只有一个，重复调用unpark也不会积累permit

也就是说

当调用park方法时：

- 如果permit大于0，也就是可消费的，那么消费掉这个permit然后退出
- 如果permit等于0，那么必须堵塞等待permit

当调用unpark方法时：

- 会增加一个permit，但是permit最多只能等于1，累加无效

那么如果唤醒两次，堵塞两次最后结果怎么样呢？也就是调用两次unpark，两次park，==最后结果是堵塞的==

```java
public class LockSupportDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {

            System.out.println(Thread.currentThread().getName() + "\t来了");
            //调用两次park
            LockSupport.park();
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t被唤醒");
        },"t1");
        t1.start();

        try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}

        Thread t2 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t来了");
            //调用两次unpark
            LockSupport.unpark(t1);
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName() + "\t通知");;
        },"t2");
        t2.start();
    }
}
```

![image-20201221130523209](http://cdn.noteblogs.cn/image-20201221130523209.png)

结果是一直堵塞在这里！



# 二、AQS

### 2.1 简介

AQS(AbstractQueuedSynchronizer)抽象队列同步器，AQS是构成JUC的基石，JUC包下的大部分同步类都是基于AQS实现的。例如CountDownLatch、ReentrantLock、CyclicBarrier、Semaphore。AQS是一种提供了原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的简单框架

Java并发下会导致共享资源被占用，那么就需要一定的阻塞等待机制来保证锁的分配，这个机制在AQS中使用的是CLH队列来实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象体现，它将请求资源的线程封装成队列的节点一个一个Node，通过CAS，自旋以及LockSupport的方式，维护state变量的状态，是并发提供同步的控制效果。

也就是说其实Java很多并发包都需要这么一种框架模型来管理阻塞线程队列和唤醒队列的功能，而到了具体的实现只需要继承这个框架即可，这也就是面向对象的体现

### 2.2 AQS框架体系

AQS使用一个volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成对资源获取的排队工作，将每条要去抢占资源的线程封装成一个node节点来实现锁的分配，通过CAS来实现state值得修改

![image-20201222094424965](http://cdn.noteblogs.cn/image-20201222094424965.png)







##### 2.2.1 AQS中的数据结构

类似于HashMap中的Node<K,V>键值对，既然是队列同步器，那么肯定要涉及到数据结构，而AQS中的数据结构为Node节点

![image-20201222094937607](http://cdn.noteblogs.cn/image-20201222094937607.png)

| 方法或者属性 |                         含义                         |
| :----------: | :--------------------------------------------------: |
|  waitStatus  |       当前节点，其实就是当前线程在队列中的状态       |
|    thread    |                 表示处于该节点的线程                 |
|     prev     |                     前驱节点指针                     |
|     next     |                     后继节点指针                     |
| predecessor  | 返回前驱节点，如果没有则抛出异常NullPointerException |

简单来说，AQS = state + CLH队列，而Node就是该队列里的一个一个节点，Node = waitStatus + 前后指针指向

线程获取两种锁的方式也被定义在了Node中：

|   模式    |                       含义                        |
| :-------: | :-----------------------------------------------: |
|  SHARED   |   表示线程以共享的模式等待锁，例如ReadWriteLock   |
| EXCLUSIVE | 表示线程正在以独占的方式等待锁，例如ReentrantLock |

waitStatus有以下几个枚举值，分别对应不同的状态

| **枚举**  |                      含义                      |
| :-------: | :--------------------------------------------: |
|     0     |        当一个Node被初始化的时候的默认值        |
| CANCELLED |      为1，表示线程获取锁的请求已经取消了       |
| CONDITION |  为-2，表示节点在等待队列中，节点线程等待唤醒  |
| PROPAGATE | 为-3，当前线程处在SHARED情况下，该字段才会使用 |
|  SIGNAL   |   为-1，表示线程已经准备好了，就等资源释放了   |

##### 2.2.2 AQS中的state

AQS = state + CLH队列，这个公式简单的说了AQS中的结构，state是AQS中的同步状态是由Volatile修饰的，用于展示当前临界资源的获锁情况

![](http://cdn.noteblogs.cn/image-20201222094243390.png)

那么有变量，就会有set和get方法

|                    方法                     |              含义              |
| :-----------------------------------------: | :----------------------------: |
|       protected final int getState()        |           获取state            |
| protected final void setState(int newState) |           设置state            |
| protected final void setState(int newState) | 通过CAS自旋的方式修改state的值 |

这几个方法都是Final修饰的，说明子类中无法重写它们。我们可以通过修改State字段表示的同步状态来实现多线程的独占模式和共享模式（加锁过程）。

### 2.3 如何加锁

ReentrantLock是AQS实现的自定义同步器之一，那么就以ReentrantLock为例理解AQS是如何进行加锁与解锁的。首先复习一些前置知识：

- ReentrantLock提供了公平锁与非公平锁，默认使用构造器，不传入参数是非公平锁
- ReentrantLock(boolean fair)可以指定一个布尔值，用来确定当前锁是公平锁还是非公平锁

![image-20201222102846821](http://cdn.noteblogs.cn/image-20201222102846821.png)

下面通过一个流程图来梳理一下ReentrantLock的lock过程

![AQS](http://cdn.noteblogs.cn/AQS.jpg)

1. 通过ReentrantLock的加锁方法Lock进行加锁操作
2. 调用内部的sync.lock()方法，这是个抽象方法，由ReentrantLock实现的公平锁fairSync#Lock与非公平锁NonfairSync#Lock，执行内部的lock方法
3. 接着AQS的acquire再次获取锁，如果获取失败，那么执行下面的逻辑，将该线程加入到队列中

### 2.4 线程加入队列

现在通过一个例子来说明AQS是如何工作的，现在有3个顾客来bank办理业务，而这个银行只有一个窗口能办理业务，A顾客需要20分钟，B顾客需要15分钟，C顾客需要15分钟

```java
//资源类
class Bank{
    public void service(){
        System.out.println(Thread.currentThread().getName() + "\t正在办理业务");
    }
}

public class AQSDemo {
    private static Lock lock = new ReentrantLock();
    public static void main(String[] args) {
        Bank bank = new Bank();
        new Thread(() -> {
            try {
                lock.lock();
                bank.service();
                try { TimeUnit.MINUTES.sleep(20);} catch (InterruptedException e) {e.printStackTrace();}
            }finally {
                lock.unlock();
            }
        },"A").start();
    try { TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        
        new Thread(() -> {
            try {
                lock.lock();
                bank.service();
                try { TimeUnit.MINUTES.sleep(15);} catch (InterruptedException e) {e.printStackTrace();}
            }finally {
                lock.unlock();
            }
        },"B").start();
        
        new Thread(() -> {
            try {
                lock.lock();
                bank.service();
                try { TimeUnit.MINUTES.sleep(15);} catch (InterruptedException e) {e.printStackTrace();}
            }finally {
                lock.unlock();
            }
        },"C").start();

    }
```

1. 当A,B,C三个线程同时来到银行网点办理业务时，由于只有一个窗口，A,B,C三个线程需要争抢，由于在第21行休眠了，所以一定是A先获得锁。根据上面的流程图，最后需要调用NonfairSync#Lock方法

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}
```

compareAndSetState方法前面已经说到过，就是通过CAS的方式判断当前的state是不是为0，如果是那么修改为1，并且返回true。由于A线程是第一个进来的，state的值为0，所以修改为1，并且调用setExclusiveOwnerThread将当前线程设置为A线程，也就是说现在应该如图

![image-20201222125818107](http://cdn.noteblogs.cn/image-20201222125818107.png)

2. 当B线程调用lock方法时，由于stata已经为1了，所以将不能获取到锁，开始走加入等待队列的逻辑。也就是走下图红线![AQS -- 获取锁逻辑](http://cdn.noteblogs.cn/AQS%20--%20%E8%8E%B7%E5%8F%96%E9%94%81%E9%80%BB%E8%BE%91.jpg)

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

acquire(1)里面的if判断一共有3个方法，分别进行探究

##### 2.4.1 tryAcquire方法

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

直接点开tryAcquire会发现，AQS中直接给抛出了个异常，这是典型的模板设计模式，父类方法抛出异常使子类不得不重写方法，由于是非公平锁，那么我们应该找NonfairSync#tryAcquire()方法

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
final boolean nonfairTryAcquire(int acquires) {
    //当前线程A
    final Thread current = Thread.currentThread();
    //获取状态，那么此时的状态为1
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //getExclusiveOwnerThread表示获取正在占用资源的线程，也就是A线程
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

根据分析，可见tryAcquire方法返回的是false，取返为true。

##### 2.4.2 addWaiter方法

接着分析addWaiter(Node.EXCLUSIVE), arg)方法，传入的是EXCLUSIVE说明是独占的方式

```java
private Node addWaiter(Node mode) {
    //将当前线程封装为node节点
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

由于B线程是第一个要加入到阻塞队列里的线程，所以pred为空，即调用enq方法

```java
//这里参数的node是当前线程所在的node
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

A线程是第一个，所以还没有初始化，那么需要先初始化出一个头结点出来，也叫虚拟节点，==这里不是直接初始化当前节点==，而是需要添加一个虚拟节点，然后进行比较并交换头结点，将尾节点也指向这个虚拟节点，看下图

![image-20201222181500121](http://cdn.noteblogs.cn/image-20201222181500121.png)

接着，由于这是一个for(;;)循环，接着走else分支，else分支就将B线程加入到了等待队列里

![](http://cdn.noteblogs.cn/AQS%E7%BA%BF%E7%A8%8B%E5%8A%A0%E5%85%A5%E9%98%9F%E5%88%97%20(1).jpg)

##### 2.4.3 acquireQueued方法

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //获取当前节点的前驱节点
            final Node p = node.predecessor();
            //如果当前节点是一个线程进来的节点，那么再次尝试获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获取锁失败
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

acquireQueued方法还会调用tryAcquire也就是再次尝试获取锁，如果获取失败接着调用shouldParkAfterFailedAcquire(p,node)方法，这个时候的p是node的前节点，也就是虚拟节点，虚拟节点的waitStatas=0所以调用compareAndSetWaitStatus方法，将虚拟节点的waitStatas值改为了-1，而且返回false

但是acquireQueued方法里面是一个for循环，即再次尝试获取锁，获取失败那么调用shouldParkAfterFailedAcquire方法，此时不一样了因为上一个循环已经将虚拟节点的waitStatus改为了-1，所以返回true

![AQS线程加入队列 (1)](http://cdn.noteblogs.cn/AQS%E7%BA%BF%E7%A8%8B%E5%8A%A0%E5%85%A5%E9%98%9F%E5%88%97%20(1).jpg)

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)	//Node.SIGNAL = -1
        return true;
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

接着调用parkAndCheckInterrupt

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

这个时候，B线程真真正正的休眠了，因为调用了 LockSupport.park

那么线程加入队列结束

下面的流程图来整理一下线程加入等待队列

![AQS进入等待队列](http://cdn.noteblogs.cn/AQS%E8%BF%9B%E5%85%A5%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97.jpg)

## 2.5 如何解锁

同样还是调用sync.release方法

```java
public void unlock() {
    sync.release(1);
}
```

解锁时并不区分公平锁和非公平岁锁

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

本质上是在调用LockSupport的unpark方法，判断条件是h != null && h.waitStatus != 0

```
h == null Head还没初始化。初始情况下，head == null，第一个节点入队，Head会被初始化一个虚拟节点。所以说，这里如果还没来得及入队，就会出现head == null 的情况。

h != null && waitStatus == 0 表明后继节点对应的线程仍在运行中，不需要唤醒。

h != null && waitStatus < 0 表明后继节点可能被阻塞了，需要唤醒。
```

最后来一个图总结

![AQS--加入队列](http://cdn.noteblogs.cn/AQS--%E5%8A%A0%E5%85%A5%E9%98%9F%E5%88%97.jpg)