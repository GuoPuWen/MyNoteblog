# 一、执行器

SqlSession提供了一个接口，设置了增删改查、事务相关的方法，但是只是提供了一个接口，提供了一个门面，真正执行的是Executor，ExecutorType下定义了3个执行器：

- `SIMPLE` 在每次执行完成后都会关闭 `statement` 对象；
- `REUSE` 会在本地维护一个容器，当前 `statement` 创建完成后放入容器中，当下次执行相同的 `sql` 时会复用 `statement` 对象，执行完毕后也不会关闭；
- `BATCH` 会将修改操作记录在本地，等待程序触发或有下一次查询时才批量执行修改操作

前面章节介绍过Mybatis的执行分为两步：

- 解析配置文件到Configuration
- 执行sql

而执行器便是执行sql这边的核心组件，下面是Executor的子类图

![image-20210222154044366](http://cdn.noteblogs.cn/image-20210222154044366.png)

在执行sql这个过程中，下面四个组件是必须的：

- **执行器：**Executor, 处理流程的头部，主要负责缓存、事务、批处理。一个执行可用于执行多条SQL。它和SQL处理器是1对N的关系。
- **Sql处理器：**StatementHandler 用于和JDBC打道，比如基于SQL声明Statement、设置参数、然后就是调用Statement来执行。它只能用于一次SQL的执行
- **参数处理器：**ParameterHandler，用于解析SQL参数，并基于参数映射，填充至PrepareStatement。同样它只能用于一次SQL的执行
- **结果集处理器：**ResultSetHandler，用于读取ResultSet 结果集，并基于结果集映射，封装成JAVA对象。他也只用用于一次SQL的执行。

## 1.1 SimpleExecutor

SimpleExecutor是默认的简单执行器，使用看下面代码

```java
private Configuration configuration;
private Connection connection;
private JdbcTransaction jdbcTransaction;

@Before
public void init() throws Exception {
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
    configuration = sessionFactory.getConfiguration();
    connection = DriverManager.getConnection(JDBC.URL, JDBC.USERNAME, JDBC.PASSWORD);
    jdbcTransaction = new JdbcTransaction(connection);
}


@Test
public void simpleTest() throws SQLException {
    SimpleExecutor simpleExecutor = new SimpleExecutor(configuration, jdbcTransaction);
    MappedStatement ms = configuration.getMappedStatement("cn.mybatis.dao.UserMapper.findById");
    List<Object> list = simpleExecutor.doQuery(ms, 3, RowBounds.DEFAULT, SimpleExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    simpleExecutor.doQuery(ms, 3, RowBounds.DEFAULT, SimpleExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    System.out.println(list.get(0));
}
```

还记得MappedStatement对象吗？在处理mapper映射器的时候mybatis会将sql的所有信息保存在MappedStatement对象中，所以执行sql是必不可少的，doQuery的参数声明：

- MappedStatement对象保存sql的所有信息
- 参数
- 分页，可以使用RowBounds的默认
- 默认结果处理集
- BoundSql生成sql

![image-20210222155228720](http://cdn.noteblogs.cn/image-20210222155228720.png)

可以看到无论sql是否一样，每次都会进行预编译

## 1.2 ReuseExecutor

ReuseExecutor使用代码和上面一样，ReuseExecutor可重复执行器的特点是，可重复使用JDBC中的Statement，能减少预编译的次数，该执行器会把Statement缓存起来，下次遇到相同的sql，就直接取出来使用，减少预编译的次数

```java
@Test
public void reuseTest() throws SQLException {
    ReuseExecutor reuseExecutor = new ReuseExecutor(configuration, jdbcTransaction);
    MappedStatement ms = configuration.getMappedStatement("cn.mybatis.dao.UserMapper.findById");
    List<Object> list = reuseExecutor.doQuery(ms, 3, RowBounds.DEFAULT, ReuseExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    reuseExecutor.doQuery(ms, 3, RowBounds.DEFAULT, ReuseExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    System.out.println(list.get(0));
}
```

![image-20210222192243838](http://cdn.noteblogs.cn/image-20210222192243838.png)

可以看到执行两次相同的sql，只使用了一次预编译

## 1.3 BatchExecutor

BatchExecutor 是批处理执行器，每次的执行操作不会立即进行，而是把对应的Statement填充好参数之后存储起来，当调用flushStatements 的时候会一次性提交到数据库，它可以用于批处理插入的场景，效果相当于SQL的拼装，需要注意的是，JDBC的批处理并不适用于查询语句，例如：

```java
@Test
public void batchTest() throws SQLException{
    BatchExecutor batchExecutor = new BatchExecutor(configuration, jdbcTransaction);
    MappedStatement ms = configuration.getMappedStatement("cn.mybatis.dao.UserMapper.findById");
    List<Object> list = batchExecutor.doQuery(ms, 3, RowBounds.DEFAULT, BatchExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    batchExecutor.doQuery(ms, 3, RowBounds.DEFAULT, BatchExecutor.NO_RESULT_HANDLER, ms.getBoundSql(3));
    System.out.println(list.get(0));
}
```

![image-20210222193345295](http://cdn.noteblogs.cn/image-20210222193345295.png)**可以看到还是预编译了两次，BatchExecutor执行update**

## 1.4 基础执行器

Mybatis是具有一级缓存和二级缓存的，可以发现如果使用上面的方法执行sql，是没有走到缓存相关的逻辑的，而缓存相关的逻辑是在BaseExecutor中实现的，具体结构可见下图

![image-20210223083549358](http://cdn.noteblogs.cn/image-20210223083549358.png)

当我们执行query，其实是执行了BaseExecutor中的query方法，里面定义了缓存相关的逻辑

```java
@Test
public void testBase() throws SQLException {
    SimpleExecutor simpleExecutor = new SimpleExecutor(configuration, jdbcTransaction);
    MappedStatement ms = configuration.getMappedStatement("cn.mybatis.dao.UserMapper.findById");
    List<Object> list = simpleExecutor.query(ms, 3, RowBounds.DEFAULT, SimpleExecutor.NO_RESULT_HANDLER);
    simpleExecutor.query(ms, 3, RowBounds.DEFAULT, SimpleExecutor.NO_RESULT_HANDLER);
    System.out.println(list.get(0));
}
```

将会先去查询一级缓存，如果一级缓存中有那么直接返回结果，如果没有则调用子类执行器SimpleExecutor、ReuseExecutor、BatchExecutor中的query方法，所以上述代码的执行值进行了一次编译sql

![image-20210223084124501](http://cdn.noteblogs.cn/image-20210223084124501.png)

下图是BaseExecutor中的query方法

![image-20210223084409118](http://cdn.noteblogs.cn/image-20210223084409118.png)

## 1.5 CachingExecutor

上面提到一级缓存相关的执行逻辑，也就是调用BaseExecutor的query方法，如果一级缓存没有命中则直接调用子类的doQuery方法进行数据库的查询，那么二级缓存呢？ CachingExecutor就是二级缓存执行的具体逻辑类

通过前面说SqlSession的运行流程就知道，正常来说实现进行二级缓存的查询，然后进行一级缓存，那么mybatis使用了装饰者模式，在CachingExecutor中有一个delegate指向一级缓存器BaseExecutor

> 装饰者模式：在不改变原有类的结构和继承的情况下，通过包装原对象区扩展一个新功能

![image-20210223093128098](http://cdn.noteblogs.cn/image-20210223093128098.png)

下面来测试一下二级缓存，注意在此之前要开启二级缓存

```java
public void testCache() throws SQLException{
    Executor simpleExecutor = new SimpleExecutor(configuration, jdbcTransaction);
    MappedStatement ms = configuration.getMappedStatement("cn.mybatis.dao.UserMapper.findById");

    CachingExecutor cachingExecutor = new CachingExecutor(simpleExecutor);

    List<Object> list = cachingExecutor.query(ms, 3, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
   	//需要手动提交，这个是必须的	
    cachingExecutor.commit(true);
    cachingExecutor.query(ms, 3, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
    cachingExecutor.query(ms, 3, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
    cachingExecutor.query(ms, 3, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
    System.out.println(list.get(0));
}
```

![image-20210223093305904](http://cdn.noteblogs.cn/image-20210223093305904.png)

可以看到已经执行了二级缓存相关的逻辑，并且命中率也在上升，从代码中也可以看到在new CachingExecutor对象中传入了一个SimpleExecutor作为delegate，进行装饰，那么上述代码的流程为：

CachingExecutor的二级缓存query方法 ---> BaseExecutor的query方法 ---> SimpleExecutord的doQuery方法

## 1.6 总结

上面的代码都是直接使用Executor，下面我们通过最基本的方法来回顾上述mybatis的执行器的执行流程

```java
@Test
public void testSqlSession() throws Exception {
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
    SqlSession sqlSession = sessionFactory.openSession(true);
    List<Object> list = sqlSession.selectList("cn.mybatis.dao.UserMapper.findById", 3);
    sqlSession.commit();
    sqlSession.selectList("cn.mybatis.dao.UserMapper.findById", 3);
    sqlSession.selectList("cn.mybatis.dao.UserMapper.findById", 3);
    sqlSession.selectList("cn.mybatis.dao.UserMapper.findById", 3);
    System.out.println(list.get(0));
}
```

SqlSession是一个门面，比如说通过服务员点餐和直接通过大厨点餐是一样的，只不过服务员提供了一个门面，通过debug查看SqlSession的属性

![image-20210223094513535](http://cdn.noteblogs.cn/image-20210223094513535.png)

上述代码的执行流程为：

CachingExecutor的query方法(二级缓存) --> BaseExecutor的query(一级缓存) --> SimpleExecutor的doQuery方法(数据库查询)

通过一个图来总结：

![image-20210223095003252](http://cdn.noteblogs.cn/image-20210223095003252.png)

# 二、一级缓存

mybatis是默认开启一级缓存的，一级缓存的作用域是一个会话级别的，但是要想命中一级缓存需要下面几个条件：

- SQL传入参数一致
- 同一个会话（SqlSession对象，一级缓存属于会话级缓存）
- 方法名和类名必须一样（Statement ID必须一样如：com.xxx.XXXMapper.findById()）
- 行范围一样 rowbound
- 不手动清空缓存 (-cleanCache -commit rollback)
- 没有Update操作
- 缓存作用域不能是STATEMENT
- 未配置flushCash为false

Spring集成mybatis会造成缓存失效的原因是，spring会为每一个查询新建一个会话，这就破坏了mybatis的一级缓存条件，要解决这个问题可以通过事务解决