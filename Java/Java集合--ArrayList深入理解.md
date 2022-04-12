来源：Java核心技术，博客

### 一、概述

ArrayList是我们很常用的一个集合类，但是其底层却是有很丰富的代码逻辑。Arraylist的底层源码解读开始。

##### 1.类注释解读

借助有道翻译，对照ArrayList的类注释(JDK1.8)大概可以得出以下几点

- including null ，允许null值
- its capacity grows automatically，会自动扩容
- add、size、isEmpty、get、set的时间复杂都都是O(n)
- 不是线程安全的
- 增强for循环，或使用迭代器迭代过程中，如果数组大小被改变，会快速失败，抛出异常



##### 2.ArrayList实现的接口

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable,java.io.Serializable
```

- RandomAccess：表明其**支持快速随机访问**，也就是说类似于数组一样，能够通过下标访问，也确实如此，ArrayList的底层技术就是数组。但是ArrayList的随机访问形式是通过get()方式。LinkedList没有实现这个RandomAccess，因为LinkedList的底层是链表。

> 数组支持随机访问， 查询速度快， 增删元素慢； 链表支持顺序访问， 查询速度慢， 增删元素快。所以对应的 `ArrayList` 查询速度快，`LinkedList` 查询速度慢，

- Cloneable：允许在堆中克隆出一块和原对象一样的对象，并将这个对象的地址赋予新的引用，这样显然我对新引用的操作，不会影响到原对象。
- Serializable：实现这个接口以便序列化

##### 

##### 3.ArrayList整体架构

ArrayList的底层实现是数组，对于一个集合首先要想到的就是CRUD的操作。源码解读也是围绕这个CRUD的操作来展开的，当然还有ArrayList的扩容操作。在ArrayList的源码解读中，有3个属性或者说是概念需要了解：

- DEFAULT_CAPACITY ：在JDK1.8中该值为10，这个常量表示数组的初始大小
-  size：表示当前数组的大小，没有使用volatile修饰，是非线程安全的
- modCount：统计当前数组的被修改的次数，如果数组结构有变动则加1



### 二、源码解析

##### 1.初始化

ArrayList有3种方式初始化一个ArrayList：分别是无参数、指定初始化大小、指定初始化数据。

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

注意：ArrayList在使用无参构造器时，默认大小时空数组，我之前一直以为一初始化就将默认大小设置为10，设置为10时在第一次使用add的时候，后面说到add的源码的时候可以很好的明白这一点

##### 2.add和扩容方法

ArrayList是由自动扩容的功能的，也就是说在每次使用add方法时，都需要判断是否需要扩容

```java
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
}
```

很显然，扩容要2步：

1. 判断是否需要扩容
2. 直接给数组添加值

```java
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
//初始化大小时，若没有给定初始值，那么初始大小为10
private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
        //记录数组被修改
    	modCount++;

        // 如果期望的最小容量大于当前数组的容量，那么扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
    	//oldCapacity >> 1是指oldCapacity/2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
    	//扩容后的值 <  期望的大小值，那么扩容后的值为期望值minCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
    	//如果扩容后的值超过了jvm能分配的最大值，就用Integer的最大值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
    	//复制扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
}
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```

这里需要注意：

- 扩容的规则是扩大1.5倍，这个数是因为。JDK开发人员通过大量的数据发现1.5倍是最好的，因为如果一次性扩太多，那么容易浪费更多的内存，如果一次性扩容太小，那么需要多次扩容，对性能消耗比较严重
- Arraylist会检查扩容后的值，这个值不能小于当前值，而且不能大于Integer.MAX_VALUE
- 前面说过，ArrayList中可以添加null值，在源码中发现ArrayList并没有去检查添加的值是否为null
- 扩容完成之后，直接使用elementData[size++] = e进行扩容，这种简答的扩容，没有锁进行控制，所以是线程不安全的

扩容本质探究

扩容的主要代码是```Arrays.copyOf(elementData, newCapacity);```，而我们知道copyOf的底层使用的是System.arraycopy方法

当然add还有一个重载的方法

```java
public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
}
```

这个方法先检查输入的下标合不合法，如果合法检查是否需要扩容，最后也是用System.arraycopy方法来完成通过下标插入数据的核心操作。

##### 3.删除

###### 1.通过值来删除

```java
public boolean remove(Object o) {
    //如果删除的是null值，则遍历找到第一个null值并删除
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        //不为null，找到第一个和要删除的值相等的值删除
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

- 因为新增的时候可以添加null值，所以删除时也可以删除null值
- 通过值来查找下边是通过equals方法的，所以如果数组不是基本类型以及包装类，要注意对equals方法的重写

- 需要注意，这里remove使用的仍然是System.arraycopy方法来进行删除操作，而且最后size会--

```java
ArrayList<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(2);
    list.add(3);
//①i++删除
    for (int i = 0; i < list.size(); i++) {
        if(list.get(i) == 2){
            list.remove(i);
        }
        System.out.println(list);  
}
```

注意这个输出结果，看似应该没有问题，但是输出结果是[1, 2, 3]，问题就出在remove使用的仍然是System.arraycopy的方式，当每删除一个元素时size也--了，应该要使用下面的方式

```java
for (int i = list.size()-1; i >= 0 ; i--) {
    if(list.get(i) == 2){
        list.remove(i);
    }
}
System.out.println(list);
```

###### 2.通过下标来删除

通过下标来删除和通过值来删除大同小异

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
    numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

##### 4.迭代器

如果要有迭代器只需要实现Iterator接口即可

```java
private class Itr implements Iterator<E> {}
```

ArrayList使用内部类的方式实现了Iterator接口。在这个类里面有3个成员变量

```java
//迭代过程中的下一个元素的位置
int cursor;       // index of next element to return
//表示上一次迭代过程中索引的位置，如果是删除则为-1
int lastRet = -1; // index of last element returned; -1 if no such
int expectedModCount = modCount;
```

看迭代器源码无非是看它的3个方法：next、hasNext、remove



###### 1.hasNext

```java
public boolean hasNext() {
    return cursor != size;
}
```

hasNext方法很简单，判断是否变量cursor是否与size数组大小值相等，如果不相等返回true，表示还有元素可以迭代，如果相等，则返回false，没有元素可以迭代

###### 2.next

```java
public E next() {
    //判断版本号有无修改，如果修改则抛出异常
    checkForComodification();
    //本次迭代过程中，元素的索引位置
    int i = cursor;
    if (i >= size)
    throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
    throw new ConcurrentModificationException();
    //下一次迭代过程中，索引的位置
    cursor = i + 1;
    //返回元素值
    return (E) elementData[lastRet = i];
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

###### 3.remove

在类注释解读中，其中有一条：增强for循环，或使用迭代器迭代过程中，如果数组大小被改变，会快速失败，抛出异常。

```java
Iterator<Integer> iterator = list.iterator();
    while(iterator.hasNext()){
        Integer next = iterator.next();
        if(next == 2){
        list.remove(2);
    }
}
```

​	很显然，这样会抛出一个异常ConcurrentModificationException，因为在使用迭代器的next方法时，checkForComodification()方法便是检查版本号有没有被修改。

​	使用迭代器提供的remove方法，则不会产生这个问题

```java
public void remove() {
    //如果数组的索引为0则无法进行删除抛出异常
    if (lastRet < 0)
    throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        //-1表示元素已经删除，防止重复删除操作
        lastRet = -1;
        //将modcount的值与expectedModCount一致，这样checkForComodification()判断时就不会报错
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
    	throw new ConcurrentModificationException();
    }
}
```

- lastRet = -1的目的是为了防止重复删除
- expectedModCount = modCount，删除了元素，modcount的值就会发生变化，这里吧expectedModCount 重新赋值一下，两者的值就相等