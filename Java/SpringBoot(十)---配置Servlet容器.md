1环境：

- jdk14
- idea2020
- SpringBoot2.2.6.RELEASE

### 一、嵌入式的Servlet容器自动配置原理

启动SpringBoot，只要在主程序的main方法里运行即可，那么很显然SpringBoot使用了嵌入式的Tomcat

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\36.png)



嵌入式的Tomcat的自动配置都在EmbeddedWebServerFactoryCustomizerAutoConfiguration这个类下

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication
@EnableConfigurationProperties(ServerProperties.class)
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass({ Tomcat.class, UpgradeProtocol.class })
	public static class TomcatWebServerFactoryCustomizerConfiguration {

		@Bean
		public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(Environment environment,
				ServerProperties serverProperties) {
			return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
		}

	}
```



