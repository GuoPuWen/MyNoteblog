# 一、HashSet线程安全？

HashSet的底层是HashMap，由前面的文章可以知道HashMap是线程不安全的，那么理所应当的HashSet也是线程不安全的，例如可看如下代码：

```java
public class HashSetDemo {
    public static void main(String[] args) {
        Set<Integer> set = new HashSet<>();
        for (int i = 0; i < 30; i++) {
            final int temp = i;
            new Thread(() -> {
                set.add(temp);
                System.out.println(set);
            },String.valueOf(temp)).start();
        }
    }
}
```

![image-20210316105612607](http://cdn.noteblogs.cn/image-20210316105612607.png)

同样报出ConcurrentModificationException异常，可见HashSet也是线程不安全的，同样，如何解决HashSet的线程安全问题？

# 二、SynchronizedSet

和List、map解决线程安全的方式一样，Collections类下也为我们提供了一个线程安全的Set，那就是SynchronizedSet这个类，SynchronizedSet这个类和SynchronizedList的套路一致，同样是对传入的HashSet的各个方法增加Synchronized锁



# 三、CopyOnWriteArraySet

Set作为一个不重复的List，那么List具有CopyOnWriteArrayList只需要其稍加改变就可以变为CopyOnWriteArraySet，而CopyOnWriteArraySet也正是这么做的，

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;
```

可以看到CopyOnWriteArraySet的底层数据结构就是CopyOnWriteArrayList，CopyOnWriteArrayList是采用写时复制、读写分离这种思想的，所以CopyOnWriteArraySet在特定的写少读多的场景下，也具有性能高的优点

看看CopyOnWriteArraySet的add方法

```java
public boolean add(E e) {
    return al.addIfAbsent(e);
}
private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        if (snapshot != current) {
            // Optimize for lost race to another addXXX operation
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                return false;
        }
        Object[] newElements = Arrays.copyOf(current, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

这段代码也很好理解就是首先检查原来的数组里面有没有要添加的元素，如果有的话就不要再添加了，如果没有的话，创建一个新的数组，复制之前数组元素并且添加新的元素

接着看看remove方法，也是直接调用CopyOnWriteArrayList的remove方法

```java
public boolean remove(Object o) {
    return al.remove(o);
}
```

