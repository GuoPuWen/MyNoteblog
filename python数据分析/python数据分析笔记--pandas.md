numpy能够帮助我们处理数值，但是pandas除了处理数值之外(基于numpy)，还能够帮助我们处理其他类型的数据，pandas的常用数据结构有两种:

1.Series 一维，带标签数组

2.DataFrame 二维，Series容器

# 一、Series 

### 1.1 创建Series

```python
pd.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
```

==创建的方式--一般创建==

```python
In [4]: pd.Series(np.arange(10))		#隐式索引，从0开始
Out[4]:
0    0
1    1
2    2
3    3
4    4
5    5
6    6
7    7
8    8
9    9
dtype: int32

In [5]: pd.Series(np.arange(3),index=list("abc"))	#显示索引
Out[5]:
a    0
b    1
c    2
dtype: int32

In [6]: pd.Series(np.arange(3),index=list("abc"),dtype='int64')
Out[6]:
a    0
b    1
c    2
dtype: int64
    
In [8]: type(pd.Series(np.arange(10)))
Out[8]: pandas.core.series.Series
```

==通过字典创建==

```python
In [9]: map = {'a':1,'b':2,'c':3}

In [10]: map
Out[10]: {'a': 1, 'b': 2, 'c': 3}

In [11]: pd.Series(map)
Out[11]:
a    1
b    2
c    3
dtype: int64
```

### 1.2 Series的索引和切片

因为Series只有一列，因此一般只对行进行操作

==取索引列与值列==

```python
In [14]: t
Out[14]:
a    6
b    8
c    8
d    2
e    6
dtype: int32

In [15]: t.index
Out[15]: Index(['a', 'b', 'c', 'd', 'e'], dtype='object')

In [17]: t.values
Out[17]: array([6, 8, 8, 2, 6])

In [18]: type(t.index)
Out[18]: pandas.core.indexes.base.Index

In [19]: type(t.values)
Out[19]: numpy.ndarray
```

==通过索引的方式取值==

```python
In [20]: t
Out[20]:
a    6
b    8
c    8
d    2
e    6
dtype: int32

In [21]: t['a']
Out[21]: 6

In [22]: t[['a','b']]
Out[22]:
a    6
b    8
dtype: int32


In [24]: t['a':'c']
Out[24]:
a    6
b    8
c    8
dtype: int32

In [25]: t.loc['a':'c']
Out[25]:
a    6
b    8
c    8
dtype: int32

In [26]: t.loc[['a','c']]
Out[26]:
a    6
c    8
dtype: int32
```

### 1.3 基本操作

##### **1.3.1 显示Series部分数据内容**

* **s.head(n)** 该函数代表的意思是显示前多少行，可以指定显示的行数，不写n默认是前5行
* **s.tail(n)** 该函数代表的意思是显示后多少行，可以指定显示的行数，不写n默认是前5行

```python
In [28]: t.head()
Out[28]:
a    6
b    8
c    8
d    2
e    6
dtype: int32

In [29]: t.head(3)
Out[29]:
a    6
b    8
c    8
dtype: int32

In [30]: t.tail()
Out[30]:
a    6
b    8
c    8
d    2
e    6
dtype: int32

In [31]: t.tail(3)
Out[31]:
c    8
d    2
e    6
dtype: int32
```

##### **1.3.2 Series去重操作**

```python
In [36]: t
Out[36]:
a    1
b    2
c    2
d    3
e    4
dtype: int64


In [38]: t.unique()
Out[38]: array([1, 2, 3, 4], dtype=int64)

In [39]: type(t.unique())
Out[39]: numpy.ndarray
```

==去重之后返回是一个ndarray类型==

# 二、DataFrame

### 2.1 创建DataFrame

DataFrame()函数的参数index的值相当于行索引，若不手动赋值，将默认从0开始分配。columns的值相当于列索引，若不手动赋值，也将默认从0开始分配。

```python
In [41]: pd.DataFrame(np.arange(12).reshape(3,4))
Out[41]:
   0  1   2   3
0  0  1   2   3
1  4  5   6   7
2  8  9  10  11

In [42]: pd.DataFrame(np.arange(12).reshape(3,4),index=list('abc'),columns=[1,2,3,4])
Out[42]:
   1  2   3   4
a  0  1   2   3
b  4  5   6   7
c  8  9  10  11
```

==使用字典创建==

```python
In [46]: data
Out[46]:
{'性别': ['男', '女', '女', '男', '男'],
 '姓名': ['小明', '小红', '小芳', '大黑', '张三'],
 '年龄': [20, 21, 25, 24, 29]}

In [47]: pd.DataFrame(data)
Out[47]:
  性别  姓名  年龄
0  男  小明  20
1  女  小红  21
2  女  小芳  25
3  男  大黑  24
4  男  张三  29
```

### 2.2 一些基本操作

|   函数(属性)   |            作用            |
| :------------: | :------------------------: |
|  DataFrame()   |   创建一个DataFrame对象    |
|   df.values    |   返回ndarray类型的对象    |
|    df.index    |         获取行索引         |
|   df.columns   |         获取列索引         |
|    df.axes     |       获取行及列索引       |
|      df.T      |         行与列对调         |
|   df. info()   |  打印DataFrame对象的信息   |
|   df.head(i)   |      显示前 i 行数据       |
|   df.tail(i)   |      显示后 i 行数据       |
| df.describe()  | 快速查看数据按列的统计信息 |
|   df.shape()   |         行数，列数         |
|    df.ndim     |          返回维度          |
| df.sort_values |      对某一列进行排序      |

- df.values 返回ndarray类型的对象

- df.info查看DataFrame对象的信息

