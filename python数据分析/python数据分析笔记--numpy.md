### 一、简介

 一个在Python中做科学计算的基础库，重在数值计算，也是大部分PYTHON科学计算库的基础库，多用于在大型、多维数组上执行数值运算

### 二、创建数组对象以及数组对象属性和方法

| 函数              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| arange            | 类似python内置的range，但返回的是一个ndarray而不是列表       |
| array             | 将输入的数据（列表、元祖、数组或其他序列类型）转换为ndarray。要么推断出dtype，要么显示指定dtype。默认直接复制输入的数据 |
| asarray           | 将输入的数据转换为ndarray，如果输入本身就是一个ndarray就不进行复制 |
| ones、ones_like   | 根据指定的形状和dtype创建一个全1的数组。ones_like以另一个数组为参数，并根据其形状和dtype创建一个全1的数组 |
| zeros、zeros_like | 类似于ones和ones_like ，只不过产生全0的数组                  |
| empty、empty_like | 创建新数组，只分配内存空间但是不填充任何值                   |
| eye、identity     | 创建一个正方的NxN单位矩阵(对角巷为1，其余为0)                |

| 属性/方法 | 说明                                   |
| --------- | -------------------------------------- |
| ndim      | 维度                                   |
| size      | 元素个数                               |
| shape     | 数组的形状                             |
| itemsize  | 每个元素所占的字节                     |
| nbytes    | 总占字节                               |
| real      | 针对复数类型的实数部分                 |
| imag      | 针对复数类型的虚数部分                 |
| dtype     | 用于指定或输出数组对象元素的类型       |
| reshape   | 分割数组成指定形状:data.reshape(2,3,4) |
| astype    | 类型转换:arr.astype(np.float64)        |

```python
import numpy as np
a = np.array([1,2,3,4,5])
b = np.array(range(1,6))
c = np.arange(1,6)
```

上面a，b，c的内容相同都为：

```python
array([1, 2, 3, 4, 5])
```

查看数据的类型：

```python
a.dtype
dtype('int32')
```

- 创建多维数组

```python
np.arange(1,25).reshape(4,6)		# reshape可以指定数组的形状
array([[ 1,  2,  3,  4,  5,  6],
       [ 7,  8,  9, 10, 11, 12],
       [13, 14, 15, 16, 17, 18],
       [19, 20, 21, 22, 23, 24]])
```

- 创建全为0或者1的数组与上面类似，调用属性或者方法也类似

==注意：对于一些方法的使用，可以直接使用pycharm查看源码，源码上已经有很详细的注释还有一些例子：例如zeros方法，按住ctrl查看其源码，可以看到该如何使用这些方法==

```python
def zeros(shape, dtype=None, order='C'): # real signature unknown; restored from __doc__
    """
    zeros(shape, dtype=float, order='C')
    
        Return a new array of given shape and type, filled with zeros.
    
        Parameters
        ----------
        shape : int or tuple of ints
            Shape of the new array, e.g., ``(2, 3)`` or ``2``.
        dtype : data-type, optional
            The desired data-type for the array, e.g., `numpy.int8`.  Default is
            `numpy.float64`.
        order : {'C', 'F'}, optional, default: 'C'
            Whether to store multi-dimensional data in row-major
            (C-style) or column-major (Fortran-style) order in
            memory.
```

### 三、数组的类型

- 关于数组中元素的类型，可以使用dtype属性来查看，更多的属性可以看下表

![image-20201127092058325](C:\Users\VSUS\Desktop\笔记\python数据分析\img\1.png)

### 

- 也可以在创建数组时指定数据的类型，例如

```python
In [3]: np.array([1,0,1,0],dtype=np.bool)
Out[3]: array([ True, False,  True, False])
```

```python
In [8]: c = np.random.rand(3,3)

In [9]: c
Out[9]:
array([[0.56375592, 0.79636384, 0.46828949],
       [0.25391213, 0.95462444, 0.85507318],
       [0.87857127, 0.17334742, 0.64066803]])
# 保留固定位数的小数
In [10]: np.round(c,2)
Out[10]:
array([[0.56, 0.8 , 0.47],
       [0.25, 0.95, 0.86],
       [0.88, 0.17, 0.64]])
```

