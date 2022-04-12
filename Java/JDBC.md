# 一、概念

- Java DataBase Connectivity  Java数据库连接  Java语言操作数据库
- 本质：

# 二、JDBC连接步骤

1. 导入jar包
   1. 复制jar包
   2. ADD As Library

2. 注册驱动
3. 获取数据库连接对象
4. 定义sql语句
5. 获取执行sql语句的对象 Statement
6. 执行sql，接受返回结果
7. 处理结果
8. 释放资源

# 三、各个对象

## 1.DriverManager：驱动管理对象

- 功能

  1. 注册驱动：告诉程序该使用哪一个数据库驱动jar包

     ```
     static void registerDriver(Driver driver, DriverAction da) 
     注册与给定的驱动程序 DriverManager 
     ```

     但实际代码：Class.forName("com.mysql.jdbc.Driver");

     查看源代码后发现在com.mysql.jdbc.Driver中存在静态代码块

     ```java
         static {
             try {
                 DriverManager.registerDriver(new Driver());
             } catch (SQLException var1) {
                 throw new RuntimeException("Can't register driver!");
             }
         }
     ```

     注意：mysql5之后的驱动jar包可以省略注册驱动的步骤

  2. 获取数据库连接

     - 方法：

       ```java
       static Connection getConnection(String url, String user, String password) 
       ```

       

     - 参数
       1. url：指定连接的路径
          - 语法：jdbc：mysql://ip地址(域名):端口号/数据库名称
          - 注意：如果是本机mysql服务器，并且mysql端口号默认是3306，url可以写成：
            jdbc:mysql:///数据库名称
       2. user:用户名
       3. password:密码

## 2.Connection:数据库连接对象

1. 获取执行sql的对象
2. 管理事务

## 3.Statement 执行静态sql的对象

1. 执行sql
   1. boolean execute(String sql) ：可以执行任意sql语句
   
   2. int executeUpdate(String sql)：
   
      **可以执行DML(insert、update、delete)语句、DDL(create、drop、alter)**
      返回值：影响的行数
   
   3. ResultSet executeQuery(String sql)执行DQLselect语句
      返回一个结果集
      
      

## 4.ResultSet结果集对象

1. boolean next() 

   将光标从当前位置向前移动一行。`ResultSet`光标最初位于第一行之前;第一次调用方法`next`使第一行成为当前行;第二个调用使第二行成为当前行，依此类推。当调用`next`方法返回`false`时，光标位于最后一行之后。 

   

2. xxx getxxx(参数)方法

   - xxx代表数据类型，如 int getInt()
   - 参数有2种
     1. int ：代表列的编号
     2. String：代表列名称

## 5.preparedStatment：执行预编译sql对象

1. PreparedStatement：执行sql的对象
   1. SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题
   		1. 输入用户随便，输入密码：a' or 'a' = 'a
   		2. sql：select * from user where username = 'fhdsjkf' and password = 'a' or 'a' = 'a' 

   	2. 解决sql注入问题：使用PreparedStatement对象来解决
   	3. 预编译的SQL：参数使用?作为占位符
   	4. 步骤：
   		1. 导入驱动jar包 mysql-connector-java-5.1.37-bin.jar
   		2. 注册驱动
   		3. 获取数据库连接对象 Connection
   		4. 定义sql
   			* 注意：sql的参数使用？作为占位符。 如：select * from user where username = ? and password = ?;
   		5. 获取执行sql语句的对象 PreparedStatement  Connection.prepareStatement(String sql) 
   		6. 给？赋值：
   			* 方法： setXxx(参数1,参数2)
   				* 参数1：？的位置编号 从1 开始
   				* 参数2：？的值
   		7. 执行sql，接受返回结果，不需要传递sql语句
   		8. 处理结果
   		9. 释放资源

   	5. 注意：后期都会使用PreparedStatement来完成增删改查的所有操作
   		1. 可以防止SQL注入
   		2. 效率更高

# 四、JDBC工具类

对于JDBC的8个步骤来说，可以创建一个工具类，将一些常用的步骤封装起来。

1. 注册驱动 获取数据库连接对象

   这个步骤使用静态代码块来实现，每次调用JDBC工具类时都会先编译这部分代码。通过配置文件来实现通用性

   ```java
   static{
       //1.使用ProPerties类
       Properties pro = new Properties();
   
   
       //2.使用load方法加载配置文件
       try {
           //获取src路径下的文件的方式
           ClassLoader classLoader = JDBCUtil.class.getClassLoader();
           URL res_url = classLoader.getResource("pro.properties");
           String path = res_url.getPath();
   
           pro.load(new FileReader(path));
   
           url = pro.getProperty("url");
           password = pro.getProperty("password");
           user = pro.getProperty("user");
           driver = pro.getProperty("driver");
           Class.forName(driver);
       } catch (IOException | ClassNotFoundException e) {
           e.printStackTrace();
   	}
       public static Connection getConnection(){
           try {
               return DriverManager.getConnection(url, user, password);
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return null;
       }
   ```

   pro.properties

   ```java
   url=jdbc:mysql://localhost:3306/db2
   user=root
   password=213
   driver=com.mysql.jdbc.Driver
   ```

   

