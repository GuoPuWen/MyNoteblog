前面说到Spring的Ioc的基本思想，这章记录Spring ioc的具体使用，Spring的使用有两种方式，一种是基于注解的，一种是基于xml的。这章节介绍基于xml的，这里使用的是maven工程。

### 一、快速入门Spring

##### 1.导入坐标

```xml
 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
</dependency>
```

##### 2.创建业务层接口

​	这里主要解决的是业务层和持久层之间的依赖关系，所以这里需要创建业务层接口以及实现类，创建持久层接口以及实现类。

持久层接口以及实现类

com.SpringDemo.Dao

```JAVA
public interface AccountDao {
    /**
     * 保存账户
     */
    void saveAccount();
}
```

com.SpringDemo.Dao.Impl

```JAVA
public class AccountDaoImpl implements AccountDao {
    public void saveAccount() {
        System.out.println("保存账户");
    }
}
```

业务层接口以及实现类

com.Service

```java
public interface AccountService {
    void saveAccount();
}
```

com.Service.Impl

```java
public class AccountServiceImpl implements AccountService {
    //主要是解决这里的依赖问题
    private AccountDao accountDao;
    public void saveAccount() {
        accountDao.saveAccount();
    }
}
```

##### 3.在resource文件夹下创建配置文件

配置文件的名称是随意的，这里把它命名为bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 配置Service层 -->
    <bean id="accountService" class="com.SpringDemo1.Service.Impl.AccountServiceImpl"></bean>
    <!-- 配置Dao层 -->
    <bean id="accountDao" class="com.SpringDemo1.Dao.Impl.AccountDaoImpl"></bean>

</beans>
```

- bean标签：配置要Spring帮我们创建的对象，并放与ioc容器中
  - id：对象的唯一标识
  - class 属性：指定要创建对象的全限定类名

##### 4.创建展现层测试代码是否成功

```java
public class Cilent {
    public static void main(String[] args) {
        //加载配置文件，获取ioc容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //根据bean的id获取对象
        AccountService accountService = ac.getBean("accountService", AccountService.class);
         AccountDao accountDao = ac.getBean("accountDao", AccountDao.class);
        //检测对象是否创建成功
        System.out.println(accountDao);
        System.out.println(accountService);

    }
}
```

##### 5.运行结果

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\1.png)

### 二、Sprin工厂中类结构图

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\3.png)

- BeanFactory：容器的底层接口
- ApplicationContext：常用的接口
- ClassPathXmlApplicationContest：基于xml的实现类
- FileSystemXmlApplicationContest：基于xml的实现类
- AnnotationConfigApplicationContest：基于注解配置的实现类

> BeanFactory和ApplicationContext的区别：ApplicationContext是BeanFactory的子接口，它们创建对象的时间不一样，ApplicationContext：只要一读取配置文件，默认情况下就会创建对象；BeanFactory：什么使用什么时候创建对象。



>ClassPathXmlApplicationContext：
>
>它是从类的根路径下加载配置文件 推荐使用
>
>FileSystemXmlApplicationContext：
>
>它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。 
>
>AnnotationConfigApplicationContext：
>
>当我们使用注解配置容器对象时，需要使用此类来创建 spring 容器。它用来读取注解。

### 三、IOC容器中的bean

##### 1.bean标签

在配置文件中，要将某一对象交给Spring来管理，使用xml的方式需要在配置文件中beans标签中使用bean标签

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 配置Service层 -->
    <bean id="accountService" class="com.SpringDemo1.Service.Impl.AccountServiceImpl"></bean>
    <!-- 配置Dao层 -->
    <bean id="accountDao" class="com.SpringDemo1.Dao.Impl.AccountDaoImpl"></bean>

</beans>
```

注意：我们知道ioc是通过反射来创建对象，那么在默认情况下当配置bean标签之后，默认使用改类i中的无参构造函数，如果该类中没有无参构造函数那么无法创建成功

该标签的一些属性：

- id：给对象在容器中提供一个唯一标识。用于获取对象。

- class：指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。

- scope：指定对象的作用范围

  - singleton ：默认值，单例的
  - prototype ：多例的
  -  request ：WEB 项目中，Spring 创建一个 Bean 的对象,将对象存入到 request 域中
  -  session ：WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到 session 域中.

  -  global session ：WEB 项目中，应用在集群环境，如果没有集群环境那么 

    globalSession 相当于 session.

- init-method：指定类中的初始化方法名称
- destroy-method：指定类中销毁方法名称

当把scope属性设置为prototype时，每次使用该对象都会创建一个该对象实例，也就是多，例的。例如：下面默认情况下：

bean.xml

```xml
 <bean id="accountDao" class="com.SpringDemo1.Dao.Impl.AccountDaoImpl" ></bean>
```

测试代码

```java
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        AccountService accountService = ac.getBean("accountService", AccountService.class);
        AccountDao accountDao = ac.getBean("accountDao", AccountDao.class);
        AccountDao accountDao2 = ac.getBean("accountDao", AccountDao.class);
        System.out.println(accountDao);
        System.out.println(accountDao2);
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\4.png)

当设置为scope=”prototype“时

```xml
<bean id="accountDao" class="com.SpringDemo1.Dao.Impl.AccountDaoImpl" scope="prototype"></bean>
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\5.png)

##### 2.bean的生命周期

单例对象：scope="singleton"或者不指定值。默认为单例：

- 对象出生：当应用加载，创建容器时，对象就被创建了。 

- 对象活着：只要容器在，对象一直活着。 

- 对象死亡：当应用卸载，销毁容器时，对象就被销毁了。 

多例对象：scope="prototype"，每次访问对象时，都会重新创建对象实例

- 对象出生：当使用对象时，创建新的对象实例。 

- 对象活着：只要对象在使用中，就一直活着。 

- 对象死亡：当对象长时间不用时，被 java 的垃圾回收器回收了。 

### 四、实例化bean的两种方式

##### 1.使用无参构造函数

使用这个方式，就是上面我们配置的方式，需要bean中有默认的无参构造函数，如果没有那么会创建失败

##### 2.用静态工厂的方法

静态工厂生产AccountService对象：

```java
public class AccountServiceBeanFactory {
    public static AccountService acccountServiceFactory(){
        return new AccountServiceImpl();
    }
}
```

bean.xml

```xml
<bean id="accountService" class="com.SpringDemo1.Service.Factory.AccountServiceBeanFactory" factory-method="acccountServiceFactory">
</bean>
```

测试代码：

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
AccountService accountService = ac.getBean("accountService", AccountService.class);
System.out.println(accountService);
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\6.png)

注意：

- factory-method 属性：指定生产对象的静态方法



