# 一、辅助类介绍

### 1.1 CountDownLatch

需求：班上六个人上自习，班长最后要将教室锁门，也就是说班长只有等待着六个人上完自习后才能将教室锁住，也就是说这里需要进程的先后执行，只有这6个人执行完后，班长所在的进程才能执行。使用join方法可以完成这个需求，但是JUC中有强大的工具类CountDownLatch提供使用

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for(int i = 1;i <= 6;i++){
            new  Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t离开了教室");
                countDownLatch.countDown();
            },"" + i).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThrea 0.d().getName()+"\t****** 班长关门  走人，main线程是班长");
    }
}
```

原理：

 * CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，这些线程会阻塞。
 * 其它线程调用countDown方法会将计数器减1(调用countDown方法的线程不会阻塞)，
 * 当计数器的值变为0时，因await方法阻塞的线程会被唤醒，继续执行。

### 1.2 CyclicBarrier

需求：一个小组开会，组长只有在等待其他组员全部到来的时候才能开会，也就是说必须等到组长这个线程到来时，其他线程才能继续干活

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
		//设置 需要多少个线程进来才执行这个方法的线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("组长开会----");
        });

        for (int i = 1; i <= 7; i++) {
            new Thread(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + "\t组员到来");
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

![029](http://cdn.noteblogs.cn/029.png)

组员到来的时间不一样，但是组长这个线程一定是在最后得

原理： CyclicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。线程进入屏障通过CyclicBarrier的await()方法。

Cyclicbarrier和CountDownLatch比较相似，不过Cyclicbarrier是做加法，CountDownLatch是做减法

### 1.3 Semaphore

需求：现在停车场上有3个停车位，但是有6辆汽车，要求这些停车位一旦有空闲的，汽车就要过去停车。

这个有点类似于PV操作对临界区资源的访问

```java
public class SemaphoreDemo {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);	//设置3个停车位
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();	//获取停车位资源
                    System.out.println(Thread.currentThread().getName() + "\t抢占到了车位");	
                    TimeUnit.SECONDS.sleep(5);	//休眠5秒
                    System.out.println(Thread.currentThread().getName() + "\t离开到了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();	//释放停车位资源
                }
            },String.valueOf(i)).start();

        }
    }
}
```

![30](http://cdn.noteblogs.cn/30-1615942586595.png)

原理：

 在信号量上我们定义两种操作：

 * acquire（获取） 当一个线程调用acquire操作时，它要么通过成功获取信号量（信号量减1），要么一直等下去，直到有线程释放信号量，或超时。
 * release（释放）实际上会将信号量的值加1，然后唤醒等待的线程。

信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

# 二、ReentrantReadWriteLock读写锁

例如下例子，有5个线程向资源类里写数据，5个线程读数据

```java
class MyCache {
     //volatile:，保证可见性，不保证原子性，一个线程修改后，通知更新
    private volatile Map<String, Object> map = new HashMap<>();


    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "\t 正在写" + key);
        //暂停一会儿线程
        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "\t 写完了" + key);
        System.out.println();

    }

    public Object get(String key) {
        Object result = null;
        System.out.println(Thread.currentThread().getName() + "\t 正在读" + key);
        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        result = map.get(key);
        System.out.println(Thread.currentThread().getName() + "\t 读完了" + result);
        return result;
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.put(num + "", num + "");
            }, String.valueOf(i)).start();
        }
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.get(num + "");
            }, String.valueOf(i)).start();
        }

    }
}
```

输出结果

![image-20201213123226463](http://cdn.noteblogs.cn/31-1615942598595.png)

这里已经可以看出问题了，由于没有加锁，写和读同时进行会造成脏读，那加把锁就可以解决了这个问题！加入Synchronized锁，但是可以有更好的方法？

我们知道，读一个数据是不需要加锁的，例如前面说过的写时复制的思路，我们可以在写入的时候加入锁，防止其他人写入，但是在读取的时候就不需要加锁了。这就是读写分离

ReentrantReadWriteLock就是一种读写分离锁，看下面代码

```java
class MyCache {
     //volatile:，保证可见性，不保证原子性，一个线程修改后，通知更新
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();	//读写锁

    public synchronized void put(String key, Object value) {
        rwLock.writeLock().lock();	//加锁
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在写" + key);
            //暂停一会儿线程
            TimeUnit.MILLISECONDS.sleep(300);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t 写完了" + key);
            System.out.println();
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            rwLock.writeLock().unlock();	//解锁
        }
    }

    public synchronized Object get(String key) {
        rwLock.readLock().lock();
        try {
            Object result = null;
            System.out.println(Thread.currentThread().getName() + "\t 正在读" + key);
             TimeUnit.MILLISECONDS.sleep(300);

            result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t 读完了" + result);
            return result;
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            rwLock.readLock().unlock();
        }
        return null;
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.put(num + "", num + "");
            }, String.valueOf(i)).start();
        }
        .
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.get(num + "");
            }, String.valueOf(i)).start();
        }

    }
}

