# 一、概述

在学习完JAVASE阶段的高级框架例如Spring、Mybatis等等，总会去思考这些框架是如何工作的，其中的底层原理是什么？但是要了解这些高级框架的设计原理，首先Java基础得扎实，Java基础阶段的反射、注解、动态代理与设计模式构成了一些高级框架的基础。其实在基础篇的学习中，我已经反复的学习过反射，但是仅仅是当时学习时使用到，到后面的学习中就没有在使用过反射，因此重新写一遍反射的相关知识

```java
List<Integer> list = new ArrayList<>();
```

上面是最简单的new 一个对象，而这种new的前提是我已经知道了初始化的对象是什么，所以在编译阶段就对这个类进行实例化了，那么思考，如果是在运行阶段我们需要去new一个对象呢，也就是说这个对象我们事先不知道它的具体类型是什么，例如在Spring中的IOP中可以在运行中对标注了特定注解或者在配置文件中配置了全限定类名的类进行实例化，很显然使用new的这种是无法做到的，因为配置文件是多变的

这个时候就需要用到我们Java强大的反射

`Java 反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法，对于任意一个对象都能够调用它的任意一个属性和方法。这种在运行时动态的获取信息以及动态调用对象的方法的功能称为 Java 的反射机制。`(==注意关键字运行时==)

一个Java类有三部分：实例变量(数据成员)、构造方法、实例方法，那么反射机制的作用就是能在运行时获取一个Java类的这三部分并执行，具体如下：

- 在==运行时==判断任意一个对象所属的类型
- 在==运行时==构造任意一个类的对象
- 在==运行时==判断任意一个类所具有的成员变量和方法
- 在==运行时==调用任意一个对象的方法，甚至可以调用private方法。

具体运用上，Java反射就是一组在java.lang.reflect下的API

| 类包                          | 作用                                     |
| ----------------------------- | ---------------------------------------- |
| java.lang.Class               | 代表一个类                               |
| java.lang.reflect.Constructor | 代表类的构造方法                         |
| java.lang.reflect.Field       | 代表类的成员变量                         |
| java.lang.reflect.Method      | 代表类的方法                             |
| java.lang.reflect.Modifier    | 用于判断和获取某个类、变量或方法的修饰符 |

下面分别介绍这些类的方法，然后最后给出一个案例

# 二、反射API中的方法

### 2.1 获取一个类

获取一个类有三种方法：

1. 使用.class方法获取一个类
2. 用Object类的getClass()方法
3. 使用Class.forName()方法

```java
//使用.class方法
Class<Person> personClass = Person.class;
//使用Object类的getClass()方法
Person person = new Person();
Class<? extends Person> personClass1 = person.getClass();
//使用Class.forName()方法
Class<?> personClass2 = Class.forName("cn.Reflect.Person");
```

### 2.2 Class类中的方法

获取到Class类对象后，可以调用下面这些方法获取类中的其他部分

==1.获取成员变量==

| 方法                                | 描述                                |
| ----------------------------------- | ----------------------------------- |
| Field[] getFields()                 | 获取所有public修饰的成员变量        |
| Field getField(String name)         | 获取指定名称的 public修饰的成员变量 |
| Field[] getDeclaredFields()         | 获取所有的成员变量，不考虑修饰符    |
| Field getDeclaredField(String name) | 获取指定名称的成员变量              |



==2.获取构造方法==

这些方法与获取成员变量相似，不作过多描述

| 方法                                                         | 描述 |
| ------------------------------------------------------------ | ---- |
| Constructor<?>[] getConstructors()                           |      |
| Constructor<T> getConstructor(类<?>... parameterTypes)       |      |
| Constructor<T> getDeclaredConstructor(类<?>... parameterTypes) |      |
| Constructor<?>[] getDeclaredConstructors()                   |      |

==3.获取成员方法==

