# **一、insert**

## **1、插入操作**

 

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class CRUDTests {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void testInsert(){
        User user = new User();
        user.setName("Helen");
        user.setAge(18);
        user.setEmail("55317332@qq.com");
        int result = userMapper.insert(user);
        System.out.println(result); //影响的行数
        System.out.println(user); //id自动回填
    }
}
```

**注意：**数据库插入id值默认为：全局唯一id

![img](file:///C:/Users/VSUS/Documents/My Knowledge/temp/d4f92e2e-0ba2-4406-bad2-bdb95a46c95d/128/index_files/93ff417f-c9f7-4225-b395-2afe2776183d.jpg)

## 2、主键策略

**（1）ID_WORKER**

MyBatis-Plus默认的主键策略是：ID_WORKER  *全局唯一ID*

**参考资料：分布式系统唯一ID生成方案汇总：**https://www.cnblogs.com/haoxinyue/p/5208136.html

**（2）自增策略**

* 要想主键自增需要配置如下主键策略

* * 需要在创建数据表的时候设置主键自增
  * 实体字段中配置 @TableId(type = IdType.AUTO)

 

```
@TableId(type = IdType.AUTO)
private Long id;
```

要想影响所有实体的配置，可以设置全局主键配置

 

```
#全局设置主键生成策略
mybatis-plus.global-config.db-config.id-type=auto
```

其它主键策略：分析 IdType 源码可知

 

```
@Getter
public enum IdType {
    /**
     * 数据库ID自增
     */
    AUTO(0),
    /**
     * 该类型为未设置主键类型
     */
    NONE(1),
    /**
     * 用户输入ID
     * 该类型可以通过自己注册自动填充插件进行填充
     */
    INPUT(2),
    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 全局唯一ID (idWorker)
     */
    ID_WORKER(3),
    /**
     * 全局唯一ID (UUID)
     */
    UUID(4),
    /**
     * 字符串全局唯一ID (idWorker 的字符串表示)
     */
    ID_WORKER_STR(5);
    private int key;
    IdType(int key) {
        this.key = key;
    }
}
```

# 二、update

## **1、根据Id更新操作**

**注意：**update时生成的sql自动是动态sql：UPDATE user SET age=? WHERE id=? 

 

```
    @Test
    public void testUpdateById(){
        User user = new User();
        user.setId(1L);
        user.setAge(28);
        int result = userMapper.updateById(user);
        System.out.println(result);
    }
