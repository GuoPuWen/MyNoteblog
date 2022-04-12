## 一、ArrayList的不安全

```java
public class ArrayListDemo {

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

运行如上方法，创建了30个线程写的同时也进行读，那么这将会抛出一个异常ConcurrentModificationException，出现异常的原因很简单，ArrayList集合类不是线程安全的，那么多个线程同时操作读和写那么将会抛出异常，那么问题来了？如何解决List集合下的线程安全问题？Vector？这是一个好的办法吗？

## 二、Vector

讲到解决List集合的安全问题，可能大部分同学都会回答Vector，Vector肯定可以解决List集合的线程安全问题，下面来简单的分析一下Vector是如何解决list集合线程安全问题的

Vector的底层和ArrayList一样也是使用了数组，下面贴出一些源码出来看看

add方法

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```

get方法

```java
public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

可以看到在Vector中都使用了Synchronized关键字，这样做的后果就是效率非常低

## 三、SynchronizedList

说到线程安全的List，Collections包下的SynchronizedList不得不提一提，看这个类的名字就可以知道这是在List集合中加了Synchronized

看看这个类的一些方法

```java
static class SynchronizedList<E>
        extends SynchronizedCollection<E>
        implements List<E> {
        private static final long serialVersionUID = -7754090372962971524L;

        final List<E> list;

        SynchronizedList(List<E> list) {
            super(list);
            this.list = list;
        }
        SynchronizedList(List<E> list, Object mutex) {
            super(list, mutex);
            this.list = list;
        }

        public boolean equals(Object o) {
            if (this == o)
                return true;
            synchronized (mutex) {return list.equals(o);}
        }
        public int hashCode() {
            synchronized (mutex) {return list.hashCode();}
        }

        public E get(int index) {
            synchronized (mutex) {return list.get(index);}
        }
        public E set(int index, E element) {
            synchronized (mutex) {return list.set(index, element);}
        }
        public void add(int index, E element) {
            synchronized (mutex) {list.add(index, element);}
        }
        public E remove(int index) {
            synchronized (mutex) {return list.remove(index);}
        }

        public int indexOf(Object o) {
            synchronized (mutex) {return list.indexOf(o);}
        }
        public int lastIndexOf(Object o) {
            synchronized (mutex) {return list.lastIndexOf(o);}
        }

        public boolean addAll(int index, Collection<? extends E> c) {
            synchronized (mutex) {return list.addAll(index, c);}
        }

        public ListIterator<E> listIterator() {
            return list.listIterator(); // Must be manually synched by user
        }

        public ListIterator<E> listIterator(int index) {
            return list.listIterator(index); // Must be manually synched by user
        }

        public List<E> subList(int fromIndex, int toIndex) {
            synchronized (mutex) {
                return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                            mutex);
            }
        }

    }
```

可以看到的是在SynchronizedList是实现了List接口的，这也就意味着它的可扩展性，可以传入一个List对象，不过从上面方法中看出来还是对每一个add、get方法加入Synchronized锁的方式来实现线程安全的，所以SynchronizedList的效率也不是很高

官方文档中提到

```java
List list = Collections.synchronizedList(new ArrayList());
...
    synchronized (list) {
    Iterator i = list.iterator(); // Must be in synchronized block
    while (i.hasNext()){}
        foo(i.next());
}
```

很奇怪的是既然Synchronized是线程安全的，那么在迭代的时候又何必在加一层锁呢？在SynchronizedList的实现代码中没有对iterator()再次进行封装Synchronized也就是说如果使用迭代器进行读操作在高并发的情况下并没有实现线程安全，所以我们使用的话需要手动加一个Synchronized

所以SynchronizedList适用于不需要高性能、不使用Iterator的情况

SynchronizedList和Vector的主要区别如下：

1. Vector扩容为原来的2倍长度，ArrayList扩容为原来1.5倍
2. SynchronizedList有很好的扩展和兼容功能。他可以将所有的List的子类转成线程安全的类。
3. **使用SynchronizedList的时候，进行遍历时要手动进行同步处理 。**
4. SynchronizedList可以指定锁定的对象。



## 四、CopyOnWriteArrayList

既然Vector、SynchronizedList的性能都不是很高，那有没有性能高的呢？CopyOnWriteArrayList在特定的写少读多的情况下性能比上面两个集合高很多

CopyOnWriterArrayList是在java.util.concurrent包下的，是JDK为我们提供的一个在高并发环境下的高性能List。CopyOnWriterArrayList是其实也是一种读写分离的思想，CopyOnWriterArrayList在add的时候会先复制一份容器，然后往容器里添加完元素之后再将原容器的引用指回新的容器，这种做法的好处是在读的时候不需要加锁，可以高并发的进行读

下面看看add源码

```java
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array
    
    
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

源码写的非常清楚，首先获取当前数组，然后使用Arrays的copyOf方法复制当前数组一份，由于这是局部变量这只是存在当前线程的工作区的，所以对这个复制后的容器的操作一定是线程安全的，接着添加元素，然后在将新的容器的地址给原先的容器

get方法就使用索引返回数据，所以CopyOnWriteArrsyList读是可以并发执行的，而写时加了锁的。所以这种容器非常适用于读多写少的情况

还有一个细节需要关注，CopyOnWriteArrayList中的数组是加了Volatile修饰的，这样做的目的是要及时的保证可见性，要不然复制为新的数组之后其他线程看不到，这样就麻烦了！

总体来说，CopyOnWriteArrayList是一种读写分离的思想，就是读和写不在一个容器上，这样做不能保证数据的实时一致性，但是可以保证数据得到最终一致性，也只适用于读多写少的情况，如果是写多的情况那么copy一份新的数组给内存消耗是很大的