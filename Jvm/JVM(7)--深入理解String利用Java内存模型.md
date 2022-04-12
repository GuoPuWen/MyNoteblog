参考：《深入理解Java虚拟机》

​			《宋红康JVM教程》

之前学Java基础的时候写过一篇关于String类的思考（[对String的一些思考](https://blog.csdn.net/weixin_44706647/article/details/104999761)），但是那个时候还不懂Java的内存模型，当时为了证明是否正确，使用了反编译，还查阅了字节码指令。但是String远远没有那么简单，在理解了Java的内存模型之后，尤其是听了宋红康老师的String的教程，对String有更加深刻的理解。

### 一、引题开篇

首先给出一些常见的面试题，如果下面这些面试题，各位心里都有非常肯定的答案，那么这篇文章并不适合你

1. 下面操作一共创建了几个对象

```java
String str1 = new String("1");	//2个(不包括引用类型)
String str2 = new String("1") + new String("2");	//6个(不包括引用类型)
```

2. true or false?

```java
String str1 = new String("1");
String srt2 = str1.intern();
String str3 = "1";
System.out.println(str1 == srt2);	//jdk6: false jdk7/jdk8:false
System.out.println(srt2 == str3);	//jdk6:	true  jdk7/jdk8:true
```

3. true or false?

```java
String s1 = new String("1") + new String("1");
s1.intern();
String s2 = "11";
System.out.println(s1 == s2);	//jdk6:false	jdk7/jdk8:true
```

### 二、String的基本特性复习

- 创建一个字符串总体上有两种方式：

```java
String s1 = "hello";	//字面量的定义方式，这里声明的字符串在字符串常量池中
String s2 = new String("hello");
```

- String是不可被继承的，是被声明为final
- String实现了Serializable接口：表示字符串是支持序列化的。 实现了Comparable接口：表示String可以比较大小
- ==String在jdk8及以前内部定义了final char[]，value用于存储字符串数据。jdk9时改为final byte[]==

- String的不变性的理解：
  - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值
  - 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
  - 当调用String的replace（）方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

### 三、字符串常量池的一些理解

##### 1.为什么要有字符串常量池

- 字符串的分配，和其他的对象分配一样，耗费高昂的时间与空间代价，==大量频繁的创建字符串，极大程度地影响程序的性能==
- JVM==为了提高性能和减少内存开销==，在实例化字符串常量的时候进行了一些优化，所以为字符串开辟一个字符串常量池，类似于缓存区
- 创建字符串常量时，首先检查字符串常量池是否存在该字符串
- ==字符串常量池中是不会存储相同内容的字符串的==
- ==常量池就类似一个JAVA系统级别提供的缓存==

##### 2. 演进细节

- 在jdk6中，字符串常量池放在永久代（方法区在hostpot的具体实现）
- 在jdk7（jdk8 方法区已经改为元空间，在本地内存实现，jdk7相当于是永久代和元空间的过渡时间）及以上版本，字符串常量池放在堆上
- ![](http://cdn.noteblogs.cn/35.png)

![](http://cdn.noteblogs.cn/37.png)

为什么要调整字符串常量池的位置？

​	jdk开发人员要作如上改变，肯定是照顾到整体的性能的。在jdk6的时候，运行时常量池在永久代中，永久代的垃圾回收频率是很低的（永久代的垃圾回收要触发Full GC），甚至在Jvm虚拟机规范中并没有要求方法区要进行垃圾回收，但是字符串又是使用频率很高的，需要对一些不用的常量进行垃圾回收。如果将字符串常量池放在堆上，堆是垃圾回收的重点区域，所以能对字符串常量池进行及时的回收

##### 3.验证字符串常量池的存在

```JAVA
System.out.println();//3279
System.out.println("1");
System.out.println("2");
System.out.println("3");
System.out.println("4");
System.out.println("5");
System.out.println("6");
System.out.println("7");
System.out.println("8");
System.out.println("9");
System.out.println("10");//3289

System.out.println("1");//3289
System.out.println("2");//3289
System.out.println("3");//3289
System.out.println("4");//3289
System.out.println("5");//3289
System.out.println("6");//3289
System.out.println("7");//3289
System.out.println("8");//3289
System.out.println("9");//3289
System.out.println("10");//3289
```

输出10个字符串，非常简单的程序，前十个数的输出，因为字符串常量池中没有所以会创建，但是后面10个，因为有字符串常量池，所以直接从字符串常量池里取值，所以String的数目不会增加，可以使用调试该程序，然后调出memory窗口查看String的数量，具体看下图：

![](http://cdn.noteblogs.cn/38.png)

![](http://cdn.noteblogs.cn/39.png)

![](http://cdn.noteblogs.cn/40.png)

如果没有字符串常量池，那么后面10个也会再次创建字符串，内存中的字符串数目应该会增加，但是数目没有增加，可见Java中有字符串常量池这么一个结构，用来提高效率，节省内存。

### 四、字符串的拼接操作

因为String的不可变性，那么字符串的拼接操作之后的字符串是在字符串常量池呢，还是是个对象存储在堆上呢？先给出结论

- ==常量与常量的拼接结果在常量池，原理是编译期优化==
- ==只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder==
- ==如果拼接的结果调用intern（）方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址==

例子1：

```java
@Test
public void test1(){
    String s1 = "a" + "b" + "c";    //编译期优化：等同于"abc"
    String s2 = "abc";              //"abc"一定是放在字符串常量池中，将此地址赋给s2
    System.out.println(s1 == s2);    //true
    System.out.println(s1.equals(s2));  //true

}
```

例子2：

```java
@Test
public void test2(){
    String s = "a";
    String s1 = "b";
    String s3 = "ab";
    String s4 = s + s1;
    System.out.println(s3 == s4);//false

}
```

底层字节码：

```java
 0 ldc #6 <a>
 2 astore_1
 3 ldc #7 <b>
 5 astore_2
 6 ldc #8 <ab>
 8 astore_3
 9 new #9 <java/lang/StringBuilder>
12 dup
13 invokespecial #10 <java/lang/StringBuilder.<init>>
16 aload_1
17 invokevirtual #11 <java/lang/StringBuilder.append>
20 aload_2
21 invokevirtual #11 <java/lang/StringBuilder.append>
24 invokevirtual #12 <java/lang/StringBuilder.toString>
27 astore 4
29 getstatic #3 <java/lang/System.out>
32 aload_3
33 aload 4
35 if_acmpne 42 (+7)
38 iconst_1
39 goto 43 (+4)
42 iconst_0
43 invokevirtual #4 <java/io/PrintStream.println>
46 return
```

由字节码文件可知，“+”拼接操作（如果有变量的话）的底层使用了StringBuilder，也就是说上面的例子中底层的操作应该是：

```java
StringBuilder s = new StringBuilder();
s.append("a")
s.append("b")
s.toString()  --> 约等于 new String("ab")
```

### 五、append与“+”的效率比较

```java
@Test
public void test3(){
    double startTime  = System.currentTimeMillis();
    //method1(100000);  //3450
    method2(100000);    //15
    double endTime = System.currentTimeMillis();
    System.out.println(endTime - startTime);
}
public void method1(int highLevel){
    String src = "";
    for (int i = 0; i < highLevel; i++) {
        src = src + "a";
    }
}
public void method2(int highLevel){
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < highLevel; i++) {
        sb.append("a");
    }
    String src = sb.toString();
}
```

使用字符串拼接的方式，花费了3450ms，而使用append方法则只使用了15ms可见，使用append来拼接字符串大幅度的提高了效率，可以从以下两方面考虑：

- StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象，而使用String的字符串拼接方式：创建过多个StringBuilder和String的对象         

- 使用String的字符串拼接方式：内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；如果进行GC，需要花费额外的时间。

### 六、intern方法

##### 1.一共创建几个对象

String有个intern方法，是当前的字符对象（通过new出来的对象）可以使用intern方法从常量池中获取，如果常量池中不存在该字符串，那么就新建一个这样的字符串放到常量池中。

通俗点讲，Intern方法就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池（String Intern Pool）。

那么现在回到开篇的问题，下列代码创建多少个对象：

```java
String str1 = new String("1");	//2个(不包括引用类型)
String str2 = new String("1") + new String("2");	//6个(不包括引用类型)
```

==对于 new String("1")：==

查看字节码指令(注意：这里不包括引用str1)：

```java
 0 new #2 <java/lang/String>
 3 dup
 4 ldc #3 <1>
 6 invokespecial #4 <java/lang/String.<init>>
 9 pop
10 return
```

**所以这里创建了2个对象：**

- new 关键字在堆空间创建的

- 字符串常量池中的对象“1”

这里有一个疑问，当时在听课的时候没明白，为什么字符串常量池里面的也是对象？其实首先在学习Java的时候就已经强调过，万事万物都是对象。反过来说，如果这里的不是对象本身，而是一个引用，那么它是怎么做到让不同对象引用相同的字符串常量池里面的值呢，也就是下面的代码怎么会正确呢？

```java
String s1 = "1";
String s2 = "1";
System.out.println(s1 == s2);		//true
```

最正确的答案便是查找官方文档，官方文档里已经说了，this String object，所以这里的也是对象

![](http://cdn.noteblogs.cn/41.png)

对于 ==new String("1") + new String("2");==

查看字节码指令：

```java
 0 new #2 <java/lang/StringBuilder>
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>>
 7 new #4 <java/lang/String>
10 dup
11 ldc #5 <1>
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>
22 dup
23 ldc #8 <2>
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>
34 astore_1
35 return
```

* 对象1：new StringBuilder()
* 对象2： new String("1")
* 对象3： 常量池中的"1"
* 对象4： new String("2")
* 对象5： 常量池中的"2"
* 对象6 ：new String("12")

==这里有个很关键的点==，对象6有没有在字符串常量池里面生成对象，如果是直接使用new String("12")，那么必然在字符串里面有一份拷贝，会生成一个对象（前提是之前没有“12”在字符串常量池里面）。但是这里是Stringbuffer里面的toString方法==是不会在字符串常量池里面生成"12"对象的==

所以综上，总共生成了6个对象。

##### 2. 几道面试题详解

参考[美团技术团队](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)的文章，其实文章里已经说得很清楚了，在这里重新敲一下也只是想加深一下影响，争取从画蛇添足到画龙点睛！

```java
public void test5(){
    String s = new String("1");	①
    s.intern();					②		
    String s2 = "1";		·	③
    System.out.println(s == s2);④

    String s3 = new String("1") + new String("1");	⑤
    s3.intern();									⑥
    String s4 = "11";								⑦
    System.out.println(s3 == s4);					⑧
}
```

关于intern方法的题目，一定要清楚不同jdk版本之间的变化，在jdk6之前，字符串常量池是在永久代中的，而jdk7开始，字符串常量池已经被改到堆空间，那么回答这种问题肯定要反版本讨论。

上面的结果是：

* jdk6 下`false false`
* jdk7 下`false true`

上述代码中第①行使用new String的方式创建了一个字符串，这种方法会创建2个字符串，s指向堆中的实例，第②行使用intern方法，会查找字符串常量池中是否有“1”，查找有，所以这行没什么实际效果，第③行，因为字符串常量池中已经有“1”，那么直接将该对象返回给s2，所以不管是jdk6还是jdk7（或者更高），s指向的是堆，s2指向的是字符串常量池，所以打印肯定都是false

**下面说第二段代码：**

第二段代码，第⑤行总共创建4个对象：new StringBuilder一个、new String("1")二个，("11").toString一个，这里为第一段代码已经在字符串常量池中创建了“1"。==那这里其实要关注的核心问题就是，字符串常量池中是否有”11“这个对象呢？是没有在字符串常量池创建”11“对象的，因为调用的是StringBuilder的toString方法==。第⑥行调用intern方法，因为字符串常量池中没有“11“这个对象，所以调用intern'方法必定会在字符串常量池中生成”11“对象。到这里就有版本的区别，

在jdk6中，字符串常量池在永久代中，那么创建字符串就单独的创建一份，将值拷贝到字符串常量池中，所以这个字符串常量池中“11”的地址和堆中“11”的地址不一样，所以返回false

在jdk7中，字符串常量池在堆中，Java虚拟机为了节省空间或者提升效率，会直接将堆中的“11”对象的地址复制给字符串常量池中的“11”，也就是此时，堆中的“11”和字符串常量池中的“11”是同一个对象，那么使用=比较自然也是一样的值

##### 3.总结

jdk1.6中，将这个字符串对象尝试放入串池。

* ➢如果字符串常量池中有，则并不会放入。返回已有的串池中的对象的地址
* ➢如果没有，会把此==对象==复制一份，放入串池，并返回串池中的对象地址

Jdk1.7起，将这个字符串对象尝试放入串池。

* ➢如果字符串常量池中有，则并不会放入。返回已有的串池中的对象的地址
* ➢如果没有，则会把对象的==引用地址==复制一份，放入串池，并返回串池中的引用地址

##### 4. 扩展

```java
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();	//intern方法没有什么作用，因为此时字符串常量池中已经有“11”
    System.out.println(s3 == s4);
```

* jdk6 下`false false`
* jdk7 下`false false`