### 一、环境准备

- jdk14
- idea2020
- SpringBoot 2.2.6.RELEASE

使用Springboot的向导功能创建web，自动导入以下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 二、SpringMVC自动加载原理

Springboot中的web使用的是SpringMVC，那么自动配置类WebMvcAutoConfiguration中包含了SpringMVC自动配置的所有组件，查看官方文档，给我们列出来了SpringMVC的自动配置

[SpringMVC自动配置官方文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-developing-web-applications)

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  ==自动配置了ViewResolver（视图解析器）==

- Support for serving static resources, including support for WebJars 

  ==支持静态资源文件夹路径==[SpringBoot对静态资源的映射规则](https://blog.csdn.net/weixin_44706647/article/details/105716849)

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
  ==自动注册了 Converter , GenericConverter , Formatter beans类型转换器==

- Support for `HttpMessageConverters` 
  ==SpringMVC用来转换Http请求和响应的==

- Automatic registration of `MessageCodesResolver` 
  ==定义错误代码生成规则==

- Static `index.html` support
  ==支持静态首页访问==

- Custom `Favicon` support 
  ==支持Favicon.icon图标访问==

- Automatic use of a `ConfigurableWebBindingInitializer` bean 

    ==初始化WebDataBinder；将请求的参数转化为对应的JavaBean，并且会结合上面的类型、格式转换一起使用==

那么，该如何定制自己的SpringMVC呢？官网里也有详细的说明：

> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.

### 三、扩展SpringMVC

##### 1.视图映射

根据官方文档里的第一句话，要配置拦截器，格式化处 理器，视图控制器等，可以添加自己的`WebMvcConfigure`类型 的 `@Configuration` 类，而不需要注解`@EnableWebMvc` 。

那么打开WebMvcConfigurerAdapter

```java
@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
```

打开WebMvcConfigurer：

```java
	/**
	 * Add {@link Converter Converters} and {@link Formatter Formatters} in addition to the ones
	 * registered by default.
	 */
	default void addFormatters(FormatterRegistry registry) {
	}

	/**
	 * Add Spring MVC lifecycle interceptors for pre- and post-processing of
	 * controller method invocations and resource handler requests.
	 * Interceptors can be registered to apply to all requests or be limited
	 * to a subset of URL patterns.
	 */
	default void addInterceptors(InterceptorRegistry registry) {
	}

	/**
	 * Add handlers to serve static resources such as images, js, and, css
	 * files from specific locations under web application root, the classpath,
	 * and others.
	 */
	default void addResourceHandlers(ResourceHandlerRegistry registry) {
	}
	/**
	 * Configure simple automated controllers pre-configured with the response
	 * status code and/or a view to render the response body. This is useful in
	 * cases where there is no need for custom controller logic -- e.g. render a
	 * home page, perform simple site URL redirects, return a 404 status with
	 * HTML content, a 204 with no content, and more.
	 */
	default void addViewControllers(ViewControllerRegistry registry) {
	}
```

那么添加视图映射的方法就是addViewControllers。

添加视图映射的基本步骤“

1. 实现WebMvcConfigurer，重写里面的方法
2. 不需要注解`@EnableWebMvc` 。

```java
/**
 * author by four and ten
 * create by 2020/4/28 9:01
 */
@Configuration
public class myConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("success");
    }
}
```



![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\27.png)

==注意：SpringBoot1.x中是继承WebMvcConfigurerAdapter，在SpringBoot2.x中这种方法已经弃用，使用的是实现WebMvcConfigurer的方法==

##### 2.拦截器

定义拦截器和定义视图映射的方法一样，这里就直接上源码：

```java
/**
 * author by four and ten
 * create by 2020/4/28 9:01
 */
@Configuration
public class myConfig implements WebMvcConfigurer {
    /**
     * 自定义资源映射
     * @param registry
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("success");
    }

    /**
     * 自定义拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HandlerInterceptor() {
            @Override
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
                System.out.println("自定义拦截器");
                return true;
            }

            @Override
            public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

            }

            @Override
            public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

            }
        }).addPathPatterns("/**");
    }
}
```

当浏览器访问localhost:8080/hello时控制台台输出

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\28.png)

### 四、全面接管SpringMVC

全面接管SpringMVC的意思就是不需要SpringBoot对SpringMVC的所有自动配置，而所有的配置都由自己配置，这种方式是不被推荐的。

在官方文档里的描述中，只要加入@EnableWebMvc注解，那么SpringMVC的所有自动配置就失效了，分析以下其中的原理即可。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

接着打开DelegatingWebMvcConfiguration

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

在WebMvcAutoConfiguration中添加了

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)，也就是说如果没有WebMvcConfigurationSupport这个类下面的配置才会生效，而刚好如果导入了EnableWebMvc，则会导入DelegatingWebMvcConfiguration这个类继承了WebMvcConfigurationSupport，所以WebMvcAutoConfiguration这个类失效了，所以所有的自动配置都失效了。