```

![image-20201213124900799](http://cdn.noteblogs.cn/32-1615942608389.png)

# 三、BlockingQueue阻塞队列       

### 3.1 概念

阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。这两个附加操作支持阻塞的插入和移除方法 ，使用BlockingQueue使得我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了

阻塞队列常用于生产者消费者模型，生产者是向队列里添加元素的线程，消费者是向队列里取元素的线程

### 3.2 JAVA中的阻塞队列

java中提供了7中阻塞队列：

- ArrayBlockingQueue：一个有数组组成的有界阻塞队列

ArrayBlockingQueue是一个用数组实现的有界阻塞队列，此队列按照FIFO的原则对元素进行排序，默认情况下不保证线程公平的访问队列，也就是说对先等待的线程是不公平的，当队列可以用时，阻塞的线程都可以争夺访问队列的资格，有可能先阻塞的线程最后才访问队列

- LinkedBlockingQueue：由链表结构组成的有界（但大小默认值为integer.MAX_VALUE）阻塞队列。

- PriorityBlockingQueue：支持优先级排序的无界阻塞队列。
- DelayQueue：使用优先级队列实现的延迟无界阻塞队列。·
- SynchronousQueue：不存储元素的阻塞队列，也即单个元素的队列。每一个put操作必须等待一个take操作
- LinkedTransferQueue：由链表组成的无界阻塞队列。
- LinkedBlockingDeque：由链表组成的双向阻塞队列。

### 3.3 核心方法

这些队列支持阻塞的插入和移除方法

- 插入方法：当队列满时，队列会阻塞插入元素的线程，知道队列不满
- 移除方法：当队列为空时，获取元素的线程会等待队列变为非空

| 方法     | 抛出异常  | 返回特殊值 | 一直阻塞 | 超时退出           |
| -------- | --------- | ---------- | -------- | ------------------ |
| 插入方法 | add(e)    | offer()    | put()    | offer(e,time,unit) |
| 移除方法 | remove(e) | poll()     | take()   | poll(time,unit)    |
| 检查方法 | element() | peek()     | 不可用   | 不可用             |

#####   3.3.1 抛出异常

当队列满时，如果再往队列里插入元素，会抛出IllegalStateException异常，当队列为空时获取元素会抛出NoSuchElementException异常

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
System.out.println(blockingQueue.add("a"));
System.out.println(blockingQueue.add("b"));
System.out.println(blockingQueue.add("c"));
System.out.println(blockingQueue.add("d"));
```

![image-20201213164127406](http://cdn.noteblogs.cn/33-1615942616482.png)

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
System.out.println(blockingQueue.add("a"));
System.out.println(blockingQueue.add("b"));
System.out.println(blockingQueue.add("c"));
//System.out.println(blockingQueue.add("d"));

System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
System.out.println(blockingQueue.remove());
```

![image-20201213164127406](http://cdn.noteblogs.cn/34-1615942625669.png)

##### 3.3.2 返回特殊值

当队列插入元素时，会返回元素是否插入成功，成功放回true，如果是移除方法，则从队列里取出一个元素，如果没有则放回null

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

System.out.println(blockingQueue.offer("a"));
System.out.println(blockingQueue.offer("b"));
System.out.println(blockingQueue.offer("c"));
System.out.println(blockingQueue.offer("x"));

System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
System.out.println(blockingQueue.poll());
```

![](http://cdn.noteblogs.cn/35-1615942647226.png)

##### 3.3.3 一直阻塞

当堵塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出，当队列为空时，如果消费者线程从队列里take元素，队列会阻塞消费者线程，知道队列不为空

```java
//写入
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
blockingQueue.put("a");
blockingQueue.put("b");
blockingQueue.put("c");
blockingQueue.put("x");
```

![](http://cdn.noteblogs.cn/36-1615942652410.png)

可见线程一直堵塞在这里

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
//取出
blockingQueue.put("a");
blockingQueue.put("b");
blockingQueue.put("c");
//blockingQueue.put("x");
System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
System.out.println(blockingQueue.take());
```

![](http://cdn.noteblogs.cn/37-1615942657038.png)

##### 3.3.4 超时退出

当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生成者消费者线程一段时间，如果超时了指定的时间，生成者线程将会退出

```java
BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
System.out.println(blockingQueue.offer("a"));
System.out.println(blockingQueue.offer("b"));
System.out.println(blockingQueue.offer("c"));
System.out.println(blockingQueue.offer("a",3L, TimeUnit.SECONDS));
```

线程会先阻塞3秒钟，然后直接退出

### 3.4 阻塞队列的实现原理

如果队列是空的，消费者会一直等待，当生产者添加元素时，消费者是如何知道当前队列有元素的呢？

==使用通知模式实现==，所谓通知模式，就是当生产者往满的队列中添加元素时会阻塞住生产者，当消费者消费了一个队里中的元素后，会通知生产者当前队列可以用，

ArrayBlockingQueue使用了Condition来实现其中的通知机制