### 1.数组复制方法中效率高低？

java中常见的有以下几种数组复制的方法：

- for循环逐一复制
- System.arraycopy()
- System.copyof()
- 使用clone方法

先来说说这几个怎么用吧

```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

可见这是一个native方法：

- src - 源数组。
- srcPos - 源数组中的起始位置。
- dest - 目标数组。
- destPos - 目标数据中的起始位置。
- length - 要复制的数组元素的数量

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
public static int[] copyOfRange(int[] original, int from, int to) {
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    int[] copy = new int[newLength];
    System.arraycopy(original, from, copy, 0,
                     Math.min(original.length - from, newLength));
    return copy;
}
```

可见，copyOf方法底层调用的是 System.arraycopy方法，那么它们两的效率要比较的话肯定是arraycopy方法更高，copyOfRange本质上也是调用arraycopy方法

下面将通过程序来验证，上面

```java
public class test {
    public static final int size = 1000000;

    public static void copyByArrayCopy(String[] strArray){
        Long startTime = System.currentTimeMillis();
        String[] destArray = new String[size];
        System.arraycopy(strArray,0,destArray,0,strArray.length);
        Long endTime = System.currentTimeMillis();
        System.out.println("copyByArrayCopy cost time is "+(endTime-startTime));
    }

    public static void copyByLoop(String[] strArray){
        Long startTime = System.currentTimeMillis();
        String[] destArray = new String[size];
        for(int i = 0;i<strArray.length;i++){
            destArray[i] = strArray[i];
        }
        Long endTime = System.currentTimeMillis();
        System.out.println("copyByLoop cost time is "+(endTime-startTime));
    }

    public static void copyByClone(String[] strArray){
        Long startTime = System.currentTimeMillis();
        String[] destArray = strArray.clone();
        Long endTime = System.currentTimeMillis();
        System.out.println("copyByClone cost time is "+(endTime-startTime));
    }

    public static void main(String args[]){
        String arr1[] = new String[size];
        for(int i=0;i<size;i++){
            arr1[i] = "this is a test"+i;
        }
        String arr2[] = new String[size];
        for(int i=0;i<size;i++){
            arr2[i] = "this is a test"+i;
        }
        String arr3[] = new String[size];
        for(int i=0;i<size;i++){
            arr3[i] = "this is a test"+i;
        }
        copyByClone(arr1);
        copyByLoop(arr2);
        copyByArrayCopy(arr3);

    }
    public static void printArr(String[] strArray){
        for(String str:strArray){
            System.out.println(str);
        }
    }
}
```

执行结果如下：

```
copyByClone cost time is 5
copyByLoop cost time is 19
copyByArrayCopy cost time is 5
```

由此可见效率：

arraycopy() = clone() > for循环

