上篇文章介绍了mybatis入门，在入门案例中使用了查询所有操作，这篇文章介绍mybatis中对数据库里的CRUD操作，在这之前，先导入日志分析工具log4j

## 一、日志分析工具log4j

1. 导入依赖jar包

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
</dependency>
```

2. 添加配置文件

   该配置文件只需要添加至resource文件夹下即可，log4j会自动读取。文件名为log4j.properties

```
log4j.rootLogger=DEBUG,A1
log4j.logger.org.apache=DEBUG
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n
```

3. 结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\3.png)

配置完成之后控制台会出现很多的日志信息

## 二、CRUD操作

注意：这里只给出Dao接口类和对应的UserDaoConfig配置文件以及测试类的代码，对于CRUD操作只需要修改这两个地方

##### 1. 查询操作

1. 查询所有

com.Dao.userDao

```java
    /**
     * 查询所有
     */
    List<User> findAll();
```

UserDaoConfig配置文件

```xml
    <select id="findAll" resultType="com.Domain.User">
        select * from user
    </select>
```

为避免测试类出现大量的重复代码，使用如下简化：

```java
public class Test {
    InputStream in;
    SqlSessionFactoryBuilder builder;
    SqlSessionFactory build;
    SqlSession session;
    UserDao userDao;
    @Before
    public void init() throws IOException {
        in = Resources.getResourceAsStream("MapperConfig.xml");
        builder = new SqlSessionFactoryBuilder();
        build = builder.build(in);
        session = build.openSession();
        userDao = session.getMapper(UserDao.class);
    }

    /**
     * 测试查询所有
     * @throws IOException
     */
    @org.junit.Test
    public void testFindALll() throws IOException {
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
    }
    @After
    public void destory() throws IOException {
        //手动提交事务
        session.commit();
        in.close();
        session.close();
    }

}
```



2. 根据id查询

com.Dao.userDao

```java
    /**
     * 根据id查询
     * @param userId
     * @return
     */
    User findById(Integer userId);
```

UserDaoConfig配置文件

```xml
    <select id="findById" parameterType="int" resultType="com.Domain.User">
        select * from user where id = #{id}
    </select>
```

- select 标签：书写查询sql语句
- id属性：用于指定该sql语句对应那个方法
- parameterType：用于指定传入参数的类型，这里处基本数据类型外，其他类要写全限定类
- resultType：用于指定结果集的类型。
- #{}：表示占位符，与jdbc中的?作用类似

测试代码

```java
@org.junit.Test
public void testFindById(){
	User user = userDao.findById(42);
	System.out.println(user);
}
```

##### 2.保存操作

1. 新增用户

com.Dao.userDao

```
/**
     * 新增用户
     * @param user
     */
    void saveUser(User user);
