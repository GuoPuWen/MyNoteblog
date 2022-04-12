# 一、生产者消费者模型

题目：现在两个线程，可以操作初始值为零的一个变量，  实现一个线程对该变量加1，一个线程对该变量-1， 实现交替，来10轮，变量初始值为0

对于多线程问题：线程 操作 资源类

### 1.1 使用synchronized

```java
//资源类
class T {
    private int num = 0;
    public synchronized void numadd()  {
        if(num != 0){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "\t" + this.num );
        this.notifyAll();
    }
    public synchronized void numsub()  {
        if(num == 0){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "\t" + this.num );
        this.notifyAll();
    }


}
public class ThreadNotifyWaitDemo {
    public static void main(String[] args) {
        T t = new T();
        //线程 操作
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numadd(); },"A").start();
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numsub(); },"B").start();
    }
}
```

### 1.2 虚假唤醒问题

题目：现在四个线程，可以操作初始值为零的一个变量，  实现两个线程对该变量加1，两个线程对该变量-1， 实现交替，来10轮，变量初始值为0

分析：现在线程增加了两个，如果只是简单的将线程数复制，将会出现虚假唤醒的问题

```java
public static void main(String[] args) {
    T t = new T();
    new Thread(() -> { for(int i = 0;i < 10;i++) t.numadd(); },"A").start();
    new Thread(() -> { for(int i = 0;i < 10;i++) t.numsub(); },"B").start();
    new Thread(() -> { for(int i = 0;i < 10;i++) t.numadd(); },"C").start();
    new Thread(() -> { for(int i = 0;i < 10;i++) t.numsub(); },"D").start();
}
```

