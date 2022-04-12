# 一、介绍

​	ThreadLocal官方描述：

> This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).
> Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).
>
> 大致意思：
>
> ThreadLocal这个类提供线程局部变量，而这个变量在高并发的环境下能保证每个线程的变量相对于独立其他线程的变量，ThreadLocal实例通常用private static修饰，用于关联线程和上下文

也就是说ThreadLocal是在高并发的环境下解决==变量在线程之间隔离而在线程方法组件之间通信的问题==

## 1.1 基本使用

下面是ThreadLocal的一些常用方法：

| 方法声明                  | 描述                       |
| ------------------------- | -------------------------- |
| ThreadLocal()             | 创建ThreadLocal对象        |
| public void set( T value) | 设置当前线程绑定的局部变量 |
| public T get()            | 获取当前线程绑定的局部变量 |
| public void remove()      | 移除当前线程绑定的局部变量 |

一个小例子：

```java
class MyDemo{
    private String content;

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
public class ThreadLocalDemo {
    public static void main(String[] args) {
        MyDemo demo = new MyDemo();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------------------");
                System.out.println(Thread.currentThread().getName() + "--->" +demo.getContent());
            },"线程" + i).start();
        }
    }
}
```

线程 操作 资源类，那么上面new了5个线程分别取操作MyDemo，由于CPU的时间调度，5个线程运行的情况我们是不能把控的，所以上面的代码肯定会有线程安全问题，看输出

