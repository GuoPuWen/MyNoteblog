# 一、概述

jdk1.8是Java语言开发的一个重要版本，Java 8 是oracle公司于2014年3月发布，可以看成是自Java 5 以来最具革命性的版本。Java 8为Java语言、编译器、类库、开发 工具与JVM带来了大量新特性。

[What's New in JDK 8](https://www.oracle.com/java/technologies/javase/8-whats-new.html)

jdk1.8的新特性大概有以下几点：

- Lambda表达式
- 函数式接口
- 方法引用与构造器引用
- Stream API
- Optional
- 新的日期和时间API
- 其他新特性

下面一一介绍，jdk8新特性具有以下特点：

- 速度更快
- 代码更少
- 强大的Stream API
- 便于并行
- 最大化减少快指针异常：Optional
- Nashorn引擎，允许JVM运行JS应用

# 二、Lambda表达式

Lambda是一个匿名函数，可以把Lambda表达式理解为==一段可以传递的代码==(将代码像数据一样进行传递)，使用它可以写出更简洁、更灵活的代码，《Java核心技术》上说Lambda是简洁优雅的生产力

### 2.1从匿名内部类到Lambda

例如，在多线程代码中，当然这是一个假的多线程，这里只是举匿名内部类的例子

```java
public class demo1 {
    public static void main(String[] args) {
        Runnable r1 = new Runnable(){
            @Override
            public void run() {
                System.out.println("这是一个假的多线程");
            }
        };
        r1.run();
    }
}
```

**使用Lambda表达式**

```java
Runnable r2 = () -> System.out.println("这是一个假的多线程");
r2.run();
```

可以看出来使用Lambda是代码更加简洁，这样简化是有道理的，原本在匿名内部类的写法中，是有第6行是有效的，这段代码才是我们开发者需要关注的

### 2.2 Lambda语法

Lambda是一种语法，对于语法的学习，熟悉就是理解

Lambda需要使用一种新的语法元素和操作符，这个操作符为“->”，所以也叫箭头函数，这个操作符称为Lambda操作符。这个操作符将Lambda分为了两个部分

- 箭头左侧：指定了Lambda表法式需要的参数列表

- 箭头右侧：指定了Lambda体，是抽象方法的实现逻辑，也就是我们最需要关心的地方

准备工作：为了更好演示Lambda表达式，先给出5个接口，分别对应：无参无返回值，一个参数无返回值，两个参数(多个)无返回值，无参有返回值，一个参数有返回值

```java
public interface NoReturnNoParam {
    void method();
}
public interface NoReturnOneParam {
    void method(int a);
}
public interface NoReturnMutilParam {
    void method(int a , int b);
}
public interface ReturnNoParam {
    int method();
}
public interface ReturnOneParam {
    int method(int a);
}
```

==Lambda语法一：无参无返回值==

```java
NoReturnNoParam nrnp = () -> { System.out.println("method方法执行了"); };
nrnp.method();
```

==Lambda语法二：一个参数无返回值==

```java
NoReturnOneParam nrop = (int a) -> { System.out.println(a + "\tmethod方法执行了"); };
nrop.method(1);
//也可以省略参数类型
NoReturnOneParam nrop = (a) -> { System.out.println(a + "\tmethod方法执行了"); };
nrop.method(1);
//接着 可以把小括号去掉
NoReturnOneParam nrop = a -> { System.out.println(a + "\tmethod方法执行了"); };
nrop.method(1);
//在Lambda体中只有一条语句，可以省略大括号
NoReturnOneParam nrop = a -> System.out.println(a + "\tmethod方法执行了");
nrop.method(1);
```

==Lambda语法三：多个参数无返回值==

```java
NoReturnMutilParam nrmp = (int a , int b) -> { System.out.println(a + b + "\tmethod方法执行了"); };
nrmp.method(1, 2);
//可以省略参数类型
NoReturnMutilParam nrmp = (a , b) -> { System.out.println(a + b + "\tmethod方法执行了"); };
nrmp.method(1, 2);
//这里就不能省略括号了，因为有两个参数
//在Lambda体中只有一条语句，可以省略大括号
```

==Lambda语法四：无参有返回值==

```java
ReturnNoParam rnp = () -> {return 1;};
System.out.println(rnp.method());
//Lambda体中只有一条语句，可以省略return与大括号
ReturnNoParam rnp = () -> 1;
System.out.println(rnp.method());
```

==Lambda语法五：一个参数有返回值==

​	你已经懂了 

# 三、 函数式接口

Lambda能让代码更加简洁，但是什么时候可以使用Lambda呢？从上面的实例中可以知道，当一个接口只包含一个抽象方法时可以使用Lambda表达式，这种接口也称为函数式接口
		我们可以在一个接口上使用 **@FunctionalInterface** 注解，这样做可以检 查它是否是一个函数式接口。同时 javadoc 也会包含一条声明，说明这个 接口是一个函数式接口。例如Runnablke是一个·函数式接口

![image-20210126123321490](http://cdn.noteblogs.cn/image-20210126123321490.png)

前面定义的5个接口也是函数式接口，但是可想而知前面手动定义的函数式接口是不可以通用的，jdk 内置了四大函数式接口

| 函数式接口              | 参数类型 | 返回类型 | 抽象方法          | 用途                                                   |
| ----------------------- | -------- | -------- | ----------------- | ------------------------------------------------------ |
| Consumer<T>消费型接口   | T        | void     | void accept(T t)  | 对类型为T的对象应用操作                                |
| Supplier<T>供给型接口   | 无       | T        | T get()           | 返回类型为T的对象                                      |
| Functuon<T,R>函数型接口 | T        | R        | R apply(T t)      | 对类型为T的对象应用操作，并返回结果，结果是R类型的对象 |
| Predicate<T> 断定型接口 | T        | boolean  | boolean test(T t) | 确定类型为T的对象是否满足约束条件                      |

下面是一些举例

Consumer是消费型接口，有一个参数无返回类型，意味你给我东西消费

```java
public static void main(String[] args) {
    happyTime(100, money -> System.out.println("我消费了" + money));
}

public static void happyTime(double money, Consumer<Double> con){
    con.accept(money);
}
```

```java
public static void main(String[] args) {
    List<String> list1 = Arrays.asList("天津", "北京", "南京", "东京", "西京");
    List<String> list = filterString(list1, str -> str.contains("京"));
    System.out.println(list);
}

public static List<String> filterString(List<String> list, Predicate<String> pred){
    List<String> res = new ArrayList<>();
    for(String str : list){
        if(pred.test(str)){
            res.add(str);
        }
    }
    return res;
}
```

除去上面比较常用的四种函数式接口之外，jdk还提供了许多其他的接口

![image-20210126144535374](http://cdn.noteblogs.cn/image-20210126144535374.png)

# 四、方法引用与构造器引用

### 4.1 方法引用

当要传递给Lambda体的操作已经有实现的方法了，就可以使用方法引用。方法引用可以看做是对Lambda表达式深层次的表达，==方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法==

要能够使用方法引用必须要求接口的抽象方法的方法签名(也就是参数列表和放回值类型)必须和方法引用的方法的方法签名一致，方法引用使用“::”来标识

方法引用有三种情况：

- 对象 :: 实例方法名
- 类 :: 静态方法名
- 类 :: 实例方法名

==情况一：对象 :: 实例方法名==

```java
Consumer<String> con = str -> System.out.println(str);
con.accept("北京");
```

对于Consumer中的accept方法：void accept(T t)，System.out中的println方法：void println(T t)，所以可以使用方法引用

```java
/**
     * Consumer中的accept方法：void accept(T t)
     * System.out中的println方法：void println(T t)
     * 所以可以使用方法引用
     */
@Test
public void test1(){
    Consumer<String> con = str -> System.out.println(str);
    con.accept("北京");

    PrintStream ps = System.out;
    Consumer cons = ps :: println;
}
```

```java
/**
     * Function接口中的apply方法： R apply(T t);
     * ArrayList对象实例中的get方法：E get(int index);
     */
@Test
public void test2(){
    List<Integer> list = Arrays.asList(1, 2, 3, 4);
    
    Function<Integer,Integer> func1 = (index) -> list.get(index);
    System.out.println(func1.apply(1));

    Function<Integer,Integer> func2 = list :: get;
    System.out.println(func2.apply(1));
}
```

==情况二：类 :: 静态方法==

```java
/**
     * Integer类的compare方法 int compare(int x, int y)
     * Comparator的compare方法：int compare(T o1, T o2);
     */
@Test
public void test3(){
    Comparator<Integer> comp = (t1, t2) -> Integer.compare(t1, t2);
    System.out.println(comp.compare(1, 2));

    Comparator<Integer> comp2 = Integer::compare;
    System.out.println(comp2.compare(1,-1));
}
```

==情况三：类 :: 实例方法==

```java
/**
     * String类中的 int str1.compareTo(str2)
     * Comparator中的 int compare(str1, str2)
     */
@Test
public void test4(){
    Comparator<String> comp = (str1, str2) -> str1.compareTo(str2);
    System.out.println(comp.compare("abc", "abd"));

    Comparator<String> comp2 = String::compareTo;
    System.out.println(comp2.compare("abc", "abd"));
}
```

### 4.2 构造器引用

构造器引用和前面的方法引用其实是一样的，因为构造器也是方法

```java
@Test
public void test5(){
    Supplier<Employee> sup = () -> new Employee();
    System.out.println(sup.get());
    Supplier<Employee> sup2 = Employee::new;
    System.out.println(sup2.get());
}
```

# 五、Stream API

在Hadoop框架中有MapReduce，其中map阶段可以理解为分就是做映射，reduce阶段可以理解为合，我理解的Stream其实就是这种计算方式。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

在开发中，数据来源可能来自于mysql，那么可有在mysql中先处理好结果再返回java层面，但是如果数据来自于Radis那么这些NoSQL的数据必须由java层面来处理。而jdk8中提供的stream API就是提供了这么一种计算方式

Stream其实就是一组API，在最后会提供一个案例demo来学习

### 5.1 Stream概述

Stream提供了对集合操作的API，Stream讲的是计算，但是要注意：

- Stream不会存储元素
- Stream不会改变源对象，而是会生成一个新的Stream
-  Stream操作是延迟执行的，因为Stream有3个阶段，Stream会在需要结果的时候才执行

==Stream的三个操作步骤==

**1.创建Stream：**从一个数据源(集合或者数组)中获取一个流

**2.中间操作：**一个中间操作链，对数据源的数据进行处理

**3.终止操作：**一旦执行终止操作，才执行中间操作链，并产生结果，之后不能再被使用

![image-20210127104228518](http://cdn.noteblogs.cn/image-20210127104228518.png)

### 5.2 创建Stream

 ==1.通过集合==

jdk8中的Collection接口被扩展，提供了两个获取Stream流的方法：

- default Stream<E> stream()
- default Stream<E> parallelStream()

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);
Stream<Integer> stream = list.stream();
```

==2.通过数组==

jdk8的Arrays的静态方法stream()可以获取数组流

![image-20210127105458316](http://cdn.noteblogs.cn/image-20210127105458316.png)

```java
IntStream stream1 = Arrays.stream(new int[]{1, 2, 3, 4});
```

==3.通过Stream的of()==

可以调用Stream类的静态方法of()，通过显示值创建一个流，它可以接收任意数量的参数

public static<T> Stream<T> of(T... values)

```java
Stream<Integer> stream2 = Stream.of(1, 2, 3);
```

### 5.3 Stream的中间操作

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理，而在终止操作时一次性全部处理

==1.筛选与切片==

|        方法         |                           描述                           |
| :-----------------: | :------------------------------------------------------: |
| filter(Predicate p) |              传入Lambda，从流中排除某些元素              |
|     distinct()      | 筛选，通过流所产生元素的hashcode()和equals()去除重复元素 |
| limit(long maxSize) |              截断流，使其元素不超过给定数量              |
|    skip(long n)     |  跳过元素，返回一个去除了前n个元素的流，与limit方法相反  |

==2.映射==

|              方法               |                             描述                             |
| :-----------------------------: | :----------------------------------------------------------: |
|         map(Function f)         | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素 |
| mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的DoubleStream |
|    mapTolnt(TolntFunction f)    | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的IntStream |
|   mapToLong(ToLongFunction f)   | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的LongStream |
|       flatMap(Function f)       | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |

==3.排序==

| 方法                   | 描述                               |
| ---------------------- | ---------------------------------- |
| sorted()               | 产生一个新流，其中按自然顺序排序   |
| sorted(Comparator com) | 产生一个新流，其中按比较器顺序排序 |

### 5.4 Stream的终止操作

Stream的终止操作会从流水线产生结果，其结果可以不是任何不是流的值，例如：List，Integer。**一旦流进行了终止操作，不能在此使用**

==1.匹配与查找==

| 方法                           | 描述                     |
| ------------------------------ | ------------------------ |
| boolean allMatch(Predicate P)  | 检查是否匹配所有元素     |
| boolean anyMatch(Predicate P)  | 检查是否至少匹配一个元素 |
| boolean noneMatch(Predicate P) | 检查是否没有匹配元素     |
| Optional<T> findFirst()        | 返回第一个元素           |
| Optional<T> findAny()          | 返回当前流中的任意元素   |
| long count()                   | 返回流中元素个数         |
| Optional<T> max(Comparator c)  | 返回流中最大值           |
| Optional<T> min(Comparator c)  | 返回流中最小值           |
| forEach(Consumer c)            | 内部迭代，就是遍历元素   |

==2.规约==

| 方法                             | 描述                                                      |
| -------------------------------- | --------------------------------------------------------- |
| reduce(T iden, BinaryOpreator b) | 可以将流中的元素反复结合起来，得到一个值，返回T           |
| reduce(BinaryOpreator b)         | 可以将流中的元素反复结合起来，得到一个值，返回Optional<T> |

==3.收集==

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| collect(Collector c) | 将流转换为其他形式，接收一个Collector参数接口方法的实现，用于给Stream中的元素做汇总的方法 |

### 5.5 一些案例

```java
public class StreamExample {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("abc", "bc", "", "edg", "abcd", "jkl");
        //检查空字符串的个数
        long count = list.stream().filter(str -> str.isEmpty()).count();
        System.out.println("空字符串的个数:" + count);
        System.out.println("字符串长度大于2的");
        list.stream().filter(str -> str.length() > 2).forEach(System.out::println);
        List<String> list1 = list.stream().filter(str -> str.length() > 2).collect(Collectors.toList());
        System.out.println("列表为：" + list1);
        String s = list.stream().filter(str -> !str.isEmpty()).collect(Collectors.joining(", "));
        System.out.println("合并字符串" + s);

        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 6);
        List<Integer> squaresList = numbers.stream().map(i -> i * i).distinct().collect(Collectors.toList());
        System.out.println("Squares List: " + squaresList);
        IntSummaryStatistics summaryStatistics = numbers.stream().mapToInt(i -> i).summaryStatistics();
        System.out.println("列表中最大的数 :" + summaryStatistics.getMax());
        System.out.println("列表中最小的数 : " + summaryStatistics.getMin());
        System.out.println("所有数之和 : " + summaryStatistics.getSum());
        System.out.println("平均数 : " + summaryStatistics.getAverage());
        System.out.println("随机数: ");
    }
```

# 六、Optional

相比C++，Java去除了指针的概念，但是尽管如此，空指针异常还是困扰开发者的一大难题，而jdk8新增的Optional可以很好的解决空指针异常

Optional< T>类是一个容器类，它可以保存类型为T的值，代表这个值存在，也可以保存null，表示这个值不存在。Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional中的一些有用的方法如下：

==1.创建Optional类对象的方法==

| 方法                                       | 描述                            |
| ------------------------------------------ | ------------------------------- |
| static <T> Optional<T> empty()             | 返回空的 Optional 实例          |
| static <T> Optional<T> of(T t)             | 创建一个option实例，t必须非空   |
| static <T> Optional<T> ofNullable(T value) | 创建一个option实例，t可以为null |

==2.判断Optional容器是否包含对象==

| 方法                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| boolean isPresent()                          | 判断是否包含对象                                             |
| void ifPresent(Consumer<? super T> consumer) | 如果有值，会执行Consumer接口的实现代码，并且该值会作为参数传给它 |

==3.获取Optional容器的对象==

| 方法                                                   | 描述                                               |
| ------------------------------------------------------ | -------------------------------------------------- |
| T get()                                                | 如果调用镀锡包含该值，则返回该值，否则抛出异常     |
| T orElse(T other)                                      | 如果有值将其返回，否则返回指定的other对象          |
| T orElseThrow(Supplier<? extends X> exceptionSupplier) | 如果有值将其返回，否则抛出由Supplier接口实现的异常 |

一些案例

```java
public class OptionalDemo {
    public static void main(String[] args) {
        OptionalDemo demo = new OptionalDemo();
        Integer value1 = null;
        Integer value2 = new Integer(2);
        Optional<Integer> a = Optional.ofNullable(value1);
        Optional<Integer> b = Optional.of(value2);
        System.out.println(demo.sum(a, b));
    }

    public Integer sum(Optional<Integer> num1, Optional<Integer> num2){
        System.out.println("第一个参数值存在" + num1.isPresent());
        System.out.println("第二个参数值存在" + num2.isPresent());

        Integer value1 = num1.orElse(new Integer(0));
        Integer value2 = num2.get();
        return value1 + value2;
    }
}
```

![image-20210129102718181](http://cdn.noteblogs.cn/image-20210129102718181.png)

# 七、日期API

日期API使用最多的便是格式化当前日期，在java8之前我们一般这样做

```java
String pat = "yyyy-MM-dd";
SimpleDateFormat format = new SimpleDateFormat(pat);
System.out.println(format.format(new Date()));
```

这样做不仅繁琐，而且DateFormat是线程不安全的，如果多个线程调用，则会出现意外，使用老的日期API具有如下缺点：

- Java的`java.util.Date`和`java.util.Calendar`类易用性差，不支持时区，而且他们都不是线程安全的；
- 用于格式化日期的类`DateFormat`被放在`java.text`包中，它是一个抽象类，所以我们需要实例化一个`SimpleDateFormat`对象来处理日期格式化，并且`DateFormat`也是非线程安全，这意味着如果你在多线程程序中调用同一个`DateFormat`对象，会得到意想不到的结果。
- 对日期的计算方式繁琐，而且容易出错，因为月份是从0开始的，从`Calendar`中获取的月份需要加一才能表示当前月份。

基于以上问题，java8提供了一套用于处理时间和时区的API

### 7.1 日期/时间

jak8主要提供了3个类用于处理时间和日期(其实还有不少，这3个类最为常见)

- LocalDate：表示一个具体的日期，但是不包含具体的时间，也不包含时区信息
- LocalTime：表示一个具有的时间，但是不包含日期信息
- LocalDateTime：是LocalDate和LocalTime的结合体

==LocalDate==

```java
LocalDate date = LocalDate.of(2020, 1, 29);
int year = date.getYear();
Month month = date.getMonth();
int dayOfMonth = date.getDayOfMonth();
DayOfWeek dayOfWeek = date.getDayOfWeek();     // 一周的第几天
int length = date.lengthOfMonth();             // 月份的天数
boolean leapYear = date.isLeapYear();          // 是否为闰年
//获取当前日期
LocalDate now = LocalDate.now();
```

==LocalTime==

```java
LocalTime localTime = LocalTime.of(17, 23, 52);     // 初始化一个时间：17:23:52
int hour = localTime.getHour();                     // 时：17
int minute = localTime.getMinute();                 // 分：23
int second = localTime.getSecond();                 // 秒：52
```

==LocalDateTime==

```java
LocalDateTime ldt1 = LocalDateTime.of(2017, Month.JANUARY, 4, 17, 23, 52);

LocalDate localDate = LocalDate.of(2017, Month.JANUARY, 4);
LocalTime localTime = LocalTime.of(17, 23, 52);
LocalDateTime ldt2 = localDate.atTime(localTime);
```

==日期格式化==

```java
LocalDateTime dateTime = LocalDateTime.now();
String strDate1 = dateTime.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20170105
String strDate2 = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2017-01-05
String strDate3 = dateTime.format(DateTimeFormatter.ISO_LOCAL_TIME);    // 14:20:16.998
String strDate4 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));   // 2017-01-05
String strDate5 = dateTime.format(DateTimeFormatter.ofPattern("今天是：YYYY年 MMMM DD日 E", Locale.CHINESE)); // 今天是：2017年 一月 05日 星期四
```

### 7.2 时区

Java 8中的时区操作被很大程度上简化了，新的时区类`java.time.ZoneId`是原有的`java.util.TimeZone`类的替代品。`ZoneId`对象可以通过`ZoneId.of()`方法创建，也可以通过`ZoneId.systemDefault()`获取系统默认时区：

```java
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
ZoneId systemZoneId = ZoneId.systemDefault();
```

有了`ZoneId`，我们就可以将一个`LocalDate`、`LocalTime`或`LocalDateTime`对象转化为`ZonedDateTime`对象：

```java
LocalDateTime localDateTime = LocalDateTime.now();
//2021-01-29T11:17:47.135+08:00[Asia/Shanghai]
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, shanghaiZoneId);
```

