## 一、会话技术概述

1. **会话**：一次会话中包含多次请求和响应。一次会话是指，浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止。
2. 功能：在一次会话的范围内的多次请求间，共享数据
3. 方式：
   1.  客户端会话技术：Cookie
   2. 服务器端会话技术：Session
4. 理解：http是无状态的协议，客户每次读取web页面时，服务器都打开新的会话，而且服务器也不会自动维护客户的上下文信息。如果说我们要实现购物车添加或删除商品的功能的话，我们就需要使用到Cookie和Session技术。也就是说，Cookie和Session是域对象。所谓域就相当于给存储的内容设置一个边界，将存储的内容存储到这片区域内。
5. 为什么要有session和cookie技术？之前说到过，HttpServletRequest与ServletContext对象都可以存储数据，HttpServletRequest是针对每次请求，ServletContext是针对整个web容器，如果我们有类似与将商品加入购物车这样的需求，HttpServletRequest与ServletContext就无法完成，HttpServletRequest是每次请求，但是加入购物车和付款是不能请求，会造成数据丢失；ServletContext是整个web容器，无法分辨那个商品是哪个用户添加的。所以就需要session和cookie技术

## 二、cookie

##### 1. 概念：

客户端会话技术，将数据保存到客户端

##### 2.cookie的基本原理

基于响应头set-cookie和请求头cookie实现

![](C:\Users\four and ten\Desktop\笔记\JAVA\会话技术\1.jpg)

1. 客户端在浏览器的地址栏中键入Web服务器的URL，浏览器发送读取网页的请求。 
2. 服务器接收到请求后，产生一个Set-Cookie报头，放在HTTP报文中一起回传客户端，发起一次会话。 
3. 客户端收到应答后，若要继续该次会话，则将Set-Cook-ie中的内容取出，形成一个Cookie.txt文件储存在客户端电脑上
4. 当客户端再次向服务器发出请求时，浏览器先在电脑里寻找对应该网站的Cookie.txt文件。如果找到，则根据此Cookie.txt产生Cookie报头，放在HTTP请求报文中发给服务器。
5. 服务器接收到包含Cookie报头的请求，检索其Cookie中与用户有关的信息，生成一个客户端所请示的页面应答传递给客户端。 浏览器的每一次网页请求，都可以传递已存在的Cookie文件，例如，浏览器的打开或刷新网页操作。

##### 3.cookie的实现

- 创建Cookie对象，绑定数据

  new Cookie(String name, String value) 

- 发送Cookie对象

   response.addCookie(Cookie cookie) 

- 获取Cookie，拿到数据]

   Cookie[]  request.getCookies()  

##### 4.Cookie的特点和作用

1. 特点
   1. cookie存储数据在客户端浏览器
   2. 浏览器对于单个cookie 的大小有限制 以及 对同一个域名下的总cookie数量也有限制

2. 作用
   1. cookie一般用于存储少量的不太敏感的数据
   2. 在不登录的情况下，完成服务器对客户端的身份识别

##### 5.注意事项

1. cookie可以一次发送多个，从获取cookie的方法的返回值是一个cookie数组可以看出

2. cookie默认在浏览器关闭之后（即有一方断开连接，会话结束），cookie数据被销毁，但是可以通过设置，来使cookie持久化存储。

   ```java
   /**
   * 设置cookie持久化储存
   * 参数：
   *      1. 正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效
   * 		2. 负数：默认值
   * 		3. 零：删除cookie信息
   */
   cookie.setMaxAge(60 * 60 * 30);
   ```
   
   注意：设置cookie数据保存时间一定要在发送cookie之前

3. cookie存储中文问题，在tomcat 8 之前 cookie中不能直接存储中文数据，要想存储中文

一般采用URL编码，然后使用URL解码，在tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析。

4. cookie共享问题。假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享？默认情况下cookie不能共享。但可以通过设置。

   ```
   setPath(String path)
   ```

   置cookie的获取范围。默认情况下，设置当前的虚拟目录，如果要共享，则可以将path设置为"/"

5. 不同的tomcat服务器间cookie共享问题

   ```
   setDomain(String path):如果设置一级域名相同，那么多个服务器之间cookie可以共享
   setDomain(".baidu.com"),那么tieba.baidu.com和news.baidu.com中cookie可以共享
   ```

   

## 三、Session

##### 1.概念

服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中。

##### 2.原理

 	session是依赖Cookie实现的。session是服务器端对象
  	当用户第一次使用session时（表示第一次请求服务器），服务器会创建session，并创建一个Cookie，在Cookie中保存了session的id，发送给客户端。这样客户端就有了自己session的id了。但这个Cookie只在浏览器内存中存在，也就是说，在关闭浏览器窗口后，Cookie就会丢失，也就丢失了sessionId。
	  当用户第二次访问服务器时，会在请求中把保存了sessionId的Cookie发送给服务器，服务器通过sessionId查找session对象，然后给使用。也就是说，只要浏览器容器不关闭，无论访问服务器多少次，使用的都是同一个session对象。这样也就可以让多个请求共享同一个session了

##### 3.使用

使用Servlet实现Session技术的是HttpSession对象

1. 获取HttpSession对象：
		HttpSession session = request.getSession();
2. 使用HttpSession对象：
    Object getAttribute(String name)  
    void setAttribute(String name, Object value)
    void removeAttribute(String name)  



