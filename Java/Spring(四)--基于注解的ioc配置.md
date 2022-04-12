基于注解的ioc配置，要实现的功能和基于xml的ioc配置是一样的，只是把xml配置文件里的内容全部用注解替换掉。本文将先介绍仍然要使用xml来配置注解也就是半注解半配置文件，后面在解决这个问题全部使用注解。至于为社么会这样，请往下看

##### 1.导入坐标

导入的坐标和基于xml的配置的坐标一样

```xml
 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
</dependency>
```

使用maven来管理jar包的好处之一就是maven会把相关的jar包全部导入至工程中来。



##### 2.在Spring的xml配置文件中开启注解

注意在xml文件中必须导入context的名称空间的约束

bean.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/context
 http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.SpringDemo1"></context:component-scan>

</beans>   
```

context:component-scan：告知spring创建容器要扫描的包。这里发现我们仍然要使用到xml配置文件里的内容，因为要通过配置文件来开启spring对注解的支持，当然spring也有纯注解的解决办法，后面会说到。



##### 3.创建Ioc容器的注解

在之前的xml配置中，我们使用bean标签来将对象交给spring的ioc容器来管理，例如：

```xml
 <bean id="accountService" class="com.SpringDemo1.Service.Impl.AccountServiceImpl" ></bean>
  <bean id="accountDao" class="com.SpringDemo1.Dao.Impl.AccountDaoImpl" scope="prototype"></bean>
```

替换这些标签的注解@Compoent @Controller @Service @Repository，看到这些注解是否有些熟悉，没错了后面三个分别对应着三层，它们都是语义化注解。

- Compoent：把资源让给spring来管理，相当于在xml中配置一个bean，里面有一个value的属性，对应着bean的id值，如果不指定value属性，那么默认bean的id是当前的类名(注意：首字母是小写的)

- Controller：语义化标签，和Compoent一样，一般用于表现层的注解
- Service：语义化标签，和Compoent一样，一般用于业务层的注解
- Repository：语义化标签，和Compoent一样，一般用于持久层的注解





##### 4.依赖注入的注解



###### 1.@Value

@Value用于注入基本数据类型和String类型

```java
@Component("accountService")
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    @Value("zhangsan")
    private String name;

    public AccountDao getAccountDao() {
        return accountDao;
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public String getName() {
        return name;
    }
	//@Value("zhangsan")
    public void setName(String name) {
        this.name = name;
    }

    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```

Value可以在属性上也可以在set方法上



###### 2.@Resource

直接按照 Bean 的 id 注入。它也只能注入其他 bean 类型。

```java
@Component("accountService")
public class AccountServiceImpl implements AccountService {
    @Resource(name = "accountDao")
    private AccountDao accountDao;


    private String name;

    public AccountDao getAccountDao() {
        return accountDao;
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```

当注入accountDao之后就可以使用saveaccount方法，需要注意的是Resource的name指定的是bean的id，也就是注入的对象也必须要在ioc容器内



###### 3.@Autowired

​	自动按照类型注入，由于ioc容器是map结构，有key和value。key存放id值，value存放对象，当使用自动类型注入时，spring会在ioc容器里找符合该对象的value，如果只有一个那么直接注入给该对象，如果有多个，通过key来判断也就是通过id值与要注入对象的变量名来判断，如果一致则注入，如果不一致那么报错。

​	为了模拟上述情况，将AccountDaoImpl类复制一份

```java
@Component("accountDao1")
public class AccountDaoImpl1 implements AccountDao {
    public void saveAccount() {
        System.out.println("保存账户");
    }
}

```

```java
@Component("accountDao2")
public class AccountDaoImpl2 implements AccountDao {
    public void saveAccount() {
        System.out.println("保存账户");
    }
}
```

AccountServiceImpl

```java
@Component("accountService")
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;
    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```

可以看到，现在在ioc容器中有3个对象

| key            | value              |
| -------------- | ------------------ |
| accountDao1    | AccountDao类型     |
| accountDao2    | AccountDao类型     |
| accountService | AccountService类型 |

当在AccountServiceImpl中要注入类型为AccountDao时，spring查找ioc容器，发现有两个AccountDao类型的值，这时spring拿出要注入的类型的变量名accountDao与ioc容器中的key比较，发现没有key符合，那么抛出异常

```
 No qualifying bean of type 'com.SpringDemo1.Dao.AccountDao' available: expected single matching bean but found 2: accountDao1,accountDao2
```

这个时候只要将accountDao修改为accountDao1,accountDao2就行，主要看你使用哪一个。

###### 4.Qualifier

spring还提供了一个配套的注解@Qualifier，注意这个注解必须要和@Autowired一起使用，这个注解的存在就是为了解决上述，不想去修改变量名的问题，可以在@Autowired的下面添加@Qualifier。其中的value属性用于指定bean的id。

```java
@Component("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    @Qualifier("accountDao1")
    private AccountDao accountDao;

    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```



##### 5.用于改变作用范围@Scope

作用：这个标签相当于scope属性值

```
<bean id="" class="" scope="">
```

属性：指定范围的值。 

- singleton ：单例
- prototype ：多例
- request ：request域，需要在web环境
- session ：session域，需要在web环境
- application： context域，需要在web环境
- globalsession 集群环境的session域，需要在web环境
  

##### 6.生命周期相关的

@PostConstruct ：相当于init-method

@PreDestroy ：相当于destroy-method