| 方法                                                         | 描述 |
| ------------------------------------------------------------ | ---- |
| Method[] getMethods()                                        |      |
| Method getMethod(String name, 类<?>... parameterTypes)       |      |
| Method[] getDeclaredMethods()                                |      |
| Method getDeclaredMethod(String name, 类<?>... parameterTypes) |      |

==4.获取全类名==

-  String getName()  

==5.获取修饰符==

- int getModifiers()

==6.返回一个新实例==

- Object newInstance()

### 2.3 Field类中的方法

| 方法                               | 描述                                    |
| ---------------------------------- | --------------------------------------- |
| void set(Object obj, Object value) | 设置值                                  |
| Object  get(Object obj)            | 获取值                                  |
| setAccessible(true)                | 获取private修饰的值前，应该调用这个方法 |
| AnnotatedType getAnnotatedType()   | 获取该属性的注解                        |
| String getName()                   | 获取字段名                              |
| int getModifiers()                 | 获取字段权限                            |

### 2.4 Constructor类中的方法

| 方法                              | 描述                   |
| --------------------------------- | ---------------------- |
| T newInstance(Object... initargs) | 通过构造方法构造该对象 |
| String getName()                  | 获取字段名             |
| int getModifiers()                | 获取字段权限           |

### 2.5 Method类中的方法

| 方法                                      | 描述         |
| ----------------------------------------- | ------------ |
| Object invoke(Object obj, Object... args) | 执行方法     |
| String getName()                          | 获取字段名   |
| int getModifiers()                        | 获取字段权限 |

# 三、一些案例

### 3.1 输出类下的全部信息

来自Java核心技术，输入一个全限定类名，输出该类的全部信息

```java
package cn.Reflect;

import javafx.scene.input.InputMethodTextRun;

import java.lang.reflect.*;
import java.util.Scanner;

public class ReflectDemo2 {
    private StringBuffer sb;
    private String name;
    private Class clazz;
    public ReflectDemo2(){
        sb = new StringBuffer();
        //输入全限定类名
        Scanner sc = new Scanner(System.in);
        String name = sc.next();
        try {
            clazz = Class.forName(name);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    public void printSuper(){
        //获取父类
        Class<?> superclass = clazz.getSuperclass();
        //  public class java.lang.Double extends java.lang.Number
        //getModifiers方法返回一个int类型来标识，通过Modifier类转为具体的类型如public等等
        String modifiers = Modifier.toString(clazz.getModifiers());
        if(modifiers.length() > 0){
            sb.append(modifiers + " ");
        }
        sb.append("class" + name);
        //判断是否有父类并且该父类不为Object
        if(superclass != null && superclass != Object.class){
            sb.append("extends" + superclass.getName());
        }
        sb.append("\n{\n");
    }
    public void printConstructors(){
        //获取所有构造方法
        Constructor[] constructors = clazz.getDeclaredConstructors();
        for (Constructor c : constructors) {
            sb.append("\t");
            //public java.lang.Double(java.lang.String);
            //获取权限修饰符
            String modifiers = Modifier.toString(c.getModifiers());
            sb.append(modifiers + " ");
            //获取构造器方法名称
            String name = c.getName();
            //获取参数
            Class[] parameters = c.getParameterTypes();
            sb.append(name + "(");
            for(int i = 0;i < parameters.length;i++){
                if(i > 0) sb.append(",");
                sb.append(parameters[i].getName());
            }
            sb.append(");\n");
        }
    }
    public void printMethod(){
        //public int compareTo(java.1ang.Object);
        //获取所有方法
        Method[] declaredMethods = clazz.getDeclaredMethods();
        for (Method m : declaredMethods) {
            sb.append("\t");
            //获取权限修饰符
            String modifiers = Modifier.toString(m.getModifiers());
            sb.append(modifiers + " ");
            //获取返回值
            Class<?> returnType = m.getReturnType();
            sb.append(returnType.getName() + " " + m.getName() + "(" );
            //获取方法参数
            Class<?>[] parameters = m.getParameterTypes();
            for(int i = 0;i < parameters.length;i++){
                if(i > 0) sb.append(",");
                sb.append(parameters[i].getName());
            }
            sb.append(");\n");
        }
        sb.append("}");
    }
    public void printField(){
        //public static final double POSITIVE_INFINITY;

        Field[] fields = clazz.getDeclaredFields();
        for (Field f : fields) {
            sb.append("\t");
            //获取权限修饰符
            String modifiers = Modifier.toString(f.getModifiers());
            sb.append(modifiers + " ");
            //获取类型
            Class<?> type = f.getType();
            sb.append(type.getName() + " " + f.getName() + ";\n");
        }

    }
    public void printInfo(){
        this.printSuper();
        this.printField();
        this.printConstructors();
        this.printMethod();
        System.out.println(sb.toString());
    }
    public static void main(String[] args) {
        ReflectDemo2 reflectDemo2 = new ReflectDemo2();
        reflectDemo2.printInfo();
    }
}
```

