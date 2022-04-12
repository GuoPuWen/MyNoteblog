来源：java核心技术，博客

String类型是我们java开发中比较常见的一个类，但是这个常见的类却隐藏着很多的知识点，现做一个总结。

### 一、创建String的方式

##### 1. String str1 = “hello”

​	要深入明白这种创建String的方式，就不得不提到字符串常量池

> 字符串常量池：为了减少在JVM中创建的字符串的数量，字符串类维护了一个字符串池，每当代码创建字符串常量时，JVM会首先检查字符串常量池。如果字符串已经存在池中，就返回池中的实例引用。如果字符串不在池中，就会实例化一个字符串并放到池中。
>
> 注意：常量池在java用于保存在编译期已确定的，已编译的class文件中的一份数据。

使用这种方式创建字符串，在编译期的时候就对常量池进行判断是否存在该字符串，如果存在则不创建直接返回对象的引用；如果不存在，则先在常量池中创建该字符串实例再返回实例的引用给str1

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\1.png)

##### 2.String str2 = new Stirng(“hello”)

​	str2的创建使用new关键字，首先，JVM会检查字符串常量池，如果该字符串在常量池中，那么不再在字符串常量池创建该字符串对象，而直接堆中复制该对象的副本，然后将堆中对象的地址赋值给引用str2，如果字符串不存在常量池中，就会实例化该字符串并且将其放到常量池中，然后在堆中复制该对象的副本，然后将堆中对象的地址赋值给引用str2。

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\2.png)

在jdk7之后，字符串常量池被移到了堆中了

##### 3.常见问题，加深理解

```java
String str1 = "hello";
String str2 = "hello";
//true
System.out.println(str1 == str2);
String str3 = new String("hello");
String str4 = new String("hello");
//false
System.out.println(str3 == str4);
//false
System.out.println(str1 == str3);
```

分析：“==”比较的是该变量的值，也就是说，比较的是str1和str2的内存地址，而不是比较str1和str2指向的内容。对于str1和str2都是使用字面量的方式创建的，str1和str2都是指向常量池中的“hello”，所以为true；对于str3和str4，指向的是在堆中的不同"hello"；str1与str3比较很显然是false，指向的内存区域都不一样

str1与str2比较

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\3.png)

str3与str4比较

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\4.png)

为验证上述分析是否正确，现在反编译一下，为了使反编译结果更加清晰，选择先将后面两个比较代码注释掉。

```java
String str1 = "hello";
String str2 = "hello";
```

反编译结果：javap -c 

```java
Compiled from "StringDemo.java"
public class com.Demo.StringDemo {
  public com.Demo.StringDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String hello
       2: astore_1					
       3: ldc           #2                  // String hello
       5: astore_2
       6: return
}
```

查询jvm指令码

> ldc ：将int, float或String型常量值从常量池中推送至栈顶
>
> astore_1：将栈顶引用型数值存入第二个本地变量
>
> astore_2 ：将栈顶引用型数值存入第三个本地变量 

然后查看常量池：javap -verbose

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\5.png)

#2 代表的就是"hello"，那么从反编译的结果来看，确实str1和str2引用了常量池中的同一个对象。

接下来，分析str3和str4，为了避免其他因素，把字符串的值改一下

```java
String str3 = new String("hello world");
String str4 = new String("hello world");
```

反编译结果：

```Java
Compiled from "StringDemo.java"
public class com.Demo.StringDemo {
  public com.Demo.StringDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/String
       3: dup
       4: ldc           #3                  // String hello world
       6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
       9: astore_1
      10: new           #2                  // class java/lang/String
      13: dup
      14: ldc           #3                  // String hello world
      16: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
      19: astore_2
      20: return
}

```

查看常量池：

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\6.png)

这也证实了，使用new的方式创建一个字符串，要先判断常量池中是否有有该对象，如果没有那么要先在常量池中创建该对象，然后拷贝到堆中，然后将地址值赋值给变量str。这里也有一个很常见的问题

​	上述代码在执行的过程中创建了几个对象？

​	很显然，通过反编译可以得知在常量池内创建了一个对象，在对堆内创建了2个对象

