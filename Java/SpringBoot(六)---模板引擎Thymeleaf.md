### 一、模板引擎简介

- 模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的[HTML](https://baike.baidu.com/item/HTML/97049)文档。(百度百科)

- 最常用的模板引擎有JSP、Velocity、Freemarker、Thymeleaf。

- Spring Boot推荐使用Thymeleaf模板引擎

### 二、快速使用thymeleaf

环境：

jdk14，idea2020，Spring Boot2.2.6，thymeleaf3.0.11

##### 1.创建Sping Boot项目

直接使用Spring Boot的向导为我们创建Spring Boot项目，在创建时勾选thymeleaf，web

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\23.png)

创建完成之后，自动给我们导入了以下坐标：

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

查看thymeleaf版本：

```xml
<thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
```

##### 2.使用thymeleaf

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
```

ThymeleafProperties是thymeleaf自动配置与配置文件属性想绑定的类，也就是如果使用默认配置，只要在classpath:/templates/并且是以.html为后缀的页面，thymeleaf会自动渲染

##### 3.编写Controller

```java
    @RequestMapping("/hello")
    public ModelAndView hello() {
        ModelAndView mv = new ModelAndView();
        mv.addObject("welcome","Hello Thymeleaf");
        mv.setViewName("hello");
        return mv;
    }
```

##### 4.在/templates/编写hello.html

这里要使用thymeleaf，那么就需要导入thymeleaf的名称空间：

```xml
<html xmlns:th="http://www.thymeleaf.org">
```

hello.html：

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h5 th:text="${welcome}">欢迎，Spring Boot</h5>
</body>
</html>
```

##### 5.运行结果

启动Spring Boot，在浏览器输入localhost:8080/hello

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\24.png)

##### 6.结论

==Thymeleaf 最为显著的特征是增强属性，任何属性都可以通过th:xx 来完成交互，例如th:value最终会覆盖value属性==。在上面中如果是直接访问hello.html静态页面，得到的是欢迎，Spring Boot，如果使用了Thymeleaf那么th:text将会覆盖原先的值

### 三、基本语法

[官方文档下载地址](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.pdf)

##### 1.th:xx

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\26.png)

- 片段包含，类似于jsp中的：jsp:include
  - th:insert
  - th:replace
- 遍历，类似于jsp中的：c:forEach
  - th:each
- 条件判断，相当于：c:if
  - th:if 
  - th:unless 
  - th:switch
  - th:case 
- 属性修改
  - th:attr
  - th:attrprpend
- 声明变量c:set
  - th:object 
  - th:with
- 修改指定属性的默认值
  - th:value
  - th:href
  - th:src
  - ......

- 修改标签体的内容
  - th:text 转义字符
  - th:utext 不转义字符

- 声名片段
  - th:fragment
  - th:remove

##### 2.循环迭代th:each

```html
<tr th:each="num,index : ${numList} ">
    <td th:text="${num}"></td>
    <td th:text="${index}"></td>
</tr>
```

th:each用于循环迭代数据，这些数据可以是数组，List，Set，Map等等，格式如上。thymeleaf还可以获取迭代状态，在th:each属性中定义一些状态变量

- index 属性：当前迭代索引，从0开始

- count 属性：当前的迭代计数，从1开始

- size 属性：迭代变量中元素的总量

- urrent 属性：每次迭代的 iter 变量，即当前遍历到的元素

- even/odd 布尔属性：当前的迭代是偶数还是奇数。

- first 布尔属性：当前的迭代是否是第⼀个迭代

- last 布尔属性：当前的迭代是否是最后⼀个迭代。
  

##### 3.条件判断th:if

很多时候只有在满⾜某个条件时，才将⼀个模板⽚段显示在结果中，否则不进行显示

- 如果表达式结果为布尔值，则为 true 或者 false

- 如果表达式的值为 null，th:if 将判定此表达式为 false

- 如果值是数字，为 0 时，判断为 false；不为零时，判定为 true

- 如果 value 是 String，值为 “false”、“off”、“no” 时，判定为 false，否则判断为 true，字符串为空时，也判断为 true

- 如果值不是布尔值，数字，字符或字符串的其它对象，只要不为 null，则判断为 true
  

##### 4.th:unless

​     th:unless 是 th:if 的反向属性，它们判断的规则一致，只是 if 当结果为true 时进行显示，unless 当结果为 false进行显示。

##### 5.th:switch 

 th:switch / th:case 与 Java 中的 switch 语句等效，有条件地显示匹配的内容。

##### 6.使用内置的对象

\#ctx : the context object. 

\#vars: the context variables. 

\#locale : the context locale. 

\#request : (only in Web Contexts) the HttpServletRequest object. 

\#response : (only in Web Contexts) the HttpServletResponse object. 

\#session : (only in Web Contexts) the HttpSession object. 

\#servletContext : (only in Web Contexts) the ServletContext object. 

更多的使用还是查看官方文档

