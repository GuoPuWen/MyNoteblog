# 一、概述

1. 概念：一门客户端脚本语言
   - 运行在客户端浏览器中的。每一个浏览器都有JavaScript的解析引擎
   - 脚本语言：不需要编译，直接就可以被浏览器解析执行了

2. 功能
   - 可以来增强用户和html页面的交互过程，可以来控制html元素，让页面有一些动态的效果，增强用户的体验。

# 二、基本语法

## 1.与html结合方式

1.  内部JS ：
   				* 定义script，标签体内容就是js代码

2. 外部JS：
   				* 定义script，通过src属性引入外部的js文件

3. 注意：

   1. script标签可以定义在html页面的任何地方。但是定义的位置会影响执行顺序。

   2.  script可以定义多个。

## 2.数据类型

1. 原始数据类型：

   1. number：数字。；整数/小数/NaN(not a number 一个不是数字的数字类型)

      > 注意：NaN类型参与的==全部为false
      >
      > ```javascript
      >     <script>
      >         var a = 'abc';
      >         document.write(a == NaN + "<br>");
      >         var b = NaN;
      >         document.write(b == NaN);
      >     </script>
      > ```
      >
      > 结果是都为false
      >
      > 如果要判断一个值是否为NaN，那么要使用Global对象下的方法isNaN()方法

   2. string：字符串。 字符串  "abc" "a" 'abc'

   3. boolean：true和false

   4. null：一个对象为空的占位符

   5. undefined：未定义。如果一个变量没有给初始化值，则会被默认赋值为undefined

2. 引用数据类型：对象

## 3.变量

- 变量：一小块存储数据的内存空间

- Java语言是强类型语言，而JavaScript是弱类型语言。
  - 强类型：在开辟变量存储空间时，定义了空间将来存储的数据的数据类型。只能存储固定类型的数据
  - 弱类型：在开辟变量存储空间时，不定义空间将来的存储数据类型，可以存放任意类型的数据。

- JS变量语法
  - var 变量名 = 初始化值;

- typeof运算符：获取变量的类型。
  - null运算得到的是object运算符

## 4.运算符

注意：很多语法与java一样，只列举不一样的地方

1. 一元运算符：只有一个运算数的运算符

   - +(-)：正负号

     - 注意：在JS中，如果运算数不是运算符所要求的类型，那么js引擎会自动的将运算数进行类型转换

     - 其他类型转number：① string转number：按照字面值转换。如果字面值不是数字，则转为NaN（不是数字的数字）② boolean转number：true转为1，false转为0

2.  比较运算符

   1. 比较方式：

      1. 类型相同：直接比较
              * 字符串：按照字典顺序比较。按位逐一比较，直到得出大小为止。

       2. 类型不同：先进行类型转换，再比较
           * ===：全等于。在比较之前，先判断类型，如果类型不一样，则直接返回false

3. 逻辑运算符

   ​	&& || !

   - 其他类型转boolean：
     1. number：0或NaN为假，其他为真
     2. string：除了空字符串("")，其他都是true
     3. null&undefined:都是false
     4. 对象：所有对象都为true

4. 流程控制
   1. switch ：在JS中,switch语句可以接受任意的原始数据类型

5. 特殊语法：
   1. 语句以;结尾，如果一行只有一条语句则 ;可以省略 (不建议)
   2. 变量的定义使用var关键字，也可以不使用
         * 用： 定义的变量是局部变量
         * 不用：定义的变量是全局变量(不建议)

# 三、基本对象

## 1. Function：函数(方法)对象

1. 创建：

   - function 方法名称(形式参数列表){
                             方法体
                         }

   - var 方法名 = function(形式参数列表){
                             方法体
                        }

2. 属性：
   1.  length:代表形参的个数

3.  特点：
   1.  方法定义是，形参的类型不用写,返回值类型也不写。
   2.  方法是一个对象，如果定义名称相同的方法，会覆盖
   3. 在JS中，方法的调用只与方法的名称有关，和参数列表无关
   4. 在方法声明中有一个隐藏的内置对象（数组），arguments,封装所有的实际参数

##  2.Array:数组对象

1. 创建：
   1. var arr = new Array(元素列表);
   2. var arr = new Array(默认长度);
   3. var arr = [元素列表];

