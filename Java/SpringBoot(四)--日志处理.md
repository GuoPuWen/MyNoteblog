### 一、概述

有一个日志对于程序来说是非常重要的，有句话说的好：”只有在程序出问题以后才会知道打一个好的日志有多么重要“。

目前java市场常用的日志框架有JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j.... 日志有日志门面和日志实现之分

- 日志门面：日志的抽象层，有JCL（Jakarta Commons Logging）、slf4j、jboss-logging
- 日志实现：log4j、jul、log4j2、logback

常见的日志介绍：

- **Commons Logging** Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging
- **Slf4j** 类似于Commons Logging，是一套简易Java日志门面，本身并无日志的实现。（Simple Logging Facade for Java，缩写Slf4j）。

- **Log4j** Apache Log4j是一个基于Java的日志记录工具。它是由Ceki Gülcü首创的，现在则是Apache软件基金会的一个项目。 Log4j是几种Java日志框架之一。
- **Log4j 2** Apache Log4j 2是apache开发的一款Log4j的升级产品。
- **Logback** 一套日志组件的实现(Slf4j阵营)。
- **Jul** (Java Util Logging)，自Java1.4以来的官方日志实现，java自带的

==而Spring Boot选用的日志框架组合是slf4j和logback==

### 二、使用slf4j

##### 1.快速使用

具体可以查看官方文档：[slf4j快速使用](http://www.slf4j.org/manual.html)

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\17.png)

这是官方文档里的一张图：

![下·](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\16.png)

根据这张图可以看到使用什么日志实现框架需要导入的jar包，那么要使用slf4j和logback就需要导入slf4j-api.jar和logback-classic.jar、logback-core.jar。

可以查看Spring Boot的pom文件依赖树，看到确实导入了这些jar包：

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\18.png)



##### 2.输出格式

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\20.png)





### 三、slf4j整合其他日志框架

当需要统一日志记录时，slf4j还提供了一种解决办法来将不同的日志框架统一的使用slf4j

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\19.png)

例如要将Jul日志框架统一为slf4j，那么需要有以下3步：

- 将系统中的jul框架jar包排除出去
- 用中间包jul-toslf4j.jar来替换原有的日志狂啊及
- 导入slf4jjar包