### 四、数组的形状

与数组的形状相关的属性为shape，数组的形状可以理解为数组的维度

```python
In [4]: a = np.array([[1,2,3,4,5],[6,7,8,9,0]])

In [5]: a.shape
Out[5]: (2, 5)
```

上面a是一个2*5的二维数组，所以shape[0]代表的是数组的行，shape[1]代表的是数组的列，可以通过reshape来改变数组的维度

```java
In [7]: a = np.arange(24)

In [8]: a
Out[8]:
array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
       17, 18, 19, 20, 21, 22, 23])

In [9]: a.shape
Out[9]: (24,)

In [10]: a.reshape(4,6)
Out[10]:
array([[ 0,  1,  2,  3,  4,  5],
       [ 6,  7,  8,  9, 10, 11],
       [12, 13, 14, 15, 16, 17],
       [18, 19, 20, 21, 22, 23]])
```

==flatten()方法可以快速的将数组化为1维度数据==

```python
In [11]: a.flatten()
Out[11]:
array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
       17, 18, 19, 20, 21, 22, 23])
```

### 五、数组的计算

- 广播机制

![image-20201127095525368](C:\Users\VSUS\Desktop\笔记\python数据分析\img\2.png)

```python
In [16]: a
Out[16]:
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])

In [17]: a+1
Out[17]:
array([[ 1,  2,  3,  4,  5],
       [ 6,  7,  8,  9, 10]])

In [18]: a*3
Out[18]:
array([[ 0,  3,  6,  9, 12],
       [15, 18, 21, 24, 27]])
```

数组与数组之间的计算

```python
In [23]: a
Out[23]:
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])

In [24]: b
Out[24]:
array([[10, 11, 12, 13, 14],
       [15, 16, 17, 18, 19]])

In [25]: a+b
Out[25]:
array([[10, 12, 14, 16, 18],
       [20, 22, 24, 26, 28]])

In [26]: a*b
Out[26]:
array([[  0,  11,  24,  39,  56],
       [ 75,  96, 119, 144, 171]])

In [27]: a/b
Out[27]:
array([[0.        , 0.09090909, 0.16666667, 0.23076923, 0.28571429],
       [0.33333333, 0.375     , 0.41176471, 0.44444444, 0.47368421]])
```

不同维度之间的数组计算

```python
In [35]: a
Out[35]:
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])

In [36]: c
Out[36]: array([1, 2, 3, 4, 5])

In [37]: a+c
Out[37]:
array([[ 1,  3,  5,  7,  9],
       [ 6,  8, 10, 12, 14]])


In [39]: a*c
Out[39]:
array([[ 0,  2,  6, 12, 20],
       [ 5, 12, 21, 32, 45]])
```

```python
In [51]: a
Out[51]:
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])

In [52]: d
Out[52]:
array([[0],
       [1]])

In [53]: a+d
Out[53]:
array([[ 0,  1,  2,  3,  4],
       [ 6,  7,  8,  9, 10]])
```

### 六、numpy读取数据

> 什么是csv文件？
>
> * CSV （逗号分隔值文件格式）广义的csv文件可以不是逗号分隔；
>   CSV文件最早用在简单的数据库里，由于其格式简单，并具备很强的开放性，所以起初被扫图家用作自己图集的标记。CSV文件是个纯文本文件，每一行表示一张图片的许多属性。你在收一套图集时，只要能找到它的CSV文件，用专用的软件校验后，你对该图集的状况就可以了如指掌。 每行相当于一条记录，是用“，”分割字段的纯文本数据库文件。
> * csv文件的显示: 以Excel表格的方式打开;

numpy使用loadtxt()方法读取文件数据：

```python
np.loadtxt(fname,dtype=np.float,delimiter=None,skiprows=0,usecols=None,unpack=False)
```

![image-20201127124246922](C:\Users\VSUS\Desktop\笔记\python数据分析\img\3.png)

### 七、索引与切片

