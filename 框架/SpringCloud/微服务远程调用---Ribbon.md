# 一、简介

Ribbon是一款服务调用的产品，是Netflix发布的云中间层服务开源项目，其主要功能是提供客户端实现负载均衡算法。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，Ribbon是一个客户端负载均衡器，我们可以在配置文件中Load Balancer后面的所有机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器，我们也很容易使用Ribbon实现自定义的负载均衡算法

- 负载均衡：将用户的请求平摊的分配到多个服务上，从而达到系统的高可用(HA)，常见的负载均衡器软件有Nginx

这里还需要注意的是Nginx负载均衡和Ribbon负载均衡之间的不同：Nginx是服务器之间的负载均衡，客户端将所有请求提交到Nginx上，然后由nginx实现转发请求，而Ribbon是本地负载均衡，在调用微服务接口的时候，会在注册中心上获取注册信息·服务列表之后缓存在本地JVM，从而在本地实现RPC远程调用技术

下图展示了Eureka使用Ribbon时的大致架构： 


![img](http://cdn.noteblogs.cn/20180401083228491)

上图告诉我们两点信息：

- 服务提供者需要注册在EurekaServer上或者其他注册中心上
- 当服务消费者需要调用某个服务时，EurekaServer会告诉Ribbon调用的服务提供者的地址

# 二、使用Ribbon

要使用某一款产品，自然要先导入对应的pom依赖坐标

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

但是由于在前面搭建过Eureka集群环境，强烈建议读者先看这篇文章，因为环境都是基于这篇文章搭建的[微服务注册中心--Eureka](https://blog.csdn.net/weixin_44706647/article/details/114142044?spm=1001.2014.3001.5501)

在Eurka的jar依赖里面已经天生的带着ribbon的依赖，所以可以不再倒导入坐标依赖

![image-20210302184849154](http://cdn.noteblogs.cn/image-20210302184849154.png)

其实在Eureka的搭建过程中已经使用过了Ribbon的远程调用，就是RestTemplate调用，但是这只是简单使用，没有使用到负载均衡，因为服务的提供者只有一个，那么在上述Eureka集群环境的基础上，搭建Ribbon负载均衡

`微服务提供者`

cloud-provider-payment8001

controller/PaymentController

```java
@RestController
@Slf4j
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @Value("${server.port}")
    private String port;

    @PostMapping("/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        return result > 0 ? new CommonResult(200,"插入数据成功",result) : new CommonResult(444,"插入数据库失败");
    }

    @GetMapping("/payment/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.findById(id);
        return payment != null ? new CommonResult(200, "查询成功" + "端口" + port, payment) : new CommonResult(444,"没有记录");
    }

}
```

dao/PaymentDao

```java
@Mapper
public interface PaymentDao {
    int create(Payment payment);
    Payment findById(@Param("id") Long id);
}
```

mapper/PaymentMapper.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.noteblogs.springcloud.dao.PaymentDao">
    <select id="findById" parameterType="Long" resultType="cn.noteblogs.entities.Payment">
        select * from payment where id = #{id}
    </select>

    <insert id="create" parameterType="cn.noteblogs.entities.Payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values(#{serial})
    </insert>
</mapper>
```

service/impl/PaymentServiceImpl

```java
@Service
public class PaymentServiceImpl implements PaymentService {
    @Autowired
    private PaymentDao paymentDao;

    @Override
    public int create(Payment payment) {
        return paymentDao.create(payment);
    }

    @Override
    public Payment findById(Long id) {
        return paymentDao.findById(id);
    }
}
```

PaymentMain8001

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}
```

application.yml

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cloud2020?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 213213
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.noteblogs.springcloud.entities  # 所有Entity 别名类所在包

eureka:
  client:
    # 注册进 Eureka 的服务中心
    register-with-eureka: true
    # 检索 服务中心 的其它服务
    fetch-registry: true
    service-url:
      # 设置与 Eureka Server 交互的地址
      defaultZone: http://eureka6001.com:6001/eureka/,http://eureka6002.com:6002/eureka/
```

cloud-provider-payment8002与上述一致，只需更改server.port端口号

`服务消费端`

cloud-customer-order80

controller/OrderController

```java
@RestController
public class OrderController {
    public static final String URL = "http://CLOUD-PAYMENT-SERVICE";

    @Autowired
    private RestTemplate restTemplate;



    @GetMapping("/consume/payment/{id}")
    public CommonResult get(@PathVariable Long id){
        return restTemplate.getForObject(URL + "/payment/" + id, CommonResult.class);
    }

    @GetMapping("/consume/payment/create")
    public CommonResult create(Payment payment){
        return restTemplate.postForObject(URL + "/payment/create/" , payment, CommonResult.class);
    }
}
```

config/ApplicationContextConfig，注意这个注解@LoadBalanced必不可少，到后面会介绍负载均衡算法

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

启动五个微服务：

- cloud-eureka-server6001
- cloud-eureka-server6002
- cloud-provider-payment8001
- cloud-provider-payment8002
- cloud-customer-order80

浏览器访问页面：localhost/consume/payment/1，并且进行多次点击，发现使用的端口号不一样，也就是说Ribbon调用了负载均衡的功能，那么简单的使用Ribbon作为负载均衡的简单环境就搭建好了

![image-20210303154523000](http://cdn.noteblogs.cn/image-20210303154523000.png)

# 三、RestTemplate的使用

这块API[的使用可以直接查看文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)

简要的说明其方法：

- getForObject/getForEntity方法
- postForObject/postForEntity方法

getForObject：返回对象为响应体数据转化成的对象，基本上可以理解为Json对象

getForEntity：返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

```java
@GetMapping("/consume/payment/{id}")
public CommonResult get(@PathVariable Long id){
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(URL + "/payment/" + id, CommonResult.class);
    if(entity.getStatusCode().is2xxSuccessful()){
        return entity.getBody();
    }else{
        return new CommonResult(444, "操作失败");
    }
}
```

# 四、负载均衡算法问题

可以看到在环境搭建的过程中标注了注解@LoadBlance这个注解标定使用负载均衡，然后默认的使用轮询这种负载均衡算法，Ribbon中有一个组件IRule，IRule表示根据特定的算法从服务列表中选取一个要访问的服务，IRule是一个接口

```java
public interface IRule{
    /*
     * choose one alive server from lb.allServers or
     * lb.upServers according to key
     * 
     * @return choosen Server object. NULL is returned if none
     *  server is available 
     */

    public Server choose(Object key);
    
    public void setLoadBalancer(ILoadBalancer lb);
    
    public ILoadBalancer getLoadBalancer();    
}
```

![image-20210303161615733](http://cdn.noteblogs.cn/image-20210303161615733.png)

以下简要说明7种主要的负载均衡算法，这些负载均衡算法均是抽象类com.netflix.loadbalancer.AbstractLoadBalancerRule的实现，而给抽象类实现了IRule接口：

- com.netflix.loadbalancer.RoundRobinRule：轮询，为默认的负载均衡算法。
- com.netflix.loadbalancer.RandomRule：随机。
- com.netflix.loadbalancer.RetryRule：先按照RoundRobinRule（轮询）的策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务。
- com.netflix.loadbalancer.WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择。
- com.netflix.loadbalancer.BestAvailableRule：先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。
- com.netflix.loadbalancer.AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例。
- com.netflix.loadbalancer.ZoneAvoidanceRule：复合判断Server所在区域的性能和Server的可用性选择服务器。
  
  