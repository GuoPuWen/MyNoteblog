在Servlet一节里记录了Servlet的使用，生命周期以及原理，这一节介绍Servlet中的重要对象：Request、Response

- request和response对象是由服务器创建的。我们来使用它们
- request对象是来获取请求消息，response对象是来设置响应消息

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

![](servlet\4.png)

![](servlet\5.png)

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

  **(*)String getHeader(String name)**

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

  **String getParameter(String name)**

- 根据参数名称获取参数值的数组(用户多选框)

   String[] getParameterValues(String name)

- 获取所有请求的参数名称

  Enumeration<String> getParameterNames()

- 获取所有参数的map集合

   **Map<String,String[]> getParameterMap()**

##### 3.中文乱码

在获取参数前，设置request的编码

request.setCharacterEncoding("utf-8");

##### 4.请求转发(forward)

**一种在服务器内部的资源跳转方式**

1. **步骤：**

   1. **通过request对象获取请求转发器对象**

   ​	**RequestDispatcher getRequestDispatcher(String path)**

   2. **使用RequestDispatcher对象来进行转发**

      **forward(ServletRequest request, ServletResponse response)**

2. **特点：**
   
   > 1. **浏览器地址栏路径不发生变化
   > 2. **只能转发到当前服务器内部资源中。**
   > 3. 转发是一次请求

##### 5.共享数据

- 域对象：一个有作用范围的对象，可以在范围内共享数据

- request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据
- 方法：
  - void setAttribute(String name,Object obj):存储数据
  - Object getAttitude(String name):通过键获取值
  - void removeAttribute(String name):通过键移除键值对

##### 6.小案例

登陆案例：在页面上输入用户名，密码。通过操作数据库判断是否登陆成功，若成功，跳转到成功登陆页面(要求该页面展示用户名)；若失败，跳转失败页面。

![](C:\Users\four and ten\Desktop\笔记\JAVA\servlet\6.png)

cn.Request.domain.User

```java
package cn.Request.domain;

public class User {
    private String username;
    private String password;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }
    public User(){

    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    
    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object obj) {
        User _user = (User)obj;
        if(_user.getPassword() == this.password && _user.getUsername() == this.username){
            return true;
        }else{
            return false;
        }
    }
}
```

cn.Request.dao.Userdao 操作数据库

```java
package cn.Request.dao;

import cn.Request.Util.Util;
import cn.Request.domain.User;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class UserDao {

    public User login(User loginuser) throws SQLException {
        DataSource ds = Util.get_DataSource();
        Connection conn = ds.getConnection();
        String sql = "select * from user where username = ? and password = ? ";
        PreparedStatement pst = conn.prepareStatement(sql);
        pst.setString(1,loginuser.getUsername());
        pst.setString(2,loginuser.getPassword());
        ResultSet res = pst.executeQuery();

        if(res.next()){
            return loginuser;
        }
        return null;


    }

}
```

cn.Request.Util  数据库连接池工具类

```java
package cn.Request.Util;

import com.alibaba.druid.pool.DruidConnectionHolder;

import com.alibaba.druid.pool.DruidDataSourceFactory;
import org.apache.commons.beanutils.PropertyUtilsBean;
import org.junit.Test;

import javax.sql.DataSource;
import javax.sql.StatementEvent;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

public class Util {

    private  static DataSource ds;
    static {
        Properties pro = new Properties();
        InputStream rs = Util.class.getClassLoader().getResourceAsStream("druid.properties");
        try {
            pro.load(rs);
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static DataSource get_DataSource(){
        return ds;
    }
}
```

cn.Reques.Servlet.ServletLogin

```java
package cn.Request.Servlet;

import cn.Request.dao.UserDao;
import cn.Request.domain.User;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.sql.SQLException;

/**
 * author by four and ten
 * create by 2020/3/1 17:31
 */
@WebServlet("/ServletLogin")
public class ServletLogin extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        System.out.println(username);
        System.out.println(password);
        User loginuser = new User(username,password);
        try {
            User login = new UserDao().login(loginuser);
            if(login == null){
                request.getRequestDispatcher("/FailingLogin").forward(request,response);
            }else {
                request.setAttribute("user", loginuser);
                request.getRequestDispatcher("/SuccessLogin").forward(request,response);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```



```java
package cn.Request.Servlet;

import cn.Request.domain.User;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * author by four and ten
 * create by 2020/3/1 19:11
 */
@WebServlet("/SuccessLogin")
public class SuccessLogin extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        User user = (User)request.getAttribute("user");
        String username = user.getUsername();
        response.getWriter().write("欢迎您回来" + username);


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```



```java
package cn.Request.Servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * author by four and ten
 * create by 2020/3/1 19:11
 */
@WebServlet("/FailingLogin")
public class FailingLogin extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        response.getWriter().write("登陆失败");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

