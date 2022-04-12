来源：Java核心技术、网上博客

### 一、数组的概念

数组是一种数据结构， 用来存储同一类型值的集合。通过一个整型下标可以访问数组中 

的每一个值。二维数组是指数组中的数组，二维数组的每一个元素是一个一维数组

### 二、定义数组的三种方式

##### 1.一维数组

```java
// 数据类型[] 数组名称 = new 数据类型 [长度];
int[] arr1 = new int[5];
int[] arr2 = new int[]{1,2,3,4,5};
int[] arr3 = {1,2,3,4,5};
```

注意：

- int[] 变量名也可以写成int 变量名[]
- 创建一个整形数组时，其中的元素会被默认初始化为0，boolean 数组的元素会初始化为 fals% 对象数组的元素则初始化为一个特殊值 null, 这表示这些元素（还）未存放任何对象。

##### 2.二维数组

```java
int[][] arr = new int[3][5];
int[][] arr = {{2,5},{1},{3,2,4},{1,7,5,9}};
```

### 三、数组的遍历

##### 1.一维数组

数组的遍历有1.for,  2.while,   3.do…while,   4.增强for循环(foreach)，4种方法，这里主要说明第4种。

 for(数据类型 变量：数组（集合）){

​		输出（变量）；

}

```java
int[] a = {2,1,3,4};
for (int temp : a) {
    System.out.println(temp);
}
```

##### 2.二维数组

二维数组本质上和一维数组的遍历方式一致，主要搞清楚，什么代表行数，什么代表列数

```java
int[][] arr = {{1,2},{3,4},{5,6}};
    for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr[i].length; j++) {
    	System.out.println(arr[i][j]);
    }
}
```

### 四、数组的常用方法

操作数组的常用类是Arrays，注意不是Array

>Array类主要提供了动态创建和访问 Java 数组的方法。
>
>Arrays包含用来操作数组（比如排序和搜索）的各种方法。此类还包含一个允许将数组作为列表来查看的静态工厂

##### 1.toString

返回指定数组的内容的字符串表示形式

```java
// static String toString(Object[] a) 
int[] arr3 = {1,2,3,4,5};
String s = Arrays.toString(arr3);
System.out.println(s);
```

##### 2.aslist

返回由指定数组支持的固定大小的列表

static <T> List<T>  asList(T... a)  

```java
String[] str = new String[]{"hello","java"};
List<String> strings = Arrays.asList(str);
```

注意：

- 看似这个方法返回一个List类型，但其实追踪到其源码

```Java
public static <T> List<T> asList(T... a) {
	return new ArrayList<>(a);
}
```

好像是返回了一个ArrayList集合，但是打开这个ArrayList发现这是Arrays自己包下的，所以这就导致了，这个集合不能进行add()、remove()、clear()等方法

![](C:\Users\four and ten\Desktop\笔记\JAVA\数组\1.png)

- 该方法适用于对象型数据的数组（String、Integer...），如果是基本类型的数组则无法进行转换

```java
System.out.println(Arrays.asList(new String[]{"a","b"}));
System.out.println(Arrays.asList(new Integer[]{3,4}));
System.out.println(Arrays.asList(new int[]{1,2}));
//输出
[a, b]
[3, 4]
[[I@1b6d3586]
```

- 该方法将数组与List列表链接起来：当更新其一个时，另一个自动更新

##### 3.arraycopy

这个方法是System包下的，方法原型是

```java
   public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

很显然这是一个本地方法，从src数组的下标为srcPos的索引开始，到后面length个元素放到dest数组的下标为destPos的索引开始的位置。注意：这里不能超出数组dest的最大存储空间，否则报ArrayIndexOutOfBoundsException异常

```java
int[] a = new int[]{1,2,3,4,5};
int[] b = new int[]{6,7,8,9,0};
//将a数组下标为2的数开始，长度为3结束，复制到数组b下标为3的数开始，也就是输出为[6,7,8,3,4]
System.arraycopy(a,2,b,3,2);
for (int i = 0; i < b.length; i++) {
	System.out.println(b[i]);
}
```

##### 4.copyOf和copyOFRange方法

这两个方法是Arrays包下的。方法原型是：

- static short[] copyOf(short[] original, int newLength) (注意：这个方法有很多重载)

从数组的第一个元素开始复制，复制长度为length，若长度超过数组原长，则超出元素为默认值0

该方法返回一个数组

```java
int[] c = Arrays.copyOf(a, 3);
for (int i = 0; i < c.length; i++) {
	System.out.println(c[i]);
}
```

- public static double[] copyOfRange(double []original,int from,int to) (注意：这个方法有很多重载)

original下标为from的位置开始复制，到to-1的位置结束，返回一个长度为to-from的数组

##### 5.数组的排序sort

这个方法是Arrays包下的

public static void sort(doule a[]) 将数组按升序全排序
public static void sort(doule a[],int start,int end);从索引为start到索引为end-1的位置，升序排序

```java
int[] a = new int[]{5,4,3,7,2};
Arrays.sort(a);
for (int i = 0; i < a.length; i++) {
	System.out.print(a[i] + " ");
}
```

但是如果要对该数组进行降序排序呢，可以使用sort的另外一个重载方法，传入一个实现了Comparator接口，重写了compare方法的实现类。方法原型是：

public static <T> void sort(T[] a, Comparator<? super T> c) 

```java
//现了Comparator接口，重写了compare方法的实现类
class MyComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 > o2 ? -1 : 1;
    }
}
public class demo1 {
    public static void main(String[] args) {
     Integer[] c = new Integer[]{3,2,7,1,5};
        System.out.println();
        Arrays.sort(c,new MyComparator());
        for (int i = 0; i < c.length; i++) {
            System.out.print(c[i] + " ");
        }
    }
}
```

当然也可以使用java.util.Collections包下的reverseOrder()，这个方法也返回一个Comparator实现类

```java
Integer[] b = new Integer[]{3,2,7,1,5};
    Arrays.sort(b,Collections.reverseOrder());
    for (int i = 0; i < b.length; i++) {
    System.out.print(b[i] + " ");
}
```

注意：这里的数组类型不能是基本数据类型，显然也用到了自动解装箱。

##### 6.binarySearch

在数组中查找一个数的方法，看这个名字很显然使用了二分搜索算法，方法原型是

public static int binarySearch(double [] a,double number)

```java
int a[] = {2,1,3,4};
int i = Arrays.binarySearch(a, 3);
System.out.println(i);
//输出结果是2
```

查看一下该方法的源码

```java
// Like public version, but without range checks.
private static int binarySearch0(int[] a, int fromIndex, int toIndex,
int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
    int mid = (low + high) >>> 1;
    int midVal = a[mid];

    if (midVal < key)
    low = mid + 1;
    else if (midVal > key)
    high = mid - 1;
    else
    return mid; // key found
    }
    return -(low + 1);  // key not found
}
```

