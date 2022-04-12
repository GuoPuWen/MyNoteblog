# 一、前言

tomcat从JavaEE基础的学习一直伴随着我们，而Tomact到底是什么东西，能不能自己模拟一个tomcat呢？这个问题其实从开始接触tomcat的时候就已经在我心底留下了。我对于tomcat的这个软件的认识只在于使用阶段，只知道要将项目部署到tomcat上，用户才能访问。而本篇文章先从http协议开始，然后自己手动模拟一个tomcat，最后介绍tomcat的底层架构，读懂tomcat的源码

### 1.1 tomcat做了什么

我们知道软件架构有两种：

- C/S架构：客户端/服务器端
- B/S架构：浏览器/服务器端

而tomcat就是一种B/S服务器端的软件，能够接受用户的请求，处理请求，并且做出响应，常见的web服务器有以下几种：

- webLogic：oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
- webSphere：IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的
-  JBOSS：JBOSS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
- Tomcat：Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范
  servlet/jsp。开源的，免费的  

tomcat是一个Servlet容器，这句话恐怕都听说过，也就是说tomcat可以处理应用了Servlet规范的请求。

![image-20210112203943671](http://cdn.noteblogs.cn/image-20210112203943671.png)

我的理解是tomat服务器向用户屏蔽了如何封装Http请求，以至于用户只需要去关心具体的业务逻辑。

### 1.2 HTTP的工作原理

我们首先需要知道，当时我们在浏览器上输入流一个网址例如www.baidu.com时，发生了什么事情？

1. 域名解析，浏览器查找该域名的IP地址，也就是DNS解析
2. 浏览器根据ip地址，向web服务器发送一个Http请求
3. 服务器收到请求并处理
4. 服务器放回一个请求
5. 浏览器对该请求进行渲染

输入一个网址，大致的过程就是上面几个步骤，而其中的每一个步骤展开细讲都是一道面试题。下面来看看Http的工作原理

![image-20210112210032311](http://cdn.noteblogs.cn/image-20210112210032311.png)

1. 用户通过浏览器进行了一个操作，比如输入网址并回车，或者是点击链接，接着浏览
   器获取了这个事件。
2. 浏览器向服务端发出TCP连接请求。
3. 服务程序接受浏览器的连接请求，并经过TCP三次握手建立连接。
4. 浏览器将请求数据打包成一个HTTP协议格式的数据包。
5. 浏览器将该数据包推入 网络，数据包经过网络传输，最终达到端服务程序。
6.  服务端程序拿到这个数据包后，同样以HTTP协议格式解包，获取到客户端的意图。
7. 得知客户端意图后进行处理，比如提供静态文件或者调用服务端程序获得动态结果。
8.  服务器将响应结果（可能是HTML或者图片等）按照HTTP协议格式打包。
9. 服务器将响应数据包推入网络，数据包经过网络传输最终达到到浏览器。
10. 浏览器拿到数据包后，以HTTP协议的格式解包，然后解析数据，假设这里的数据是
    HTML。
11.  浏览器将HTML文件展示在页面上  

### 1.3 HTTP的请求报文格式

![img](http://cdn.noteblogs.cn/2012072810301161.png)

下面看一个具体的例子

```
GET /index.html HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
```

### 1.4 HTTP的响应报文格式

![img](http://cdn.noteblogs.cn/20150126110634828)

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 333
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>手写tomcat<h1>
</body>
</html>
```

# 二、手写简易的Tomcat

思路：使用socket监听8080端口，当有客户端连接时，放回字符串"hello tomcat"，这个好实现使用BIO就可以，看下面代码

```java
public class Bootstrap {
    /**
     * 监听端口号
     */
    private static final int port = 8080;

    public void start() throws IOException {
        ServerSocket socket = new ServerSocket(port);
        System.out.println("---------start port : " + port);
        while(true){
            Socket accept = socket.accept();
            OutputStream outputStream = accept.getOutputStream();
            outputStream.write("hello tomcat".getBytes());
            accept.close();
        }
    }

    public static void main(String[] args) throws IOException {
        new Bootstrap().start();
    }
}
```

然后执行这段代码，在浏览器上输入localhost:8080，毫不意外，是行不通的？



![image-20210113134730675](http://cdn.noteblogs.cn/image-20210113134730675.png)

仔细想想其中的道理，其实前言部分已经有了答案，就是服务器这边直接放回的是字符串"hello tomcat"，但是浏览器能解析吗？我们都知道当输入localhost:8080，用的应用层协议是HTTP，所以对应的返回数据也要是HTTP报文，那么可以有一个工具类来实现报文的封装

```java
public class HttpUtil {
    /**
     * 响应体报文格式：状态行、首部行、和实体体
     *
     *
     * @return
     */
    public static String addHeader(int len){
        StringBuffer sb = new StringBuffer("HTTP/1.1 200 OK\n");
        sb.append("Content-Type: text/html; charset=UTF-8 \n");
        sb.append("Content-Length: " + len + " \n" + "\r\n");   //http响应体中用\r\n换行
        return sb.toString();
    }

    /**
     * 响应200
     * @param content
     * @return
     */
    public static String res_200(String content){
        return addHeader(content.length()) + content;
    }
}
```



然后主类中使用HttpUtil工具类

```java
public class Bootstrap {
    /**
     * 监听端口号
     */
    private static final int port = 8080;

    public void start() throws IOException {
        ServerSocket socket = new ServerSocket(port);
        System.out.println("---------start port : " + port);
        while(true){
            Socket accept = socket.accept();
            OutputStream outputStream = accept.getOutputStream();
            outputStream.write(HttpUtil.res_200("hello tomcat").getBytes());	//注意这里
            accept.close();
        }
    }

    public static void main(String[] args) throws IOException {
        new Bootstrap().start();
    }
}
```

然后执行，浏览器访问

![image-20210113135254058](http://cdn.noteblogs.cn/image-20210113135254058.png)

那么到现在已经完成了简单的响应字符串的功能，这个响应JSON字符串在SpringMVC中可以使用@ResponseBody

接下来进行第二步，响应html页面。那么思路是很明确的，当一个请求过来时，获取到它的请求路径，然后找到对应资源，将这个资源送到输出流里面，当然也是HTTP格式的报文

关键是如何获取响应路径，具体看下面代码先构建一个Request封装类，将HTTP请求报文封装在Request中

```java
package cn.noteblogs;

import java.io.IOException;
import java.io.InputStream;

public class Request {
    /**
     * 请求方法
     */
    private String method;


    /**
     * 请求url
     */
    private String url;
    /**
     * 输入流
     */
    private InputStream inputStream;
    
    public Request(){}

    public Request(InputStream inputStream) throws IOException {
        this.inputStream = inputStream;
        int count = 0;
        while(count == 0){
            count = inputStream.available();
        }
        byte[] b = new byte[count];
        inputStream.read(b);
        String reqStr = new String(b);
        System.out.println("请求体：" + reqStr);
        String[] split = reqStr.split("\n");
        //System.out.println(split[0]);   //第一行信息 GET /index.html HTTP/1.1  GET /favicon.ico HTTP/1.1
        String reqInfo[] = split[0].split(" ");
        System.out.println("请求方法：" + reqInfo[0]);
        System.out.println("请求路径：" + reqInfo[1]);
        this.method = reqInfo[0];
        this.url = reqInfo[1];
    }
    //get和set方法
}

```



再次封装一个Response封装类，将需要响应的逻辑放在里面

```java
public class Response {
    private OutputStream outputStream;

    public Response(OutputStream outputStream){
        this.outputStream = outputStream;
    }

    public void outPutStr(String conent) throws IOException {
        outputStream.write(HttpUtil.res_200(conent).getBytes());
        outputStream.flush();
        outputStream.close();
    }
    public void outPutHtml(String url) throws IOException {
        if(url.equals("/favicon.ico")){
            return;
        }
        outputStream.write(HttpUtil.res_200(Resourceutil.readFile(url)).getBytes());
        outputStream.flush();
        outputStream.close();
    }
    
}
```

获取资源的String方法

```java
public class Response {
    private OutputStream outputStream;

    public Response(OutputStream outputStream){
        this.outputStream = outputStream;
    }

    public void outPutStr(String conent) throws IOException {
        outputStream.write(HttpUtil.res_200(conent).getBytes());
        outputStream.flush();
        outputStream.close();
    }
    public void outPutHtml(String url) throws IOException {
        if(url.equals("/favicon.ico")){
            return;
        }
        outputStream.write(HttpUtil.res_200(Resourceutil.readFile(url)).getBytes());
        outputStream.flush();
        outputStream.close();
    }
    
}
```

画一个示意图来解释上面3个类的作用和位置



![image-20210113141708724](http://cdn.noteblogs.cn/image-20210113141708724.png)



接下来就是处理动态请求了，我们知道tomcat是实现了Servlet规范的，所谓规范就是指http请求在接收到请求之后将请求交给Servlet容器来处理，Servlet容器通过Servlet接口来调用不同的业务类，这一整套称作Servlet规范

Servlet规范接口

```java
/**
 * @author 四五又十
 * @create 2021/1/12 15:41
 */
public interface Servlet {	
    void init() throws Exception;	//这里需要先把异常处理好
    void destory() throws Exception;
    void service(Request request, Response response) throws Exception;
}
```

实现了Servlet规范的HttpServlet抽象类

```java
public abstract class HttpServlet implements Servlet{
    public abstract void doGet(Request request, Response response) throws Exception;
    public abstract void doPost(Request request, Response response) throws Exception;

    @Override
    public void service(Request request, Response response) throws Exception {
        if(request.getMethod().equalsIgnoreCase("GET")){
            doGet(request,response);
        }else{
            doPost(request,response);
        }
    }
}
```

具体的Servlet

```java
public class MyServlet extends HttpServlet {
    @Override
    public void doGet(Request request, Response response) throws IOException, InterruptedException {
        String content = "GET业务请求";
        response.outPutStr(content);
        //try { TimeUnit.SECONDS.sleep(10);} catch (InterruptedException e) {e.printStackTrace();}
    }

    @Override
    public void doPost(Request request, Response response) throws IOException {
        String content = "<h2> POST业务请求 </h2>";
        response.outPutStr(content);
    }

    @Override
    public void init() throws Exception {

    }

    @Override
    public void destory() throws Exception {

    }
}
```

然后接下来，模仿web.xml的格式将每一个路径映射到一个具体的Servlet类中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app>

    <servlet>
        <servlet-name>test</servlet-name>
        <servlet-class>cn.noteblogs.MyServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>test</servlet-name>
        <url-pattern>/test</url-pattern>
    </servlet-mapping>

</web-app>
```

所以需要一个工具类来解析xml，最终的结果是一个Hashmap<请求路径，Servlet实现类>

```java
public class Xmlutil {
    public static HashMap loadServlet(){
        InputStream inputStream = Xmlutil.class.getClassLoader().getResourceAsStream("web.xml");
        SAXReader saxReader = new SAXReader();
        HashMap<String, HttpServlet> urlPatternMap = new HashMap<String, HttpServlet>();
        try {
            Document document = saxReader.read(inputStream);
            Element rootElement = document.getRootElement();
            List<Element> servletNode = rootElement.selectNodes("//servlet");
            for (Element element : servletNode) {
                String servletName = ((Element)element.selectSingleNode("servlet-name")).getStringValue();
                String servletClass = ((Element)element.selectSingleNode("servlet-class")).getStringValue();
                Element servletMapping = (Element)rootElement.selectSingleNode("/web-app/servlet-mapping[servlet-name='" + servletName + "']");
                String urlPattern = servletMapping.selectSingleNode("url-pattern").getStringValue();
                urlPatternMap.put(urlPattern, (HttpServlet)Class.forName(servletClass).newInstance());
            }

        } catch (DocumentException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return urlPatternMap;
    }

}

```

那么，最终的Socket代码如下：

```java
public class Bootstrap {
    /**
     * 监听端口号
     */
    private static final int port = 8080;
    private static HashMap<String, HttpServlet> urlPatternMap;
    private static final ExecutorService executor = Executors.newCachedThreadPool();
    static {
        urlPatternMap = Xmlutil.loadServlet();
    }


    public void start() throws IOException {
        ServerSocket socket = new ServerSocket(port);
        System.out.println("---------start port : " + port);
        while(true){
            Socket accept = socket.accept();
            Request request = new Request(accept.getInputStream());
            Response response = new Response(accept.getOutputStream());
            if(request.getUrl().contains(".html")){
                response.outPutHtml(request.getUrl());

            }else {
                if(!urlPatternMap.containsKey(request.getUrl())){
                    response.outPutStr(request.getUrl() + "is not found ...");
                }else{
                    HttpServlet httpServlet = urlPatternMap.get(request.getUrl());
                    try {
                        httpServlet.service(request, response);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
            accept.close();
        }
    }

    public static void main(String[] args) throws IOException {
        new Bootstrap().start();
    }
}
```

那么一个简单版的tomcat就制作好了，可以处理动态请求和静态请求，项目结构为

![image-20210113195126118](http://cdn.noteblogs.cn/image-20210113195126118.png)

参考：[手写一个简易版的tomcat](https://juejin.cn/post/6905770007142088711)

[项目地址](https://github.com/GuoPuWen/SimpleTomcat)

