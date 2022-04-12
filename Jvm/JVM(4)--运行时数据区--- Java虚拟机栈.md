参考：《深入理解Java虚拟机第三版》

​			《宋红康JVM教程》

### 一、概述

还是需要拿出那张运行时数据区的示意图：

![](http://cdn.noteblogs.cn/7.PNG)

Java虚拟机栈和PC寄存器（程序计数器）一样也是线程私有的，它的生命周期和线程一样的。我们比较清楚的是，栈管运行，程序如何执行，如何处理数据；堆处理的是数据存储的问题，这种划分很广义。其中的栈其实就是虚拟机栈。

虚拟机栈描述的是Java方法的执行的内存模型：每个方法被执行的时候都会有同时创建一个栈帧用于存储局部变量表、操作栈、动态链接、方法出口、一些附加信息。

每一个方法的被调用的直至完成的过程，就对应一个栈帧在虚拟机栈中入栈和出栈的过程。



### 二、对Java虚拟机栈的理解

##### 1.虚拟机栈的特点

- 栈这种数据结构，是先进后出（后进先出）的
- Jvm对栈的操作只有两个：方法执行入栈，方法结束出栈
- 对于Java虚拟机栈来说不存在垃圾回收问题

##### 2.通过程序来理解

```java
package com.java;

/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/16 19:47
 */
public class StackFrameTest {
    public static void main(String[] args) {
        StackFrameTest test = new StackFrameTest();
        test.method1();
    }

    public void method1(){
        System.out.println("method1()开始执行。。。");
        method2();
        System.out.println("method1()执行结束。。。");
    }

    public int method2(){
        System.out.println("method2()开始执行。。。");
        int i = 10;
        int m = (int) method3();
        System.out.println("method2()执行结束。。。");
        return i+m;
    }

    public double method3(){
        System.out.println("method3()开始执行。。。");
        double j = 20.0;
        System.out.println("method3()执行结束。。。");
        return j;
    }

}
```

上面的程序的结果是很显然的：

```html
method1()开始执行。。。
method2()开始执行。。。
method3()开始执行。。。
method3()执行结束。。。
method2()执行结束。。。
method1()执行结束。。。
```

使用idea自带的debug工具进行调试的，便可以很清楚的明白这些方法之间的调用关系是通过栈这种数据结构来管理的

![](http://cdn.noteblogs.cn/9.png)

##### 3.栈中可能出现的异常

>  java虚拟机规范允许**Java栈的大小是动态的或者是固定不变的**

那么可能出现的异常就对应两种情况：

①如果栈的大小是固定的，线程（前面说过Java虚拟机栈是线程私有的）请求的栈的深度大于虚拟机所允许的深度，将会抛出**StackOverFlowError**

> 按照Java的异常处理机制，**StackOverFlowError**应该归属于错误类，但是这里的“异常”指的是广义上的异常，也就是把出现的错误和异常都归属于异常

```java
public class demo4 {
    public static void main(String[] args) {
        main(args);
    }
}
```

以上程序执行便会抛出一个java.lang.StackOverflowError异常

②如果虚拟机栈的大小是可以动态扩展的话，当扩展到无法申请足够的内存时，将会抛出一个**OutOfMemoryError**

##### 4.设置栈的内存大小

我们可以使用参数-Xss选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。 （IDEA设置方法：Run-EditConfigurations-VM options 填入指定栈的大小-Xss256k）

可以维护一个count变量来统计栈的大小：

```java
public class demo4 {
    private static int count = 1;
    public static void main(String[] args) {
        System.out.println(count);
        count++;
        main(args);
    }
}
```

##### 5.Java虚拟机栈的内部结构

一个线程独享一个Java虚拟机栈，Java虚拟机栈内部有很多栈帧，一个方法对应一个栈帧，一个栈帧内部具有局部变量表、操作数栈、动态链接、和方法返回地址这些信息，看如下图：

![](http://cdn.noteblogs.cn/10.PNG)

在编译执行代码时，栈帧中需要多大的局部变量表，多深的操作数栈已经被确定了，并且写入到方法表的code属性中，如下代码：

```java
package com.java;

/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/16 23:51
 */
public class StackTest {
    public static void main(String[] args) {
        StackTest test = new StackTest();
        test.methodA();
    }

    public void methodA() {
        int i = 10;
        int j = 20;
        
        methodB();
    }

    public void methodB(){
        int k = 30;
        int m = 40;
    }
}
```

使用idae中的反编译插件jclasslib得到

![](http://cdn.noteblogs.cn/12.png)

下面便依次说明Java虚拟机栈内部的具体结构

### 三、Java虚拟机栈的内部结构

##### 1.局部变量表

###### 1.1 局部变量表的大小确定

局部变量表是一组变量值储存空间，用在存放方法参数和方法内部定义的局部变量，反编译class文件，在code属性的max_locals数据项中确定了该方法需要分配的最大局部变量包表的容量

例如，如上程序，对于main这个静态方法来说，反编译之后

![](http://cdn.noteblogs.cn/13.png)

max_locals的值为2，也就是局部变量中记录了2个变量，事实也是如此，参数上的名为args的变量和方法体中名为test的变量

对于methodA这个普通方法来说，反编译之后可能会认为只有2个变量，i和j，但是事情却不是如此

![](http://cdn.noteblogs.cn/14.png)

通过反编译后，得到的methodA方法中变量表有3个，那么还有一个是什么？

不要忘记了，在普通方法中有一个this引用，可以指向当前对象的引用，那么在methodA中，局部变量表的数量为3。

###### 1.2 局部变量表的存储结构

局部变量表定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量这些数据类型包括各类基本数据类型、对象引用（reference），以及returnAddressleixing。

这个数组中最小的单位是变量槽（Variable Solt），虚拟机规范中（java8）并没有规定一个Slot应占用的空间大小，在HotSpot虚拟机中，2位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double）占用两个slot。

局部变量表既然为一个数组，那么他的下表索引是从index0开始的，

![](http://cdn.noteblogs.cn/16.png)

long和double类型占据两个Solt槽。

接着使用反编译工具来查看main这种静态方法和methodA这种普通方法的局部变量表的内容

![](http://cdn.noteblogs.cn/17.png)

- Start PC ：起始的PC寄存器行号
- Length：作用范围
- index：Solt槽的下表

从上可以看出，Solt槽的确是从下表index0开始的。

methodA这种普通方法反编译后得到的局部变量表

![](http://cdn.noteblogs.cn/18.png)

从上可以看出来，确实对于普通方法methodA来说，Slot槽的索引下标index0是this指针。

==这也可以很好地解释为什么在static静态方法中不能通过this去调用本身类的一些属性，因为static静态方法中的局部变量表中没有this==

###### 1.3 Solt的重复利用

例如下代码：

```java
public void methodC(){
    int a = 0;
    {
        int b = 0;
        b = a+1;
    }
    //变量c使用之前以及经销毁的变量b占据的slot位置
    int c = a+1;
}
```

b变量包裹在代码块中，显然b变量的作用域是超不过该代码块中的。如此虚拟机将会设计一个可以重复利用的Solt槽

![](http://cdn.noteblogs.cn/19.png)

变量b的Length长度只有4也就是再该代码块中可以使用，当程序执行完该代码块中内容后，变量c占用变量b的Solt槽，这样就完成了Solt槽的重复利用。

###### 1.4 局部变量表在使用前注意

==局部变量在使用前必须赋予初值==

在类加载子系统章节中，类变量有两次赋予初值的机会，一次在准备过程中，赋予系统初值，另外一次在初始化阶段，赋予程序员定义的初值，所以类变量在使用前不需要赋予初值也不会报错，例如下面程序

```java
    private static int num1;
    private int num2;

    @Test
    public void test(){
        System.out.println(this.num2);
        System.out.println(StackTest.num1);
    }
```

但是局部变量不一样了，一个局部变量在使用前如果是没有赋予初值的话是不能使用的；

```java
    public void methodA() {
        //错误
        int i ;
        System.out.println(i);
    }
```

编译器在编译该代码时也可以检测到该错误

###### 1.5局部变量表小结

1. 局部变量表，最基本的存储单元是Slot(变量槽)
2. 局部变量表所需的容量大小是在编译期确定下来的，并保存在方法的Code属性的max_locals（maximum local variables）数据项中。在方法运行期间是不会改变局部变量表的大小的
3. 局部变量表中的变量只在当前方法调用中有效。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁。
4. 如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数表顺序排列。
5. 由于局部变量表是建立在线程的栈上，是线程私有的数据，因此不存在数据安全问题
6. ==局部变量在使用前必须赋予初值==

##### 2.操作数栈

###### 2.1.概述

操作数栈也被成为操作栈，是一个后入先出的栈。与局部变量表一样，操作数栈的最大深度在编译时已经被确定，被写入到code属性中的max_stacks数据项之中

![](http://cdn.noteblogs.cn/20.png)

当一个方法执行时，这个方法的操作数栈是空的，在方法的执行过程中，会有各种的字节码指令向操作数栈中写入和提取内容，也就是入栈和出栈的操作。

操作数栈的作用类似于CPU的作用，在控制着整个代码的运行，而局部表量表类似于硬盘，只是做一个存储作用

###### 2.2 通过程序来理解

```java
    public void methodC(){
        byte i =15;
        int j = 8;
        int k = i + j;
    }
```

如上程序反编译后得到的结果是：

```java
 public void methodC();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        15
         2: istore_1
         3: bipush        8
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: return
      LineNumberTable:
        line 39: 0
        line 40: 3
        line 41: 6
        line 42: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/java/StackTest;
            3       8     1     i   B
            6       5     2     j   I
           10       1     3     k   I
```

在code属性中标明了操作数栈的深度 stack = 2，局部变量表的大小 locals = 4 ，args_size是方法参数的个数，这里是1，因为普通方法都有一个隐藏的this参数

 接下来中间的一段是操作指令：

```java
 0: bipush        15
2: istore_1
```

**①15入栈；②存储15，15进入局部变量表**

![](http://cdn.noteblogs.cn/21.png)

```java
3: bipush        8
5: istore_2
```

**③压入8；④存储8，8进入局部变量表；**

![](http://cdn.noteblogs.cn/22.png)

```java
6: iload_1
7: iload_2
8: iadd
```



**⑤从局部变量表中把索引为1和2的是数据取出来，放到操作数栈；⑥iadd相加操作，8和15出栈**

![](http://cdn.noteblogs.cn/23.png)



```java
9: istore_3
```



**⑦iadd操作结果23入栈；⑧将23存储在局部变量表索引为3的位置上**

![](http://cdn.noteblogs.cn/24.png)

###### 2.3 栈顶缓存技术

基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数

由于操作数是存储在内存中的，因此频繁地执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM的设计者们提出了栈顶缓存技术，**将栈顶元素全部缓存在内存里CPU的寄存器中，以此降低对内存的读/写次数，提升执行程序的执行效率**

##### 3. 动态链接

###### 3.1 概述

每一个栈帧内部都包含一个指向运行时常量池或该栈帧所属方法的引用。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接。比如invokedynamic指令

在Java源文件被编译成字节码文件中时，所有的变量和方法引用都作为符号引用（symbolic Refenrence）保存在class文件的常量池里。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。**

例如：

```java
public void methodA() {
    int j = 20;
    methodB();
}
```

反编译一下

![](C:\Users\VSUS\Desktop\笔记\JVM\img\25.png)

动态链接的作用就是将这些符号引用转换为调用方法的直接引用

![](http://cdn.noteblogs.cn/26.png)

###### 3.2 方法的调用

- 静态链接：当一个 字节码文件被装载进JVM内部时，如果被调用的目标方法在编译期可知，且运行期保持不变时。这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接。
- 动态链接：如果被调用的方法在编译期无法被确定下来，也就是说，只能够在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接。

- 早期绑定：**早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。**与静态链接相对应

- **晚期绑定（动态绑定）**：如果被调用的方法在编译期无法被确定下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式也就被称之为晚期绑定。与动态链接相对应

###### 3.3 虚方法和非虚方法

- 虚方法：可以被重写的方法
- 非虚方法：如果方法在编译器就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为非虚方法，==**静态方法、私有方法、final方法、实例构造器、父类方法都是非虚方法**==

###### 3.4 方法重写的本质

1 找到操作数栈的第一个元素所执行的对象的实际类型，记作C。

2.如果在类型C中找到与常量中的描述符合简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回java.lang.IllegalAccessError异常。

3.否则，按照继承关系从下往上依次对c的各个父类进行第二步的搜索和验证过程。

4.如果始终没有找到合适的方法，则抛出java.lang.AbstractMethodError异常。 **IllegalAccessError介绍** 程序视图访问或修改一个属性或调用一个方法，这个属性或方法，你没有权限访问。一般的，这个会引起编译器异常。这个错误如果发生在运行时，就说明一个类发生了不兼容的改变。

###### 3.5 虚方法表

在动态链接中，需要不断的去找到合适的方法，这样就会影响虚拟机执行的效率，jvm采用在类的方法区建立一个虚方法表（virtual method table）（非虚方法不会出现在表中）来实现。使用索引表来代替查找。	

虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始值准备完成之后，jvm会把该类的方法表也初始化完毕。

##### 4.方法返回地址

一个方法被执行有两种方式退出这个方法：

- 正常完成出口：
- 异常完成出口：在方法的执行过程中遇到了异常，而且这个异常在本方法非异常表里没有找到匹配的异常处理器

无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，**调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址。**而通过异常退出时，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。