- 一维数组通过冒号分隔切片参数 **start:stop:step** 来进行切片操作：

```python
In [6]: a
Out[6]: array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

In [7]: a[2:7:2]
Out[7]: array([2, 4, 6])

In [8]: a[5]
Out[8]: 5

In [10]: a[2:]
Out[10]: array([2, 3, 4, 5, 6, 7, 8, 9])

In [11]: a[2:5]
Out[11]: array([2, 3, 4])

In [12]: a[:6]
Out[12]: array([0, 1, 2, 3, 4, 5])
```

- 二维数组[行,列]，逗号前是分割行，逗号后是分割列，同样有**start:stop:step** 这个规则，如果只切割行，那么逗号可以省略

```python
In [4]: a
Out[4]:
array([[1, 2, 3],
       [3, 4, 5],
       [4, 5, 6]])

In [5]: a[1:]
Out[5]:
array([[3, 4, 5],
       [4, 5, 6]])

In [6]: a[:,2]
Out[6]: array([3, 5, 6])

In [7]: a[:,1:3]
Out[7]:
array([[2, 3],
       [4, 5],
       [5, 6]])

In [8]: a[[0,1],[0,1]]
Out[8]: array([1, 4])


In [12]: a[:2,:]
Out[12]:
array([[1, 2, 3],
       [3, 4, 5]])

In [13]: a[:2,:2]
Out[13]:
array([[1, 2],
       [3, 4]])
```

- 布尔索引

我们可以通过一个布尔数组来索引目标数组。

布尔索引通过布尔运算（如：比较运算符）来获取符合指定条件的元素的数组。

```python
In [24]: a
Out[24]:
array([[1, 2, 3],
       [3, 4, 5],
       [4, 5, 6]])

In [25]: a[a>3]
Out[25]: array([4, 5, 4, 5, 6])
In [26]: a[a>3] = 10

In [27]: a
Out[27]:
array([[ 1,  2,  3],
       [ 3, 10, 10],
       [10, 10, 10]])    
```

- 三元运算符

```python
In [27]: a
Out[27]:
array([[ 1,  2,  3],
       [ 3, 10, 10],
       [10, 10, 10]])

In [28]: np.where(a>2,100,0)
Out[28]:
array([[  0,   0, 100],
       [100, 100, 100],
       [100, 100, 100]])
```

### 八、numpy中的nan

==nan(NAN,Nan):not a number表示不是一个数字==

什么时候numpy中会出现nan：

- 当我们读取本地的文件为float的时候，如果有缺失，就会出现nan

- 当做了一个不合适的计算的时候(比如无穷大(inf)减去无穷大)

==inf(-inf,inf):infinity,inf表示正无穷，-inf表示负无穷==

什么时候回出现inf包括（-inf，+inf）

- 比如一个数字除以0，（python中直接会报错，numpy中是一个inf或者-inf）

==nan的注意事项==

- 两个nan是不相等的
- np.nan != np.nan
- 判断nan的个数

```python
In [32]: b
Out[32]:
array([[ 1., nan,  3.],
       [ 3.,  4., nan],
       [ 4.,  5.,  6.]])

In [41]: b==b
Out[41]:
array([[ True, False,  True],
       [ True,  True, False],
       [ True,  True,  True]])

In [42]: b!=b
Out[42]:
array([[False,  True, False],
       [False, False,  True],
       [False, False, False]])
# 统计nan的个数
In [43]: np.count_nonzero(b!=b)
Out[43]: 2
```

解释：count_nonzero函数统计不是0的个数，b!=b中如果b中有nan，那么返回true代表值为1，那么1是不为0的就把nan的个数统计出来

- 过滤nan

```python
In [32]: b
Out[32]:
array([[ 1., nan,  3.],
       [ 3.,  4., nan],
       [ 4.,  5.,  6.]])

In [36]: b[b==b]
Out[36]: array([1., 3., 3., 4., 4., 5., 6.])
```

- nan和任何值计算都是nan

