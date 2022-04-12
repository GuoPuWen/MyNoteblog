	参考的是b站上黑马老师的视频，那个黑马的老师太硬核了，对mybatis框架的理解很深入，而且手写了框架的代码，下面的是我自己的一些理解

##### 1.创建前准备

​	创建mybatis框架需要导入mybatis的jar包，还有数据库mysql本身的jar包。所以创建maven工程，更好的导入jar包。

##### 2.在pom.xml中导入jar包

​	数据库用的5.几版本的，mybatis也不是用的最新的，我怕版本不兼容，因为idea不是最新的，反正之前学jdbc的时候用最新的8.x版本的数据库，一直没有成功。顺便还导入junit测试jar包

```xml
 <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>
    </dependencies>
```

##### 3.编写实体类

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\1.png)

那么实体类的属性（就是java类中的get和set方法后面，将首字母小写的一个字符串。例如 getName，那么name就是属性）要和数据库的列名保持一致（如果不一致也可以，后面文章中会写到），这个java类

```java
package com.Domain;
import java.util.Date;

/**
 * author by four and ten
 * create by 2020/3/12 12:40
 */
public class User {
    private Integer id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", sex='" + sex + '\'' +
                ", birthday=" + birthday +
                ", address='" + address + '\'' +
                '}';
    }
}
```

##### 4.编写Dao操作类

使用的是编写接口，然后使用代理类的方式。

com.Dao.UserDao

```
public interface UserDao {
    /**
     * 查询所有
     */
    void findAll();
}
```

##### 5.创建Dao操作类的配置文件

配置文件都写在resource文件夹下，这里将Dao操作类的配置文件放在和Dao操作类具有相同目录结构下，这里需要注意的是这个配置文件一定要放在resource下的一个文件夹下，不能直接放在resource目录下，否则会报错，所以这里最好放在和Dao操作类具有相同目录结构下，结构清晰。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace属性要和操作Dao的接口的全限定类名一样-->
<mapper namespace="com.Dao.UserDao">
    <!--配置查询所有 id值要和该接口的方法名一致 resultType是结果封装的类-->
    <select id="FindAll" resultType="com.Domain.User">
        select * from user
    </select>
</mapper>
```

这里Mapper中namespace的定义本身是没有限制的，只要不重复即可，但如果要使用Mybatis的DAO接口动态代理，则namespace必须为DAO接口的全路径，

##### 6.创建主配置文件

在maven中，配置文件一般是放在resource文件夹中，名字无所谓。我的叫MapConfig.xml，直接放在resource主目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis的主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/user"/>
                <property name="username" value="root"/>
                <property name="password" value="213213"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <mapper resource=""com/Dao/UserDaoConfig.xml"/>
    </mappers>
</configuration>
```

使用mybatis框架有两种方式，xml和注解的方式，这里使用的是xml配置的方式

##### 7.编写测试代码

测试代码写在test文件夹下

```java
public class Test {
    @org.junit.Test
    public void testFindAll() throws IOException {
        //加载主配置文件
        InputStream in = Resources.getResourceAsStream("MapperConfig.xml");
        //创建sqlSessoin对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = builder.build(in);
        SqlSession session = build.openSession();
        //创建代理Dao对象
        UserDao Dao = session.getMapper(UserDao.class);
        List<User> users = Dao.FindAll();
        for (User user : users) {
            System.out.println(user);
        }
    }

}
```

##### 8.整个项目分析

从test测试代码开始

1. 加载主配置文件，

```java
InputStream in = Resources.getResourceAsStream("MapperConfig.xml");
```

这里的Resources类是**org.apache.ibatis.io.Resources**包下的，打开这个包看源码的话，其实就是通过类加载器来加载类目录下的文件，当然也可以直接通过类加载器来加载主配置文件

```java
InputStream in = Test.class.getClassLoader().getResourceAsStream("MapperConfig.xml");
```

2. 创建SqlSession对象，这里创建SqlSession对象使用了工厂模式，而创建该工厂使用了构建者模式，

> ##### 构造者模式的优点
>
> ①使用构造者模式可以使客户端不必知道产品内部组成的细节。
>
> ②具体的构造者类之间是相互独立的，这有利于系统的扩展。
>
> ③具体的构造者相互独立，因此可以对建造的过程逐步细化，而不会对其他模块产生任何影响。

```java
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory build = builder.build(in);
SqlSession session = build.openSession();
```

3. 创建Dao接口实现类，使用了代理模式

```
UserDao Dao = session.getMapper(UserDao.class);
```

##### 9.整个项目结构

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\2.png)

