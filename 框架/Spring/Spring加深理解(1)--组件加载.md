本文主要介绍Spring常用的注解开发，使用注解来替代xml配置文件，也为后面理解Spring的运行过程打下基础，同时为使用SpringBoot也提供基础。一整个注解驱动开发是参考雷丰阳老师的Spring注解驱动开发课程。Spring的注解分为配置类注解、容器类注解、web相关的注解、Aop相关的注解等等

# 一、配置类注解

## 基于xml配置文件的开发

之前写Spring的工程时，通常会建一个bean.xml配置文件，例如：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="person" class="cn.noteblogs.bean.Person">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="45"></property>
    </bean>

</beans>
```

person实体类，这里使用了lombok插件

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Person {
    private String name;
    private Integer age;
}
```

那么就可以在测试类中使用：

```xml
    @Test
    public void test1(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
        Object person = applicationContext.getBean("person");
        System.out.println(person);
    }
```

相关的pom文件这里也给出

```xml
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.13.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

上面是基于配置文件的开发，下面体验一下基于注解的开发

## 基于注解的开发

基于注解的开发，省去了配置文件，但是需要增加一个配置类，这个配置类就相当于配置文件

config/ApplicationConfig

```java
@Configuration
public class ApplicationConfig {
    @Bean
    public Person person(){
        return new Person("zhang", 45);
    }
}
```

```java
    @Test
    public void test1(){
//        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
//        Object person = applicationContext.getBean("person");
//        System.out.println(person);
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        Object person = applicationContext.getBean("person");
        System.out.println(person);
    }
```

同样也是可以实例化bean

`Configuration`Spring中配置类上的注解，本质上也是一个Component

# 二、容器组件添加注解

## @bean

```xml
@Configuration
public class ApplicationConfig {
    @Bean
    public Person person(){
        return new Person("zhang", 45);
    }
}
```

使用上面的方式可以初始化一个bean容器，但是我们使用者关心的是这个ioc容器里面的key是怎么定义的？

- 默认情况下，方法名为key
- 可以指定@Bean("person01")，那么key为person01

可以使用ApplicationContext #getBeanDefinitionNames获取容器内的key的name

```java
private ApplicationContext applicationContext;
@Before
public void before(){
    applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
}
@Test
public void printIOCName(){
    String[] names = applicationContext.getBeanDefinitionNames();
    for (String name : names) {
        System.out.println(name);
    }
}
```

![image-20210225103508016](http://cdn.noteblogs.cn/image-20210225103508016.png)

## @ComponentScan

ComponentScan可以定义扫描包的规则，与配置文件中的context:component-scan效果一样

```xml
<context:component-scan base-package="cn.noteblogs"></context:component-scan>
```

只需要在配置类上头@ComponentScan注解即可，里面内置的includeFilters属性和excludeFilters属性可以让我们配置扫描一些包和不扫描一些包

## @Scope

@Scope注解能够设置组件的作用域，可以设置如下值：

- ConfigurableBeanFactory#SCOPE_PROTOTYPE 也就是prototype
- ConfigurableBeanFactory#SCOPE_SINGLETON也就是 singleton
- org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST web中的request域
- org.springframework.web.context.WebApplicationContext#SCOPE_SESSION web中的session域

设置上面这些值，都比较好理解，主要是设置不同的值，ioc容器创建的时间问题，例如对于person类设置单例的话，

```java
@Configuration
@ComponentScan(basePackages = "cn.noteblogs")
public class ApplicationConfig {

    @Bean
    @Scope("singleton") // @Scope("prototype")
    public Person person01(){
        return new Person("zhang", 45);
    }
}
```

执行以下代码：

```
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
```

**如果设置为单例的话，则会发现在执行以上代码的时候就会进行IOC容器的初始化，而设置为多例的话则会在需要的时候进行加载，也就是延迟加载**

这里需要注意的是，在多线程环境下，如果多个线程共享一个bean的话则有可能会造成共享变量线程不安全问题

## @Lazy

@Lazy注解可以为某个bean实现懒加载的方式，所谓懒加载就是说在Spring的IOC容器启动的时候不进行bean的初始化，而是在第一次使用的时候进行bean的初始化工作，在@Scope注解里面可以知道，**如果设置为单例的话是饿汉式就是非懒加载的，在容器一被创建的时候就进行初始化工作，而设置为prototype则是懒加载的形式**

```java
@Configuration
@ComponentScan(basePackages = "cn.noteblogs")
public class ApplicationConfig {

