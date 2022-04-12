# 一、HashMap线程安全？

## 1.1 JDK1.7

在jdk1.7中HashMap在多线程环境下有可能出现循环链表死循环的问题，主要是因为JDK1.7是采用头插法，当两个线程同时进入到 transfer方法时，如果其中一个线程被挂起而另外一个线程完成了transfer方法时，可能就会造成线程安全问题，这里可以参考https://www.coolshell.cn/articles/9606.html这篇文章写的

## 1.2 JDK1.8

在jdk1.8中解决了上面那个问题，将头插法改为了尾插法，但是在高并发环境下还是具有线程安全问题，具体可看如下代码：

```java
public class HashMapDemo {
    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        for (int i = 0; i < 30; i++) {
            final int j = i;
            new Thread(() -> {
                map.put(j, UUID.randomUUID().toString().replace("-", ""));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```

![image-20210315192647489](http://cdn.noteblogs.cn/image-20210315192647489.png)

运行上面代码直接会抛出ConcurrentModificationException异常，HashMap是线程不安全的，那么线程安全的map有哪些呢？

# 二、HashTable

HashTable解决了HashMap的线程不安全的问题，得益于HashTable在每个方法上都加上了Synchronized，看看它的put和get方法

```java
public synchronized V get(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return (V)e.value;
        }
    }
    return null;
}

public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```

很显然，HashTable是对方法级别进行加锁，所以在高并发的环境下只能有一个线程操作HashTable，get和put方法也不能同时进行，所以Hashtable的效率很低，一般不使用HashTable作为线程安全的Map集合

下面在陈述一下HashMap和HashTable两者的区别：

- 在线层是否安全方面，HashMap是线程不安全的，而HashTable是线程安全的，HashTable的大部分方法都被加入Synchronized方法修饰
- 在效率方面，因为HashTable是方法级别的加锁，所以很影响效率，效率比较低
- 对null的处理：`HashMap`可以存储key和value都为null，但是key如果为null只能存一份，`HashTable`不允许key为null和value为null
- 底层数据结构：`HashMap`的底层使用数组＋链表 + 红黑树，但是HashTable没有这么复杂的机制
- 初始容量：`HashMap`如果不指定初始容量那么为16，`HashTable`不指定初始容量则大小为11，另外如果给HashMap指定了初始容量，HashMap会将容量变为大于当前值的最小2次幂
- 扩容方面：`HashMap`扩容会变为原容量的两倍，而`HashTable`则扩容为2n+1倍

# 三、SynchronizedMap

Collections也同样提供了一个线程安全的map集合，SynchronizedMap的实现也比较简单，可以看看源码

```java
public V get(Object key) {
    synchronized (mutex) {return m.get(key);}
}

public V put(K key, V value) {
    synchronized (mutex) {return m.put(key, value);}
}
```

可以看出来SynchronizedMap的实现方式也是加个对象锁，这种与HashTable好不到那里去，也是不建议使用的

# 四、**ConcurrentHashMap** 

HashTable在高并发的环境下表现效率低下的原因是所有访问HashTable的线程都必须竞争同一把锁，但是如果容器中有多把锁，每一把锁锁住其中的一部分数据，那么高并发下多线程访问容器中的不同数据时，线程之间就不会发生锁的竞争，从而可以提高多线程之间的访问效率，**ConcurrentHashMap** 使用的便是锁的分段技术，首选将数据分成一段一段存储，然后给每一段数据加上一把锁，当一个线程占用锁的时候，其他段的数据不是这把锁锁住的，所以可以被其他线程同时访问

ConcurrentHashMap在jdk1.6、1.7、1.8之间的变化比较大，这里也只说明jdk1.7和jdk1.8版本的ConcurrentHashMap的一些底层原理

## 4.1 JDK1.7

jdk1.7中的ConcurrentHashMap底层数据结构是数组＋链表，但是因为要分段， 所以ConcurrentHashMap额外维护了一个Segment的数组，在Segment里存在一个HashEntry用于存放真正的数据，结构示意图如下：

![JAVA 7 ConcurrentHashMap](http://cdn.noteblogs.cn/concurrenthashmap_java7.png)

### 	put方法

总体思路是：计算key在哪一个Segment上，然后再计算key在HashEntry的下标。得到下标之后在将值和处理HashMap一样的方式处理

**ConcurrentHashMap** 的hash方法也加了一系列的扰动函数目的还是为了增大hash值减少hash冲突，

```java
private int hash(Object k) {
    int h = hashSeed;

    if ((0 != h) && (k instanceof String)) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // Spread bits to regularize both segment and index locations,
    // using variant of single-word Wang/Jenkins hash.
    h += (h <<  15) ^ 0xffffcd7d;
    h ^= (h >>> 10);
    h += (h <<   3);
    h ^= (h >>>  6);
    h += (h <<   2) + (h << 14);
    return h ^ (h >>> 16);
}
```

计算Segment的下标的方法为取哈希码的高sshift位，具体在ConcurrentHashMap的构造方法上

```java
this.segmentShift = 32 - sshift;
this.segmentMask = ssize - 1;
```

put方法为

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}

```

定位到Segment之后接着处理HashMap的方式去处理下标索引，其中Segment的数组大小和HashEntry数组大小都保持在2的次方幂其中原因和HashMap的数组长度为2的次方幂一致，另外点击put的重载方法可以看到使用了ReentrantLock进行加锁，Segment也是继承了ReentrantLock所以对于加锁解锁的操作非常方便

### get方法

对于读操作，是不需要加锁的，但是要保证数据的可见性，所以**ConcurrentHashMap** 将要使用到的变量都设置为`volitile`类型，定义成volatile的变量在线程之间能够保持可见性，能够被多线程同时读，并且不会读到过期的值

ConcurrentHashMap使用USAFE提供的getObjectVolatile方法直接从内存中读取最新的Segment值，这样读就可以并发的执行，同时读取HashEntry也是使用同样的方法

```java
s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)
HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
```

### size方法

因为ConcurrentHashMap的数据是分段的，那么如果要统计整个ConcurrentHashMap的数据大小就需要将每一个Segment数组中的元素求和，有一个办法在需要获取size的时候锁住全部的Segment，然后去统计数量，但是在这个过程中ConcurrentHashMap无法进行读写操作

ConcurrentHashMap没有选择上面这种方案，在每次累加count操作过程中，之前累加过的count发送变化的几率非常小，所以ConcurrentHashMap的做法是先尝试两次通过不加锁的方式去获取Segment大小，如果在统计过程中容器的count发生了变化，则在采用加锁的方式来统计所有的Segment的大小，并且ConcurrentHashMap使用modCount来判断在统计的时候容器是否发生了变化，因为在put、remove、clean方法操作元素都会将变量modCount进行加1

## 4.2 JDK1.8

ConcurrentHashMap1.7版本实现了分段锁，也就是说理论上Segment数组有多少，ConcurrentHashMap1.7可以支持的并发数就有多少，但是为了进一步提升ConcurrentHashMap1.7的效率在JDK1.8的时候放弃了分段锁的方案，而是直接使用一个较大的数组，对首节点进行加锁，同时为了提升查找效率JDK1.8版本的ConcurrentHashMap在链表长度超过一定的阈值(8)之后将链表转为红黑树，将寻址时间复杂度从N降为LogN

同时锁从ReentrantLock换到了Synchronized，这点让人疑惑？Synchronized不是重量锁吗，其实JDK官方对Synchronized进行了一些优化，在ConcurrentHashMap这种锁的粒度比较低的情况下Synchronized的性能并不会比ReentrantLock更差

### put方法

ConcurrentHashMap1.8的put方法的流程如下：

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    	//如果key和value有一个为null直接抛出异常
        if (key == null || value == null) throw new NullPointerException();
    	//计算hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                //如果数组没有初始化，那么初始化数组调用initTable方法
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //如果没有hash冲突，则直接使用CAS进行插入
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //如果还在进行扩容操作，就先进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //当前为红黑树
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```