2. 释放资源

   使用一个函数重载传入参数

   全部代码：

   ```java
   package com.JDBCUtil;
   
   import com.mysql.jdbc.ConnectionGroup;
   
   import java.io.FileInputStream;
   import java.io.FileReader;
   import java.io.IOException;
   import java.net.URL;
   import java.sql.*;
   import java.util.Properties;
   
   /**
    * @author四五又十
    * @create 2020/1/25 16:03
    */
   public class JDBCUtil {
       private static String url;
       private static String password;
       private static String user;
       private static String driver;
       /**
        * 加载配置文件
        */
       static{
           //1.使用ProPerties类
           Properties pro = new Properties();
   
   
           //2.使用load方法加载配置文件
           try {
               //获取src路径下的文件的方式
               ClassLoader classLoader = JDBCUtil.class.getClassLoader();
               URL res_url = classLoader.getResource("pro.properties");
               String path = res_url.getPath();
   
               pro.load(new FileReader(path));
   
               url = pro.getProperty("url");
               password = pro.getProperty("password");
               user = pro.getProperty("user");
               driver = pro.getProperty("driver");
               Class.forName(driver);
           } catch (IOException | ClassNotFoundException e) {
               e.printStackTrace();
           }
   
   
   
       }
   
       /**
        * 获取数据库连接
        * @return
        */
       public static Connection getConnection(){
           try {
               return DriverManager.getConnection(url, user, password);
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return null;
       }
   
       /**
        * 释放资源
        * @param coon
        * @param stat
        */
       public static void closeAllResource(Connection coon, Statement stat){
           if(stat != null){
               try {
                   stat.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
           if(coon != null) {
               try {
                   coon.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
       }
   
       public static void closeAllResource(Connection coon, Statement stat, ResultSet rst){
           if(rst != null) {
               try {
                   rst.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
           if(stat != null){
               try {
                   stat.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
           if(coon != null) {
               try {
                   coon.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
   
       }
   
   
   }
   ```



# 五、举例

1.修改db5下的account中的name = A 的balance = 1

```java
package com.jdbcdemo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * @author四五又十
 * @create 2020/1/21 20:15
 */
public class demo3 {
    private final static String url = "jdbc:mysql://localhost:3306/db5";
    private final static String user = "root";
    private final static String password = "213";

    public static void main(String[] args) {
        Connection conn = null;
        Statement stat = null;
        //1.注册驱动
        try {
            Class.forName("com.mysql.jdbc.Driver");
            //2.连接数据库
            conn = DriverManager.getConnection(url, user, password);
            //3.创建执行sql对象 Statement
            stat = conn.createStatement();
            //4.定义sql语句
            String sql = "update account set balance = 1 where id = 3";
            int count = stat.executeUpdate(sql);
            System.out.println(count);
            if( count >= 0){
                System.out.println("添加成功!");
            }else{
                System.out.println("添加失败");
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }finally {
            if(stat != null){
                try {
                    stat.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        if(conn != null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }


    }
}

```



2. 根据用户输入的用户名和密码，判断是否成功登录

   

```java
package com.jdbcdemo;

import com.JDBCUtil.JDBCUtil;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Scanner;

/**
 * 测试：根据用户输入的用户名和密码，判断是否成功登录
 * @author四五又十
 * @create 2020/1/25 19:04
 */
public class demo6 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入用户名");
        String username = sc.nextLine();
        System.out.println("请输入密码");
        String password = sc.nextLine();
        if(new demo6().login(username, password)){
            System.out.println("登陆成功");
        }else{
            System.out.println("用户名或密码错误");
        }
    }

    public boolean login(String username, String password){
        if(username == null || password == null) {
            return false;
        }else {
            //连接数据库
            Connection conn = JDBCUtil.getConnection();
            //定义sql
            String sql = "select * from user where username = '"+username+"' and password = '"+password+"'";
            //获取执行sql对象
            Statement stat = null;
            ResultSet res = null;
            try {
                stat = conn.createStatement();
                //执行sql
                res = stat.executeQuery(sql);
                //处理结果
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                return res.next();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return false;
    }
}
```

3. 解决sql注入问题代码：

   ```java
   package com.jdbcdemo;
   
   import com.JDBCUtil.JDBCUtil;
   
   import java.sql.*;
   import java.util.Scanner;
   
   /**
    * @author四五又十
    * @create 2020/1/26 19:49
    */
   public class demo7 {
       public static void main(String[] args) {
           Scanner sc = new Scanner(System.in);
           System.out.println("请输入用户名");
           String username = sc.nextLine();
           System.out.println("请输入密码");
           String password = sc.nextLine();
           if(new demo7().login(username, password)){
               System.out.println("登陆成功");
           }else{
               System.out.println("用户名或密码错误");
           }
       }
   
       public boolean login(String username, String password){
           if(username == null || password == null) {
               return false;
           }else {
               //连接数据库
               Connection conn = JDBCUtil.getConnection();
               //定义sql
               String sql = "select * from user where username = ? and password = ?";
               //获取执行sql对象
               PreparedStatement pstat = null;
               ResultSet res = null;
               try {
                   pstat = conn.prepareStatement(sql);
                   //给？赋值
                   pstat.setString(1, username);
                   pstat.setString(2, password);
                   //执行sql
                   res = pstat.executeQuery();
                   //处理结果
               } catch (SQLException e) {
                   e.printStackTrace();
               }
               try {
                   return res.next();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
           return false;
       }
   }
   ```

4.转账问题