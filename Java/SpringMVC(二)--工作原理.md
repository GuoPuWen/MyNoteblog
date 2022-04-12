### 一、回顾

在 [SpringMVC(一)](https://blog.csdn.net/weixin_44706647/article/details/105367408)介绍了SpringMVC的快速入门，简单的回顾一下SpringMVC环境的搭建：

1. 导入相关依赖
2. 在web.xml文件中配置核心的控制器

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
<!--  SpringMVC的核心控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

3. 配置springmvc的配置文件，在配置文件内需要配置视图解析器

```xml
    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.SpringMVCDemo"></context:component-scan>

    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

```

3. 使用SpringMVC

### 二、SpringMVC的工作原理

##### 1.组件

在讲解SpringMVC的工作原理前，需要介绍SpringMVC的各大组件

1. **DispatcherServlet：前端控制器**

用户请求到达前端控制器，它就相当于 mvc 模式中的 c，dispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性。

2. **HandlerMapping：处理器映射器**

HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

3. **Handler：处理器**

它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理

4. **HandlAdapter：处理器适配器**

通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理 器进行执行。 

5. **View Resolver：视图解析器**

View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。 

6. **View：视图**

SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView等。我们最常用的视图就是 jsp。 

##### 2.工作原理

下图是SpringMVC的工作原理图，图来源于网络

![](C:\Users\four and ten\Desktop\笔记\JAVA\SpringMVC\11.png)

1. 当浏览器请求：localhost/hello时，前端控制器dispatcherServlet 接受用户的请求
2. 前端控制器dispatcherServlet 收到请求调用HandlerMapping 处理器映射器
3. HandlerMapping 处理器映射器找到处理器（Handler），并生成处理器对象，返回给前端控制器dispatcherServlet 
4. 前端控制器dispatcherServlet 调用HandlerAdapter处理器适配器。
5. HandlerAdapter处理器适配器调用具体的处理器，返回ModelAndView给前端控制器
6. 前端控制器将ModelAndView交给View Resolver视图解析器，视图解析器解析ModelAndView将View返回给前端控制器
7. 前端控制器将view进行渲染后返回给用户

**从以上流程中可以看出，前端控制器dispatcherServlet是整个SpringMVC工作时的核心组件，它将收到的数据进行分发，分配任务至各大相应组件，组件处理完数据后又将数据返回至dispatcherServlet，最终dispatcherServlet生成view视图展示给用户**

##### 3.配置文件

很显然，当我们需要某个组件时，都需要加载该组件，比如说我们需要视图解析器，那么需要将视图解析器加载至ioc容器中，如果说每一个组件都需要去手动配置时，那是很麻烦的，因为组件有好几个，SpringMVC提供了一个标签

```xml
<mvc:annotation-driven/>
```

只要在springmvc的配置文件内加上了该标签，那么相当于配置好了处理器映射器、处理器适配器、视图解析器，这样就方便的多

