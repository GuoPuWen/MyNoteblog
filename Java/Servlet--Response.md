## 一、Response对象

##### 1.功能

设置响应消息

##### 2.设置响应行

响应行格式：协议/版本 响应状态码 状态码描述

- 设置状态码：setStatus(int sc) 

##### 3.设置响应头

- setHeader(String name, String value) 

##### 4. 设置响应体

步骤：1.  获取输出流

- 字符输出流：PrintWriter getWriter()
- 字节输出流：ServletOutputStream getOutputStream()

2. 使用输出流，将数据输出到客户端浏览器

   注意：乱码问题：response.getWriter()获取的流的默认编码是ISO-8859-1，与页面的解码不一致就会导致乱码问题。解决办法：是在获取流之前设置编码

   response.setContentType("text/html;charset=utf-8");

## 二、重定向(redirect)

##### 1.redirect与forward区别

重定向与request请求转发类似，都是一种资源跳转的方式。

重定向的特点:redirect

> 1. 地址栏发生变化
> 2. 重定向可以访问其他站点(服务器)的资源
> 3. 重定向是两次请求。不能使用request对象来共享数据

request请求转发特点：forward

> 1. 转发地址栏路径不变
> 2. 转发只能访问当前服务器下的资源
> 3. 转发是一次请求，可以使用request对象来共享数据

##### 2.实现步骤

1. 方法1

```java
 //设置状态码
response.setStatus(302);
//设置响应头location
response.setHeader("location","/Request/Redirectdemo2" );
```

2. 方法2

   ```java
    response.sendRedirect("/Request/Redirectdemo2");
   ```

##### 3.路径写法

web项目里的路径写法规则：判断定义的路径是给谁用的？判断请求将来从哪儿发出

- 给客户端浏览器使用：需要加虚拟目录(项目的访问路径)， 建议虚拟目录动态获取：request.getContextPath()，

- 给服务器使用：不需要加虚拟目录

## 三、ServletContext对象

##### 1.概念

代表整个web应用，可以和程序的容器(服务器)来通信

##### 2. 获取

1. 通过request对象获取
   request.getServletContext();

2. 通过HttpServlet获取

   this.getServletContext();

##### 3. 功能

1. 获取MIME类型：

-  MIME类型:在互联网通信过程中定义的一种文件数据类型。格式：大类型/小类型   text/html		image/jpeg

- 获取：String getMimeType(String file)  



2. 域对象：共享数据

- setAttribute(String name,Object value)
- getAttribute(String name)
- removeAttribute(String name)

ServletContext对象范围：所有用户所有请求的数据



3. 获取文件的真实(服务器)路径
   - 方法：String getRealPath(String path)  

例如：下面文件

![](C:\Users\four and ten\Desktop\笔记\JAVA\servlet\8.png)



```java
 		ServletContext servletContext = this.getServletContext();
        String realPath = servletContext.getRealPath("/c.txt");
        System.out.println(realPath);

        String realPath1 = servletContext.getRealPath("/WEB-INF/d.txt");
        System.out.println(realPath1);

        String realPath2 = servletContext.getRealPath("/WEB-INF/classes/src.txt");
        System.out.println(realPath2);
```

输出：

![](C:\Users\four and ten\Desktop\笔记\JAVA\servlet\9.png)

src下的文件对应着Tomcat部署的web项目下的WEB-INF/classes下的文件

