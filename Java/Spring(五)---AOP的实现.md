Spring中关于AOP中的思想与概念在Spring的第一节中已经介绍过，所以这节直接介绍如何让实现Spring中的AOP。

案例：在service层中需要增加日志的管理代码，使用spring的aop来增强

### 一、基于xml的AOP配置的快速入门

##### 1.导入依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
		<!-- AOP的依赖-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
        </dependency>
</dependencies>
```

##### 2.创建Spring的配置文件导入约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

##### 3.编写业务逻辑代码

service层这里为了简便使用的是模拟操作

com.SpringDemo3.service.IAccountService

```java
/**
 * 账户的业务层接口
 */
public interface IAccountService {

    /**
     * 模拟保存账户
     */
    void saveAccount();

    /**
     * 模拟更新账户
     * @param i
     */
    void updateAccount(int i);

    /**
     * 删除账户
     * @return
     */
    int  deleteAccount();
}
```

com.SpringDemo3.service.impl.AccountServiceImpl

```java
/**
 * 账户的业务层实现类
 */
public class AccountServiceImpl implements IAccountService {

    @Override
    public void saveAccount() {
        System.out.println("执行了保存");
    }

    @Override
    public void updateAccount(int i) {
        System.out.println("执行了更新"+i);

    }

    @Override
    public int deleteAccount() {
        System.out.println("执行了删除");
        return 0;
    }
}

```

com.SpringDemo3.utils.Logger

```java
/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
public class Logger {

    /**
     * 用于打印日志：计划让其在切入点方法执行之前执行（切入点方法就是业务层方法）
     */
    public  void printLog(){
        System.out.println("Logger类中的pringLog方法开始记录日志了。。。");
    }
}
```

##### 4.在配置文件中配置aop

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd">
 
    <!-- 配置srping的Ioc,把service对象配置进来-->
    
    <bean id="accountService" class="com.SpringDemo3.service.impl.AccountServiceImpl"></bean>
    
     <!-- 配置Logger类 -->
    <bean id="logger" class="com.SpringDemo3.utils.Logger"></bean>
    
     <!--配置AOP-->
    <aop:config>
         <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
              <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(public void com.SpringDemo3.service.impl.AccountServiceImpl.saveAccount())">				</aop:before>
        </aop:aspect>
   </aop:config>
</beans>
```

##### 5.测试代码

```java
public class AOPTest {

    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        IAccountService as = (IAccountService)ac.getBean("accountService");
        //3.执行方法
        as.saveAccount();
    }
}
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\10.png)

### 二、基于xml的AOP配置的细节

可以发现，在配置AOP中核心就是bean.xml这个配置文件，主要是在这个配置文件内进行通知，切点的配置，介绍细节也是说这里面的标签

bean.xml

```XML
aop:aspect<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd">
 
    <!-- 配置srping的Ioc,把service对象配置进来-->
    
    <bean id="accountService" class="com.SpringDemo3.service.impl.AccountServiceImpl"></bean>
    
     <!-- 配置Logger类 -->
    <bean id="logger" class="com.SpringDemo3.utils.Logger"></bean>
    
     <!--配置AOP-->
    <aop:config>
         <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
              <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(public void com.SpringDemo3.service.impl.AccountServiceImpl.saveAccount())">				</aop:before>
        </aop:aspect>
   </aop:config>
</beans>
```

- aop:config：开始AOP的配置
- aop:aspect：配置切面
  -  id属性：是给切面提供一个唯一标识
  - ref属性：是指定通知类bean的Id。

在aop:aspect标签的内部使用对应的标签来配置通知的类型：

- aop:before：配置前置通知：在切入点方法执行之前执行
- aop:after-returning：配置后置通知：在切入点方法正常执行之后值。它和异常通知永远只能执行一个
- aop:after-throwing：配置异常通知：在切入点方法执行产生异常之后执行。它和后置通知永远只能执行一个
- aop:after：配置最终通知：无论切入点方法是否正常执行它都会在其后面执行

- aop:around ：配置环绕通知

**每一个通知都对应着三个属性：**

- method属性：用于指定哪个方法是前置通知
- pointcut属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强

切入点表达式的写法：execution(切入点表达式)

总体写法：访问修饰符 返回值  包名.包名.包名...类名.方法名(参数列表)

这样写太麻烦，简写规则：

1.  访问修饰符可以省略
2. 返回值可以使用通配符*，表示任意返回值*
3. *包名可以使用通配符，表示任意包。但是有几级包，就需要写几个*.
4. 包名可以使用..表示当前包及其子包
5.  类名和方法名都可以使用*来实现通配
6.   参数列表：可以直接写数据类型：基本类型直接写名称    int；引用类型写包名.类名的方式   java.lang.String；可以使用通配符表示任意类型，但是必须有参数；可以使用..表示有无参数均可，有参数可以是任意类型

全通配写法：

```
* *..*.*(..)
```

上面的案例中一般写成：

```
* com.SpringDemo3.service.impl..*(..)
```

- pointcut-ref：可以将切入点表达式写在通知的外面，这样就可以让多个通知使用该切入点表达式

```xml
<aop:config>
    <!-- 写在aop:aspect的外面-->
    <aop:pointcut id="pt1" expression="execution( * com.SpringDemo3.service.impl..*(..))"/>
    <aop:aspect id="logAdvice" ref="logger">
        <aop:before method="printLog"  pointcut-ref="pt1">					</aop:before>
    </aop:aspect>
