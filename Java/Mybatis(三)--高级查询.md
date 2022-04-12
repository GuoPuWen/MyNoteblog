### 一、mybatis的连接池

我们知道使用连接池技术可以有很多的好处：

- 资源重用
- 加快响应速度
- 利于资源分配

还有等等好处，常见的数据库连接池技术有c3p0，druid等等。mybatis也为我们封装好了它自己的连接池技术，在主配置文件中，配置数据源的时候，

```xml
<dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
</dataSource>
```

指定了type属性，其中的POOLED的值便是使用了数据库连接池技术，所以只要指定type属性为POOLED，则就是使用了数据库连接池。

### 二、mybatis的事务提交

事务的简短回顾：

- 事务定义：一个最小的不可再分的工作单元；通常一个事务对应一个完整的业务(例如银行账户转账业务，该业务就是一个最小的工作单元)
- 事务四大特征(ACID)：
  - 原子性(A)：事务是最小单位，不可再分
  - 一致性(C)：事务要求所有的DML语句操作的时候，必须保证同时成功或者同时失败
  - 隔离性(I)：事务A和事务B之间具有隔离性
  - 持久性(D)：是事务的保证，事务终结的标志(内存的数据持久到硬盘文件中)

- 开启事务：Start Transaction
- 事务结束：End Transaction
- 提交事务：Commit Transaction
- 回滚事务：Rollback Transaction

在mybatis的CRUD操作的时候，如果不手动提交事务，是无法将数据库保存在数据库里的，提交事务使用Sqlsession的commit方法

```java
 @After
    public void destory() throws IOException {
        //手动提交事务
        session.commit();
        in.close();
        session.close();
    }
```

设置自动提交事务的办法是在创建Sqlsession对象的时候传入一个参数boolean类型的，用于表明是否自动提交事务。

```java
session = factory.openSession(true);
```

### 三、mybatis的动态sql语句

##### 1.if标签

if标签支持在sql语句中多条件查询。例如：输入参数为user对象，如果有某一个属性为空，需要其他属性同时成立，比如如果用户名字段为空，那么只要地址匹配即可，如果不为空那么需要用户名和地址同时匹配

```xml
<select id="findByUser" resultType="com.Domain.User" parameterType="com.Domain.User">
        select * from user where 1=1
        <if test="username != null and username !=''">
            and username like #{username}
        </if>
        <if test="address != null and address != ''">
            and address like #{address}
        </if>
</select>
```

##### 2.where标签

在if标签中使用了where 1= 1，这样写不太方便，mybatis提供了一个where标签

```xml
 <select id="findByUser" resultType="com.Domain.User" parameterType="com.Domain.User">
        select * from user
        <where>
            <if test="username != null and username !=''">
                and username like #{username}
            </if>
            <if test="address != null and address != ''">
                and address like #{address}
            </if>
        </where>
</select>
```

##### 3.foreach

例如sql语句：select * from user where id in(42,43,50,51)

这样的sql语句可以使用foreach标签来完成

```xml
    <select id="findByIds" resultType="com.Domain.User">
        select * from user where id in
        <foreach collection="ids" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </select>
```

foreach标签内的属性都是字面意思，很好理解。

```java
 List<User> findByIds(@Param("ids") Integer[] ids);
```

测试方法

```java
@org.junit.Test
public void testFindByIds(){
    Integer[] ids = {42,43,50,51};
    List<User> users = userDao.findByIds(ids);
    for (User user : users) {
    System.out.println(user);
    }
}
```

### 四、多表查询基于注解

##### 1.一对一(多对一)第一种方法	

案例是：账户与用户的关系，一个用户可以有多个账户，一个账户只能有一个用户，所以这个关系从用户出发就是一对多，从账户出发就是一对一。

user表：

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\6.png)

account表：

​	![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\7.png)

有两种方法查询：很显然，第一种方法就是建立和查询的sql语句一一对应的实体类，例如：sql语句

```sql
SELECT 
	*
FROM
	account,USER
WHERE
	account.`UID` = user.`id`
```

