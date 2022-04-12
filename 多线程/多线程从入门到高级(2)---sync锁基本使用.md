参考：《Java多线程编程核心技术》

### 一、线程安全与非线程安全

- 非线程安全：多个线程对同一个对象中的实例变量进行并发访问时产生，后果就是产生脏读(这里使用数据库的例子来解释脏读，这里的事物就相当于线程中的run方法，回滚就是线程中的run方法执行完毕的时候)

> 脏读：读到了别的事务回滚前的脏数据。比如事务B执行过程中修改了数据X，在未提交前，事务A读取了X，而事务B却回滚了，这样事务A就形成了脏读
>
> 不可重复读：事务A首先读取了一条数据，然后执行逻辑的时候，事务B将这条数据改变了，然后事务A再次读取的时候，发现数据不匹配了，就是所谓的不可重复读了。
>
> 幻读：事务A首先根据条件索引得到N条数据，然后事务B改变了这N条数据之外的M条或者增添了M条符合事务A搜索条件的数据，导致事务A再次搜索发现有N+M条数据了，就产生了幻读。

- 线程安全：获得的实例变量的值是通过同步处理的，方法内的变量是线程安全的

### 二、synchronized同步方法

先看一个例子：

```java
public class HasSelfPrivateNum {
    private int num = 0;
    public void addI(String username){
        try {
            if(username.equals("a")){
                num = 100;
                System.out.println("a set over");
                Thread.sleep(2000);
            }else{
                num = 200;
                System.out.println("b set over");
            }
            System.out.println("username= " + username + " num=" + num);
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }

}
public class ThreadA extends Thread{
    private HasSelfPrivateNum numRef;
    public ThreadA(HasSelfPrivateNum numRef) {
        this.numRef = numRef;
    }
    @Override
    public void run() {
        numRef.addI("a");
    }
}
public class ThreadB extends Thread{
    private HasSelfPrivateNum numRef;
    public ThreadB(HasSelfPrivateNum numRef) {
        this.numRef = numRef;
    }

    @Override
    public void run() {
        numRef.addI("b");
    }
}
public class Run {
    public static void main(String[] args) {
        HasSelfPrivateNum numRef = new HasSelfPrivateNum();
        new ThreadA(numRef).start();
        new ThreadB(numRef).start();
    }
}
```

HasSelfPrivateNum是一个操作业务的方法，当传入的username为“a”时，num=100，并打印“a set over”，当传入的username为“b”时，num=200，并打印“b set over”，ThreadA和ThreadB是两个线程，Run是主线程

![](C:\Users\VSUS\Desktop\笔记\多线程\img\15.png)

分析：当线程A和线程B一启动，线程A先抢占了CPU，执行了HasSelfPrivateNum的addI，并将num=100，然后执行了Thread.sleep(2000);，线程A处于休眠状态，这时线程B获得了CPU，进入方法将num=200，并打印username= b num=200，当线程A醒来的时候它并不知道num的值以及被更改为num=200，所以输出“username= a num=200”

只需要将addI方法加上synchronized同步锁

```java
public class HasSelfPrivateNum {
    private int num = 0;
    synchronized public void addI(String username){
        ......
    }
}
```

![](C:\Users\VSUS\Desktop\笔记\多线程\img\16.png)

分析：当线程A和线程B一启动，线程A先抢占了CPU，并获得当前对象的锁，执行了HasSelfPrivateNum的addI，并将num=100，然后执行了Thread.sleep(2000);，线程A处于休眠状态，此时线程B想执行addI，但是发现没有获得该对象的锁只能等待，等线程A执行完addI方法时释放锁，线程B开始拿到锁执行addI

##### 1.多个对象多个锁，无法同步

将上面的run方法改为如下：

```java
public class Run {
    public static void main(String[] args) {
        HasSelfPrivateNum numRef1 = new HasSelfPrivateNum();
        HasSelfPrivateNum numRef2 = new HasSelfPrivateNum();
        new ThreadA(numRef1).start();
        new ThreadB(numRef2).start();
    }
}
```

运行结果为：

![](C:\Users\VSUS\Desktop\笔记\多线程\img\17.png)

可见运行的顺序是交叉的，不是同步的，synchronized锁住的不是代码，而是对象，如果多个线程访问多个对象，那么JVM就会创建多个锁，所以执行的结果是异步的

##### 2.syncorized锁重入

可重入锁：自己可以再次获取自己的内部锁。比如有1条线程获取了某个对象的锁，如果该对象锁还没有释放，那么该对象可以获得该对象锁。