```python
In [44]: b
Out[44]:
array([[ 1., nan,  3.],
       [ 3.,  4., nan],
       [ 4.,  5.,  6.]])

In [45]: b + 1
Out[45]:
array([[ 2., nan,  4.],
       [ 4.,  5., nan],
       [ 5.,  6.,  7.]])
```

### 九、常用统计函数

| 作用   | 函数                                   |
| ------ | -------------------------------------- |
| 求和   | t.sum(axis=None)                       |
| 均值   | t.mean(a,axis=None) 受离群点的影响较大 |
| 中值   | np.median(t,axis=None)                 |
| 最大值 | t.max(axis=None)                       |
| 最小值 | t.min(axis=None)                       |
| 极值   | np.ptp(t,axis=None)                    |
| 标准差 | t.std(axis=None)                       |

默认返回多维数组的全部的统计结果,如果指定axis则返回一个当前轴上的结果

例如：

```python
In [46]: a
Out[46]:
array([[ 1,  2,  3],
       [ 3, 10, 10],
       [10, 10, 10]])

In [47]: a.sum(axis=0)
Out[47]: array([14, 22, 23])

In [48]: a.sum(axis=1)
Out[48]: array([ 6, 23, 30])
```

### 十、一些其他的方法

##### 10.1 数组的拼接

- 竖直拼接vstack()

```python
In [49]: t1 = np.arange(12).reshape(2,6)

In [50]: t1
Out[50]:
array([[ 0,  1,  2,  3,  4,  5],
       [ 6,  7,  8,  9, 10, 11]])

In [52]: t2 = np.arange(12,24).reshape(2,6)

In [53]: t2
Out[53]:
array([[12, 13, 14, 15, 16, 17],
       [18, 19, 20, 21, 22, 23]])


In [55]: np.vstack((t1,t2))
Out[55]:
array([[ 0,  1,  2,  3,  4,  5],
       [ 6,  7,  8,  9, 10, 11],
       [12, 13, 14, 15, 16, 17],
       [18, 19, 20, 21, 22, 23]])
```

- 水平拼接hstack

```python
In [57]: np.hstack((t1,t2))
Out[57]:
array([[ 0,  1,  2,  3,  4,  5, 12, 13, 14, 15, 16, 17],
       [ 6,  7,  8,  9, 10, 11, 18, 19, 20, 21, 22, 23]])
```

##### 10.2 行列交换

```python
In [60]: a
Out[60]:
array([[ 1,  2,  3],
       [ 3, 10, 10],
       [10, 10, 10]])

In [61]: a[[1,2],:] = a[[2,1],:]	# 行交换

In [62]: a
Out[62]:
array([[ 1,  2,  3],
       [10, 10, 10],
       [ 3, 10, 10]])

In [63]: a[[0,1],:2] = a[[0,1],:2]	#部分行交换

In [64]: a
Out[64]:
array([[ 1,  2,  3],
       [10, 10, 10],
       [ 3, 10, 10]])

In [65]: a[[0,1],:2] = a[[1,0],:2]	#列交换

In [66]: a
Out[66]:
array([[10, 10,  3],
       [ 1,  2, 10],
       [ 3, 10, 10]])

In [67]: a[:,[0,2]] = a[:,[2,0]]

In [68]: a
Out[68]:
array([[ 3, 10, 10],
       [10,  2,  1],
       [10, 10,  3]])
```

##### 10.3 获取最大最小值的位置方法

- np.argmax(t,axis=0)

- np.argmin(t,axis=1)

```python
In [75]: c
Out[75]:
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])

In [76]: np.argmax(c,axis=0)
Out[76]: array([1, 1, 1, 1, 1], dtype=int64)

In [77]: np.argmax(c,axis=1)
Out[77]: array([4, 4], dtype=int64)

In [78]: np.argmin(c,axis=0)
Out[78]: array([0, 0, 0, 0, 0], dtype=int64)

In [79]: np.argmin(c,axis=1)
Out[79]: array([0, 0], dtype=int64)
```

##### 10.4 生成随机数

![image-20201127135917116](C:\Users\VSUS\Desktop\笔记\python数据分析\img\4.png)

count += np.size((np.abs(distance) >= 20))