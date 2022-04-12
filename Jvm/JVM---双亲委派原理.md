参考：《深入理解Java虚拟机第三版》

​			《宋红康JVM教程》

前言：本文将介绍类加载器的分类，什么是双亲委派机制，这种机制有什么好处？在明白双亲委派机制的前提是要了解类加载器。

# 一、类加载器ClassLoader

在虚拟机的角度，一共有两种类加载器，==①启动类加载器，②其他的加载器==，这样划分的依据是，启动类加载器是使用C/C++编写的，其他加载器是java语言编写的。但是这样划分过于笼统。

站在Java开发人员的角度，类加载器ClassLoader分为两类：==①虚拟机自带的加载器，②用户自定义类加载器==

##### 1.虚拟机自带的加载器

###### 1.1BootStrap ClassLoader

BootStrap ClassLoader，启动类加载器具有以下特点：

- 这个类加载使用**C/C++语言实现的**，嵌套在JVM内部

- 它用来加载java的核心库（JAVA_HOME/jre/lib/rt.jar/resources.jar或sun.boot.class.path路径下的内容），用于提供JVM自身需要的类

- 并不继承自java.lang.ClassLoader,没有父加载器

- 加载拓展类和应用程序类加载器，并指定为他们的父加载器

- ==处于安全考虑，BootStrap启动类加载器只加载包名为java、javax、sun等开头的类==

###### 1.2**Extension ClassLoader**

**Extension ClassLoader**  **拓展类加载器**

- java语言编写 ，由sun.misc.Launcher$ExtClassLoader实现。

- 派生于ClassLoader类

- 父类加载器为启动类加载器

- 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。**如果用户创建的JAR放在此目录下，也会由拓展类加载器自动加载**

###### 1.3**AppClassLoader**

**AppClassLoader**，**应用程序类加载器**

- java语言编写， 由sun.misc.Launcher$AppClassLoader实现。
- 派生于ClassLoader类
- 父类加载器为拓展类加载器
- 它负责加载环境变量classpath或系统属性 java.class.path指定路径下的类库

**该类加载器是程序中默认的类加载器**，一般来说，java应用的类都是由它来完成加载

通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

```java
/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/4 9:46
 */
public class demo2 {
    public static void main(String[] args) {

        //应用程序加载器  sun.misc.Launcher$AppClassLoader@18b4aac2
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);

        //扩展类加载器  sun.misc.Launcher$ExtClassLoader@1540e19d
        ClassLoader exClassLoader = systemClassLoader.getParent();
        System.out.println(exClassLoader);

        //试图获取引导类加载器  null
        ClassLoader bootstrapClassLoader = exClassLoader.getParent();
        System.out.println(bootstrapClassLoader);
    }
}

```

通过上面程序可以发现，引导类加载器Bootstrap ClassLoader获取下来的值是null，这样好理解，因为Bootstrap ClassLoader是使用C/C++语言编写的，自然获取不到。从上面程序也可以发现，Bootstrap ClassLoader、Extension ClassLoader、AppClassLoader存在层级关系

![](http://cdn.noteblogs.cn/6.png)

上面分别介绍了这3中加载器的特点，其实这3中加载器的最主要的区别在于，==分别加载不同的类==:

**AppClassLoader**：负责加载用户自己写的类

**Extension ClassLoader**：负责加载<Java_Home>\lib\ext目录下的类

**BootStrap ClassLoader**：负责加载Java核心类库，BootStrap启动类加载器只加载包名为java、javax、sun等开头的类

例如：尝试获取加载String类的类加载器：String类是由**BootStrap ClassLoader**加载，那么输出为null

```java
//输出 null
ClassLoader classLoader = String.class.getClassLoader();
System.out.println(classLoader);
```

>获取ClassLoader的方法：
>
>1.获取当前类的ClassLoader： clazz.getClassLoader
>
>2.获取当前线程的ClassLoader：Thread.currentThread().getContextCladdLoader()
>
>3.获取系统的classLoader：ClassLoader.getSystemClassLoader()
>
>4.获取调用者的classLoader：DriverManager.getCallerClassLoader()

##### 2.用户自定义加载器

* 隔离加载类
* 修改类加载的方式
* 拓展加载源
* 防止源码泄漏

# 二、双亲委派机制

通过一个实例来恰到好处的说明双亲委派机制的好处，例如：在工作目录下创建一个类，类名和包名为：java.lang.String，这里不需要质疑，有人会问，String类不是Java自带的类吗？这样创建是合法的，编译器并不会报错。在String类中，我们只是去输出一个语句例如：

```java
package java.lang;

/**
 * @author 四五又十
 * @version 1.0
 * @date 2020/7/6 18:29
 */
public class String {
    static {
        System.out.println("自定义String类");
    }
}

```

然后在一个main方法里去new 一个String

```java
public class demo4 {
    public static void main(String[] args) {
        String s = new String();
    }
}
```

现在思考，String类的加载是会加载jdk中的String，还是我们自定义的String方法，毕竟它们都在java.lang包下？如果是加载jdk下的String类，那么控制台不输出，如果是加载自定义中的String，因为在自定义的String类中使用了静态代码块，那么控制台肯定要输出一句话“自定义String类”?

答案是控制台不输出！！也就是Java虚拟机选择加载了jdk中的String，这就是双亲委派模型在起作用

==双亲委派机制的工作过程是：如果一个类加载器收到了加载类的请求，那么它不会自己尝试去加载这个类，而是把请求都交给父类加载器，父类加载器无法加载在交给更上一层的父类加载器，依次递归，只有当父类加载器无法加载该类时，子加载器才会尝试加载。==

根据双亲委派模型，可以得出加载如上demo4的过程：类加载器收到请求要加载demo4，那么虚拟机会将该类交给父加载器**Extension ClassLoader**，**Extension ClassLoader**一看这个类我也没有办法加载，那么再交给父类加载器**BootStrap ClassLoader**，**BootStrap ClassLoader**也没法加载，那么还是向下委派，直到**AppClassLoader**，**AppClassLoader**是负责加载用户自定义类的，它可以加载，那么demo4就被加载到虚拟机中了。

再看String类的加载过程，首先**AppClassLoader**收到要加载String类的请求，**AppClassLoader**自己不会去尝试加载，要交给父类**Extension ClassLoader**，**Extension ClassLoader**一看我也没有权限加载，那么在向上委派交给**BootStrap ClassLoader**，BootStrap ClassLoader一看String是java包下的，加载这种类是属于BootStrap ClassLoader的工作，那么Stirng类就被加载至虚拟机上了，那么既然String类是由BootStrap ClassLoader加载的，那么显然是jdk中的String

在源码中，实现双亲委派机制的代码在java.lang.ClassLoader的loadClass方法中：

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

