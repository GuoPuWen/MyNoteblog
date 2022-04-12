# 一、概述

ArrayList实现了List接口，继承了AbstractList。ArrayList的底层是数组，允许大小可以变化，包括null值。

ArrayList的继承结构：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

RandomAccess是随机访问，Cloneable支持复制，java.io.Serializable可以进行***反序列和序列化***操作

# 二、重要源码

## 1.底层是数组

```java
transient Object[] elementData;
private int size;
```

这里可以看出ArrayList底层是用了一个数组，这里直接使用了泛型擦除。

## 2.重要方法(JDK12)

1. **add(E)  插入元素**

   

   ```java
   public boolean add(E e) {
       modCount++;
       add(e, elementData, size);
       return true;
   }
   private void add(E e, Object[] elementData, int s) {
       if (s == elementData.length)
           elementData = grow();
       elementData[s] = e;
       size = s + 1;
   }
   ```

- 这里add方法中的modCount域，对ArrayList内容的修改都将增加这个值，在迭代器初始化时会将这个值赋值给迭代器的expectedModCount,在迭代过程中，判断modCount与expectedModCoun是否相等，判断是否有其他线程修改了ArrayList.

- 当调用add方法时会调用下面这个add方法，这个方法可以判断是否需要扩容
- 一些扩展性的方法
  - add(int , E) 在指定位置插入元素
  - E set(int , E)在指定位置替换元素，返回之前的元素

2. remove(int) 

   ```java
   public E remove(int index) {
       Objects.checkIndex(index, size);
       final Object[] es = elementData;
   
       @SuppressWarnings("unchecked") E oldValue = (E) es[index];
       fastRemove(es, index);
   
       return oldValue;
   }
   private void fastRemove(Object[] es, int i) {
   	modCount++;
       final int newSize;
       if ((newSize = size - 1) > i)
       System.arraycopy(es, i + 1, es, i, newSize - i);
       es[size = newSize] = null;
   }
   //将src数组索引下标为srcpos的位置开始复制length个数据到dest数组索引下标destpos的位置
   public static native void arraycopy(Object src,  int  srcPos,
                                           Object dest, int destPos,
                                           int length);
   ```

   

remove的核心是采用的是System.arraycopy方法，arraycopy复制时需要将复制的内容先保存到一个临时的表里，然后在移动。remove方法在使用时，要特别注意它的底层是靠System.arraycopy。看下面这个例子

```java
package cn.ArrayList;
import java.util.ArrayList;

/**
 * 测试ArrayList的Remove
 * @author四五又十
 * @create 2020/1/30 18:58
 */
public class ArrayListDemo1 {
    public static void main(String[] args) {
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
        }
        System.out.println(list);
        //②i--删除
        for (int i = list.size()-1; i >= 0 ; i--) {
            if(list.get(i) == 2){
                list.remove(i);
            }
        }
        System.out.println(list);
    }
}
```

当使用第一种时会出现删除不干净的情况，即输出结果是[1,2,3]，因为当使用remove方法时，size也减1

