# 一、AOP环境搭建

首先复习一下AOP，AOP是什么？AOP是一种编程思想，是面向对象编码的一种补充，AOP是面向切面编程，面向对象编程将程序抽象成各个层次的对象，而面向切面编程将程序抽象成各个切面。AOP值在程序运行期间动态的将某段代码切入到指定方法、指定位置进行运行的编程方式，它的底层离不开动态代理

AOP中的一些重要术语：

- 连接点（JoinPooint）：程序执行的某个特定位置，如类的开始初始化前，类的初始化后，类的某个方法调用前/后，方法抛出异常后。一个类或者一段代码拥有一些边界性质的特定点，这就是连接点。Spring仅支持方法的连接点，即仅能在方法调用前，方法调用后，方法抛出异常时及方法调用前后这些程序执行点织入增强
- 切点（Pointcut）：每个程序类都拥有多个连接点，在多个连接点中定位我们想要的连接点。AOP通过切点定位连接点，切点和连接点不是一对一的关系，一个切点可以匹配多个连接点
- 增强（Advice）：增强是织入到目标类连接点上的一段程序代码，Spring中增强除了用于描述一段程序代码外，还拥有另一个和连接点相关的信息，执行点的方位。Spring 提供的接口都是带方位名的：BeforeAdvice、AfterReturningAdvice，等

## 1.1 导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>
```

使用Spring的AOP功能导入的是spring-aspects这个jar包，但是在此之前肯定也要导入Spring的jar

## 1.2 编写业务类

MathCalculator

```java
public class MathCalculator {
    public int div(int i, int j) {
        System.out.println("MathCalculator...div...");
        return i / j;
    }
}
```

本次AOP的使用是对这个业务类进行切点织入，进行一个日志的扫描，现在，我们希望在以上这个业务逻辑类中的除法运算之前，记录一下日志，例如记录一下哪个方法运行了，用的参数是什么，运行结束之后它的返回值又是什么，顺便可以将其打印出来，还有如果运行出异常了，那么就捕获一下异常信息。

或者，你会有这样一个需求，即希望在业务逻辑运行的时候将日志进行打印，而且是在方法运行之前、方法运行结束、方法出现异常等等位置，都希望会有日志打印出来。

## 1.3 定义切面类

AOP中的通知方法及其对应的注解与含义如下：

- 前置通知（对应的注解是@Before）：在目标方法运行之前运行
- 后置通知（对应的注解是@After）：在目标方法运行结束之后运行，无论目标方法是正常结束还是异常结束都会执行
- 返回通知（对应的注解是@AfterReturning）：在目标方法正常返回之后运行
- 异常通知（对应的注解是@AfterThrowing）：在目标方法运行出现异常之后运行
- 环绕通知（对应的注解是@Around）：动态代理，我们可以直接手动推进目标方法运行（joinPoint.procced()）
  

```java
@Aspect	//这个注解不能少，告诉Spring
public class LogAspects {


    @Pointcut("execution(public int cn.noteblogs.aop.MathCalculator.*(..))")
    public void pointCut(){}

    // @Before：在目标方法（即div方法）运行之前切入
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs(); // 拿到参数列表，即目标方法运行需要的参数列表
        System.out.println(joinPoint.getSignature().getName() + "运行......@Before，参数列表是：{" + Arrays.asList(args) + "}");

    }

    // 在目标方法（即div方法）结束时被调用
    @After("pointCut()")
    public void logEnd() {
        System.out.println("除法结束......@After");
    }

    // 在目标方法（即div方法）正常返回了，有返回值，被调用
    @AfterReturning(value = "pointCut()", returning = "result")
    public void logReturn(JoinPoint joinPoint,Object result) {
        System.out.println(joinPoint.getSignature().getName() + "正常返回......@AfterReturning，运行结果是：{" + result + "}");
    }

    // 在目标方法（即div方法）出现异常，被调用
    @AfterThrowing("pointCut()")
    public void logException() {
        System.out.println("除法出现异常......异常信息：{}");
    }

}
```

## 1.4 加入容器中

切记要将业务类和切面类都加入Spring的容器中，这样Spring才能使用动态代理为我们切入相关的日志代码,还需要注意的是EnableAspectJAutoProxy注解表示开启Spring的AOP功能，在Spring中这种注解非常多，表示开启xxx功能，但是开启功能之后还要告诉该功能的实现类，比如上面的@Aspect注解

```java
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {

    // 将业务逻辑类（目标方法所在类）加入到容器中
    @Bean
    public MathCalculator calculator() {
        return new MathCalculator();
    }

    // 将切面类加入到容器中
    @Bean
    public LogAspects logAspects() {
        return new LogAspects();
    }

}
```

## 1.5 测试

```java
public class IOCTest_AOP {

    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);

        // 不要自己创建这个对象
        // MathCalculator mathCalculator = new MathCalculator();
        // mathCalculator.div(1, 1);

        // 我们要使用Spring容器中的组件
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1, 1);

        // 关闭容器
        applicationContext.close();
    }

}
```