### Spring Boot对静态资源的映射规则

在SpringMVC的web项目中我们有webapp这个目录来存放各种静态资源资源，js、css、html等等，但是在Spring Boot中使用向导给我们创建文件时，没有自动创建webapp这个文件，说明SpringBoot有自己的静态资源映射规则。

那么这种规则是什么？那肯定在xxxAutoConfiguration类上找答案，web的自动配置都在

WebMvcAutoConfiguration这个类下，而WebMvcAutoConfiguration类上有一个静态内部类WebMvcAutoConfigurationAdapter，也就是webMVC自动配置适配器。

在这个内部类中的addResourceHandlers便是加载静态资源的方法

```java
public class WebMvcAutoConfiguration {
    //...省略
    
    @Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
        //...省略
        @Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}           
    }
}
```

##### 1.webjars

```java
if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
```

从这段代码中可以看出从所有的/web/jars/**请求都会去:/META-INF/resources/webjars/这个目录下找。

- webjars简介：**WebJars是将web前端资源（js，css等）打成jar包文件**，然后借助Maven工具，以jar包形式对web前端资源进行统一依赖管理，保证这些Web资源版本唯一性。WebJars的jar包部署在Maven中央仓库上。

可以在webjars的官网上来查看常用的静态资源对于的maven坐标：[webjars](https://www.webjars.org/)

例如导入jquery：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.0</version>
</dependency>
```

打开jquery的jar包

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\21.png)

可以访问：localhost:8080/webjars/jquery/3.5.0/jquery.js来验证静态资源是否映射成功！

##### 2./**访问规则

在addResourceHandlers方法内还有一个和webjars映射规则代码类似的情况

```java
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
```

顺着代码一直打开staticPathPattern，在WebMvcProperties类中发现

```java
	/**
	 * Path pattern used for static resources.
	 */
	private String staticPathPattern = "/**";
```

同样的方法，打开

```java
getResourceLocations(this.resourceProperties.getStaticLocations())
```

发现在ResourceProperties类中

```java
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };
	
	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

==那么可以得出结论，当用“/**”去访问资源时，Spring boot会自动的从以下目录去找资源==

```xml
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
```

在maven项目中，java和resources文件下的均被成为类路径下。也就是：

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\22.png)

==如果说这4个文件下有相同的静态资源的话，优先级是/META-INF/resources/ > /resources/ > /static/ > /public/==

##### 3.index.html页面

在上述的4个资源目录下，当文件名为index.html，当浏览器访问localhost:8080会自动跳转到该页面

例如：classpath:/static/index.html

```java
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
spring boot 首页
</body>
</html>
```

当浏览器访问：localhost:8080则默认跳转到该页面

##### 4.定义图标

在SpringBoot中，可以把ico格式的图标放在默认静态资源文件路径下，并以favicon.ico命名，应用图标会自动变成指定的图标。所有的 **/favicon.ico 都会在静态资源文件下找。

##### 5.自定义资源访问目录

在第2条规则里，发现ResourceProperties类里面有一个staticLocations，那么就可以在配置文件内修改这个属性，来重新定义我们自己的文件访问目录，但是要注意，如果是自己配置的话，那么SpringBoot的自动配置将会失效

```java
spring.resources.static-locations=classpath:/myConfig
```

