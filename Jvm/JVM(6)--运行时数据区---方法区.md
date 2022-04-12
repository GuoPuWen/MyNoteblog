参考：《深入理解Java虚拟机》

​			《宋红康JVM教程》

### 一、概述

在介绍方法区之前，简单的回顾一下运行时数据区的各个部分组成：
![](http://cdn.noteblogs.cn/7.PNG)

方法区是线程共享的一块区域，在《Java虚拟机规范》中对方法区的作用的描述是

> It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods (§2.9) used in class and instance initialization and interface initialization.

它存储每个类的结构，比如运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括在类和实例初始化以及接口初始化中使用的特殊方法。

### 二、方法区的内存结构

##### 2.1 在不同版本中，方法区的具体实现

在《Java虚拟机规范》中，有下面一段文字

> Although the method area is logically part of the heap, simple implementations may choose not to either garbage collect or compact it.

大概的意思是虽然方法区域在逻辑上是堆的一部分，但简单的实现可能选择不进行垃圾收集或压缩。方法区还有一个名字叫“非堆”（对于HotSpot虚拟机而言），就是要把堆和方法区分开，那么具体的实现在jdk8之前和之后有比较大的不同。

**在JDK1.7中方法区的具体实现是在永久代。JDK8开始讲方法区的具体实现改为元空间**，实际上在《JAVA虚拟机规范》中没有统一要求方法区的具体实现，从jdk8开始，将用永久代改为元空间，不只是名字上的变化，内存结构也发生了很大的变化。

##### 2.2 设置方法区大小

- 在JDK1.7中。设置方法区的大小用如下参数：
  - -XX：PermSize ： 永久代初始分配空间，
  - -XX ： MaxPermSize：永久代最大可分配空间。32位机器默认是64M，64位机器模式是82M

- 在jdk1.8以上的版本中，
  - -XX：MetaspaceSize：元空间初始分配空间
  - -XX ：MaxMetaspaceSize：元空间最大可分配空间，默认是-1，也就是没有限制。此时方法区以及在本地内存落地实现

使用jdk命令行工具jinfo（需要先通过jps查看对于的进程的id）可以查看这些参数的初始值是多少，例如：

```java
jinfo -flag MetaspaceSize 8352[8352是进程号]
```

##### 2.3 方法区的OOM

运行下面代码，并设定指定参数，将会发生oom异常

```java
/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/9/5 15:27
 *  jdk6/7中：
 *  -XX:PermSize=10m -XX:MaxPermSize=10m
 *  jdk8中：
 *  -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
 */
public class OOMTest extends ClassLoader {
    public static void main(String[] args) {
        int j = 0;
        try {
            OOMTest test = new OOMTest();
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

![](http://cdn.noteblogs.cn/34.png)

上面的异常是在jdk1.8环境中测试的。



##### 2.4 方法区的内部结构

> 它用于存储已被虚拟机加载的类信息、常量、静态变量、即使编译器编译后的代码等数据
>
> 《java虚拟机》

- 类型信息

①这个类型的完整有效名称（全名=包名.类名）

②这个类型直接父类的完整有效名（对于interface或是java. lang.Object，都没有父类）

③这个类型的修饰符（public， abstract， final的某个子集）

④这个类型直接接口的一个有序列表

- 域信息

①JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。

②域的相关信息包括：域名称、 域类型、域修饰符（public， private， protected， static， final， volatile， transient的某个子集）

- 方法信息

①方法名称

②方法的返回类型（或void）

③方法参数的数量和类型（按顺序）

④方法的修饰符（public， private， protected， static， final， synchronized， native ， abstract的一个子集）

⑤方法的字节码（bytecodes）、操作数栈、局部变量表及大小（ abstract和native 方法除外）异常表（ abstract和native方法除外）

⑥每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

##### 2.5 non-final静态变量和final静态变量

- non-final静态变量
  * 静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分
  * 类变量被类的所有实例所共享，即使没有类实例你也可以访问它。

```java
public class MethodAreaTest {
    public static void main(String[] args) {
        Order order = null;
        order.hello();
        System.out.println(order.count);
    }
}

class Order {
    public static int count = 1;
    public static final int number = 2;


    public static void hello() {
        System.out.println("hello!");
    }
}
```

- final静态变量

final静态变量则在编译时就已经被分配并且确定值了。可以通过反编译字节码文件来查看上面两种变量类型的具体是如何实现的

```java
  public static int count;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC

  public static final int number;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 2
```

**可以到对于number变量，在编译期就已经确定了值**

### 三、方法区的演进细节

方法区的具体实现，在java虚拟机规范中并没有明确规定，所以这里说的演进细节都说的是Hostpot虚拟机。

Hotspot中 方法区的变化：

* jdk1.6及之前：有永久代（permanent generation） ，静态变量存放在永久代上
* jdk1.7：有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中
* jdk1.8及之后： 无永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆

![](http://cdn.noteblogs.cn/35.png)

![](http://cdn.noteblogs.cn/36.png)

![](http://cdn.noteblogs.cn/37.png)

### 四、永久代为什么要被元空间代替

随着Java8的到来，HotSpot VM中再也见不到永久代了。但是这并不意味着类.的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（ Metaspace ）。

由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。

这项改动是很有必要的，原因有：

* 1）为永久代设置空间大小是很难确定的。 在某些场景下，如果动态加载类过多，容易产生Perm区的O0M。比如某个实际Web工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。 `"Exception in thread' dubbo client x.x connector’java.lang.OutOfMemoryError： PermGenspace"` 而元空间和永久代之间最大的区别在于：==元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制==。
* 2）对永久代进行调优是很困难的。

- 将 HotSpot 与 JRockit 合二为一

 