在Servlet一节里记录了Servlet的使用，生命周期以及原理，这一节介绍Servlet中的重要对象：Request、Response

- request和response对象是由服务器创建的。我们来使用它们
- request对象是来获取请求消息，response对象是来设置响应消息

## 一、HttpServletRequest

##### 1.获取请求消息数据

（1）获取请求行数据

请求行：请求方式 请求url 请求协议/版本



- 获取请求方式 ：GET/POST

  String getMethod() 

- (*)获取虚拟目录：就是在idea里的Application Context里面设置的

   String getContextPath()

```java
 protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String contextPath = request.getContextPath();
        System.out.println(contextPath);
 }
```

![](F:\笔记\JAVA\servlet\4.png)

![](F:\笔记\JAVA\servlet\5.png)

- 获取Servlet路径：资源名称

  String getServletPath()

- 获取get方式请求参数：name=zhangsan

  String getQueryString()

- (*)获取请求URI：
  -  String getRequestURI()
  - StringBuffer getRequestURL() 

- 获取协议及版本：HTTP/1.1

  String getProtocol()

-  获取客户机的IP地址

  String getRemoteAddr

（2）获取请求头数据

- 通过请求头的名称获取请求头的值

  (*)String getHeader(String name)

- 获取所有的请求头名称

  Enumeration<String> getHeaderNames()

（3）获取请求体数据

注意：只有POST请求方式，才有请求体，在请求体中封装了POST请求的请求参数- 

- 获取字符输入流

   BufferedReader getReader()

- 获取字节输入流

   ServletInputStream getInputStream()

##### 2.其他功能

- 获取请求参数通用方式：不论get还是post请求方式都可以使用下列方法来获取请求参数

  String getParameter(String name)

- 根据参数名称获取参数值的数组(用户多选框)

   String[] getParameterValues(String name)

- 获取所有请求的参数名称

  Enumeration<String> getParameterNames()

- 获取所有参数的map集合

   Map<String,String[]> getParameterMap()+

##### 3.中文乱码

在获取参数前，设置request的编码

request.setCharacterEncoding("utf-8");

##### 4.请求转发

一种在服务器内部的资源跳转方式

1. 步骤：

   1. 通过request对象获取请求转发器对象

   ​	RequestDispatcher getRequestDispatcher(String path)

   2. 使用RequestDispatcher对象来进行转发

      forward(ServletRequest request, ServletResponse response)

2. 特点：
   1. 浏览器地址栏路径不发生变化
   2. 只能转发到当前服务器内部资源中。
   3. 转发是一次请求

##### 5.共享数据

- 域对象：一个有作用范围的对象，可以在范围内共享数据

- request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据
- 方法：
  - void setAttribute(String name,Object obj):存储数据
  - Object getAttitude(String name):通过键获取值
  - void removeAttribute(String name):通过键移除键值对