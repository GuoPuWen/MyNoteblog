## 一、概述

##### 1.概念

注解是一种能被添加到java代码中的元数据，类、方法、变量、参数和包都可以用注解来修饰。注解对于它所修饰的代码并没有直接的影响。注解是jdk1.5之后的特性

##### 2.注解的用法

注解有许多用法，其中有：**为编译器提供信息** - 注解能被编译器检测到错误或抑制警告。**编译时和部署时的处理** - 软件工具能处理注解信息从而生成代码，XML文件等等。**运行时的处理** - 有些注解在运行时能被检测到。

##### 3.作用

1. 编写文档：通过代码里标识的注解生成文档【生成文档doc文档】
2. 代码分析：通过代码里标识的注解对代码进行分析【使用反射】
3. 译检查：通过代码里标识的注解让编译器能够实现基本的编译检查

## 二、JDK预定义的注解

### (1)Java 内置三大注解

##### 1.@Override

检测被该注解标注的方法是否是继承自父类(接口)的

##### 2.@Deprecated

该注解标注的内容，表示已过时

##### 3.@SuppressWarnings

压制警告，一般传递参数all  @SuppressWarnings("all")

### (2)源注解(注解的注解)

##### 1.@Target

描述注解能够作用的位置

ElementType取值：

- TYPE：可以作用于类上
- METHOD：可以作用于方法上
-  FIELD：可以作用于成员变量上

##### 2.@Retention

描述注解被保留的阶段。

```java
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME;
}
```

  SOURCE： 将被限定在Java源文件中，那么这个注解即不会参与编译也不会在运行期起任何作用，这个注解就和一个注释是一样的效果，只能被阅读Java文件的人看到；

CLASS：将被编译到Class文件中，那么编译器可以在编译时根据注解做一些处理动作，但是运行时JVM（Java虚拟机）会忽略它，我们在运行期也不能读取到；

RUNTIME：可以在运行期的加载阶段被加载到Class对象中。那么在程序运行阶段，我们可以通过反射得到这个注解，并通过判断是否有这个注解或这个注解中属性的值，从而执行不同的程序代码段。**我们实际开发中的自定义注解几乎都是使用的RetentionPolicy.RUNTIME**；

##### 3.@Documented

描述注解是否被抽取到api文档中

##### 4.@Inherited

描述注解是否被子类继承

## 三、自定义注解

自定义注解的3个流程：

- **第一步，定义注解——相当于定义标记；**
- **第二步，配置注解——把标记打在需要用到的程序代码中；**
- **第三步，解析注解——在编译期或运行时检测到标记，并进行特殊操作。**

##### 1.声明注解

注解在Java中，与类、接口、枚举类似，因此其声明语法基本一致，只是所使用的关键字有所不同`@interface`。**在底层实现上，所有定义的注解都会自动继承java.lang.annotation.Annotation接口**。注解本质上就是一个接口，该接口默认继承Annotation接口

```java
public @interface demo1 {}
```

##### 2.实现注解

在类的实现部分无非就是书写构造、属性或方法。但是，在自定义注解中，其实现部分只能声明方法，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Anno1{
    public String name();
    int age() default 20;
    int[] array();
}
```

返回值类型：

* 基本数据类型(int float boolean byte double char long short)
* String
* 枚举
* 注解
* 以上类型的数组

注解内方法(属性)的声明规则：

1. 访问修饰符必须为public，不写默认为public；
2. 该元素的类型只能是基本数据类型、String、Class、枚举类型、注解类型（体现了注解的嵌套效果）以及上述类型的一位数组；
3. 该元素的名称一般定义为名词，如果注解中只有一个元素，把名字起为value（后面使用会带来便利操作）；
4. ()不是定义方法参数的地方，也不能在括号中定义任何参数，仅仅只是一个特殊的语法；
5. `default`代表默认值，值必须和第2点定义的类型一致；

可以看出：注解类型元素的语法非常奇怪，即又有属性的特征（可以赋值）,又有方法的特征（打上了一对括号）。但是这么设计是有道理的，我们在后面可以看到：注解在定义好了以后，**使用的时候操作元素类型像在操作属性，解析的时候操作元素类型像在操作方法**。

##### 3.使用注解

```java
public class demo1 {
    @Anno1(name = "demo1", array = {1,2})
    public void show(){
        System.out.println("demo1....");
    }
}
```

在要注解的括号内，以“元素名=元素”键值对的方法填上莫有默认的注解属性

##### 4.特殊用法

1. **如果注解本身没有注解类型元素，那么在使用注解的时候可以省略()，直接写为：@注解名，它和标准语法@注解名()等效**，比如说@Override等等
2. **如果注解本身只有一个注解类型元素，而且命名为value，那么在使用注解的时候可以直接使用：@注解名(注解值)，其等效于：@注解名(value = 注解值)**

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Anno1 {
   String value();
}
```

```java
public class demo1 {
    @Anno1("demo1")
    public void show(){
        System.out.println("demo1....");
   }
}
```

## 四、解析注解

1. getAnnotation(Class<A> annotationClass)方法是获取该元素上指定的注解。之后再调用该注解的注解类型元素方法就可以获得配置时的值数据；

例子：不改变类的任何代码，可以创建任意类的对象，可以执行任意方法

分析：实现一个类ReflectTest使用注解，将任意类的包名，需要实现的方法名，作为参数传递到注解上，使用getAnnotation()获取到该注解，得到包名和方法名。然后利用反射技术实现方法

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Anno2 {
   String classname();
   String mothod();
}
```

```java
@Anno2(classname = "cn.review.Annotation",mothod = "show")
public class ReflectTest {
    public static void main(String[] args) throws ClassNotFoundException {

        //获取字节码对象
        Class<ReflectTest> reflectTestClass = ReflectTest.class;
        //获取注解
        Anno2 anno = reflectTestClass.getAnnotation(Anno2.class);
        //获取任意类的classname mothod
        String cls = anno.classname();
        String mothod = anno.mothod();
        //使用反射

    }
}

```

