## 1.final、finally、finalize

final是java中的关键字，可以用来修饰类、方法和变量：

- 修饰类时，表示该类不能被其他类所继承，final类中的成员方法也会被默认修饰为final的
- 修饰方法时，表名该方法不能被重写，防止继承类将其进行更改
- 修饰变量时，表名该变量只能被赋值一次

finally是处理异常块中的一部分，用在try/catch语句中，finally语句块在正常情况下一定会被执行，至于不执行的情况等下一个题目

finalize是一个方法定义在Object类下的，是在该对象将要被回收时的调用，也可以说是对象生命周期中的方法，死亡前调用，有可能在调用finalize方法时使该对象复活，使用场景为关闭资源等等

## 2.finally什么情况下不会被执行

- 在代码不进入try/catch语句块的时候，那么finally就不会执行，在try语句块之前代码抛出异常或者已经返回，那么finally就不会被执行  
- 代码进入try语句块，但是调用了System.exit(0)则finally不会被执行，或者当线程在执行try语句块的时候进程被kill了

## 3.进程与线程

- 进程是程序执行的最小单位，线程是任务调度的最小单位
- 一个进程可以拥有多个线程
- 线程进行上下文的切换比进程资源消耗更小

答这个题目可以结合例子，qq是一个进程，而qq中的一个聊天窗口或者其他的功能可能是一个线程

## 4. Java的异常体系

Java的异常的基类是Throwable，Throwable有两个子类为Error和==Exception(错误：RuntimeException)==，分别为系统错误和异常(注意不能说成是运行时异常了)，系统错误表示虚拟机错误是不可避免的，而Exception运行时异常可以由用户进行捕获

Exception下又分为运行时异常和可检查异常，运行时异常指程序运行时出现的异常，可能是逻辑错误啊，数组下标越界或者空指针异常，而可检查异常是指程序在编译期间出现的异常，必须显示的处理

## 5.OOM

OOM为OutOfMemberError，OOM是一种错误，是JVM内存不足的一种错误，常见的OOM有：‘

- 堆空间不够时抛出OOM:java heap space异常
- 元空间内存不足时抛出OOM:MetaSpace，但是jdk1.8开始方法区的落地实现改为本地内存
- 当本地内存不够时，抛出OOM:direct buffer member，在使用NIO程序时buffer可以分配本地内存加快传输速度，当内存不够时有可能抛出OOM:direct buffer member
- 当无法在创建线程时抛出OOM:unable to create new therad异常，线程数超过了操作系统规定的最大线程数

## 6.重定向与转发