</aop:config>
```

==各种不同的通知测试代码==

```xml
<aop:config>
    <aop:pointcut id="pt1" expression="execution( * com.SpringDemo3.service.impl..*(..))"/>
    <aop:aspect id="logAdvice" ref="logger">
        <aop:before method="beforePrintLog"  pointcut-ref="pt1"></aop:before>
        <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1"></aop:after-returning>
        <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>
        <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>
    </aop:aspect>
</aop:config>
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\11.png)

==环绕通知==

前面的4中通知都是在xml中配置通知的类型，这些类型决定需要增强的方法在那个位置执行，而环绕通知提供的一种可以在代码中手动控制增强方法何时执行的方式。

在环绕通知中，需要自己手动执行切入点方法，spring提供了一个接口ProceedingJoinPoint，其中该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。

```java
   /**
     * 环绕通知
     */
    public void arroundPrintLog(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        try {
            //前置通知
            this.beforePrintLog();
            pjp.proceed(args);
            //后置通知
            this.afterReturningPrintLog();
        } catch (Throwable throwable) {
            //异常通知
            this.afterThrowingPrintLog();
            throwable.printStackTrace();
        } finally {
            //最终通知
            this.afterPrintLog();
        }

    }
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\13.png)

### 三、基于注解的AOP配置快速入门

##### 1.导入依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
		<!-- AOP的依赖-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
        </dependency>
</dependencies>
```

##### 2.创建Spring的配置文件导入约束

这个约束文件要比基于xml的配置的约束文件要多一些context的内容

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:context="http://www.springframework.org/schema/context"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd
 http://www.springframework.org/schema/context
 http://www.springframework.org/schema/context/spring-context.xsd">
 </beans>
```

##### 3.在配置文件中开启注解AOP的支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/aop
         http://www.springframework.org/schema/aop/spring-aop.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 配置spring创建容器时要扫描的包-->
    <context:component-scan base-package="com.SpringDemo3"></context:component-scan>

    <!-- 配置spring开启注解AOP的支持 -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

##### 4.编写业务逻辑代码

```java
@Component("logger")
@Aspect
public class Logger {

    /**
     * 前置通知
     */
    @Before("execution(* com.SpringDemo3.service.impl..*(..))")
    public void beforePrintLog(){

        System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
    }
}
```

##### 5.测试代码

```java
public class AOPTest {

    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        IAccountService as = (IAccountService)ac.getBean("accountService");
        //3.执行方法
        as.saveAccount();

    }
}
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\14.png)

### 四、基于注解的AOP配置细节

可以看出使用注解时，需要配置的地方是切面类，也就是Logger类，不过需要注意的是，要把spring的ioc容器的注解也给加上。

- @Aspect：表示当前类是一个切面类
- @Before：前置通知
- @AfterReturning：后置通知
- @AfterThrowing：异常通知
- @After：最终通知

在注解的配置中也有类型pointcut-ref的用法：

- @Pointcut：配置切入点表达式，可以用于重用

```java
/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
@Component("logger")
@Aspect
public class Logger {
    @Pointcut("execution(* com.SpringDemo3.service.impl..*(..))")
    private void pt1(){}
    /**
     * 前置通知
     */
    @Before("pt1()")
    public void beforePrintLog(){

        System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
    }

    /**
     * 后置通知
     */
    @AfterReturning("pt1()")
    public void afterReturningPrintLog(){

        System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志了。。。");
    }
    /**
     * 异常通知
     */
    @AfterThrowing("pt1()")
    public void afterThrowingPrintLog() {
        System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志了。。。");
    }

    /**
     * 最终通知
     */
    @After("pt1()")
    public void afterPrintLog(){
        System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志了。。。");
    }
}
```

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring\15.png)

==环绕通知==

- Around

```java
 /**
* 环绕通知
*/ 
@Around("pt1()")
    public void arroundPrintLog(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        try {
            this.beforePrintLog();
            pjp.proceed(args);
            this.afterReturningPrintLog();
        } catch (Throwable throwable) {
            this.afterThrowingPrintLog();
            throwable.printStackTrace();
        } finally {
            this.afterPrintLog();
        }

    }
```