那么就需要对应建立一个与sql语句查询的列名相对应的实体类其中的属性有id，username，birthday，sex，address，uid，money。这里有一个技巧，可以让该实体类直接继承user类，那么只需要写3个属性的get，set方法

###### 1.建立实体类

user

```java
public class User implements Serializable {

    private Integer id;
    private String username;
    private String address;
    private String sex;
    private Date birthday;

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

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
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

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                ", sex='" + sex + '\'' +
                ", birthday=" + birthday +
                '}';
    }
}
```

account

```java
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}

```

AccountUser

```java
public class AccountUser extends User {
    private Integer id;
    private Integer Uid;
    private Double money;

    @Override
    public Integer getId() {
        return id;
    }

    @Override
    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return Uid;
    }

    public void setUid(Integer uid) {
        Uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "AccountUser{" + super.toString() +
                "id=" + id +
                ", Uid=" + Uid +
                ", money=" + money +
                '}';
    }

```

###### 2.编写Dao接口方法

```java
public interface AccountDao {
    /**
     * 查询所有账户，同时获得该账户的所属用户信息 一对一
     * @return
     */
    List<AccountUser> findAll();
}
```

###### 3.accountDao.xml配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.MybatisDemo6.Dao.AccountDao">
<select id="findAll" resultType="com.MybatisDemo6.Domain.AccountUser" >
        select * from account, user where account.uid = user.id
    </select>
</mapper>
```

###### 4.MapperConfig主配置文件

###### 5.测试代码

直接调用方法，和之前的CRUD操作一样

###### 6.检验结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\8.png)



##### 2.一对一(多对一)第二种方法

​	通过上面可以发现这种方法的灵活性不高，而且要编写很多重复性的代码，mybatis给我们提供了另外一种方法：使用resulttype的方法

###### 1.修改Account类，加入User类的对象引用

```java
private User user;

public User getUser() {
	return user;
}

public void setUser(User user) {
	this.user = user;
}
```

###### 2.修改AccountDao的方法

```java
List<Account> findAllByAccount();
```

###### 3.重新配置AccountDao配置文件

```xml
 <resultMap id="accountMap" type="com.MybatisDemo6.Domain.Account">
        <id column="aid" property="id"></id>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
        <association property="user" javaType="com.MybatisDemo6.Domain.User">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
            <result column="address" property="address"></result>
        </association>
    </resultMap>
    
    <select id="findAllByAccount" resultMap="accountMap" >
        select * from account, user where account.uid = user.id
    </select>
```

###### 4.测试代码

```java
    @Test
    public void testFindAllByAccount(){
        AccountDao accountDao = session.getMapper(AccountDao.class);
        List<Account> accounts= accountDao.findAllByAccount();
        for (Account account : accounts) {
            System.out.println(account);
            System.out.println(account.getUser());
        }
    }
```

###### 5.结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\10.png)



##### 3.一对多

一个用户可以对应多个账户，这个关系为一对多

###### 1.修改User类

```java
    public List<Account> getAccount() {
        return account;
    }

    public void setAccount(List<Account> account) {
        this.account = account;
    }

    private List<Account> account;
```

###### 2.加入方法

```
 /**
     * 查询所有的用户，同时查询该用户的所有账户 一对多
     * @return
     */
    List<User> findAllByUser();
```

###### 3.修改Dao配置文件

```xml
    <resultMap id="userMap" type="com.MybatisDemo6.Domain.User">
    <id column="id" property="id"></id>
    <result column="username" property="username"></result>
    <result column="sex" property="sex"></result>
    <result column="birthday" property="birthday"></result>
    <result column="address" property="address"></result>
        <collection property="account" ofType="com.MybatisDemo6.Domain.Account">
            <id column="aid" property="id"></id>
            <result column="uid" property="uid"/>
            <result column="money" property="money"/>
        </collection>
    </resultMap>