```python
In [55]: df.info()
<class 'pandas.core.frame.DataFrame'>
Index: 5 entries, one to five
Data columns (total 4 columns):
 #   Column  Non-Null Count  Dtype
---  ------  --------------  -----
 0   姓名      5 non-null      object
 1   性别      5 non-null      object
 2   年龄      5 non-null      int64
 3   职业      0 non-null      object
dtypes: int64(1), object(3)
memory usage: 200.0+ bytes
```

### 2.3 索引与切片

==直接通过索引的方式==

```python
In [64]: df
Out[64]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN
four   大黑  男  24  NaN
five   张三  男  29  NaN

In [65]: df[1:2]
Out[65]:
     姓名 性别  年龄   职业
two  小红  女  21  NaN

In [66]: df[:3]
Out[66]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN

In [67]: df[:3]['姓名']
Out[67]:
one      小明
two      小红
three    小芳
Name: 姓名, dtype: object
        
In [79]: df['姓名']
Out[79]:
one      小明
two      小红
three    小芳
four     大黑
five     张三
Name: 姓名, dtype: object

In [80]: type(df['姓名'])
Out[80]: pandas.core.series.Series
```

通过上面的一些例子可以明白，df[start:stop]也同样适用于dataframe

==通过loc和iloc函数==

* a.loc[行,列]，标签索引，自定义索引
* a.iloc[行,列]，位置索引，默认索引

```python
In [72]: df
Out[72]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN
four   大黑  男  24  NaN
five   张三  男  29  NaN

In [73]: df.loc['one']
Out[73]:
姓名     小明
性别      男
年龄     20
职业    NaN
Name: one, dtype: object

In [74]: df.loc['one':'five']	#可以可以看到stop位置也是可以取到的，这是一个例外
Out[74]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN
four   大黑  男  24  NaN
five   张三  男  29  NaN

In [75]: df[0:4]	#这个stop和之前一样，取不到
Out[75]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN
four   大黑  男  24  NaN

In [76]: df.iloc[0]
Out[76]:
姓名     小明
性别      男
年龄     20
职业    NaN
Name: one, dtype: object

In [77]: df.iloc[0:4]
Out[77]:
       姓名 性别  年龄   职业
one    小明  男  20  NaN
two    小红  女  21  NaN
three  小芳  女  25  NaN
four   大黑  男  24  NaN
```

### 2.4 缺失数据的处理

构造数据

```python
[27]: items2 = [{'bikes': 20, 'pants': 30, 'watches': 35, 'shirts': 15, 'shoes':8, 'suits':45},
    ...: {'watches': 10, 'glasses': 50, 'bikes': 15, 'pants':5, 'shirts': 2, 'shoes':5, 'suits':7},
    ...: {'bikes': 20, 'pants': 30, 'watches': 35, 'glasses': 4, 'shoes':10}]
In [29]: t2 = pd.DataFrame(items2,index=['store1','store2','store3'])

In [32]: t2
Out[32]:
        bikes  pants  watches  shirts  shoes  suits  glasses
store1     20     30       35    15.0      8   45.0      NaN
store2     15      5       10     2.0      5    7.0     50.0
store3     20     30       35     NaN     10    NaN      4.0
```

##### ==统计nan的个数==

可以使用isnull方法，返回dataframe类型，如果是nan返回true

```python
In [33]: t2.isnull()
Out[33]:
        bikes  pants  watches  shirts  shoes  suits  glasses
store1  False  False    False   False  False  False     True
store2  False  False    False   False  False  False    False
store3  False  False    False    True  False   True    False
```

接着调用pd中的sum方法统计个数，sum方法返回一个series类型，因为false为0，true为1所以sum方法返回的是true的个数

```python
In [34]: t2.isnull().sum()
Out[34]:
bikes      0
pants      0
watches    0
shirts     1
shoes      0
suits      1
glasses    1
dtype: int64
```

最终需要统计nan的个数可以再次调用sum方法

```python
In [36]: t2.isnull().sum().sum()
Out[36]: 3
```

##### ==缺失值处理==

处理缺失值有两种方法：

- 直接删除有nan的行或者列
- 替换nan的值

**直接删除nan**：

```python
In [40]: t2.dropna(axis=0,how='any')
Out[40]:
        bikes  pants  watches  shirts  shoes  suits  glasses
store2     15      5       10     2.0      5    7.0     50.0

In [41]: t2.dropna(axis=1,how='any')
Out[41]:
        bikes  pants  watches  shoes
store1     20     30       35      8
store2     15      5       10      5
store3     20     30       35     10
```

调用np中的dropna可以删除nan的行或者列，使用axis指定是删除行还是删除列，注意，`.dropna()` 方法不在原地地删除具有 `NaN`值的行或列。也就是说，原始 DataFrame 不会改变。你始终可以在 `dropna()` 方法中将关键字 `inplace 设为 True`，在原地删除目标行或列。

**替换nan的值**

fillna方法可以替换pd中nan的值，例如将所有nan替换为0

```python
In [42]: t2.fillna(0)
Out[42]:
        bikes  pants  watches  shirts  shoes  suits  glasses
store1     20     30       35    15.0      8   45.0      0.0
store2     15      5       10     2.0      5    7.0     50.0
store3     20     30       35     0.0     10    0.0      4.0
```

例如讲nan的值替换为当前列的均值

```python
In [45]: t2.fillna(t2.mean(axis=0))
Out[45]:
        bikes  pants  watches  shirts  shoes  suits  glasses
store1     20     30       35    15.0      8   45.0     27.0
store2     15      5       10     2.0      5    7.0     50.0
store3     20     30       35     8.5     10   26.0      4.0
```

