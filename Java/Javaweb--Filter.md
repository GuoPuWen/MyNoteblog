来源：视频资料，网上博客

### 一、Filter概述

##### 1.概念

Filter称之为过滤器，是用来做一些拦截的任务。客户端每次向服务器发送资源请求时，都会被Filter拦截，只有当Filter同意请求时，才可以访问该资源。

![](C:\Users\four and ten\Desktop\笔记\JAVA\Filter\3.png)

- 客户端发送http请求，进入到Filter拦截器中，进行相关业务逻辑
- 判定通行，进入到Servlet，Servlet执行完毕，又返回Filter，进行一些增强，最后响应回到服务器
- 判断不通行，访问超时

##### 2.一些使用例子

登录验证、统一编码处理、敏感字符过滤等等

### 二、快速入门

1. 编写java类实现Filter接口
2. 复写其方法
3. 配置拦截路径(使用注解或者是在web.xml中配置)

在web项目中，直接右键创建一个Filter，然后复写里面的doFilter方法，这里使用注解的方式

```java
//配置拦截路径
@WebFilter("/*")
public class Filter implements javax.servlet.Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        System.out.println("访问前使用了Filter");
        //放行
        chain.doFilter(req, resp);
    }

    public void init(FilterConfig config) throws ServletException {

    }

}
```

在web.xml中配置

```xml
<filter>
    <filter-name>demo1</filter-name>
    <!-- 拦截器全限定类名 -->
    <filter-class>com.Filter.Filter</filter-class>
</filter>
<filter-mapping>
<filter-name>demo1</filter-name>
    <!-- 拦截路径 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这样就配置好了一个Filter过滤器

### 三、Filter的生命周期

我们发现，在要实现Filter类的3个方法，分别是init、doFilter、destroy这和Servlet很相似，这3个方法代表着Filter的生命周期

- init：在服务器启动时，会创建Filter对象，这时会调用init方法，这个方法只执行一次，主要是用于加载资源
- doFilter：每一次请求被拦截资源时，会执行。执行多次
- destroy：在服务器关闭后，Filter对象被销毁。如果服务器是正常关闭，则会执行destroy方法。只执行一次。用于释放资源

### 四、过滤器执行过程分析

​	当配置好过滤器，并且配置好需要拦截的资源目录后(注意：这里为了简便，说的都是只有一个拦截器的情况)，如果是服务器第一次启动，那么调用Filter的init方法，当客户端访问服务器的资源时，首先会执行doFilter方法，这个doFilter方法会传递一个FilterChain对象，这个对象也有一个doFilter方法，用于判定是否通行。在调用这个方法之前的代码都是拦截前执行的代码，这个方法之后的代码在响应之后会执行，

​	也就是说chain.doFilter(req, resp);这行代码就是个分界线，前面的代码是拦截，后面的代码是做一些响应时的增强

### 五、过滤器配置

##### 1.拦截路径写法

- /index.jsp  只有访问index.jsp资源时，过滤器才会被执行，也就是拦截具体的资源路径
-  /user/*    拦截目录访问/user下的所有资源时，过滤器都会被执行
- *.jsp         访问所有后缀名为jsp资源时，过滤器都会被执行，后缀名拦截
- /*              拦截所有资源

##### 2.拦截类型配置

(1)使用注解的方式

使用注解的方式配置的是dispatcherTypes值

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebFilter {
    String description() default "";

    String displayName() default "";

    WebInitParam[] initParams() default {};

    String filterName() default "";

    String smallIcon() default "";

    String largeIcon() default "";

    String[] servletNames() default {};

    String[] value() default {};

    String[] urlPatterns() default {};

    DispatcherType[] dispatcherTypes() default {DispatcherType.REQUEST};

    boolean asyncSupported() default false;
}

public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR;

    private DispatcherType() {
    }
}

```

可以看到dispatcherTypes的默认值是REQUEST，值得是浏览器直接请求资源。DispatcherType是一个枚举类型，其值为：

- FORWARD：转发访问资源
- INCLUDE：包含访问资源
- ERROR：错误跳转资源
- ASYNC：异步访问资源
- REQUEST：默认值。浏览器直接请求资源

(2)web.xml配置
 设置dispatcher标签即可

### 六、配置多个过滤器

filter是支持配置多个过滤器的，只要该类i实现了过滤器的接口 javax.servlet.Filter就会被认为是一个过滤器。那么这里涉及到一个过滤器链执行的先后顺序问题，哪一个过滤器先执行，哪一个过滤器后执行。

当然如果过滤器的执行顺序确定了，那么请求和响应时过滤器执行的顺序应该时刚好相反的，例如：

```
过滤器1
过滤器2
资源执行
过滤器2
过滤器1 
```

对于不同的配置，有不同的判定先后顺序的方法：

(1) 注解：按照类名的字符串比较规则比较，值小的先执行

(2) xml配置：filter-mapping谁定义在上边，谁先执行

(3)即有注解，又有xml配置：那么会**优先执行web.xml中配置的Filter**