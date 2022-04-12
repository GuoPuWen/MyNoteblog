# 三、集合中的不安全类

### 3.1 ArrayList

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 30; i++) {
        new Thread(() -> {
            list.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(list);
        },String.valueOf(i)).start();
    }
}
```

上面的代码将会产生一个异常java.util.ConcurrentModificationException，出现这个异常是因为集合在迭代的过程中对其进行修改，因为有30个线程进来，一方面要对进行修改增加操作，一方面要对进行读的操作，当线程数过多时将会产生ConcurrentModificationException异常

解决办法：

1. 使用线程安全类Vector，Vector的add方式是被Synchronized修饰的

![image-20201210171554040](C:\Users\VSUS\Desktop\笔记\多线程\img\24.png)

2. 使用Collections.synchronizedList(new ArrayList());

```java
public static void main(String[] args) {
    List<String> list = Collections.synchronizedList(new ArrayList<>());
    for (int i = 0; i < 30; i++) {
        new Thread(() -> {
            list.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(list);
        },String.valueOf(i)).start();
    }
}
```

3. new CopyOnWriteArrayList()的方法

```java
public static void main(String[] args) {
    List<String> list = new CopyOnWriteArrayList<>();
    for (int i = 0; i < 30; i++) {
        new Thread(() -> {
            list.add(UUID.randomUUID().toString().substring(0,8));
            System.out.println(list);
        },String.valueOf(i)).start();
    }
}
```

第3种方法叫做写时复制

 CopyOnWrite容器即写时复制的容器。往一个容器添加元素的时候，不直接往当前容器Object[]添加，而是先将当前容器Object[]进行Copy， 复制出一个新的容器Object[] newElements，然后新的容器Object[] newElements里添加元素，添加完元素之后， 再将原容器的引用指向新的容器setArray(newElements);。这样做的好处是可以对CopyOnWrite容器进行并发的读， 而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

CopyOnWrite这种机制虽然是线程安全的，但是每次add都需要拷贝一份数组，如果是频繁的add将会很耗时，所以在使用这个集合的时候尽量避免频繁的add操作，另外，在数据量大的时候，因为是拷贝一份然后在将元素写入，这样数据过大可能会造成实时性比较低。

```java
//CopyOnWriteArrayList中的add方法源码
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### 3.2 HashSet

HashSet也是线程不安全的类，下面代码也会抛出ConcurrentModificationException异常

```java
    public static void main(String[] args) {
        HashSet<String> list = new HashSet<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
```

解决办法和ArrayList一样：

- 使用Collections.synchronizedSet
- 使用CopyOnWriteArraySet

### 3.3 HashMap

HashMap也是线程不安全类，解决办法和以上两个集合类似：

- 使用Collections.synchronizedMap
- 使用ConcurrentHashMap，这个后续会讲到

# 四、通过Callable的方式创建线程

### 4.1区别

```java
class Thread1 implements Runnable{
    @Override
    public void run() {

    }
}

class Thread2 implements Callable<String>{
    @Override
    public String call() throws Exception {
        return null;
    }
}
```

1. 实现Runnable接口创建线程重写的是run方法，实现Callable接口重写的call方法
2. 实现Callable接口创建进程，可以抛出异常
3. 实现Callable接口创建进程，有返回值，放回值要和泛型一致

### 4.2 启动一个线程

那么该如何创建一个实现了Callable的线程呢？查看官方文档可以发现，使用Callable创建的线程和Runnable相似，那么可以在Therad中启动

![image-20201212201203490](C:\Users\VSUS\AppData\Roaming\Typora\typora-user-images\image-20201212201203490.png)

Thread中是没有直接可以传入Callable接口的参数的，

![image-20201213102530382](C:\Users\VSUS\Desktop\笔记\多线程\img\25.png)

要想在Thread中使用Callable，我们可以试想一下这种即实现了Runnable，又实现了Callable的接口或者类呢？而FutureTask正是这种类，首先是实现了Runnable接口，然后构造方法允许传入Callable接口的实现类

![image-20201213103047068](C:\Users\VSUS\Desktop\笔记\多线程\img\26.png)

```java
FutureTask(Callable<V> callable) 
```

那么启动Callable线程就好办了

```java
class Thread2 implements Callable<String>{
    @Override
    public String call() throws Exception {
        System.out.println("线程启动了");
        return null;
    }
}

public class CallableTest {

    public static void main(String[] args) throws InterruptedException {
        FutureTask<String> task = new FutureTask<>(new Thread2());
        new Thread(task,"A").start();
    }
}
```

### 4.3 获取返回值

调用get方法即可获得返回值

```java
FutureTask<String> task = new FutureTask<>(new Thread2());
new Thread(task,"A").start();
task.get();
```

### 4.4 原理

调用get方法线程会堵塞知道线程执行结束，返回返回值。在主线程中需要执行比较耗时的操作时，但又不想阻塞主线程时，可以把这些作业交给Future对象在后台完成，当主线程将来需要时，就可以通过Future对象获得后台作业的计算结果或者执行状态。

```java
FutureTask<Integer> futureTask = new FutureTask(()->{
    System.out.println(Thread.currentThread().getName()+"  come in callable");
    TimeUnit.SECONDS.sleep(4);
    return 1024;
});
new Thread(futureTask,"A").start();

while(!futureTask.isDone()){
    System.out.println("***wait");
}
System.out.println(futureTask.get());
System.out.println(Thread.currentThread().getName()+" come over");
```

上面启动了一个线程，模拟计算较耗时的任务然后将最终结果放回，在主线程中，isDone方法判断是否执行结束，执行结束（完成/取消/异常）返回true，以上程序可以证明调用get方法是堵塞的，只有A线程将结果返回，才会继续执行

![image-20201213110946521](C:\Users\VSUS\Desktop\笔记\多线程\img\27.png)==还需要注意的是一个task任务只计算一次，看如下代码==

```java
FutureTask<Integer> futureTask = new FutureTask(()->{
    System.out.println(Thread.currentThread().getName()+"  come in callable");
    TimeUnit.SECONDS.sleep(4);
    return 1024;
});
new Thread(futureTask,"A").start();
while(!futureTask.isDone()){
    System.out.println("***wait");
}
System.out.println(futureTask.get());
System.out.println("--------");
System.out.println(futureTask.get());
System.out.println(Thread.currentThread().getName()+" come over");
```

![image-20201213111158770](C:\Users\VSUS\Desktop\笔记\多线程\img\28.png)

第二次get时，线程并没有休眠4秒。可见是直接返回的，所以每个task任务只会计算一次。

总结：

一般FutureTask多用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。仅在计算完成时才能检索结果；如果计算尚未完成，则阻塞 get 方法。一旦计算完成，就不能再重新开始或取消计算。get方法而获取结果只有在计算完成时获取，否则会一直阻塞直到任务转入完成状态，然后会返回结果或者抛出异常。 一个task任务只计算一次，所以一般将get方法放到最后。

