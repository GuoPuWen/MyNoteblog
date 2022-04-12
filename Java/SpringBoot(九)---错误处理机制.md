### 一、默认处理机制

##### 1.浏览器

当浏览器发送错误请求时，SpringBoot会有一个默认的错误处理，例如：

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\32.png)

SpringBoot会返回一个错误页面，这个错误页面里有状态码和错误信息

##### 

##### 2.postman

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\33.png)

会返回json数据

### 二、默认错误处理机制代码分析

错误处理机制的自动配置都在ErrorMvcAutoConfiguration这个类里，这个



##### 1.ErrorPageCustomizer

ErrorMvcAutoConfiguration添加了ErrorPageCustomizer这个组件

```java
@Bean
public ErrorPageCustomizer errorPageCustomizer() {
    return new ErrorPageCustomizer(this.serverProperties);
}
```

ErrorPageCustomizer这个组件是添加错误映射

```java
private static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {

    private final ServerProperties properties;

    protected ErrorPageCustomizer(ServerProperties properties) {
        this.properties = properties;
    }

    @Override
    public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
        ErrorPage errorPage = new ErrorPage(this.properties.getServletPrefix()
                                            + this.properties.getError().getPath());
        errorPageRegistry.addErrorPages(errorPage);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

那么一但发生错误式，会去到那个页面呢，就在getPath()方法里找答案

```java
	/**
	 * Path of the error controller.
	 */
	@Value("${error.path:/error}")
	private String path = "/error";
```

也就是说，如果浏览器一但发送错误请求，会发送/error请求



##### 2.BasicErrorController

当服务器收到/error请求时，来到BasicErrorController来处理/error请求

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
```

很显然，这个Controller就是专门用来处理/error请求的，根据一开始。当使用不同的客户端时，响应的数据格式不一样，那么具体代码就在这个Controller中

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}
```

当使用浏览器发生错误页面时：

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)

public static final String TEXT_HTML_VALUE = "text/html";
```

浏览器的请求头带有test/html，进入第一个RequestMapping，返回ModelAndView

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\34.png)

当使用其他客户端时，进入另外一个RequestMapping中处理请求，返回键值对集合，也就是json数据。



##### 3.DefaultErrorViewResolver

错误页面的Controller是带有text/html的请求头的请求

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        //生成错误页面
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}
```

实现resolveErrorView的具体实现类是DefaultErrorViewResolver，打开这个类

```java
	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot去找一个页面 error/状态码
		String errorViewName = "error/" + viewName;
        //如果模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
				this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        ////模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面 error/状态码.html
		return resolveResource(errorViewName, model);
	}
```

到这里就可以得出定制错误页面的方法，可以在静态资源文件夹下创建error/状态码.html或者在template文件夹下创建error/状态码.html



##### 4.DefaultErrorAttributes

在BasicErrorController调用了getErrorAttributes方法，而这个方法的具体实现是DefaultErrorAttributes中的，作用就是==共享错误信息==

```java
@Override
	public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, webRequest);
		addErrorDetails(errorAttributes, webRequest, includeStackTrace);
		addPath(errorAttributes, webRequest);
		return errorAttributes;
	}
```



##### 5.总结



==当浏览器（或者其他客户端）发送了错误请求时，会来到/error请求，这个/error请求映射是在ErrorPageCustomizer这个组件中完成的，BasicErrorController收到/error请求开始处理这个请求，然后根据请求头来判断是浏览器还是其他客户端，如果是浏览器，那么返回ModelAndView；如果是其他客户端返回json数据==

==DefaultErrorViewResolver这个组件便是生成错误页面的，返回一个ModelAndView对象，而这些视图对象中有一些共享的数据，例如timestamp，status等等，DefaultErrorAttributes完成了这些共享数据的解析。==



### 三、定制错误页面

DefaultErrorViewResolver的resolve方法告诉我们如何去定制自己的错误页面

```java
private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot去找一个页面 error/状态码
		String errorViewName = "error/" + viewName;
        //如果模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
				this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        ////模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面 error/状态码.html
		return resolveResource(errorViewName, model);
	}
```

##### 1.使用模板引擎

​	有模板引擎的情况下，将错误页面放在templates（thymeleaf的默认解析文件夹）的error文件夹下并命名为 ==状态码.html==，也可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的错误

DefaultErrorAttribute共享了一些错误信息，意味着我们可以在模板引擎中使用这些信息，

- timestamp：时间戳 
- status：状态码 
- error：错误提示 
- exception：异常对象 
- message：异常消息 
- errors：JSR303数据校验的错误都在这里 

这些错误信息在DefaultErrorAttributes都能找到

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\35.png)



##### 2.不使用模板引擎

如果不使用模板引擎，那么DefaultErrorAttributes中的信息无法使用。方法还是一样的，在静态资源文件夹下找error/状态码.html



##### 3.都不用

如果都不用，也就是返回的ModelAndView为空

```java
	@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}
```

那么，默认去找"error"这个视图对象，在ErrorMvcAutoConfiguration这个类中加入了这个组件

```java
private final StaticView defaultErrorView = new StaticView();

@Bean(name = "error")
@ConditionalOnMissingBean(name = "error")
public View defaultErrorView() {
    return this.defaultErrorView;
}
```

```java
private static class StaticView implements View {

		private static final MediaType TEXT_HTML_UTF8 = new MediaType("text", "html", StandardCharsets.UTF_8);

		private static final Log logger = LogFactory.getLog(StaticView.class);

		@Override
		public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
				throws Exception {
			if (response.isCommitted()) {
				String message = getMessage(model);
				logger.error(message);
				return;
			}
			response.setContentType(TEXT_HTML_UTF8.toString());
			StringBuilder builder = new StringBuilder();
			Date timestamp = (Date) model.get("timestamp");
			Object message = model.get("message");
			Object trace = model.get("trace");
			if (response.getContentType() == null) {
				response.setContentType(getContentType());
			}
			builder.append("<html><body><h1>Whitelabel Error Page</h1>").append(
					"<p>This application has no explicit mapping for /error, so you are seeing this as a fallback.</p>")
					.append("<div id='created'>").append(timestamp).append("</div>")
					.append("<div>There was an unexpected error (type=").append(htmlEscape(model.get("error")))
					.append(", status=").append(htmlEscape(model.get("status"))).append(").</div>");
			if (message != null) {
				builder.append("<div>").append(htmlEscape(message)).append("</div>");
			}
			if (trace != null) {
				builder.append("<div style='white-space:pre-wrap;'>").append(htmlEscape(trace)).append("</div>");
			}
			builder.append("</body></html>");
			response.getWriter().append(builder.toString());
		}
```

那么错误页面的默认显示就完成了