```

## 2、自动填充

项目中经常会遇到一些数据，每次都使用相同的方式填充，例如记录的创建时间，更新时间等。

我们可以使用MyBatis Plus的自动填充功能，完成这些字段的赋值工作：

**（1）数据库表中添加自动填充字段**

在User表中添加datetime类型的新的字段 create_time、update_time

**（2）实体上添加注解**

 

```
@Data
public class User {
    ......
        
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    //@TableField(fill = FieldFill.UPDATE)
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}
```

**（3）实现元对象处理器接口**

**注意：不要忘记添加 @Component 注解**

 

```
package com.atguigu.mybatisplus.handler;
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import java.util.Date;
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(MyMetaObjectHandler.class);
    @Override
    public void insertFill(MetaObject metaObject) {
        LOGGER.info("start insert fill ....");
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
    @Override
    public void updateFill(MetaObject metaObject) {
        LOGGER.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

**（4）测试**

## 3、乐观锁

**主要适用场景：**当要更新一条记录的时候，希望这条记录没有被别人更新，也就是说实现线程安全的数据更新

**乐观锁实现方式：**

* 取出记录时，获取当前version
* 更新时，带上这个version
* 执行更新时， set version = newVersion where version = oldVersion
* 如果version不对，就更新失败

***\*（1）数据库中添加version字段\****

 

```
ALTER TABLE `user` ADD COLUMN `version` INT
```



![img](file:///C:/Users/VSUS/Documents/My Knowledge/temp/d4f92e2e-0ba2-4406-bad2-bdb95a46c95d/128/index_files/7bf260f8-d483-49fe-b448-b2fbea3dddaf.png)

***\*（2）实体类添加version字段\****

并添加 @Version 注解

 

```
@Version
@TableField(fill = FieldFill.INSERT)
private Integer version;
```

***\*（3）元对象处理器接口添加version的insert默认值\****

 

```
@Override
public void insertFill(MetaObject metaObject) {
    ......
    this.setFieldValByName("version", 1, metaObject);
}
```

**特别说明:**

支持的数据类型只有 int,Integer,long,Long,Date,Timestamp,LocalDateTime整数类型下 `newVersion = oldVersion + 1``newVersion` 会回写到 `entity` 中仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!

**（4）\**在 MybatisPlusConfig 中注册 Bean\****

创建配置类

 

```
package com.atguigu.mybatisplus.config;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;
@EnableTransactionManagement
@Configuration
@MapperScan("com.atguigu.mybatis_plus.mapper")
public class MybatisPlusConfig {
    /**
     * 乐观锁插件
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```

**（5）测试乐观锁可以修改成功**

测试后分析打印的sql语句，将version的数值进行了加1操作

 

```
/**
 * 测试 乐观锁插件
 */
@Test
public void testOptimisticLocker() {
    //查询
    User user = userMapper.selectById(1L);
    //修改数据
    user.setName("Helen Yao");
    user.setEmail("helen@qq.com");
    //执行更新
    userMapper.updateById(user);
}
```

**（5）测试乐观锁修改失败**

 

```
/**
 * 测试乐观锁插件 失败
 */
@Test
public void testOptimisticLockerFail() {
    //查询
    User user = userMapper.selectById(1L);
    //修改数据
    user.setName("Helen Yao1");
    user.setEmail("helen@qq.com1");
    //模拟取出数据后，数据库中version实际数据比取出的值大，即已被其它线程修改并更新了version
    user.setVersion(user.getVersion() - 1);
    //执行更新
    userMapper.updateById(user);
}
```

# **三、select**

## **1、根据id查询记录**

 

```
@Test
public void testSelectById(){
    User user = userMapper.selectById(1L);
    System.out.println(user);
}
```

## **2、通过多个id批量查询**

完成了动态sql的foreach的功能

 

```
@Test
public void testSelectBatchIds(){
    List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3));
    users.forEach(System.out::println);
}
```

## **3、简单的条件查询**

通过map封装查询条件

 

```
@Test
public void testSelectByMap(){
    HashMap<String, Object> map = new HashMap<>();
    map.put("name", "Helen");
    map.put("age", 18);
    List<User> users = userMapper.selectByMap(map);
    users.forEach(System.out::println);
}
```

**注意：**map中的key对应的是数据库中的列名。例如数据库user_id，实体类是userId，这时map的key需要填写user_id

## 4、分页

MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

**（1）创建配置类**

此时可以删除主类中的 *@MapperScan* 扫描注解**
**

 

```
/**
 * 分页插件
 */
@Bean
public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
}
```

**（2）测试selectPage分页**

**测试：**最终通过page对象获取相关数据

 

```
@Test
public void testSelectPage() {
    Page<User> page = new Page<>(1,5);
    userMapper.selectPage(page, null);
    page.getRecords().forEach(System.out::println);
    System.out.println(page.getCurrent());
    System.out.println(page.getPages());
    System.out.println(page.getSize());
    System.out.println(page.getTotal());
    System.out.println(page.hasNext());
    System.out.println(page.hasPrevious());
}
```

控制台sql语句打印：SELECT id,name,age,email,create_time,update_time FROM user LIMIT 0,5 

**（3）测试selectMapsPage分页：结果集是Map**

 

```
@Test
public void testSelectMapsPage() {
    Page<User> page = new Page<>(1, 5);
    IPage<Map<String, Object>> mapIPage = userMapper.selectMapsPage(page, null);
    //注意：此行必须使用 mapIPage 获取记录列表，否则会有数据类型转换错误
    mapIPage.getRecords().forEach(System.out::println);
    System.out.println(page.getCurrent());
    System.out.println(page.getPages());
    System.out.println(page.getSize());
    System.out.println(page.getTotal());
    System.out.println(page.hasNext());
    System.out.println(page.hasPrevious());
}
```

# 四、delete

## **1、根据id删除记录**

 

```
@Test
public void testDeleteById(){
    int result = userMapper.deleteById(8L);
    System.out.println(result);
}
```

## **2、批量删除**

 

```
    @Test
    public void testDeleteBatchIds() {
        int result = userMapper.deleteBatchIds(Arrays.asList(8, 9, 10));
        System.out.println(result);
    }
```

## **3、简单的条件查询删除**

 

```
@Test
public void testDeleteByMap() {
    HashMap<String, Object> map = new HashMap<>();
    map.put("name", "Helen");
    map.put("age", 18);
    int result = userMapper.deleteByMap(map);
    System.out.println(result);
}
```

## 4、逻辑删除

* 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除数据
* 逻辑删除：假删除，将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录

**（1）数据库中添加 deleted字段**

 

```
ALTER TABLE `user` ADD COLUMN `deleted` boolean
```

![img](file:///C:/Users/VSUS/Documents/My Knowledge/temp/d4f92e2e-0ba2-4406-bad2-bdb95a46c95d/128/index_files/bc4cbff4-c2b8-45d5-ae8d-53439dd2330c.png)

**（2）实体类添加deleted \**字段\****

并加上 @TableLogic 注解 和 @TableField(fill = FieldFill.INSERT) 注解

 

```
@TableLogic
@TableField(fill = FieldFill.INSERT)
private Integer deleted;
```

**（3）元对象处理器接口添加deleted的insert默认值**

 

```
@Override
public void insertFill(MetaObject metaObject) {
    ......
    this.setFieldValByName("deleted", 0, metaObject);
}
```

**（4）application.properties 加入配置**

此为默认值，如果你的默认值和mp默认的一样,该配置可无

 

```
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

**（5）在 MybatisPlusConfig 中注册 Bean**

 

```
@Bean
public ISqlInjector sqlInjector() {
    return new LogicSqlInjector();
}
```

**（6）测试逻辑删除**

* 测试后发现，数据并没有被删除，deleted字段的值由0变成了1
* 测试后分析打印的sql语句，是一条update
* **注意：**被删除数据的deleted 字段的值必须是 0，才能被选取出来执行逻辑删除的操作

 

```
/**
 * 测试 逻辑删除
 */
@Test
public void testLogicDelete() {
    int result = userMapper.deleteById(1L);
    System.out.println(result);
}
```

**（7）测试逻辑删除后的查询**

MyBatis Plus中查询操作也会自动添加逻辑删除字段的判断

 

```
/**
 * 测试 逻辑删除后的查询：
 * 不包括被逻辑删除的记录
 */
@Test
public void testLogicDeleteSelect() {
    User user = new User();
    List<User> users = userMapper.selectList(null);
    users.forEach(System.out::println);
}
```

测试后分析打印的sql语句，包含 WHERE deleted=0 

SELECT id,name,age,email,create_time,update_time,deleted FROM user WHERE deleted=0

# 五、性能分析

性能分析拦截器，用于输出每条 SQL 语句及其执行时间

SQL 性能执行分析,开发环境使用，超过指定时间，停止运行。有助于发现问题

## 1、配置插件

**（1）参数说明**

参数：maxTime： SQL 执行最大时长，超过自动停止运行，有助于发现问题。

参数：format： SQL是否格式化，默认false。

**（2）在 MybatisPlusConfig 中配置**

 

```
/**
 * SQL 执行性能分析插件
 * 开发环境使用，线上不推荐。 maxTime 指的是 sql 最大执行时长
 */
@Bean
@Profile({"dev","test"})// 设置 dev test 环境开启
public PerformanceInterceptor performanceInterceptor() {
    PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
    performanceInterceptor.setMaxTime(100);//ms，超过此处设置的ms则sql不执行
    performanceInterceptor.setFormat(true);
    return performanceInterceptor;
}
```

**（3）Spring Boot 中设置dev环境**

 

```
#环境设置：dev、test、prod
spring.profiles.active=dev
```

可以针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`

也可以自定义环境名称：如test1、test2

## 2、测试

**（1）常规测试**

 

```
/**
 * 测试 性能分析插件
 */
@Test
public void testPerformance() {
    User user = new User();
    user.setName("我是Helen");
    user.setEmail("helen@sina.com");
    user.setAge(18);
    userMapper.insert(user);
}
```

**输出：**

![img](file:///C:/Users/VSUS/Documents/My Knowledge/temp/d4f92e2e-0ba2-4406-bad2-bdb95a46c95d/128/index_files/bb355a17-3cdc-4f0a-82f2-232defbd235b.png)

**（2）将maxTime 改小之后再次进行测试**

 

```
performanceInterceptor.setMaxTime(5);//ms，超过此处设置的ms不执行
```

如果执行时间过长，则抛出异常：The SQL execution time is too large, 

**输出：**

![img](file:///C:/Users/VSUS/Documents/My Knowledge/temp/d4f92e2e-0ba2-4406-bad2-bdb95a46c95d/128/index_files/1ae5ae68-b6b2-4801-ae26-29835b175b24.png)

# 六、其它

如果想进行复杂条件查询，那么需要使用条件构造器 Wapper，涉及到如下方法

**1、delete**

**2、selectOne**

**3、selectCount**

**4、selectList**

**5、selectMaps**

**6、selectObjs**

**7、update**