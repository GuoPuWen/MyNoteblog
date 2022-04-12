### 一、概述

我们知道，持久层处理dao请求，业务层处理业务逻辑，那么事物的控制肯定需要在业务层。spring为我们提供了一组事务控制的API，用来控制事务。

spring的事务控制分别有声明式控制和编程式事务控制，编程式事务控制使用的很少，一般都是使用声明式事务控制。

进行事务的控制使用的对象是DataSourceTransactionManager

### 二、基于xml的事务控制

##### 1.导入坐标依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.2.RELEASE</version>
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

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
```

spring-tx是Spring的事务控制的jar包。

##### 2.创建数据库

我是用银行的转账这个案例，每次一讲到事务就必用银行转账案例(笑哭)

```sql
CREATE DATABASE USER;
CREATE TABLE account(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20) NOT NULL,
	money DOUBLE
);
```

##### 3.创建实体类

```java
/**
 * 账户的实体类
 */
public class Account implements Serializable {

    private Integer id;
    private String name;
    private Double money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
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
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

##### 4.编写dao

要实现银行转账，那么需要findAllByName、update、save方法

```java
/**
 * 账户的持久层接口
 */
public interface IAccountDao {

    /**
     * 根据名称查询账户
     * @param accountName
     * @return
     */
    Account findAccountByName(String accountName);

    /**
     * 更新账户
     * @param account
     */
    void updateAccount(Account account);
}
```

##### 5.编写dao实现类

在实现类里，由于还没有整合mybatis那么我们使用的是spring的jdbcTemplate来帮我们执行sql

```java
/**
 * 账户的持久层实现类
 */
public class AccountDaoImpl implements AccountDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Account findAccountByName(String accountName) {
        List<Account> accounts = jdbcTemplate.query("select * from account where name = ?", new BeanPropertyRowMapper<Account>(Account.class), accountName);
        return accounts.isEmpty()?null:accounts.get(0);
    }

    @Override
    public void updateAccount(Account account) {
        String name = account.getName();
        Double money = account.getMoney();
        jdbcTemplate.update("update account set money = ? where name = ?",money ,name);
    }
}

```

##### 6.编写service接口以及实现类

```java
public interface AccountService {
    /**
     * 保存
     * @param account
     */
    void saveAccount(Account account);

    /**
     * 更新
     * @param account
     */
    void updateAccount(Account account);


    /**
     * 转账
     * @param sourceName        转出账户名称
     * @param targetName        转入账户名称
     * @param money             转账金额
     */
    void transfer(String sourceName,String targetName,Double money);

}

```

实现类

```java
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }


    @Override
    public void saveAccount(Account account) {
        accountDao.updateAccount(account);
    }

    @Override
    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }

    @Override
    public void transfer(String sourceName, String targetName, Double money) {
        System.out.println("transfer....");
        //2.1根据名称查询转出账户
        Account source = accountDao.findAccountByName(sourceName);
        //2.2根据名称查询转入账户
        Account target = accountDao.findAccountByName(targetName);
        //2.3转出账户减钱
        source.setMoney(source.getMoney() - money);
        //2.4转入账户加钱
        target.setMoney(target.getMoney() + money);
        //2.5更新转出账户
        accountDao.updateAccount(source);

     //       int i=1/0;

        //2.6更新转入账户
        accountDao.updateAccount(target);
    }
}
```

##### 7.配置xml

注意在spring的xml配置文件中，需要导入aop和事务的约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/tx
 http://www.springframework.org/schema/tx/spring-tx.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd">
<!--    &lt;!&ndash;将Dao交给Spring的ioc容器管理&ndash;&gt;-->
<!--    <bean id="accountDao" class="com.SpringDemo4.dao.impl.AccountDaoImpl"></bean>-->

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
    
    <!--配置dao-->
    <bean id="accountDao" class="com.SpringDemo4.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>
    
    <!--配置service-->
    <bean id="accountService" class="com.SpringDemo4.sercice.Impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>
    