    @Bean
    @Lazy
    public Person person(){
        return new Person("zhang", 45);
    }
}
```

测试类

```java
@Test
public void testLazy(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
}
```

可以发现没有打印person的构造方法，可见在bean上添加了@Lazy注解在初始化容器的不会进行这个bean的初始化工作，也就是实现了懒加载

再来测试一下单实例bean多次使用是否是一个对象

```java
@Test
public void testLazy(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(ApplicationConfig.class);
    Object person1 = ac.getBean("person");
    Object person2 = ac.getBean("person");
    System.out.println(person1 == person2);
}
//Person创建
//true
```

可见，@Lazy注解只是规定了单实例bean的加载时机，也就是在使用时加载按需加载，而如果多次使用则还是同一个bean的

## @Condition

@Condition能够根据条件创建bean，源码如下：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition} classes that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```

可见需要传入一个实现了Condition接口的类，那么现在有一个需求：在windows系统下加载Person("win",80)，在linux系统下加载person("linux",100)，那么根据@Condition注解需要的条件，分别写LinuxCondition和WindowsCondition

```java
public class LinuxCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("linux")){
            return true;
        }
        return false;
    }
}

public class WindowsCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

ConditionContext对象里面封装了一些条件上下文对象，根据这个对象可以获得一些条件的信息

![image-20210227091421607](http://cdn.noteblogs.cn/image-20210227091421607.png)

```java
// 1. 获取到bean的创建工厂（能获取到IOC容器使用到的BeanFactory，它就是创建对象以及进行装配的工厂）
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
// 2. 获取到类加载器
ClassLoader classLoader = context.getClassLoader();
// 3. 获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
Environment environment = context.getEnvironment();
// 4. 获取到bean定义的注册类
BeanDefinitionRegistry registry = context.getRegistry();
```

现在有了两个Conditional实现类，那么可以在配置类中进行配置

```java
    @Bean("win10")
    @Conditional({WindowsCondition.class})
    public Person person01(){
        return new Person("win10", 80);
    }

    @Bean("linux")
    @Conditional({LinuxCondition.class})
    public Person person02(){
        return new Person("linux", 100);
    }
```

测试：

```java
@Test
public void testConditional(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(ApplicationConfig.class);
    String[] names = ac.getBeanDefinitionNames();
    for (String name : names) {
        System.out.println(name);
    }
}
```

在本机(win10)环境下测试

![image-20210227091730729](http://cdn.noteblogs.cn/image-20210227091730729.png)

虚拟机可以模拟Linux环境，只需要在虚拟机参数上设置`-Dos.name=linux`，即可

## @Import

Spring向容器中注册组件的方式有：

- 包扫描+给组件标注注解(@Component、@Service、@Conptroller、@Repository)，但是这种方式局限于自己写的类
- @Bean注解，通常导入第三方包的组件
- @Import注解，快速的向Spring容器导入一个组件

```java
//值允许写在类上面
、@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
	Class<?>[] value();

}
```

根据源码上的注释，可以发现Import可以有三种使用方式：

- 直接写入class数组的方式
- ImportSelector的方式，可以批量导入
- ImportBeanDefinitionRegistrar接口方式，手工注册bean到容器中

`直接写入class数组`

新建一个Color的bean，并且在配置类上使用@Iimport注解

```java
public class Color {

}
```

```java
@Configuration
@Import(Color.class)
public class ImportConfig {

}
```

测试类：

```java
@Test
public void testImport(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(ImportConfig.class);
    String[] names = ac.getBeanDefinitionNames();
    for (String name : names) {
        System.out.println(name);
    }
}
```

可以猜测一下使用Import导入组件的key是什么？

![image-20210228084220382](http://cdn.noteblogs.cn/image-20210228084220382.png)

`ImportSelector的方式`

先来看看ImportSelector的源码

![image-20210228085050190](http://cdn.noteblogs.cn/image-20210228085050190.png)

这上面有一句话返回要注册的类名，或者如果为空的话返回一个空的数组，那么返回null会怎么样呢？接下来我们我们编写一个实现了ImportSelector的类，

![image-20210228085245036](http://cdn.noteblogs.cn/image-20210228085245036.png)

运行测试类，发现报了空指针异常，在该处打一断点能发现后来其实是调用了一个数组的length方法，那么如果返回null肯定会报一个空指针异常

正确的使用应该是

```java
public class MyImportSelector implements ImportSelector {
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"cn.noteblogs.bean.Blue","cn.noteblogs.bean.Color"};
    }
}
```

接着运行测试方法，

![image-20210228085927165](http://cdn.noteblogs.cn/image-20210228085927165.png)

`ImportBeanDefinitionRegistrar接口方式`

ImportBeanDefinitionRegistrar接口方式使用方法大致和ImportSelector的方式一致，

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean definition = registry.containsBeanDefinition("cn.noteblogs.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("cn.noteblogs.bean.Blue");
        if (definition && definition2) {
            // 指定bean的定义信息，包括bean的类型、作用域等等
            // RootBeanDefinition是BeanDefinition接口的一个实现类
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class); // bean的定义信息
            // 注册一个bean，并且指定bean的名称
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }

    }
}
```

