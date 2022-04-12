### 一、概述

Spring中的JdbcTemplate对原始Jdbc API进行了封装，Spring对数据库操作需求提供了很好的支持，并在原始JDBC基础上，构建了一个抽象层，提供了许多使用JDBC的模板和驱动模块，为Spring应用操作关系数据库提供了更大的便利。所以会使用Spring中的Jdbc是很有必要的。

### 二、快速入门

##### 1.导入jar包

```xml
 		<!--Spring的jar包-->
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
		<!--Spring JdbcTemplate的jar包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
		<!--Spring事务控制的jar包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
		<!--mysql的jar包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
```

##### 2.编写Spring的配置文件

1. 导入约束
2. 配置数据源，配置数据源，可以有多种选择，使用C3P0的数据源或者使用spring的内置数据源。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--将Dao交给Spring的ioc容器管理-->
    <bean id="accountDao" class="com.SpringDemo4.dao.impl.AccountDaoImpl"></bean>

    <!--将Spring的jdbcTemplate交给Spring的ioc容器管理-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置数据源 使用spring的内置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///user"></property>
        <property name="username" value="root"></property>
        <property name="password" value="213213"></property>
    </bean>
</beans>
```

##### 3.基本使用

```java
    @Test
    public void testInsert(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //获取jdbcTemplate对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        //执行操作
        jt.execute("insert into account values(3,'wangwu', 1000)");
    }
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\16.png)

### 三、增删改查的操作

##### 1.保存操作

```java
     //获取ioc容器
	ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
	//获取jdbcTemplate对象
	JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
	//执行操作
	jt.update("insert into account(name, money) values('zhaoliu', 1000)");
```

##### 2.更新操作

```java
 //获取ioc容器
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
//获取jdbcTemplate对象
JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
//执行操作
 jt.update("update account set money = ? where id = ?",500, 2);
```
##### 3.删除操作

```java
 //获取ioc容器
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
//获取jdbcTemplate对象
JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
//执行操作
 jt.update("delete from account where id = ?",2);
```

##### 4.查询所有操作

查询操作和其他操作不同的是，需要定义将查询出来的数据封装到哪里，有两种方式，一种是自定义封装类继承RoMapper，另外一种是直接使用Spring的JdbcTemplate提供的封装类

```java
/**
 * author by four and ten
 * create by 2020/4/6 15:02
 */
public class test {
    @Test
    public void testInsert(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //获取jdbcTemplate对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        //查询所有
        List<Account> accounts = jt.query("select * from account", new AccountRoMapper());
        for (Account account : accounts) {
            System.out.println(account);
        }
    }
}
//自定义封装类，实现RowMapper接口
class AccountRoMapper implements RowMapper<Account>{

    @Override
    public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
        Account account = new Account();
        account.setId(rs.getInt("id"));
        account.setName(rs.getString("name"));
        account.setMoney(rs.getFloat("money"));
        return account;
    }

}
```

JdbcTemplate也封装了一个类，可以供我们直接使用BeanPropertyRowMapper

```java
@Test
    public void testInsert(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //获取jdbcTemplate对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        //查询所有
        List<Account> accounts = jt.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
        for (Account account : accounts) {
            System.out.println(account);
        }
    }
```

##### 5.查询一个

查询一个往往使用查询所有的操作然后返回一个list集合，然后取一个即可

```java
@Test
    public void testInsert(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //获取jdbcTemplate对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        //查询所有
        List<Account> accounts = jt.query("select * from account where id = ?",new BeanPropertyRowMapper<Account>(Account.class),3);
        System.out.println(accounts.isEmpty() ? "没有结果" : accounts.get(0));
    }
```

##### 6.查询返回一行一列，使用聚合函数

```java
@Test
    public void testInsert(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //获取jdbcTemplate对象
        JdbcTemplate jt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        //查询所有
        Integer total = jt.queryForObject("select count(id) from account", Integer.class);
        System.out.println(total);
    }
```

### 四、在dao中使用JdbcTemplate

在dao中使用jdbcTemplate有两种方式，一种是常规的方式，但是当dao比较多时，有很多的重复代码，为了解决这个问题，jdbcTemplate提供了一个JdbcDaoSupport类，让dao直接继承这个类，可以减少这些重复性的代码。

##### 1.编写dao接口

为了方便就写一个方法

```java
public interface IAccountDao {

    /**
     * 根据Id查询账户
     * @param accountId
     * @return
     */
    Account findAccountById(Integer accountId);
}
```

##### 2.第一种方法使用注入的方式

```java

/**
 * 账户的持久层实现类
 */
@Repository
public class AccountDaoImpl2 implements IAccountDao {

   private JdbcTemplate jdbcTemplate;
    
	public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}


    @Override
    public Account findAccountById(Integer accountId) {
        List<Account> accounts = jdbcTemplate.query("select * from account where id = ?",new BeanPropertyRowMapper<Account>(Account.class),accountId);
        return accounts.isEmpty()?null:accounts.get(0);
    }

}

```

当然在bean.xml配置文件中肯定要将JdbcTemplate交给Spring来管理。在这里中重复代码便是set方法，当然解决这个问题有2种方式解决，可以使用注解，也可以使用spring提供的类JdbcDaoSupport。

##### 3.第2种方法

```java
/**
 * 账户的持久层实现类
 */
public class AccountDaoImpl extends JdbcDaoSupport implements IAccountDao {

    @Override
    public Account findAccountById(Integer accountId) {
        List<Account> accounts = super.getJdbcTemplate().query("select * from account where id = ?",new BeanPropertyRowMapper<Account>(Account.class),accountId);
        return accounts.isEmpty()?null:accounts.get(0);
    }
    
}
```

使用这种方式只需要在bean.xml中配置数据源即可。