这个案例其实就是对上面API的一些调用组合，当输入java.lang.Double时，控制台输出：

```java
public final classnullextendsjava.lang.Number
{
	public static final double POSITIVE_INFINITY;
	public static final double NEGATIVE_INFINITY;
	public static final double NaN;
	public static final double MAX_VALUE;
	public static final double MIN_NORMAL;
	public static final double MIN_VALUE;
	public static final int MAX_EXPONENT;
	public static final int MIN_EXPONENT;
	public static final int SIZE;
	public static final int BYTES;
	public static final java.lang.Class TYPE;
	private final double value;
	private static final long serialVersionUID;
	public java.lang.Double(double);
	public java.lang.Double(java.lang.String);
	public boolean equals(java.lang.Object);
	public static java.lang.String toString(double);
	public java.lang.String toString();
	public int hashCode();
	public static int hashCode(double);
	public static double min(double,double);
	public static double max(double,double);
	public static native long doubleToRawLongBits(double);
	public static long doubleToLongBits(double);
	public static native double longBitsToDouble(long);
	public volatile int compareTo(java.lang.Object);
	public int compareTo(java.lang.Double);
	public byte byteValue();
	public short shortValue();
	public int intValue();
	public long longValue();
	public float floatValue();
	public double doubleValue();
	public static java.lang.Double valueOf(java.lang.String);
	public static java.lang.Double valueOf(double);
	public static java.lang.String toHexString(double);
	public static int compare(double,double);
	public static boolean isNaN(double);
	public boolean isNaN();
	public static boolean isFinite(double);
	public static boolean isInfinite(double);
	public boolean isInfinite();
	public static double sum(double,double);
	public static double parseDouble(java.lang.String);
}
```

### 3.2 根据类名创建该对象并执行方法

需求：在properties配置文件中配置className、methodName这两个属性，需要根据配置文件来创建相应对象并且执行methodName方法

```java
public class ReflectDemo3 {
    public static void main(String[] args) {
        //1.创建properties对象
        Properties properties = new Properties();
        //获取根路径下的properties流对象
        InputStream inputStream = ReflectDemo3.class.getClassLoader().getResourceAsStream("pro.properties");
        try {
            properties.load(inputStream);
            //获取对应属性
            String className = properties.getProperty("className");
            String methodName = properties.getProperty("methodName");
            //得到该类对象
            Class<?> clazz = Class.forName(className);
            //创建该对象的无参构造
            Object obj = clazz.newInstance();
            //获取方法对象
            Method method = clazz.getDeclaredMethod(methodName);
            //暴力反射
            method.setAccessible(true);
            //执行方法
            method.invoke(obj);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

配置文件内容：

```properties
className=cn.Reflect.Person
methodName=method2
```

缺点：无法创建有参构造器的对象，以及执行有参数的方法，这点本身是配置文件properties的缺点，后续在介绍完注解后，会模拟一个Spring的IOC的创建规则创建对象