只需要在配置类上配置MyImportBeanDefinitionRegistrar类即可，上面代码的意图为如果ioc容器中存在red和blue类才创建rainBow类，这点和@Condition注解很相似，可以为组件创建添加一定的条件

## FactoryBean的方式

前面所讲的三种创建bean的方式，都是使用反射来创建bean，Spring也提供了一个接口FactoryBean，用户可以通过这个接口的方式定制化创建bean对象

下面是Spring5.2版本以上的Factorybean接口的方法，可见有三个方法

```java
public interface FactoryBean<T> {

	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    //返回由FactoryBean创建的bean实例，如果isSingleton()返回true，那么该实例会放到Spring容器中单实例缓存池中
	@Nullable
	T getObject() throws Exception;
	//返回由FactoryBean创建的bean实例的作用域是singleton还是prototype
	@Nullable
	Class<?> getObjectType();
	//返回FactoryBean创建的bean实例的类型
	default boolean isSingleton() {
		return true;
	}
}
```

FactoryBean接口在Spring中的地位比较高，其实现接口有70多个，下面我们来尝试使用这个接口

```java
public class ColorFactoryBean implements FactoryBean<Color> {
    public Color getObject() throws Exception {
        System.out.println("Color创建...");
        return new Color();
    }

    public Class<?> getObjectType() {
        return Color.class;
    }
}
```

在配置类中进行如下配置

```java
@Bean
public ColorFactoryBean colorFactoryBean(){
    return new ColorFactoryBean();
}
```

进行测试

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(ApplicationConfig.class);
Object colorFactoryBean = ac.getBean("colorFactoryBean");
System.out.println("colorFactoryBean的类型" + colorFactoryBean.getClass());
String[] names = ac.getBeanDefinitionNames();
for (String name : names) {
    System.out.println(name);
}
```

![image-20210302081708407](http://cdn.noteblogs.cn/image-20210302081708407.png)

可以发现几点：

- 由于向配置类中配置了colorFactoryBean，所以IOC容器中有以colorFactoryBean为key的实例，但是实际类型却是ColorFactoryBean中的getObjectType返回的类型，也就是说创建的是getObject中的返回的对象Color对象
- 使用beanFactory创建的对象的优先级比一般的反射创建对象更高

这下我们知道了，**beanFactory工厂创建的其实是getObject中的返回的对象**，那如果我们就是要获取Factorybean对象呢，可以在获取bean的时候加上一个&符号

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(ApplicationConfig.class);
//获取Factorybean对象
Object colorFactoryBean = ac.getBean("&colorFactoryBean");
System.out.println("colorFactoryBean的类型" + colorFactoryBean.getClass());
String[] names = ac.getBeanDefinitionNames();
for (String name : names) {
    System.out.println(name);
}
```

![image-20210302082455739](http://cdn.noteblogs.cn/image-20210302082455739.png)

可以看到类型就为colorFactoryBean的类型，这个使用的源码在BeanFactory接口里面

![image-20210302082613319](http://cdn.noteblogs.cn/image-20210302082613319.png)

请注意这里是BeanFactory，前面创建bean的工厂接口是FactoryBean，下面来说明两者的区别：

- BeanFactory：BeanFactory 是顶层容器（根容器），不能被实例化，它定义了所有 IoC 容器 必须遵从
  的⼀套原则，比如我们常用的ApplicationContext也是实现该接口

- FactoryBean 是spirng提供的工厂bean的一个接口