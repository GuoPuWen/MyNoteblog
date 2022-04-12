### 一、概述

我们知道SpringBoot有很多自动配置，但是有时候我们会根据需要去修改这个自动配置，例如修改tomcat的端口等等，所以就需要配置文件来定制我们的配置，在使用SpringBoot的向导创建的时候，自动给我们创建了一个文件resources/application.properties

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\7.png)

==SpringBoot采用一个全局的配置文件，这个配置文件名是固定的==

- application.properties
- application.yml

使用这两个之中的一个即可，后面会介绍优先级问题。

### 二、Properties语法

Properties语法规则我们应该很熟悉，在很多地方的有应用，他的基本规则介绍key=value

### 三、YMAL语法

##### 1.简介

- yaml语法用k(空格):v来表示一个键值对(空格是必须的)；
- 通过空格的方式来控制层级关系，空格数量多少并没有严格规定，只需要左对齐的一列数据这都被认为是同一个层级的
- 区分大小写

```yaml
server:
  port: 8082
```

##### 2.字面量的写法

字符串，布尔类型，数值，日期这些类型直接以k(空格):v形式来写，但是要注意字符串默认不用加双引号，如果加上了引号，双引号和单引号是有区别的

- 双引号：不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
- 单引号：

##### 3.对象的写法

```yaml
friends:
  lastName: zhangsan
  age: 20
```

注意这里的缩进，因为ymal是通过缩进来确定层级关系的，还可以写成一行的形式

```yaml
friends2: {lastName: hangsan,age: 18}
```

##### 4.集合(数组)的写法

用-表示数组中的一个元素

```yaml
pets:
  - cat
  - dog
  - pig
```

这里如果写对了的话idea会给我们提示：

![](C:\Users\four and ten\Desktop\笔记\JAVA\Spring Boot\14.png)

也有对应的行内写法

```
pets2: [cat,dog,pig]
```

### 四、配置文件值注入

##### 1.@ConfigurationProperties

配置文件值注入就是在配置文件中将一些值注入到组件中，例如有persion实体类

```java
@Component
@ConfigurationProperties(prefix = "persion")
public class persion {
    private String lastName;
    private Integer age;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

在yml配置文件中：

```yaml
persion:
  lastName: zhangsan
  age: 18
  birth: 2017/12/12
  maps: {k1: v2,k2: v2}
  list:
    - list1
    - list2
  dog:
    name: pet
    age: 12
```

那么，在persion类上使用@ConfigurationProperties注解，SpringBoot会将本类中的所有属性和配置文件中的相关配置进行绑定，测试一下：

```java
@SpringBootTest
class HelloApplicationTests {

    @Autowired
    private persion persion;

    @Test
    void contextLoads() {
        System.out.println(persion);
    }
}
//输出：
persion{lastName='zhangsan', age=18, birth=Tue Dec 12 00:00:00 CST 2017, maps={k1=v2, k2=v2}, lists=null, dog=Dog{name='pet', age=12}}
```

如果导入spring-boot-configuration-processor依赖的话再yml配置文件中会有提示信息

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

[spring-boot-configuration-processor官方文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata)

##### 2.@Value

使用@ConfigurationProperties注解可以将配置文件中相关的属性全部的注入到类i中，而使用@Value属性则是单个属性注入，例如：

```java
@Component
//@ConfigurationProperties(prefix = "persion")
public class persion {

    @Value("张三")
    private String lastName;
    @Value("19")
    private Integer age;
    @Value("${persion.birth}")
    private Date birth;
	private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
	//输出
	//persion{lastName='张三', age=19, birth=Tue Dec 12 00:00:00 CST 2017, maps=null, lists=null, dog=null}
```

在value属性内还可以用表达式的方式来引用配置文件里的内容

##### 3.**@PropertySource**

这个注解的作用是导入外部的配置文件，如果将所有的注入属性配置都放在application配置文件内，那么会显得很冗余，所以可以使用这个注解来导入外部的配置文件

```java
@Component
@ConfigurationProperties(prefix = "persion")
@PropertySource(value = {"classpath:persion.properties"})
public class persion {

    //@Value("张三")
    private String lastName;
    //@Value("19")
    private Integer age;
   // @Value("${persion.birth}")
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog;
```

这里需要注意的是==不能不写ConfigurationProperties注解，如果不写这个注解，那么将不知道去映射那个属性==

##### 4.@ImportResource**

导入Spring的配置文件，让配置文件里面的内容生效，这里还是不推荐，既然SpringBoot就是为了简化Sprin的那种配置繁琐的问题，那么就不用再绕回去使用Spring的配置文件了。

### 五、配置文件加载顺序

Spring Boot启动会扫描以下位置的application配置文件

- file:./confifig/ 
- file:./ 
- classpath:/config/
- classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；并且properties的优先级高于yml。==这4个位置的配置文件时互补配置==。



