来源：《Java核心技术卷一》、博客

## 一、常用集合框架大图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123234055859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDcwNjY0Nw==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123234157320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDcwNjY0Nw==,size_16,color_FFFFFF,t_70)
## 二、常用集合分类
> 讨论Java中的集合无非就是看其：
> 1.底层数据结构
> 2.增删改查方式
> 3.初始容量，扩容方式，扩容时机
> 4.线程是否安全
> 5.是否允许为空，是否可以重复，是否有序

（1）**Collection**接口的接口对象集合
①List接口：元素按进入先后有序保存，可重复	
|接口实现类  | 描述 |
|--|--|
|LinkedList|底层链表，插入，删除较为容易，没有同步，线程不安全 |
|ArrayList|底层数组， 随机访问较为容易，没有同步，线程不安全|
|Vector|底层数组，同步，线程安全|
|Stack|Vector的实现类|
②Set接口
|接口实现类  | 描述 |
|--|--|
|Hashset|使用Hash表，元素无序且唯一，线程不安全|


## 三、Collection
**在java中Collection接口是集合类的基本接口。** 下面给出的表格是Collection中方法
|方法|说明| 
|--|--|
|boolean add(Object e)| 增加集合中的元素  |
|boolean remove(Object e)|从容器中移除元素（注意是从容器中|
|boolean contains(Object e)|容器中是否包含该元素|
| int size()| 容器中元素的数量|
| boolean isEmpty()|判断容器是否为空| 
| void clean()| 清空容器中的所有元素|
|Iterator iterator()|获得迭代器，用于遍历元素|
|boolean containsAll(Colections e)|本容器是否包含（容器中的所有元素）| 
|boolean addAll(Collection e|将容器e中所有元素添加到本容器中|
|boolean removeAll(Collection e)|移除本容器和容器e中都包含的元素|
|boolean retainAll(Collection e| 取本容器和容器e中都包含的元素|
|Object[] toArray()|转换成Object数组| 

## ArrayList
ArrayList实现List接口的动态数组（大小可变），实现了所有可选列表操作，允许null在内的所有元素。除了实现List接口外，还提供一些其他方法。
**1,底层数据结构**
	ArrayList的底层是一个Object数组
```java*
transient Object[] elementData;
```
**2.增删改查**
（1）增加元素时，①判断索引是否合法；②检测是否需要扩容；③使用System.arraycory()完成数组的复制。

>public static native void arraycopy(Object src,  int  srcPos,Object dest, int destPos,int length);
>将原数组src从secpos位置开始复制到dest数组中，数据从destPos开始，复制长度为length
```java
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
```

（2）删除元素时，同样判断索引是否合法，删除的方式是被删除元素右边的元素左移，同样是使用System.arraycory()方法
```java
    public E remove(int index) {
        Objects.checkIndex(index, size);
        final Object[] es = elementData;

        @SuppressWarnings("unchecked") E oldValue = (E) es[index];
        fastRemove(es, index);

        return oldValue;
    }
```
（3）ArrayList提供一个清空数组的方法，方法是将所有元素设置为null，让GC自动回收没有被引用的元素
```java
    public void clear() {
        modCount++;
        final Object[] es = elementData;
        for (int to = size, i = size = 0; i < to; i++)
            es[i] = null;
    }
```
（4）修改元素时，只需要检查下标即可
```java
    public E set(int index, E element) {
        Objects.checkIndex(index, size);
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```
（5）