```

###### 4.测试代码

```java
@Test
public void testFindAllByUser(){
    AccountDao accountDao = session.getMapper(AccountDao.class);
    List<User> users = accountDao.findAllByUser();
    for (User user : users) {
        System.out.println(user);
        System.out.println(user.getAccount());
    }
}
```

###### 5.结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\11.png)

##### 4.多对多

​	一个用户可以有多个角色，一个角色可以有多个用户，所以用户与角色之间是多对多的关系。在sql中多对多的关系需要借助与第三张表来实现，作为外键分别指向两张表的主键

user用户表

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\12.png)

role角色表

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\13.png)

user_role第三张表

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\14.png)

###### 1.创建role实体类

注意添加

```java
 private List<User> users;

    public List<User> getUsers() {
        return users;
    }
```

###### 2.加入方法

```java
public interface RoleDao {
    /**
     * 查询所有角色，并且查询角色对应的用户
     * @return
     */
    List<Role> findAll();
}
```

###### 3.配置Dao配置文件

```xml
<mapper namespace="com.MybatisDemo6.Dao.RoleDao">
    <resultMap id="roleMap" type="com.MybatisDemo6.Domain.Role">
        <id column="ID" property="roleID"></id>
        <result column="role_name" property="roleName"></result>
        <result column="role_desc" property="roleDesc"></result>
        <collection property="users" ofType="com.MybatisDemo6.Domain.User">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
            <result column="address" property="address"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="roleMap">
 select u.*,r.id as rid,r.role_name,r.role_desc from role r
         left outer join user_role ur  on r.id = ur.rid
         left outer join user u on u.id = ur.uid
    </select>
</mapper>
```

###### 4.编写测试类

```java
 @Test
    public void testFindAll(){
        RoleDao roleDao = session.getMapper(RoleDao.class);
        List<Role> roles = roleDao.findAll();
        for (Role role : roles) {
            System.out.println(role);
            System.out.println(role.getUsers());
        }
    }
```

### 五、mybatis的延迟加载策略

##### 1.概念

就是在需要用到数据时才进行加载，不需要用到数据时就不加载数据。延迟加载也称懒加载

**优点**：在使用关联对象时，才从数据库中查询关联数据，大大降低数据库不必要开销。

**缺点**：因为只有当需要用到数据时，才会进行数据库查询，这样在大批量数据查询时，因为查询工作也需要耗费时间，所以可能造成用户等待时间变长，造成用户体验下降。

##### 2.开启延迟加载

在全局配置文件中加入：

```java
<settings>
		<setting name="lazyLoadingEnabled"  value="true"/>
		<setting name="aggressiveLazyLoading" value="flase"/>
</settings>
```

### 六、mybatis缓存

在Mybatis中，分为一级缓存和二级缓存

![](C:\Users\four and ten\Desktop\笔记\JAVA\mybatis\15.png)

##### 1.一级缓存

一级缓存的作用域是sqlSession，一级缓存是默认开启的，要想触发mybatis的一级缓存，要满足：同一个session中、相同的SQL和参数

##### 2.二级缓存

mybatis 的二级缓存的作用域是一个mapper的namespace ，同一个namespace中查询sql可以从缓存中命中。二级缓存不是默认开启的，要开启二级缓存：

1. 在全局配置文件中开启：

```xml
<settings>
<!-- 开启二级缓存的支持 --> <setting name="cacheEnabled" value="true"/>
</settings>
```

不过这一步由于cacheEnabled 的取值默认就为 true，所以这一步可以省略不配置

2. **配置相关的** **Mapper** **映射文件**

```
<mapper namespace="com.MybatisDemo6.Dao.AccountDao">
	<!-- 开启二级缓存的支持 -->
	<cache></cache>
</mapper>
```

3. **配置** **statement** **上面的** **useCache** **属性**

```
    <!-- 根据 id 查询 --> 
<select id="findById" resultType="user" parameterType="int" useCache="true">
    select * from user where id = #{uid}
</select>
```

注意：针对每次查询都需要最新的数据 sql，要设置成 useCache=false，禁用二级缓存