```

UserDaoConfig配置文件

```xml
    <!-- 增  -->
    <insert id="saveUser" parameterType="com.Domain.User" >
        insert into user (username,birthday,sex,address) values (#{username}, #{birthday}, #{sex}, #{address})
    </insert>
```

测试代码

```java
/**
     * 保存用户
     */
    @org.junit.Test
    public void testSaveUser(){
        User user = new User();
        user.setBirthday(new Date());
        user.setSex("男");
        user.setUsername("zhangsan");
        user.setAddress("西安市");
        System.out.println(user);
        userDao.saveUser(user);
    }
```

- insert：书写insert插入sql语句
- 注意：这里必须要手动提交事务，如果不提交事务，mybatis会自动回滚事务，那么就不能保存。

```java
session.commit();
```

注意：新增用户的id返回值。因为这里的id是AUTO_INCREMENT是自增长的，如果要返回当前新增的用户的id值，那么就要在UserDaoConfig配置文件中insert标签中加入

```java
    <!-- 增  -->
    <insert id="saveUser" parameterType="com.Domain.User" >
        <selectKey keyColumn="id" keyProperty="id" resultType="int" >
            select last_insert_id();
        </selectKey>
        insert into user (username,birthday,sex,address) values (#{username}, #{birthday}, #{sex}, #{address})
    </insert>
```

然后运行

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\4.png)

##### 3.删除操作

com.Dao.userDao

```
/**
* 删除用户
* @param id
* @return int
*/
int deleteUser(int id);
```

UserDaoConfig配置文件

```xml
    <!-- 删除用户 -->
    <delete id="deleteUser" parameterType="int">
        delete from user where id = #{id}
    </delete>
```

测试代码

```java
    @org.junit.Test
    public void testDeleteuser(){
        int deleteuser = userDao.deleteUser(42);
        System.out.println(deleteuser);
    }

```

##### 4.更新操作

com.Dao.userDao 

```java
    /**
     * 更新用户
     */
    int updateUser(User user);
```

UserDaoConfig配置文件

```xml
    <!-- 更新用户 -->
    <update id="updateUser" parameterType="com.Domain.User" >
        update user set username = #{username}, birthday = #{birthday}, sex = #{sex}, address = #{address} where id = #{id}
    </update>
```

##### 5.模糊查询

1. #{}的方式

com.Dao.userDao 

```java
/**
* 模糊查询
* @return
*/
List<User> fingByName(String username);
```

UserDaoConfig配置文件

```xml
    <!-- 模糊查询 -->
    <select id="fingByName" parameterType="String" resultType="com.Domain.User">
        select * from user where username like #{username};
    </select>

```

测试代码

```java
@org.junit.Test
public void testFindByName(){
    List<User> users = userDao.fingByName("%王%");
    for (User user : users) 
        System.out.println(user);
    }
}
```

注意：在sql语句中模糊查询时要加入%%，但是我们在配置文件内没有加入，所以要在传入参数的时候加入，不加入则没法模糊查询

2. ${}的方式

UserDaoConfig配置文件

```xml
    <select id="fingByName" parameterType="String" resultType="com.Domain.User">
        select * from user where username like '%${value}%';
    </select>
```

测试代码里就不用%%了，这中方法明显是使用拼接字符串的方式。并且需要注意的是${}里面的值只能写value写其他内容则会报错

##### 6.聚合函数

com.Dao.userDao

```java
int findtotal();
```

UserDaoConfig配置文件

```xml
    <select id="findtotal" resultType="int">
        select count(id) from user;
    </select>
```

测试代码

```java
@org.junit.Test
public void testFindTatal(){
    int count = userDao.findtotal();
    System.out.println(count);
}
```

## 三、#{}与${}

**#{}表示一个占位符号**

​	通过#{}可以实现 preparedStatement 向占位符中设置值，自动进行 java 类型和 jdbc 类型转换，#{}可以有效防止 sql 注入。 #{}可以接收简单类型值或 pojo 属性值。 如果 parameterType 传输单个简单类型值，#{}括号中可以是 value或其它名称。 

**${}表示拼接 sql 串**

​	通过${}可以将 parameterType 传入的内容拼接在 sql 中且不进行 jdbc 类型转换， ${}可以接收简单类型值或 pojo 属性值，如果 parameterType 传输单个简单类型值，${}括号中只能是 value。 

**不使用resultType的方式**

​	前面讲的CRUD的操作传入的参数都是使用了resultType这个标签来实现的，其实也可以不使用这个标签，例如：

com.Dao.userDao

```Java
/**
* 不使用resultType的方式
*/
User findById2(@Param("id") int id);
```

UserDaoConfig配置文件

```xml
	<select id="findById2" resultType="com.Domain.User">
        select * from user where id =  #{param1};
    </select>
```

可以使用@Param的方式来实现

## 四、mybatis的sql配置文件

##### 1.parameterType

parameterType在Dao的配置文件中充当传入参数的作用，它的格式为：

- 实体类：全限定类名（没有设置别名的情况下）
- 基本数据类型和String类：包名，类名，直接写类型名称

基本数据类型和String类型，可以不用写全限定类名的原因是，mybatis已经将基本数据类型和String类设置了别名，所以只需要使用别名就可以。当然，mybatis还支持自定义别名，设置方法如下：在主配置文件下，配置typeAliases标签，这个后面说明。

##### 2.resultType

​	resultType属性可以指定结果集类型，支持的结果集类型和parameterType一样，当然也有定义别名。需要注意的是如果结果集是实体类，那么实体类的属性要和数据库中的表的列名要一致。

##### 3.resultMap

​	使用resultType必须要求数据库表中的列名和实体类的属性值要一致，如果不一致则会封装不到数据，当然在数据库的层面上提供了一个解决办法，就是在sql语句中设置别名，不过这个方法有点麻烦。对于每条sql语句都要设置别名。

​	mybatis为我们提供了一个解决办法（这里还要注意：mysql在windows下是不区分大小写的）。就是使用resultMap封装结果集。

实体类的属性(getter和setter方法去掉get(set)后将首字母小写)：

```Java
public class User {
    private Integer userid;
    private String userName;
    private String userSex;
    private Date userBirthday;
    private String userAddress;

    public Integer getUserid() {
        return userid;
    }

    public void setUserid(Integer userid) {
        this.userid = userid;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getUserSex() {
        return userSex;
    }

    public void setUserSex(String userSex) {
        this.userSex = userSex;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    @Override
    public String toString() {
        return "User{" +
                "userid=" + userid +
                ", userName='" + userName + '\'' +
                ", userSex='" + userSex + '\'' +
                ", userBirthday=" + userBirthday +
                ", userAddress='" + userAddress + '\'' +
                '}';
    }
}
```

现在查询所有肯定是查询不到的

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\5.png)

但是还是有部分数据被查询到了，这是因为mysql在windows下不区分大小写的原因。接下来就是要配置dao配置文件

userDaoConfig.xml

```xml
 <resultMap id="userMap" type="com.Domain.User">
        <id column="id" property="userid"></id>
        <result column="username" property="userName"></result>
        <result column="sex" property="userSex"></result>
        <result column="address" property="userAddress"></result>
        <result column="birthday" property="userBirthday"></result>
    </resultMap>

    <!--配置查询所有-->
    <select id="findAll" resultMap="userMap">
        select id, username, sex as usersex, birthday from user
    </select>
```

- type属性：指定实体类的全限定类名
- id属性：唯一标识，在sql语句的标签内使用
- id标签：主键字段
- result标签：非主键字段
- column属性：数据库列名
- property属性：实体类属性名称

指定好resultMap的配置之后，要在sql语句标签中使用resultMap属性，值为设置的id，这样就可以解决数据库列名和实体类属性值不一样的问题了。

##### 4.sql标签

使用sql标签，可以将一些常用的sql语句封装好，要用的时候直接引用即可

```xml
<sql id="default">
        select * from user
</sql>


    <!--配置查询所有-->
    <select id="findAll" resultType="com.Domain.User">
        <include refid="default"></include>
    </select>
```



## 五、主配置文件详解

主配置文件中的常见标签以及顺序

```
configuration
    properties（属性）
        --property
    -settings（全局配置参数）
        --setting
    -typeAliases（类型别名）
        --typeAliase
        --package
    -environments（环境集合属性对象）
        --environment（环境子属性对象）
            ---transactionManager（事务管理）
            ---dataSource（数据源）
    -mappers（映射器）
        --mapper
        --package
```

**这里需要特别注意的是这些个配置文件的顺序，是不能颠倒的，xml的约束是很严格的。**

##### 1.properties

这个标签主要是改造dataSource的，在没使用这个标签之前，我们是这样配置的

```java
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/user?characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="213213"/>
            </dataSource>
```

在这里写显得比较冗余，这里可以使用一个properties标签，将这些信息写入配置文件，其中的resource属性可以用来指定配置文件的路径。

主配置文件MapperConfig.xml

```xml
  <properties resource="db.properties"></properties>        
```

db.properties

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/user?characterEncoding=UTF-8
username=root
password=213213
```

配置好了properties标签之后，在dataSource数据源中就可以直接通过${key}，这里的key指的是在配置文件内使用的key。

```xml
<!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
```

需要注意的是，db.properties需要写在类路径下，当然这个maven工程就要在resource目录下了。

properties的属性：

- resource：用于指定 properties 配置文件的位置，要求配置文件必须在类路径下
- url：指定资源的url，不常用

当然，如果不想重新建一个配置文件，也可以在properties标签下有一个子标签propertie，在子标签中有2个属性，name和value，这两个属性的用处很显然，name就是key，对应着配置文件中key，value。

```xml
    <properties>
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/user?characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="213213"/>
    </properties>
```

##### 2.typeAliases

这个标签在前面已经提到过，这个标签是用来设置别名的，在resulttype或者是resulyMap中要写全限定类名，如果每个配置都要这么书写则会很麻烦，于是mybatis给我们提供了这么一个标签，去写别名

- typeAlias ：单个定义别名
  - alias：别名名称
  - type：要设置别名的类
- package：批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） 

```xml
    <typeAliases>
        <typeAlias type="com.Domain.User" alias="user"></typeAlias>
        <package name="com.Domain"/>
    </typeAliases>
```

##### 3.mappers

这个标签是用来配置dao配置文件的映射标签，其中有个子标签mapper指定其resource属性或者class属性

