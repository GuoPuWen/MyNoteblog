前面在application配置文件中设置过了一个端口的配置：

```java
server.port=8081
```

那么我们在配置文件里究竟怎么写配置才能生效，这些配置是如何生效的，如果我们学会了Spring boot的自动配置原理，那么自己去定制这些配置也就变得简单了。

当然这些配置的内容在[Spring Boot的官方文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html)里都有提到，可以直接查找这些配置信息

### 一、复习@EnableAutoConfiguration

@EnableAutoConfiguration这个注解是给容器中自动添加一些组件的，这个注解里有@Import(AutoConfigurationImportSelector.class)这个注解，AutoConfigurationImportSelector中的getAutoConfigurationEntry方法返回一个list集合configurations，这个集合里面装载了所有关于这个场景的自动配置包，但是哲学配置包在哪找呢？在getAutoConfigurationEntry方法里调用了SpringFactoriesLoader.loadFactoryNames方法，这个方法将META-INF/spring.factories这个配置文件中提到的jar全部导入至list集合中，这样自动配置就生效了。

也就是说==Spring Boot的自动配置导入的jar包是在META-INF/spring.factories这个文件中的==，打开这个文件，发现都是xxxAutoConfiguration，这些都是自动配置类。

spring-boot-autoconfigure-2.2.6.RELEASE.jar\META-INF\spring.factories（节选）

```java
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.sprin
```

### 二、**HttpEncodingAutoConfifiguration**

Spring Boot的自动配置原理大致相同，所以就以HttpEncodingAutoConfifiguration为例探究Spring boot的自动配置

打开这个HttpEncodingAutoConfifiguration，可以spring.factories这个文件内查找到

HttpEncodingAutoConfifiguration然后直接打开，也可以CTRL+N（idea）打开

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(HttpProperties.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {
    
	private final HttpProperties.Encoding properties;

	public HttpEncodingAutoConfiguration(HttpProperties properties) {
		this.properties = properties.getEncoding();
	}

	@Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}

	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}

	private static class LocaleCharsetMappingsCustomizer
			implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {

		private final HttpProperties.Encoding properties;

		LocaleCharsetMappingsCustomizer(HttpProperties.Encoding properties) {
			this.properties = properties;
		}

		@Override
		public void customize(ConfigurableServletWebServerFactory factory) {
			if (this.properties.getMapping() != null) {
				factory.setLocaleCharsetMappings(this.properties.getMapping());
			}
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}

}
```

##### 1.@Configuration

这是我们很熟悉的注解了，其中proxyBeanMethods = false说明配置类不会被代理了。主要是为了提高性能。

##### 2.@EnableConfigurationProperties

启动指定类的ConfigurationProperties，将配置文件中的值与HttpProperties绑定起来，并把HttpProperties加入到ioc容器中，比如我们在配置文件中配置

```properties
spring.http.encoding.charset=utf-8
```

那么点开这个配置(CTRL+N)发现idea给我们自动跳转道到的类便是HttpProperties.class，在这个类中可以看到一些属性便是和配置文件里能配置的值绑定起来。也和[Spring Boot的官方文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html)规定的配置是一样的。



##### 3.ConditionalOnWebApplication

打开这个注解

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnWebApplicationCondition.class)
public @interface ConditionalOnWebApplication {
```

@Conditional是Spring的底层注解，根据不同的条件，如果满足指定的条件，整个配置类就会生效，而在这个例子中，则是判断当前应用是否是web应用，如果不是web应用则整个配置都不会生效



##### 4.@ConditionalOnClass

@ConditionalOnClass(CharacterEncodingFilter.class)判断当前ioc容器中是否有CharacterEncodingFilter这个组件，这个组件是SpringMVC解决请求参数乱码问题的过滤器。

在HttpEncodingAutoConfifiguration这个类中可以看到，SpringBoot为我们加入了这个组件

```java
	@Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

- **ConditionalOnMissingBean：当前容器中没有这个组件，才使用Spring boot的自动配置，也就是说有这个注解的存在，那么我们只需要将我们自己的CharacterEncodingFilter加入到容器中，那么就覆盖了Spring Boot自动给我们配置的组件**



##### 5.@ConditionalOnProperty

判断配置文件中是否存在某个配置，也就是说这句话的意识是查看配置文件中是否有前缀为spring.http.encoding的配置， spring.http.encoding.enabled，matchIfMissing属性的意思是默认，也就是如果没有在配置文件中配置 spring.http.encoding.enabled的值，那么默认就是true



##### 6.总结

1. Spring Boot会加载大量的自动配置类
2. 这些配置类在META-INF/spring.factories这个文件中，可以查看Spring Boot的自动配置中有那些以及写好的配置组件
3. 给容器中自动配置添加组件时，会从xxxProperties类中获取某些属性，就可以在配置文件中指定这些属性的值

### 三、@Conditional派生注解

在分析HttpEncodingAutoConfifiguration的时候会发现有很多的注解以@ConditionalOnxxx的形式，这些注解都有哪些功能呢

- @ConditionalOnJava ：系统的的java版本是否符合要求 ；
- @ConditionalOnBean：容器中存在指定Bean； 
- @ConditionalOnMissingBean：容器中不存在指定Bean； 
- @ConditionalOnExpression：满足SpEL表达式指定 ；
- @ConditionalOnClass：系统中有指定的类 ；
- @ConditionalOnMissingClass：系统中没有指定的类 ；
- @ConditionalOnSingleCandidate：容器中只有一个指定的Bean，或者这个Bean是首选Bean
- @ConditionalOnProperty ：系统中指定的属性是否有指定的值 
- @ConditionalOnResource ：类路径下是否存在指定资源文件 
- @ConditionalOnWebApplication ：判断当前是否web环境
-  @ConditionalOnNotWebApplication ：判断当前不是web环境 
- @ConditionalOnJndi JNDI存在指定项 