1. 如果key和value有一个为null直接抛出异常
2. 计算hash值，这点计算hash的时候没有进行大量的扰动，这是因为jdk1.8已经变为了红黑树的结构，即使hash冲突严重，也有比较高的查询效率

```java
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```

3. 如果数组没有初始化，那么初始化数组调用initTable方法
4. 如果没有hash冲突，则直接使用CAS进行插入

```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

5. 如果还在进行扩容操作，就先进行扩容
6. 如果存在hash冲突，就加锁来保证线程安全，这里需要判断是不是当前的结构已经是红黑树了
7. **如果链表的长度大于8则进入treeifyBin试图将链表转为红黑树，但是在treeifyBin方法中还有一层判断，就是判断当前数组的长度是否大于64这点和HashMap是一致的**

```java
static final int TREEIFY_THRESHOLD = 8;

private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        //当前数组长度大于64
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

8. 添加成功，调用addCount，统计size大小

### get方法

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    //计算当前key的hash值
    int h = spread(key.hashCode());
    
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        //判断是否是首节点，如果是首节点直接返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //1. 判断当前是否在正在扩容，如果是则调用扩容的find方法
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```



get方法的流程如下：

1. 计算当前key的hash值
2. 判断是否是首节点，如果是首节点直接返回
3. 判断当前是否在正在扩容，如果是则调用扩容的find方法
4. 以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null

### size方法

ConcurrentHashMap1.8版本不在像1.7那样只有调用了size方法才会去统计size的值，但是1.8版本可以发现在在进行了put操作之后就进行了addCount方法，也就是说ConcurrentHashMap1.8版本已经计算好了size的值，如果调用size()直接返回就可以