![image-20201210164607151](http://cdn.noteblogs.cn/image-20201210164607151.png)



问题分析：因为每个线程在完成了自己的工作之后，会调用notifyAll方法，这里会将在等待队列里的加1线程唤醒，造成了出错的问题，假如A线程首先获得CPU，发现num=0，这时num++的操作，如果很不巧这个时候C线程获得了CPU资源，发现num不等于0，进行wait()，并释放当前自己的锁，加入到等待队列中，接着假如B线程获得了CPU资源，发现num=1，可以进行减1操作，然后进行notifyAll，这下就将C线程唤醒了，==由于之前已经判断过了==，所以直接到num++这一行，这样就与题目不符合

综上，要解决虚假唤醒的问题就是循环判断，也就是说上面分析中即使C线程重新获得了CPU资源，也要重新进行判断，==将if改为while==，便可以解决虚假唤醒问题

```java
class T {
    private int num = 0;
    public synchronized void numadd() {
        //将if改为while
        while(num != 0){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "\t" + this.num );
        this.notifyAll();
    }
    public synchronized void numsub() {
        //将if改为while
        while(num == 0){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "\t" + this.num );
        this.notifyAll();
    }
}
public class ThreadNotifyWaitDemo {
    public static void main(String[] args) {
        T t = new T();
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numadd(); },"A").start();
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numsub(); },"B").start();
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numadd(); },"C").start();
        new Thread(() -> { for(int i = 0;i < 10;i++) t.numsub(); },"D").start();
    }
}
```



# 二、synchronized的八个问题

```java
//资源类
class phone {
    public synchronized void sendEmail() throws InterruptedException {	//邮件
        System.out.println("*******sendEmail");
    }
    public synchronized void sendMS(){	//短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();
        
        //Thread.sleep(100);
        
        new Thread(() -> {
            phone.sendMS();
        },"B").start();
    }
}
```

### 2.1 标准访问，打印顺序？

==不确定==，谁先获得CPU的调度权不一定，所以A,B线程谁先进入phone不确定

### 2.2 邮件方法暂停4秒钟，主线程调用B前也暂停100ms，打印顺序？

```java
class phone {
    public synchronized void sendEmail() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);	//暂停4秒
        System.out.println("*******sendEmail");
    }
    public synchronized void sendMS(){
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}


public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);	//主线程暂停100ms

        new Thread(() -> {
            phone.sendMS();
        },"B").start();
    }
}
//输出
*******sendEmail
*******sendMS
```

==先打印后sendEmail打印sendMS==

首先需要明白，synchronized锁住的是什么？==synchronized锁住的是当前对象this==

**一个对象里面如果有多个synchronized方法，某一个时刻内，只要一个线程去调用其中的一个synchronized方法了， 其他的线程都只能等待，换句话说，某一个时刻内，只能有唯一个线程去访问这些synchronized方法，锁的是当前对象this，被锁定后，其他的线程都不能进入到当前对象的其他的synchronized方法**

接着看到上面程序，由于主线程的休眠100ms，A线程先进入到phone对象中，那么这个时候B线程要想进来就必须等待，哪怕是A线程在执行方法的过程中遇到了sleep休眠4秒钟，因为==sleep是不会释放锁的！==，所以打印顺序就是先打印邮件然后MS

### 2.3 新增加一个普通方法，打印顺序？

```java
class phone {
    public synchronized void sendEmail() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
    public synchronized void sendMS(){
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone.hello();	//调用hello方法
        },"B").start();
    }
}
//输出
*******hello
*******sendEmail
```

==先打印hello后打印Email==

看上面程序，hello方法是没有加synchronized的，首先的确是A线程先进入phone，先抢到手机，但是hello方法没有被synchronized锁定，所以B线程可以调用hello方法。因为A线程在调用sendEmail方法的时候休眠了4秒钟，所以先打印hello

### 2.4 两个普通同步方法，两部手机，打印顺序？

```java
class phone {
    public synchronized void sendEmail() throws InterruptedException {  //邮件
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
    public synchronized void sendMS(){  //短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();
        phone phone2 = new phone(); //重新new一个对象

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone2.sendMS();	//调用sendMS
        },"B").start();
    }
}
//输出
*******sendMS
*******sendEmail
```

==先打印短信后打印邮件==

第2个问题中提到过，synchronized锁住的是当前对象this，那么这个题目中有两个phone对象，所以锁住的必然是两个不同的对象：

A线程获得资源调度，并对phone对象上锁，执行里面的sendEmail()方法，并且休眠4秒

B线程也获得了资源调度，并对phone2对象上锁，执行里面的sendMS()方法，但是没有休眠，所以最后的输出是先打印短信。这里有两个phone对象，所以A，B线程是井水不犯河水的，是互不相干的。

### 2.5 两个静态同步方法，一步手机，打印顺序？

```java
class phone {
    //静态方法
    public static synchronized void sendEmail() throws InterruptedException {  //邮件
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
     //静态方法
    public static synchronized void sendMS(){  //短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();
       // phone phone2 = new phone(); //一部手机

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone.sendMS();
        },"B").start();
    }
}
//输出
*******sendEmail
*******sendMS
```

==先打印sendEmail后打印sendMS==

### 2.6 两个静态同步方法，两部步手机，打印顺序？

```java
class phone {
    public static synchronized void sendEmail() throws InterruptedException {  //邮件
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
    public static synchronized void sendMS(){  //短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();
       phone phone2 = new phone(); //两部手机

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone2.sendMS();
        },"B").start();
    }
}
//输出
*******sendEmail
*******sendMS
```

==先打印邮件后打印短信==

可以将第6题和第7题放到一块去思考。首先思考，synchronized修饰在静态方法上，锁住的是什么？

==synchronized修饰在静态方法上，锁住的是当前的类class==，可以理解为类只加载一次，而静态方法也只有一份实例所以使用synchronized修饰，锁住的是class，但是synchronized修饰在普通方法上，锁住的是当前的对象，因为对象是可以创建多个的，而类实例只能有一个

分析第5题，由于是两个静态同步方法，而且又是A线程先持有这个类的锁，所以B线程只能等待A线程释放锁，分析第6题，虽然创建了两个对象，但是锁住的是class，所以A，B线程持有的是同一把锁，那么就是异步调用了，B必须等A执行完后才能执行

### 2.7 一个静态同步方法，一个普通同步方法，同一部手机，打印顺序？

```java
class phone {
    //静态同步方法
    public static synchronized void sendEmail() throws InterruptedException {  //邮件
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
    //普通同步方法
    public  synchronized void sendMS(){  //短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();
       //phone phone2 = new phone(); //重新new一个对象

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone.sendMS();
        },"B").start();
    }
}
//输出
*******sendMS
*******sendEmail
```

### 2.8 一个静态同步方法，一个普通同步方法，两部手机，打印顺序？

```java
class phone {
    //静态同步方法
    public static synchronized void sendEmail() throws InterruptedException {  //邮件
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*******sendEmail");
    }
    //普通方法
    public  synchronized void sendMS(){  //短信
        System.out.println("*******sendMS");
    }
    public void hello(){
        System.out.println("*******hello");
    }
}

public class LockDemo1 {
    public static void main(String[] args) throws InterruptedException {
        phone phone = new phone();
       phone phone2 = new phone(); //两部手机

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(() -> {
            phone2.sendMS();
        },"B").start();
    }
}
//输出
*******sendMS
*******sendEmail
```

题7和题8可以在一块讨论，首先要明白一个问题，锁住普通方法的对象锁和锁住类的class锁能一样吗？

==不一样==，很显然就是不一样的，一个是对象锁一个是class锁，那既然不一样那A，B线程拿到的锁不一样，所以和第4题一样，井水不犯河水的，是互不相干，所以打印顺序自然是先发送短信

### 2.9 总结

上面8个问题，其实都是围绕锁住的是什么来展开的，如果两个线程是持有的同一把锁，那么执行顺序必然是同步的，也就是只有一个线程释放了之后其他线程才能执行，但是如果两个线程持有的都不是同一把锁了，那么只需必然是要异步执行的，也就是执行过程没有先后。从上面8个问题可以得出以下结论：

1. 普通同步方法，锁住的是对象本身
2. 静态同步方法，锁住的是class
3. 静态同步方法和普通同步方法的锁不一样