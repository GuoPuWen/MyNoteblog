来源：视频教程+博客

### 一、概述

在三层架构中：

- 表现层：也就是我们常说的web层。它负责接收客户端请求，向客户端响应结果，通常客户端使用http协议请求 ；
- 业务层：也就是我们常说的 service 层。它负责业务逻辑处理，和我们开发项目的需求息息相关。web 层依赖业务层，但是业务层不依赖 web 层；
- 持久层：也就是我们是常说的 dao 层。

而MVC是表现层的设计模型，MVC的全名是Model View Controller：

- Model（模型） ：表示应用程序核心（比如数据库记录列表）
- View（视图）显示数据（数据库记录）
- Controller（控制器）处理输入（写入数据库记录）

最简单的的设计模型是JSP （视图）+ servlet（控制器） +javabean（模型），而SpringMVC就是一种基于MVC设计模型（注意：这里要区分设计模型和设计模式），SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架。

在ssm（Spring+SpringMVC+Mybatis）中SpringMVC处于如图位置

![](SpringMVC\1.png)

**SpringMVC** **和** **Struts2** **的优略分析**

共同点： 

- 它们都是表现层框架，都是基于 MVC 模型编写的；

- 它们的底层都离不开原始 ServletAPI；

- 它们处理请求的机制都是一个核心控制器。 

区别： 

- Spring MVC 的入口是 Servlet,，而 Struts2 是 Filter  ；

- Spring MVC 是基于方法设计的，而 Struts2 是基于类，Struts2 每次执行都会创建一个动作类。所 以 Spring MVC 会稍微比 Struts2 快些；

- Spring MVC 使用更加简洁，同时还支持 JSR303, 处理 ajax 的请求更方便。

### 二、快速入门

入门案例：访问指定页面，跳转到success.jsp页面，并在控制台打印”hello springmvc'“

##### 1.创建maven的web工程

要创建maven的web项目工程，可以选择骨架，在选择骨架的时候有两个webapp，这里注意要选择apache下的maven中的webapp，可以看下图

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\2.png)

##### 2.添加Tomcat

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\3.png)

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\4.png)

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\5.png)

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\6.png)

![7](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\7.png)

##### 3.导入相关依赖

```xml
<properties>
    <!--版本锁定-->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```

##### 4.配置核心的控制器

配置核心的控制器是在web.xml中配置的，就相当于配置一个Servlet

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
<!--  SpringMVC的核心控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

里面有一个classpath:springmvc.xml是SpringMVC的核心配置文件，也就是说稍后需要创建springmvc.xml文件

##### 5.创建springmvc的配置文件

由于webapp骨架没有帮我们自动创建好java和resources目录，需要我们手动创建，在idea2019.3的版本中，选择创建文件夹，idea可以自动的帮助我们创建这两个文件夹，而不需要手动的去mark directory ac 

由于上面的web.xml文件中输入的springmvc.xml的文件位置为classpath，那么创建springmvc.xml文件必须要在resources的根目录下

springmvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.SpringMVCDemo"></context:component-scan>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

##### 6.编写控制器类

```java
/**
 * author by four and ten
 * create by 2020/4/7 14:54
 */
@Controller("controllerDemo1")
public class ControllerDemo1 {
    @RequestMapping("/hello")
    public String testDemo1(){
        System.out.println("hello SpringMVC");
        return "success";
    }
}
```

既然使用了Spring，前面也导入了Spring的jar包，那么就要将Controller类交给spring的ioc来管理，RequestMapping指定访问改servlet的路径，在springmvc.xml配置文件中，配置了一个视图解析器，将页面解析到/WEB-INF/pages/目录下的以.jsp结尾的文件中，在Controller中返回”success“，那么视图解析器会在/WEB-INF/pages/寻找success.jsp文件，如果找到那么跳转，如果没找到报404错误

##### 7.编写success.jsp页面

上面说过，当页面访问/hello时需要跳转到success.jsp页面，那么就需要在/WEB-INF/pages/目录下创建success.jsp文件

```html
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>入门成功！</h3>
</body>
</html>
```



![](SpringMVC\8.png)

##### 7.测试结果

启动Tomcat服务器，在浏览器上输入localhost:80/hello，跳转到ssuccess.jsp页面，并在控制台打印hello SpringMVC

![](SpringMVC\9.png)

![9](SpringMVC\10.png)