```java
public class Service {
    synchronized public void service1(){
        try{
            System.out.println("begin service1 threadName=" + Thread.currentThread().getName() + " time"
                    + System.currentTimeMillis()
            );

            System.out.println("service1..");
            Thread.sleep(2000);
            System.out.println("end service1 threadName=" + Thread.currentThread().getName() + " time"
                    + System.currentTimeMillis()
            );
            service2();
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }
    synchronized public void service2(){
        try{
            System.out.println("begin service2 threadName=" + Thread.currentThread().getName() + " time"
                    + System.currentTimeMillis()
            );

            System.out.println("service2...");
            Thread.sleep(2000);
            System.out.println("end service2 threadName=" + Thread.currentThread().getName() + " time"
                    + System.currentTimeMillis()
            );
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }

}
public class MyThread2 extends Thread{
    private Service service;
    public MyThread2(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        service.service2();
    }
}
public class Run {
    public static void main(String[] args) {
        Service service = new Service();
        new MyThread1(service).start();
    }
}
```

Service类里面分别创建了都被synchronized修饰的方法service1,和service2，service1调用service2，MyThread1创建一个线程，由Run类的main启动，现在要证明的结论是线程可以再次获得该对象的锁，也就是service2方法会得以执行

![](C:\Users\VSUS\Desktop\笔记\多线程\img\18.png)

但是只是这样还不足以证明即使该线程获取了锁，再次创建一个线程MyThread2

![](C:\Users\VSUS\Desktop\笔记\多线程\img\19.png)

可以看出来线程1和线程2是异步执行的，因为线程1在执行service2方法的时候还持有该对象的锁，所以线程2只能等待。

那么如果一个线程获得了被synchronized锁，那么由上面已经知道其他线程是不能再次调用被Lock锁住的方法的，那能不能调用不被synchronized锁住的方法呢？对上面的代码修改一下：将service2方法的synchronized去掉，并将service2的sleep方法去掉，线程1调用service1方法，线程2调用service2方法，

```java
public class Service {
    synchronized public void service1(){
        ......
    }
     public void service2(){
		......
            //Thread.sleep(2000);
		......
            );
    }

}
public class MyThread1 extends Thread{
    private Service service;
    public MyThread1(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        service.service1();
    }
}
public class MyThread2 extends Thread{
    private Service service;
    public MyThread2(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        service.service2();
    }
}
```

![](C:\Users\VSUS\Desktop\笔记\多线程\img\20.png)

可见线程1和线程2是异步执行的，也就是线程2可以调用不被sync锁锁住的方法

上面执行过程分析：线程1先启动并且将锁拿到，开始调用service1方法，当遇到sleep方法时，处于睡眠状态，线程2被CPU分配到时间片，开始调用service2方法，因为service2没有被synchronized修饰，所以线程2可以调用service2方法，等线程2执行完service2方法，线程1醒来后将service1方法的后序部分执行完，结束！

重要结论：

==当A线程调用anyObject对象加入synchronized关键字的X方法时，A线程就获得了X方法所在对象的锁，所以其他线程就必须等待A线程执行完毕才可以调用X方法，而B线程如果调用了声明了synchronized关键字的非X方法，必须等待A线程将X方法执行完，也就是释放锁之后才可以调用==

注意：当存在父子类继承关系时，子类完全是可以通过“可重入锁”调用父类的同步方法的，但是同步是不具有继承性的

### 三、synchronized同步语句块

使用关键字synchronized声明方法在某些情况下是由弊端的，比如A线程调用同步方法执行一个长时间的任务，那么B线程必须等待较长的时间，那么使用synchronized只锁住一些可能会产生非线程安全问题的代码块，这就会提升一定的效率，另外使用synchronized同步方法锁住的是该对象本身，这就造成了如果只有一个对象实例的情况下，只能去异步的执行，而使用同步语句块则可以同步的去执行而不产生线程安全问题

```java
public class Task {
    public void doLongTime(){
        for (int i = 1; i <= 100; i++) {
            System.out.println("noSynchronized threadName = " + Thread.currentThread().getName() + " i=" + i);
        }
        synchronized (this){
            for (int i = 1; i <= 100; i++) {
                System.out.println("Synchronized threadName = " + Thread.currentThread().getName() + " i=" + i);
            }
        }
    }

}
public class MyThread implements Runnable {
    private Task task;

    public MyThread(Task task) {
        this.task = task;
    }

    @Override
    public void run() {
        task.doLongTime();
    }
}
public class Run {
    public static void main(String[] args) {
        Task task = new Task();
        Thread t1 = new Thread(new MyThread(task));
        t1.start();
        Thread t2 = new Thread(new MyThread(task));
        t2.start();
    }
}
```

创建一个task类，模拟长时间任务，创建两个进程，启动后发现没有被synchronized包住的代码块是异步执行的，而被synchronized包住的代码是同步执行的

![](C:\Users\VSUS\Desktop\笔记\多线程\img\21.png)

![](C:\Users\VSUS\Desktop\笔记\多线程\img\22.png)

 ==使用synchronized(this)锁住的是当前对象==

synchorinzed(x)，其中x可以是任意对象，其中可以得出3个结论

- 当多个线程同时执行synchorinzed(x)同步代码块时呈同步效果
- 当其他线程执行x对象中的synchorinzed同步方法时也是呈同步效果
- 当其他线程执行x对象方法里面的synchorinzed(this)代码块时也呈同步效果