2. 方法：

   1.  join(参数):将数组中的元素按照指定的分隔符拼接为字符串返回一个字符串

      ```javascript
       var arr = new Array(1,2,3);
       var s = arr.join("--");
      alert(s);
      ```

   2.  push()：向数组的末尾添加一个或更多元素，并返回新的长度。

3. 属性
   -  length:数组的长度

4. 特点：

   1.  JS中，数组元素的类型可变的

      ```javascript
      var array = new Array(1,"abc",true,NaN);
      document.write(array);
      ```

   2. JS中，数组长度可变的。

## 3.Date：日期对象

1. 创建：

    var date = new Date();

2. 方法：

   1. toLocaleString()：返回当前date对象对应的时间本地字符串格式

   2. getTime():获取毫秒值。返回当前如期对象描述的时间到1970年1月1日零点的毫秒值差

## 4. Math：数学对象

1. 创建：

   **无需创建它，通过把 Math 作为对象使用就可以调用其所有属性和方法。**

2. 方法：
   1. random():返回 0 ~ 1 之间的随机数。 含0不含1
   2.  ceil(x)：对数进行上舍入。
   3. floor(x)：对数进行下舍入。
   4.  round(x)：把数四舍五入为最接近的整数。

3. 属性：PI

   例子：获得1~100之间的随机整数

   ```javascript
   var number = Math.random() * 100; //[0,100)之间的随机数
   var res = Math.floor(number);    //[0,99]之间的随机整数数
   document.write(res + 1 );   //[1,100]之间的随机整数数
   ```

## 5.RegExp：正则表达式对象

1. 正则表达式：定义字符串的组成规则。
    	1.  方括号:用于查找某个范围内的字符：

| [0-9]              | 查找任何从 0 至 9 的数字。         |
| ------------------ | ---------------------------------- |
| [a-z]              | 查找任何从小写 a 到小写 z 的字符。 |
| [A-Z]              | 查找任何从大写 A 到大写 Z 的字符。 |
| [A-z]              | 查找任何从大写 A 到小写 z 的字符。 |
| [adgk]             | 查找给定集合内的任何字符。         |
| [^adgk]            | 查找给定集合外的任何字符。         |
| (red\|blue\|green) | 查找任何指定的选项。               |

​				2. 元字符（Metacharacter）是拥有特殊含义的字符：

| \w(小写) | 查找单词字符   |
| -------- | -------------- |
| \d       | 查找数字。量词 |

​			3. 量词

| 量词   | 描述                                  |
| :----- | :------------------------------------ |
| n+     | 匹配任何包含至少一个 n 的字符串。     |
| n*     | 匹配任何包含零个或多个 n 的字符串。   |
| n?     | 匹配任何包含零个或一个 n 的字符串。   |
| n{X}   | 匹配包含 X 个 n 的序列的字符串。      |
| n{X,Y} | 匹配包含 X 至 Y 个 n 的序列的字符串。 |
| n{X,}  | 匹配包含至少 X 个 n 的序列的字符串。  |
| n$     | 匹配任何结尾为 n 的字符串             |
| ^n     | 匹配任何开头为 n 的字符串             |

2. 正则对象：

   1. 创建

      1. var reg = new RegExp("正则表达式");
      2. var reg = /正则表达式/;

      注意：在第1中创建方式中，正则表达式是作为字符串的形式，所以如果使用上述的元字符时，要注意加转义字符

   ```javascript
   var rg = new RegExp("^\d{6,12}$");  //6~12位的数字
   var b = rg.test("111111");
   alert(b);   //弹出false
   var rg2 = new RegExp("^\\d{6,12}$");
   alert(rg2.test("111111"))       //弹出true
   ```

   2. 方法：

      test(参数):验证指定的字符串是否符合正则定义的规范	

##  6.Global

1. 特点：全局属性和函数可用于所有内建的 JavaScript 对象。

2. 方法：

   1. encodeURI():url编码

   2. decodeURI():url解码

   3.  encodeURIComponent():url编码,编码的字符更多

   4. decodeURIComponent():url解码

   5.  parseInt():将字符串转为数字

      ​	直接使用+本就可以将字符串类型转换为数字，但是无法转类似于”123abc“，这个会直接转换为NaN类型。但是使用parseInt()会逐一判断每一个字符是否是数字，直到不是数字为止，将前边数字部分转为number。

      ​	例如：”123abc“转成123

   6.  isNaN():判断一个值是否是NaN
   7.  eval():传入 JavaScript 字符串，并把它作为脚本代码来执行。