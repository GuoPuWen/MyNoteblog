来源：图文基本来源网络

### 一、SpringJpa简介





### 二、基本操作

基本环境

- java8
- springboot 2.3.5
- springJpa 2.3.5

##### 1.导入依赖坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

查看该SpringJpa的版本：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-ldap</artifactId>
    <version>2.3.5.RELEASE</version>
</dependency>
```

##### 2.配置文件

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useSSL=false?characterEncoding=utf-8 
spring.datasource.username=youusername
spring.datasource.password=youpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQL5Dialect

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.type=trace
spring.jpa.properties.hibernate.use_sql_comments=true
spring.jpa.properties.hibernate.jdbc.batch_size=50
logging.level.org.hibernate.type.descriptor.sql=trace
```

对于spring.jpa.hibernate.ddl-auto如下取值：

- ==create==：每次加载Hibernate时都会删除上一次生成的表（包括数据），然后重新生成新表，即使两次没有任何修改也会这样执行。适用于每次执行单测前清空数据库的场景。
- ==create-drop==：每次加载Hibernate时都会生成表，但当SessionFactory关闭时，所生成的表将自动删除。
- ==update==：最常用的属性值，第一次加载Hibernate时创建数据表（前提是需要先有数据库），以后加载Hibernate时不会删除上一次生成的表，会根据实体更新，只新增字段，不会删除字段（即使实体中已经删除）。
- ==validate==：每次加载Hibernate时都会验证数据表结构，只会和已经存在的数据表进行比较，根据model修改表结构，但不会创建新表。

不配置此项，表示禁用自动建表功能

##### 3.编写实体类

```java
@Entity
public class User {
    @Id
    @GenericGenerator(name = "idGenerator",strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;

    @Column(nullable = false, unique = true)
    private String userName;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private int age;
}
```

> 主键采用UUID策略
>  `@GenericGenerator`是Hibernate提供的主键生成策略注解，注意下面的`@GeneratedValue`（JPA注解）使用generator = "idGenerator"引用了上面的name = "idGenerator"主键生成策略

注意：

JPA自带的几种主键生成策略

* TABLE： 使用一个特定的数据库表格来保存主键
* SEQUENCE： 根据底层数据库的序列来生成主键，条件是数据库支持序列。这个值要与generator一起使用，generator 指定生成主键使用的生成器（可能是orcale中自己编写的序列）
* IDENTITY： 主键由数据库自动生成（主要是支持自动增长的数据库，如mysql）
* AUTO： 主键由程序控制，也是GenerationType的默认值

##### 4.编写repository

```java
/**
 * @author 四五又十
 * @create 2020/11/12 20:15
 */
public interface UserRepository extends JpaRepository<User,String> {
    User findByUserName(String userName);
}

```

##### 5.编写测试类

```java
@Autowired
    private UserRepository userRepository;

    @Test
    public void testFindByUserName(){
        User user = new User();
        user.setUserName("zhangsan");
        user.setAge(30);
        user.setPassword("aaabbb");
        userRepository.save(user);
        User user1 = userRepository.findByUserName("zhangsan");
        System.out.println(user1);
    }
```

测试结果：我的数据库中是没有创建user表的，因为在配置文件中添加了spring.jpa.hibernate.ddl-auto=update，那么SpringJpa会自动帮我创建user表，并且向数据库中添加user这一列，并且在实体类中配置了uuid生成id主键的策略，所以在表中id字段是uuid，最后测试findByUserName也成功在控制台输出

//图片1：数据库截图

//图片2：控制台截图

### 三、其他操作

Spring Data Jpa通过解析方法名创建查询，框架在进行方法名解析时，会先把方法名多余的前缀find…By, read…By, query…By, count…By以及get…By截取掉，然后对剩下部分进行解析，第一个By会被用作分隔符来指示实际查询条件的开始。 我们可以在`实体属性`上定义条件，并将它们与And和Or连接起来，从而创建大量查询：

对于这些关键字，官网里面已经有描述 [SpringJpa官网](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-property-expressions)

可以参考下面写法：

```java
User findByUsername(String username);

List<User> findByUsernameIgnoreCase(String username);

List<User> findByUsernameLike(String username);

User findByUsernameAndPassword(String username, String password);

User findByEmail(String email);

List<User> findByEmailLike(String email);

List<User> findByIdIn(List<String> ids);

List<User> findByIdInOrderByUsername(List<String> ids);

void deleteByIdIn(List<String> ids);

Long countByUsernameLike(String username);
```

- 自定义查询Using @Query

@Query 注解的使用非常简单，只需在声明的方法上面标注该注解，同时提供一个 JPQL 查询语句即可

### 四、分页操作

在JPA中提供了很方便的分页功能，那就是Pageable（org.springframework.data.domain.Pageable）以及它的实现类PageRequest（org.springframework.data.domain.PageRequest），详细的可以见示例代码。

UserRepository代码

```java
Page<User> findByAge(Integer age, Pageable pageable);
```

测试代码：

```java
@Test
public void testFindByUserName2(){
    Pageable pageable = PageRequest.of( 0 , 2 , Sort.Direction.DESC, "id");
    Page<User> page = userRepository.findByAge(20, pageable);
    List<User> users = page.getContent();   //获得数据
    System.out.println(page.getTotalElements());//查询总行数
    System.out.println(page.getTotalPages()); //按照当前分页大小，总页数
    System.out.println(users);
}
```

注意： ` `Pageable pageable = ``new` `PageRequest(` `0` `,` `3` `, Sort.Direction.DESC,` `"id"` `);这种方式以及不能在使用