### 二、String的不可变性

​	在String的官方API上，有这么一段话：Strings are constant; their values cannot be changed after they are created. String buffers support mutable strings，翻译过来就是字符串不变; 它们的值在创建后不能被更改。 字符串缓冲区支持可变字符串。

​	这句话我认为有两个理解：在常量池中不可能存在一模一样的字符串，对String字符串的任意操作都会创建一个新的对象，而不是使用当前对象，

### 三、String字符串“+”引发的一些问题思考

```java
String str1 = "a";
String str2 = "b";
String str3 = "a" + "b";
String str4 = "ab";
String str5 = str1 + str2;
//true
System.out.println(str3 == str4);
//false
System.out.println(str3 == str5);
```

反编译一下看看什么结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\8.png)

​	从反编译的结果很显然可以看出，对于str3由于+号两侧都是编译期确定的字符串常量，JVM会直接从常量池中将这两个字符串拼接好，那么str3 == str4的结果是true(前面说过使用这种方式创建字符串会在常量池中拿)。

​	对于str5加号两侧都是变量，都是在编译期无法确定的，那么使用”+“时，JVM会自动隐式的创建一个StringBuilder，然后调用append()方法，最后调用toString将结果返回至str5，那么str3 == str5的结果自然时false，str5与str3所指的内存区域都不一样，str3指向常量池，str5指向堆。

对上面代码进行修改

```java
final String str1 = "a";
final String str2 = "b";
String str3 = "a" + "b";
String str4 = "ab";
String str5 = str1 + str2;
//true
System.out.println(str3 == str4);
//true
System.out.println(str3 == str5);
```

同样进行反编译

![](C:\Users\four and ten\Desktop\笔记\JAVA\String\9.png)

​	可以很显然的看到，str5直接从常量池中获取字符串，这是因为当变量有final修饰时，JVM会在编译期将该变量看成一个常量，将该值拷贝到常量池中，这样以来，str5也是直接从常量池中获取值，所以结果是true。

​	从上面可以看出，对于”+“号两侧都是常量，JVM会直接从常量池中获取，效率较高，节省空间；对于对于”+“号两侧不都是常量（一个常量，0个常量）的字符串拼接，JVM会创建StringBuffered然后调用其方法append，最后使用toString将返回值赋给变量。所以在对不都是常量的字符串拼接时尽量不要使用String的”+“号，这样效率更低。

### 四、intern方法

​	String的intern是一个native本地方法，这个方法的解释是：当调用 intern 方法时，如果常量池中已经该字符串，则返回池中的字符串；否则将此字符串添加到常量池中，并返回字符串的引用。

​	通过一个例子来很好的理解这个方法，前面说过

```Java
String str1 = "hello";
String str3 = new String("hello");
//false
System.out.println(str1 == str3);
```

因为str1与str3指向的内存地址不一样，但是如果使用intern方法，则结果时true

```java
String str1 = "hello";
String str3 = new String("hello");
//true
System.out.println(str1 == str3.intern());
```

​	前面说过，str3的创建方式是先看字符串常量池中是否有该字符串，如果没有则创建一个，如果有那么将该字符串拷贝一份到堆中，然后返回该对象的引用给str3，也就是说str3指向的内存区域是堆。

​	但是一但使用intern方法，则JVM会查看常量池中是否有该字符串，如果有那么返回常量池中的字符串引用，如果没有则创建一个在返回该引用，也就是说使用intern发方法则不会再堆上创建该对象的引用。

​	使用这个方法的用处很显然，当重复使用的字符串数量很多时，例如，”hello“这个字符串要使用10000次，那么如果每次都要从堆上创建对象，那么将会很耗时而且很消耗空间。

​	在看下面代码

```java
String str1 = "hello";
String str3 = new String("hello");
str3.intern();
System.out.println(str1 == str3);
```

​	与上面代码的不同之处在于将str3另起一行，输出的结果就是false，原因很显然，前面说过字符串是不可变的，对他的任意操作都是创建一个副本，那么str3调用了intern方法，但str3还是str3还是指向了堆上的空间，结果自然为false。这就类似与强制类型转换，知识在使用副本操作。