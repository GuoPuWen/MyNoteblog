# 一、初始化和销毁

bean的生命周期一般是指bean从初始化，经过一系列的流程到销毁的过程，使用Spring框架，bean的生门周期全部由Spring管理，当然我们可以通过指定方法在bean的生命周期中进行我们逻辑的编写，会自动调用我们的方法

## 1.1 使用@Bean

使用xml配置bean实例的时候，可以通过指定init-method和destory-method方法例如：

```xml
<bean id="person" class="cn.noteblogs.bean.Person" init-method="init" destroy-method="destroy">
    <property name="name" value="zhangsan"></property>
    <property name="age" value="45"></property>
</bean>
```

init()和destroy()方法是我们在Person类中定义的方法，使用注解也同样可以实现bean的初始化和销毁方法

新建一个car类

```java
public class Car {

    public Car() {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car ... init...");
    }

    public void destroy() {
        System.out.println("car ... destroy...");
    }
}
```

新建个config/MainConfigOfLifeCycle

```java
@Configuration
public class MainConfigOfLifeCycle {

    //添加初始化和销毁方法
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car(){
        return new Car();
    }
}
```

进行测试

```java
@Test
public void testLifeCycle(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
}
```

![image-20210302084100035](http://cdn.noteblogs.cn/image-20210302084100035.png)

可见在@Bean注解上指定initMethod和destroyMethod即可执行对应的初始化和销毁方法，但是查看结果并没有运行销毁方法的代码，是因为我们没有手动关闭容器

```java
@Test
public void testLifeCycle(){
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
    ac.close();
}
```

![image-20210302084414466](http://cdn.noteblogs.cn/image-20210302084414466.png)

那么现在就执行了car对象中的我们自定义的初始化和销毁方法。

对于bean的初始化和销毁工作，最常用的便是数据源的创建和销毁，可以在初始化的时候进行数据源的创建，在销毁阶段进行数据源的关闭

## 1.2 调用的时机

bean对象的初始化方法和销毁方法调用的时机，要分两种情况对待，下面先给出结论，后面给出代码进行验证：

- 单实例bean：当对象创建完成后，并且对象的属性都已经赋值好了之后，会调用bean的初始化方法，对于单实例bean，在Spring容器创建完成后，Spring容器会自动调用bean的初始化方法；在容器关闭的时候，会调用bean的销毁方法
- 多实例bean：对多实例bean，只有在每次获取bean对象的时候，才进行bean的初始化方法；而销毁方法则Spring不会调用，也就是说Spring不会管理这个bean

单实例的测试上面已经测试过了，下面测试一下多实例，在MainConfigOfLifeCycle上配置car组件处加上@Scope("prototype")注解，

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car(){
        return new Car();
    }
}
```

进行测试

```java
@Test
public void testLifeCycle(){
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
    ac.close();
}
```

运行结果，发现没有任何输出，也就是说对于多实例bean只有在使用的时候才会创建对象，那么推理也只有在使用的时候才会调用init方法，重新测试

```java
    @Test
    public void testLifeCycle(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        Object car = ac.getBean("car");
        ac.close();
    }
```

![image-20210302094418756](http://cdn.noteblogs.cn/image-20210302094418756.png)

在使用的时候调用构造器的方法和init方法，但是始终没有调用destroy方法，也验证了前面的结论：Spring不会去管理多实例bean对象，所以也不会调用销毁方法

## 1.3 InitializingBean和DisposableBean

在1.2章节中使用的是在@Bean中指定initMethod和destroyMethod属性来执行初始化方法和销毁方法，在Spring中还提供了InitializingBean接口和DisposableBean接口来指定初始化方法和销毁方法

`InitializingBean`

Spring中提供了一个InitializingBean接口，该接口为bean提供了属性初始化后的处理方法，它只包括afterPropertiesSet方法，凡是继承该接口的类，在bean的属性初始化后都会执行该方法

```java
public interface InitializingBean {
   void afterPropertiesSet() throws Exception;
}
```

`DisposableBean`

```java
public interface DisposableBean {
	void destroy() throws Exception;
}
```

下面来使用一下种方式

```java
public class Car implements InitializingBean, DisposableBean {

    public Car() {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car ... init_Bean...");
    }

    public void destroy_Bean() {
        System.out.println("car ... destroy_Bean...");
    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("car...init_InitializingBean");
    }

    public void destroy() throws Exception {
        System.out.println("car...destroy_DisposableBean");
    }
}
```

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init", destroyMethod = "destroy_Bean")
    public Car car(){
        return new Car();
    }
}
```

测试方法

```java
@Test
public void testLifeCycle(){
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
    ac.close();
}
```

![image-20210302100019015](http://cdn.noteblogs.cn/image-20210302100019015.png)

根据输出可以看出这两种定义初始化和销毁的方法的调用顺序是：

![image-20210302100642435](http://cdn.noteblogs.cn/image-20210302100642435.png)

# 二、BeanPostProcessor

BeanPostProcessor后置处理器，BeanPostProcessor有两个方法

```java
public interface BeanPostProcessor {

	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

根据这两个方法名称可以初步判断这两个方法是在Spring容器初始化的前后进行，所以Spring容器中的每一个bean对象初始化前后，都会进行BeanPostProcessor的这两个方法

**说明白点，postProcessBeforeInitialization方法会在bean实例化和属性设置好之后，自定义初始化方法之前调用，而postProcessAfterInitialization会在自定义初始化方法之后调用，当容器中存在多个BeanPostProcessor的实现类时，会按照它们在容器中注册的顺序执行。对于自定义的BeanPostProcessor实现类，还可以让其实现Ordered接口自定义排序**



```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

}
```

在配置类中配置该组件

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init", destroyMethod = "destroy_Bean")
    public Car car(){
        return new Car();
    }

    @Bean
    public MyBeanPostProcessor myBeanPostProcessor(){
        return new MyBeanPostProcessor();
    }
}
```

测试类

```java
    @Test
    public void testLifeCycle(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        ac.close();
    }
```

![image-20210302101643275](http://cdn.noteblogs.cn/image-20210302101643275.png)

# 三、总结

上述的自定义初始化方法以及后置处理器，都是在bean已经实例化好了之后进行的，而Spring中提供了许多的扩展接口用于实例化前后，设置属性前后调用，这些扩展接口在AOP、自动注入方面起了很大的作用，但是这里不再详细的描述，后续会有一篇Spring的容器创建文章来具体描述这些接口

那么对于·上面接口方法的调用时机可以通过一图进行总结

![image-20210303084348593](http://cdn.noteblogs.cn/image-20210303084348593.png)