![image-20210320123242276](http://cdn.noteblogs.cn/image-20210320123242276.png)

那么如何解决？加锁Synchronized？加锁肯定是可以解决线程安全问题的：

```java
synchronized (ThreadLocalDemo.class){
    demo.setContent(Thread.currentThread().getName() + "的数据");
    System.out.println("-----------------------------------");
    System.out.println(Thread.currentThread().getName() + "--->" +demo.getContent());
}
```

但是加了吧对象锁在上面，导致线程之间同步进行这就给高并发的效率大打折扣了！，那么解决上面这个问题可以使用ThreadLocal

```java
class MyDemo{
    private static ThreadLocal<String> tl = new ThreadLocal<>();
    private String content;

    public String getContent() {
//        return content;
        return tl.get();
    }

    public void setContent(String content) {
//        this.content = content;
        tl.set(content);
    }
}
public class ThreadLocalDemo {
    public static void main(String[] args) {
        MyDemo demo = new MyDemo();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------------------");
                System.out.println(Thread.currentThread().getName() + "--->" +demo.getContent());
            },"线程" + i).start();
        }
    }
}
```

重新运行代码，发现如论运行多少次结果都是对的，也就是解决了上面这个线程安全问题

## 1.2 Synchronized？与ThreadLocal？

在上面的一个小case中，使用synchronized同样可以解决问题，只不过两者处理角度不同，对于线程安全问题来说，Synchronized没有它不能解决的，因为是synchronized只提供一份变量，但是访问这份变量的线程必须排队，就导致线程之间的异步进行，而ThreadLocal是将这份变量在每一个线程之间存一份，每个线程之间使用自己的这样也就不会造成多个线程之间抢占资源

|        | synchronized                                                 | ThreadLocal                                                  |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理   | 同步机制采用’以时间换空间’的方式, 只提供了一份变量,让不同的线程排队访问 | ThreadLocal采用’以空间换时间’的方式, 为每一个线程都提供了一份变量的副本,从而实现同时访问而相不干扰 |
| 侧重点 | 多个线程之间访问资源的同步                                   | 多线程中让每个线程之间的数据相互隔离                         |

举个小例子来说明上面的区别：synchronized锁机制就好比现在有100个同学，但是老师手上只准备了1只笔，那么只能一个一个同学去使用这只笔，而ThreadLocal机制就是老师直接准备100值笔，每个人用自己的笔互不影响，这两种方式实现的思路是不同的

# 二、适用场景

ThreadLocal保证了每一个线程之间的变量都是自己独有的，可是这在具体的业务场景上有什么用呢？ThreadLocal最典型的使用ThreadLocal管理数据库连接Connection，将Connection绑定在ThreadLocal这样就能保证每一个线程的操作都是同一个Connection这样就能保证事务，那么我们就来模拟这个场景，理解ThreadLocal的具体作用

简单的使用C3P0连接池技术搭建好一个类似三层架构的环境，模拟一个银行转账的业务，zhang向lisi转账，下面的代码由于没有添加事务，那么一旦出现异常，则造成zahngsan的钱少了但是lisi的钱没有增加

AccountDao

```java
public class AccountDao {
    public void out(String outUser, int money) throws  SQLException {
        String sql = "update account set money = money - ? where name = ?";

        Connection conn = JDBCUtil.getConnection();
        PreparedStatement pstm = (PreparedStatement) conn.prepareStatement(sql);
        pstm.setInt(1,money);
        pstm.setString(2,outUser);
        pstm.executeUpdate();

        JDBCUtil.release(pstm,conn);
    }

    public void in(String inUser, int money) throws SQLException {
        String sql = "update account set money = money + ? where name = ?";

        Connection conn = JDBCUtil.getConnection();
        PreparedStatement pstm  = (PreparedStatement) conn.prepareStatement(sql);
        pstm.setInt(1,money);
        pstm.setString(2,inUser);
        pstm.executeUpdate();

        JDBCUtil.release(pstm,conn);
    }

}
```

AccountService

```java
public class AccountService {
    public boolean transfer(String outUser, String inUser, int money) {
        AccountDao ad = new AccountDao();
        try {
            // 转出
            ad.out(outUser, money);
            //模拟异常
            int i = 1/0;
            // 转入
            ad.in(inUser, money);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
}
```

JDBCUtil

```java
public class JDBCUtil {
    static ComboPooledDataSource dataSource = null;
    static Connection conn;
    static {
        dataSource = new ComboPooledDataSource("mysql");
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    public static void closeConn(Connection conn){
        try {
            if(conn!=null && conn.isClosed()){
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void release(PreparedStatement pstm, Connection conn){
        if(pstm != null){
            try {
                pstm.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(conn != null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void commitAndClose(Connection conn) {
        try {
            if(conn != null){
                //提交事务
                conn.commit();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    public static void rollbackAndClose(Connection conn) {
        try {
            if(conn != null){
                //回滚事务
                conn.rollback();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

main

```java
public class main {
    public static void main(String[] args) {
        AccountService accountService = new AccountService();
        accountService.transfer("zhangsan", "lisi", 100);
    }
}
```

所以说必须要添加事务！既然是事务，那么使用的Conn一定是要一样的，而我们在Service层获取到的Connection如何向下传递呢？很显然可以使用添加参数的办法，也就是说在调用dao层的方法的时候将coon连接向下传递过去，但是这样就会导致代码模块之间的耦合程度比较高，不利于维护，那么ThreadLocal就可以派上用场了

JDBCUtil

```java
public class JDBCUtil {
    private static ThreadLocal<Connection> tl = new ThreadLocal<>();
    private static ComboPooledDataSource dataSource = null;
    static {
        dataSource = new ComboPooledDataSource("mysql");
    }

    public static Connection getConnection() throws SQLException {
        Connection conn = tl.get();
        if(conn == null){
            conn = dataSource.getConnection();
            tl.set(conn);
        }
        return conn;
    }



    public static void release(PreparedStatement pstm, Connection conn){
        if(pstm != null){
            try {
                pstm.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(conn != null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void commitAndClose() {
        try {
            Connection conn = getConnection();
            if(conn != null){
                //提交事务
                conn.commit();
                //解除绑定
                tl.remove();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    public static void rollbackAndClose() {

        try {
            Connection conn = getConnection();
            if(conn != null){
                //回滚事务
                conn.rollback();
                //解除绑定
                tl.remove();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

AccountService

```java
public class AccountService {
    public boolean transfer(String outUser, String inUser, int money) {
        AccountDao ad = new AccountDao();
        try {
            Connection connection = JDBCUtil.getConnection();
            connection.setAutoCommit(false);
            // 转出
            ad.out(outUser, money);
            //模拟异常
            int i = 1/0;
            // 转入
            ad.in(inUser, money);
            //事务提交
            JDBCUtil.commitAndClose();
        } catch (Exception e) {
            e.printStackTrace();
            JDBCUtil.rollbackAndClose();
            return false;
        }
        return true;
    }
}
```

上面代码使用ThreadLocal保证了每一个线程拿到的Conn对象在不同得组件service或者dao上都是一样的并且确保了事务

从上面的例子中，可以明显感受到ThreadLocal的好处：

- 传递数据：保存每个线程绑定的数据，例如上面的Connection对象，确保在该线程需要使用该Connection对象的时候直接获取，保证线程安全问题
- 线程之间隔离：不同的线程拿到的数据时独立的一份，保证了线程安全

# 三、ThreadLocal的内部结构

一种可能的方案是，ThreadLocal只维护一个Map对象，其中key是Thread，value是该ThreadLocal在该Thread上的值，也就是如下图

![ThreadLocal side Map](http://cdn.noteblogs.cn/VarMap.png)

这样设计的方案就有以下缺陷：

- 由于多个线程操作一个Map，那么这个Map必须要加锁
- 如果ThreadLocal的实例数比较多的时候，那么Map的key就比较多
- 线程结束的时候，需要保证它所访问的所有ThreadLocal中对应的映射都删除，否则会有内存泄漏问题

早期的ThreadLocal使用的是如上方案，但是现在的设计刚好相反，==每个线程都有自己的Map，这个map的key是ThreadLocal，而值就是真正要存储的value==

![ThreadLocal side Map](http://cdn.noteblogs.cn/ThreadMap.png)

这种设计和上面的方案是刚好相反的，这也有一个比较：Thread实例多呢？还是ThreadLocal多，而大量的数据证明hreadLocal的数量要少于Thread的数量。那么单凭这两者的数量来说，设计成以ThreadLocal为key的方案更加优秀，同时这种方法由于是将map存在线程工作区空间，也就不是共享变量，所以不存在线程安全问题，也就不需要加锁，这在一定程度上提高了效率

ThreadLocal在每个线程内维护一个Map，而这个Map就是ThreadLocalMap，ThreadLocalMap没有实现map接口，也就是说这是ThreadLocal内部自己重新写的一个map

# 四、ThreadLocal的核心源码

TheradLocal对外暴露的方法比较简单set、get、remove、initialValue方法，这些方法的源码也比较简单可以理解，重点是ThreadLocalMap这个内部维护的map结构

| 方法声明                   | 描述                         |
| -------------------------- | ---------------------------- |
| protected T initialValue() | 返回当前线程局部变量的初始值 |
| public void set( T value)  | 设置当前线程绑定的局部变量   |
| public T get()             | 获取当前线程绑定的局部变量   |
| public void remove()       | 移除当前线程绑定的局部变量   |

`set方法`

```java
public void set(T value) {
    //获取当前线程
    Thread t = Thread.currentThread();
    //获取此线程维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    //判断是否为空
    if (map != null)
        //不为空加入
        map.set(this, value);
    else
        //为空创建加入
        createMap(t, value);
}
```

`get方法`

```java
public T get() {
    //获取当前线程
    Thread t = Thread.currentThread();
    //通过当前线程获取ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 以当前的ThreadLocal 为 key，调用getEntry获取对应的存储实体e
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //获取值
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
} 
```

`remove方法`

```java
public void remove() {
    //通过当前线程获取ThreadLocalMap
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

通过对这三个核心方法的分析，发现都非常简单，而这种简单是依靠于ThreadLocalMap的，分析ThreadLocalMap才是我们的关键

# 五、ThreadLocalMap

ThreadLocalMap没有实现map接口，而是独立实现了一个map，下面是它的继承结构

![在这里插入图片描述](http://cdn.noteblogs.cn/20210124143602480.png)

`成员变量`

```java
/**
         * The initial capacity -- MUST be a power of two.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;

        /**
         * The number of entries in the table.
         */
        private int size = 0;

        /**
         * The next size value at which to resize.
         */
        private int threshold; // Default to 0

```

与HashMap相似的是ThreadLocalMap的初始容量INITIAL_CAPACITY也是16，table是一个Entry数组，里面包含了key和value，size是当前table数组的大小，threshold表示扩容阈值

`Entry`

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

可见，Entry是继承WeakReference，Entry的key是一个弱引用，简单的复习四种引用：

> 1. 强引用：是指创建一个对象并把这个对象赋给一个引用变量，我们一般创建的都是强引用
> 2. 软引用：如果一个对象具有软引用，那么只要内存空间足够，垃圾回收器就不会回收它，软引用需要使用SoftReference来实现
> 3. 弱引用：如果一个对象具有弱引用，那么不管内存空间够不够，垃圾回收器都会回收它，弱引用需要使用WeakReference来实现
> 4. 虚引用：虚引用不影响对象的生命周期，如果一个对象与虚引用关联，那么垃圾回收器任何时候都有可能回收它

## 5.1 弱引用和内存泄漏

在使用ThreadLocal时有时会出现内存泄漏的问题，首先先分清楚两个概念：

- 内存溢出：通常报出OOM错误，也就是没有足够的内存给申请者使用
- 内存泄漏：指程序中已经动态分配的内存出于某种原因无法进行正常回收，导致这篇内存无法回收也无法进行使用，导致系统资源的浪费

### 5.1.1 key是强引用怎么办

那么思考，如果key使用了强引用会导致内存泄漏吗？

假如在代码中，ThreadLocal使用完了，那么ThreadLocalRef被回收了，但是不要忘记了ThreadLocalMap是和线程绑定的，也就是说如果线程还在工作，那么即使Entry中的key(ThreadLocalRef)被回收了，这个Entry也是不会被回收的

![在这里插入图片描述](http://cdn.noteblogs.cn/20210124144330949.png)

那么在ThreadLocal使用完成之后，依然存在引用链：CurrentThreadRef --> CurrentThread --> Map --> Entry，那么Entry就造成了内存泄漏

### 5.1.2 key是弱引用怎么办

那么key是弱引用能避免上面那个问题吗？不能

仔细想想，key是否是强弱引用对上述出现的问题没有影响，key是强弱引用只影响ThreadLocal是否正确的被回收，但是Entry对象还是存在的，在ThreadLocal的get/setEntry方法中，存在一层判断，如果当前的key为空，会将value也置为空，也就是说如果没有移除Entry对象，Entry就不会被回收，就造成了Entry的内存泄漏

通过上面两个问题的分析，可以发现出现内存泄漏的根本原因是ThreadLocalMap的生命周期是和线程绑定在一块的，如果没有手动的删除Entry对象，那么就只有让线程结束，但是我们一般使用了池技术线程是不会结束的，就造成了内存泄漏

### 5.1.3 为什么是弱引用

其实在上面已经解释了，如果key是强引用，那么在业务代码中使用完ThreadLocal时，想要清除掉ThreadLocalRef，但是抱歉，Entry中的key强引用于ThreadLocal，因此在进行可达性分析的时候，ThreadLocalRef依然可达无法进行回收，造成内存泄漏

如果key是弱引用，那么在业务代码使用完ThreadLocal的时候，由于是弱引用作用在ThreadLocalRef，这个时候进行可达性分析GC就会将ThreadLocalRef进行回收，对于的value在下一次ThreadLocalMap调用set、get、remove方法时就会被清理掉，从而在一定程度上可以避免内存泄漏

## 5.2 hash冲突

ThreadLocalMap还有比较有意思的一点就是它使用的解决hash冲突的方法，在HashMap中解决hash冲突使用的链地址法，而在ThreadLocalMap中解决hash冲突的方法是`开放地址法`，具体可以查看一下ThreadLocalMap中的set方法

```java
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    //计算下标，可见也要保住长度为2次幂
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= thold)
        rehash();
}
//当发生hash冲突时，具体的开放地址法的算法
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

从set方法可以很明显的看到ThreadLocalMap是使用线性地址法来解决hash冲突的，并且当出现hash冲突时判断下一个位置是否有hash冲突，那么这里可以理解Entry为一个环形数组