    <!--配置事务管理器-->
    <bean id="DataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!--配置事务的通知-->
    <tx:advice id="txAdvice" transaction-manager="DataSourceTransactionManager">
        <tx:attributes>
            <tx:method name="*" read-only="false" propagation="REQUIRED"/>
            <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
        </tx:attributes>
    </tx:advice>
    
    <!--配置aop--->
    <aop:config>
        <aop:pointcut id="pt1" expression="execution(* com.SpringDemo4.sercice.Impl.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
    </aop:config>

</beans>
```

配置声明式事务的步骤：

1. **将DataSourceTransactionManager交给ioc容器，这个对象是管理事务的对象**
2. **配置事务的通知，tx:advice配置事务通知，其下的子标签tx:attributes用于配置事务的属性：**

- read-only：是否是只读事务。默认 false，不只读
- isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
- propagation：指定事务的传播行为
- timeout：指定超时时间。默认值为：-1。永不超时
- rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。 没有默认值，任何异常都回滚。
- no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，任何异常都回滚

3. **配置aop切入点表达式**

##### 8.测试

```java
	 @Test
    public void testTransfer(){
        //获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        AccountService accountService = ac.getBean("accountService", AccountService.class);
        accountService.transfer("zhangsan","lisi",100d);
    }
```

### 三、基于注解的事务控制

使用基于注解的配置，只需要是使用一个注解@Transactional，非常方便。配置注解的事务控制，只需要改变xml配置文件里的内容和业务层代码上添加一个注解即可

##### 1.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/tx
 http://www.springframework.org/schema/tx/spring-tx.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
<!--    &lt;!&ndash;将Dao交给Spring的ioc容器管理&ndash;&gt;-->
<!--    <bean id="accountDao" class="com.SpringDemo4.dao.impl.AccountDaoImpl"></bean>-->
    <!--配置扫描的包-->
    <context:component-scan base-package="com.SpringDemo4.sercice"></context:component-scan>

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

    <!--配置dao-->
    <bean id="accountDao" class="com.SpringDemo4.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>

    <!--配置service-->
    <bean id="accountService" class="com.SpringDemo4.sercice.Impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>

    <!--配置事务管理器-->
    <bean id="DataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 开启 spring 对注解事务的支持 -->
    <tx:annotation-driven transaction-manager="DataSourceTransactionManager"/>

</beans>
```

##### 2.service实现类

```java
@Transactional
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }


    @Override
    public void saveAccount(Account account) {
        accountDao.updateAccount(account);
    }

    @Override
    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }


    @Override
    public void transfer(String sourceName, String targetName, Double money) {
        System.out.println("transfer....");
        //2.1根据名称查询转出账户
        Account source = accountDao.findAccountByName(sourceName);
        //2.2根据名称查询转入账户
        Account target = accountDao.findAccountByName(targetName);
        //2.3转出账户减钱
        source.setMoney(source.getMoney() - money);
        //2.4转入账户加钱
        target.setMoney(target.getMoney() + money);
        //2.5更新转出账户
        accountDao.updateAccount(source);

            int i=1/0;

        //2.6更新转入账户
        accountDao.updateAccount(target);
    }
}
```

Transactional注解和在基于xml的方式中配置tx:attributes中的内容一致。

##### 3.总结

总结一下基于注解的事务控制

1. 在xml文件中开启扫描的包，并开启对象事务控制的支持

```xml
  <!--配置扫描的包-->
    <context:component-scan base-package="com.SpringDemo4.sercice"></context:component-scan>  
	<!-- 开启 spring 对注解事务的支持 -->
    <tx:annotation-driven transaction-manager="DataSourceTransactionManager"/>
```

2. 配置事务管理器

```xml
 <!--配置事务管理器-->
    <bean id="DataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

在需要事务控制类上添加@Transactional注解，里面的属性和基于xml的方式中配置tx:attributes中的内容一致。