# 一、初始化方法

ArrayList与数组的最大区别是，ArrayList实现了动态扩容而数组的大小是固定的，ArrayList的底层是数组实现的

![image-20210321102453535](http://cdn.noteblogs.cn/image-20210321102453535.png)

下面先来看看ArrayList的一些变量

```java
	//默认大小
    private static final int DEFAULT_CAPACITY = 10;

	//空数组对象
    private static final Object[] EMPTY_ELEMENTDATA = {};

	//默认无参构造器的空数组，与EMPTY_ELEMENTDATA区分开是为了知道何时扩容了多少当第一个元素添加的时候
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

	//存放数据的缓存变量，在添加第一个元素的时候将被扩展为DEFAULT_CAPACITY
    transient Object[] elementData; // non-private to simplify nested class access

	//元素数量
    private int size;

```

ArrayList具有三个初始化方法

```java

    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

- 如果不传入初始容量，则使用默认容量，设置`elementData`为`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`
- 传入初始容量，会判断initialCapacity的值是否大于0，如果大于0就新new一个数组，等于0就直接设置为

EMPTY_ELEMENTDATA

- 传入一个Collection，则先调用toArray将elementData赋值给它，同样会判断它的长度是否为0，如果为0，设置`elementData`为`EMPTY_ELEMENTDATA`。

# 二、动态扩容

首先是来到add方法

```java
public boolean add(E e) {
    //确保容量足够
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //添加元素
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
    // 判断elementData是不是默认的空数组
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 取得两个参数中的最大值：DEFAULT_CAPACITY --> 10
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    //记录变更的次数与线程的安全性有关
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

当当前需要的minCapacity容量大于elementData.length的时候就需要进行扩容，也就是说整个扩容的核心方法在grow上，

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //新的容量大小为：之前容量的大小 + (之前容量的大小 / 2) 注：“>>”的意思为除以2的1次方
    int newCapacity = oldCapacity + (oldCapacity >> 1);
     // 扩容后的容量小于前容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 扩容后的容量大于arraylist最大容量时
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

首先搞清楚三个关键变量的含义：

- minCapacity：这次扩容需要的最小容量
- oldCapacity：扩容前原始数组容量
- newCapacity：扩容后的预计到达的容量

这里面最大的疑点就是newCapacity - MAX_ARRAY_SIZE

```java
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

MAX_ARRAY_SIZE的值是 Integer.MAX_VALUE - 8，这一块如果看过我的Synchronized锁实现原理就应该清楚，虚拟机对于数组对象会在对象头里面额外存放一个32bit也就是8Byte的数组大小，那么这个减8也就能理解

| 长度     | 内容                   | 说明                       |
| -------- | ---------------------- | -------------------------- |
| 32/64bit | Mark Word              | 存储对象的hashcode或锁信息 |
| 32/64bit | Class Metadata Address | 存储对象类型数据的指针     |
| 32/32bit | Array length           | 数组的长度                 |

接着往下追hugeCapacity方法

```java
private static int hugeCapacity(int minCapacity) {
    //溢出了 则变为负数
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

发现这里做了一个判断minCapacity > MAX_ARRAY_SIZE，也就是说如果minCapacity (当前需要的容量)大于MAX_ARRAY_SIZE则直接将数组容量赋值为Integer.MAX_VALUE？那前面不是说数组的最大容量为Integer.MAX_VALUE - 8，怎么这会又直接复制了呢？注意到MAX_ARRAY_SIZE的源码解释上说是有些虚拟机，如果这个时候minCapacity 确实大于MAX_ARRAY_SIZE，那么虚拟机也不管了，直接赋值到Integer.MAX_VALUE

# 三、快速失败机制

Java中的集合提供了一种叫做fail-fast的错误机制，它只能用来检测错误，当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast机制

例如在迭代的过程中调用remove方法就会抛出ConcurrentModificationException

```java
public class FastFailTest {
    public static void main(String[] args) {
        // 构建ArrayList
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        for (int i : list) {
            System.out.println(i);
            list.remove(1);
        }
    }
}
```

上面方法在迭代的过程汇总使用了remove方法，会抛出异常，原因是在迭代的过程中会检查modCount的值，但是调用remove方法会导致modCount++，从而判断两次的modCount不一样就抛出异常

正确的做法应该是使用迭代器的remove方法，因为使用迭代器的remove方法会修改expectedModCount的值，这样就不会触发fail-fast机制

在高并发的环境下，ArrayList是线程不安全的也会触发ail-fast机制，具体解决可以参考[多线程从入门到高级(4)---多线程环境下的集合安全类-List](https://blog.csdn.net/weixin_44706647/article/details/114829889)

