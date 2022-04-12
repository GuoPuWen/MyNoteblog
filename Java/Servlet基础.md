## 	1.概念

运行在服务器端的小程序(server applet)

-  Servlet就是一个接口，定义了Java类被浏览器访问到(tomcat识别)的规则。
- 将来我们自定义一个类，实现Servlet接口，复写方法。

## 2.使用

1. 创建JavaEE项目
2. 定义一个类，实现Servlet接口
3. 实现接口中的抽象方法
4. 配置Servlet

## 3.配置Servlet

##### 3.1在web.xml中配置：

```xml
<servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>cn.Servlet.ServletDemo1</servlet-class>
</servlet>
<servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <url-pattern>
            /demo1
        </url-pattern>
<servlet-mapping>
```

- **servlet-mapping**：注册servlet映射——把输入的url地址映射到servlet名
-  **url-pattern**：资源名称

```
url-pattern的配置模式：

1. /xxx：路径匹配
2. /xxx/xxx:多层路径，目录结构
3. *.do：扩展名匹配
```

- **servlet-name**：给某个servlet类起名
- **servlet-class**：写servlet类的全限定名

##### 3.2 Servlet3.0的简单配置

1. 好处：支持注解配置。可以不需要web.xml了。
2. 用法：在类上使用@WebServlet("资源名称")注解，进行配置

```java
@WebServlet("/demo1")
public class demo1 implements Servlet {
}
```



## 4.访问方法

- 在浏览器上，输(因为已经将端口改成80)localhost:/虚拟目录/资源名称。资源名称就是@WebServlet("资源名称")这里的，虚拟目录是
  ![](servlet\1.png)

  ![2](servlet\2.png)



## 5.执行原理

>1.  当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径
>2. 查找web.xml文件，是否有对应的<url-pattern>标签体内容。
>3. 如果有，则在找到对应的<servlet-class>全类名
>4. tomcat会将字节码文件加载进内存，并且创建其对象(反射)
>5.  调用其方法

## 6.Servlet中的生命周期

1. 被创建：执行init方法，只执行一次

   注意：默认情况下，第一次被访问时，Servlet被创建。

   可以配置执行Servlet的创建时机。

```
在<servlet>标签下配置
	1.第一次被访问时，创建
		<load-on-startup>的值为负数
	2. 在服务器启动时，创建
	 <load-on-startup>的值为0或正整数
```

servlet的init方法，只执行一次，说明一个Servlet在内存中只存在一个对象，Servlet是单例的，多个用户同时访问时，可能存在线程安全问题。所以尽量不要在Servlet中定义成员变量。即使定义了成员变量，也不要对修改值。

2. 提供服务：

   执行service方法，执行多次，每次访问Servlet时，Service方法都会被调用一次。

3.  被销毁：
   执行destroy方法，只执行一次， Servlet被销毁时执行。服务器关闭时，Servlet被销毁。 只有服务器正常关闭时，才会执行destroy方法。destroy方法在Servlet被销毁之前执行，一般用于释放资源

## 7.Servlet的体系结构	

Servlet -- 接口
					|
		GenericServlet -- 抽象类
				|
		HttpServlet  -- 抽象类

- GenericServlet：将Servlet接口中其他的方法做了默认空实现，只将service()方法作为抽象

- HttpServlet：对http协议的一种封装，简化操作
  1. 定义类继承HttpServlet
  2. 复写doGet/doPost方法

![](F:\笔记\JAVA\servlet\3.png)

## 8.Servlet一般写法

知道了Servlet的生命周期之后，我们一般只需要重写service方法，但是直接实现servlet接口有其他方法又不得不需要重写实现，所有我们一般选择继承GenericServlet或者HttpServlet。实际上，我们一般继承HttpServlet，第7节里说到，HttpServlet是对http协议的一种封装，简化操作，

HttpServlet会自动判断当前请求是get方式或者是post方式，然后调用doget或者dopost方法。原理很简单，在HttpServlet中的service方法内进行条件选择即可实现对于的do方法

然后，我们怎么知道那个请求是那种方式呢，一般像下面这么写：

```java
@WebServlet(name = "Servletdemo")
public class Servletdemo extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

每次写servlet这样写比较麻烦，可以在idea设置里加入此模板。

## 9.Servlet的线程安全

整个生命周期是单实例，但每次访问时为多线程。

- 单实例变成多实例，但过时了，因为耗费资源多，服务器压力大。
- 加线程锁，但数据会重复出现(没有同步机制)，且运行效率低。
- 解决线程安全问题的最佳办法：不要写全局变量，而写局部变量(即改变变量的作用域)。