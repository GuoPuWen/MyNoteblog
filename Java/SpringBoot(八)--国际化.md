### 一、环境准备

##### 1.概述

国际化是指对于各个世界不同国家的访问者来说，可以切换不同的语言，因为今天的软件系统不再是简单的单机程序，往往都是一个开放的系统，需要面对来自全世界各个地方的访问者，因此，国际化成为商业系统必不可少的一部分。



##### 2.环境

- jdk14
- idea2020
- SpringBoot2.2.6.RELEASE
- thymeleaf3.0.11.RELEASE



##### 3.依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```



##### 4.页面

要国家化的页面，index.html

```html
<body>
    用户名<input type="text" name="username"><br>
    密码<input type="text" name="password"><br>
    <input type="submit" value="登陆" th:value=""><br>
    <a href="">中文</a>
    <a href="">English</a>
</body>
```



### 二、实现步骤

##### 1.创建配置文件

在resources目录下，编写国际化配置文件，抽取页面需要显示的国际化消息

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\30.png)

idea会自动给我们补全目录 Resource bundle ’login‘。

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\29.png)



##### 2.编写配置文件

直接在idea提供的Resource Bundle界面编写国际化消息’

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\31.png)

