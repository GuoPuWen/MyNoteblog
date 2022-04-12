# 第四章 shell程序设计

### 1. shell中的符号

##### 1.1 引号

==双引号==

- 消除元字符的特殊含义，除$ ` \ “ 外，均作为普通字符
- 保留空白字符

```shell
#!/bin/bash
echo "current directory is `pwd`"
echo "home directory is $HOME"
```

==单引号==

- 屏蔽shell中元字符的功能

- 除单引号自身以外，其他元字符都作为普通字符

```shell
#!/bin/bash
echo 'current directory is `pwd`'
echo 'home directory is $HOME'
```

![image-20201107184131753](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\25.png)

==``反引号==

反引号。反引号括起来的内容是系统命令，在Bash中会先执行它。和$()作用一样,不过推荐使用$ ()，因为反引号非常容易看错。



##### 1.2 通配符

==通配符==

用于匹配文件名，完全匹配

| 通配符 | 作用                                  |
| ------ | ------------------------------------- |
| ？     | 匹配一个任意字符                      |
| *      | 匹配0个或者任意多个字符，匹配任意内容 |
| []     | 匹配括号中的任意一个字符              |
| [-]    | 匹配括号中任意一个字符，-代表一个范围 |
| [^]    | 逻辑非，表示匹配不是括号中的一个字符  |

##### 1.3 命令连接符

==顺序执行；==

格式：  命令1；命令2；……；命令n；

功能： 各条命令从左到右依次执行

例子： pwd；who|wc -l；cd/home/bin

==逻辑与&&==

格式：  命令1&&命令2

功能： 先执行命令1，如果成功，再执行命令2；否则不执行命令2

==逻辑或||==

格式： 命令1||命令2

功能： 先执行命令1，如果不成功，再执行命令2；否则不执行命令2

##### 1.4 I/O重定向

==输入重定向==

让命令从指定文件取得输入数据

格式： 命令 < 文件名

例子：bash < ex1

​			wc < fileinfo

==输入重定向操作符 < <==

以 < 操作符开始 以 < 操作符结束

两个字符之间的就是输入的内容

==输出重定向==

把命令的标准输出重定向到指定文件中

格式：  命令 > 文件名

​    命令 > > 文件名

说明：输出附加定向符（>>）的作用是把命令/程序的输出附加到指定文件的末尾

##### 1.5 管道操作符

```shell
ls -l | more
ls -l /etc | grep init
ps | sort > passort
```

##### 1.6 后台命令符

后台命令

将当前命令送到后台执行

例子：find/-name file2.txt&

##### 1.7 成组命令

格式： (命令序列)

​		    {命令序列}

功能： 配对括号之间所有命令作为一个整体执行

### 2. shell中的变量

##### 2.1 用户定义变量

- 以字母或下划线开头
-  由字母、下划线、数字
-   变量名区分大小写

说明：只在当前shell中有效 ，没有类型和存储类的限制，边定义和边限制

==变量赋值==

格式： 变量名=字符串  **//等号两边不能有空格**

==引用变量值==

​    变量名前面加$符号

```shell
 echo $a $b
```

查看所有变量----set 命令

清除变量----unset命令

```shell
unset a
```

用户定义变量说明：

shell默认将任何赋值给变量的值都解释为字符串，如果值中包含空格，赋值时必须用引号括起来

例子： x=”Hello world”

##### 2.2 环境变量

功能： 系统定义的变量，这些变量用于初始化shell的启动环境，因此被称为环境变量

说明： 习惯上，环境变量的变量名用大写字母表示

环境变量一般在用户登录时由系统设置，也可以由用户修改。当用户注销后，环境变量会随之复原

查看某个环境变量------echo $HOME

查看所有环境变量------env

常用的环境变量

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\26.jpg)



==export==

功能： 用户所创建的shell变量，默认为局部变量。export可以拓展变量的使用环境

格式： export 变量名

说明： 在一个进程内部，同名局部变量的值优先使用

##### 2.3 位置变量

功能：  用于接收命令/shell脚本的参数，也称为位置参数

参数：

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\27.jpg)

##### 2.4 特殊变量

主要用来查看脚本的运行信息

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\28.jpg)



### 3. 输入输出命令

==echo==

功能： 在屏幕上显示字符串或变量的值

选项： 

​    ![img](file://C:/Users/VSUS/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

==read==

功能： 从键盘读入数据赋值给变量

格式： read 变量1 变量2......

说明： 变量个数与给定数据个数相同，则依次对应赋值

​    变量个数少于数据个数，则从左到对应赋值，最后一个数据被赋予剩余的所有数据

### 4. 算术运算

==expr==

功能： 处理整数运算

格式： expr 算数表达式

说明： 

- 可以使用的操作符有+、-、*、/（取整）、%（取余）等

- 表达式的元素之间必须有空格

- 字符*（乘）在shell中有特殊含义，因此前面必须有转义字符“\”

- 当有变量参与运算时，需要在变量名前加$

```shell
 x=10
 expr $x + 1
 y=`expr $x + 1`
 z=$(expr $x + 1)
```



==let==

功能： 处理整数算术运算

格式： let 算数表达式

说明： 

- 可以使用的操作符有+、-、*、/（取整）、%（取余）等

- 表达式的操作符两边没有空格

- 对于字符*不需要用转义字符”\”

- 当有变量参与运算时，不需要用$

```shell
let x=x+1
((x=x+1))
```

### 5.条件测试

==test命令==

功能： 用来判断表达式的值为真还是为假

格式： test 测试条件

​    [ 测试条件 ] **//方括号内需要有空格**

##### 5.1文件属性的测试

![img](file:///C:/Users/VSUS/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\29.jpg)

```shell
test -f file1

[ -f file1 ]
```

##### 5.2 字符串的测试

![image-20201108190915546](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\37.png)

```shell
test -n $str2
[ -n $str2 ]
[ $str1 = $str2 ] //等号两边加空格
[ $str1 \< $str2 ]
```

   

##### 5.3 整数的测试

![image-20201108190953990](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\38.png)

```shell
 [ “$x” -lt 1 ]
 [ “$y” -gt 10 ]
```

说明： 如果用“<、>、>=、<=”时需要加双括号

```shell
#!/bin/bash
if [ $1 -gt 0 ]
then
        echo "$1 is positive"
else
        echo "$1 is negative "
fi
```



##### 5.4 逻辑运算符

![image-20201108191143258](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\40.png)

例子：

![img](file:///C:/Users/VSUS/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

### 6. 控制结构

##### 6.1 分支结构

if-then 结构

if-then-else 结构

if-then-elif 结构

脚本练习

```shell
编写shell脚本分别完成以下功能
（1）输入的命令行参数必须是hello，才会正确显示；否则，显示错误提示
（2）检测某个文件是否是一个普通文件
（3）比较两个字符串str1和str2是否相等 
（4）判断一个数字是否是正数
（5）判断给定的数字是否介于1到10之间
 (6)判断给定用户名已经登录系统


（1）
#!/bin/bash
if [ $1 = "hello" ]
then
       echo "yes"
else
        echo "no"
fi
（2）
#!/bin/bash
if [ -f $1 ]
then
        echo "yes"
else
        echo "no"
fi
（3）
#!/bin/bash
read -p "input first str:" str1
read -p "input second str:" str2
if [ $str1 = $str2 ]
then
        echo "yes"
else
        echo "no"
fi
（4）
#!/bin/bash
if [ $1 -ge 0 ]
then
        echo "yes"
else
        echo "no"
fi
（5）
#!/bin/bash
if [ $1 -le 10 -a $1 -ge 1 ]
then
        echo "yes"
else
        echo "no"
fi
（6）
#!/bin/bash
read -p "input username" user
if who | grep $user
then
        echo "$user is login"
else
        echo "$user is no login"
fi
```

##### 6.2 case语句

语法：

![image-20201107202722792](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\30.png)

```shell
#!/bin/bash
echo "Is it morning?Enter yes or no"
read time
case $time in
        yes | y | YES | Yes | Y) echo "yes";;
        n* | N*) echo "no";;
        *) echo "error";;

esac
```

### 7. 循环结构

##### 7.1 while循环

功能： 循环条件为真就进入循环，测试条件为假，退出循环

格式： 

![image-20201107220128614](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\31.png)

```shell
输出1-10的和
#!/bin/bash
count=1
sum=0
while [ $count -le 10 ]
do
        let sum=sum+count
        let count=count+1
done
echo $sum
输出所有参数的累加和
#!/bin/bash
sum=0
if [ $# -eq 0 ]
then
        echo "error"
fi
while [ $# -gt 0 ]
do
        let sum=sum+$1
        shift
done
echo $sum
判断所给参数是否为普通文件，要求可以输出多个参数
#!/bin/bash
if [ $# -eq 0 ]
then
        echo "error"
fi
while [ $1 ]
do
        if [ -f $1 ]
        then
                echo "display: $1"
        else
                echo "$1 is not a file name"
        fi
        shift
done

```

注意：当脚本的参数很多时，可以使用shift命令来访问多个参数

##### 7.2 until循环

功能： 循环条件为假时，执行循环体。当测试条件为真时，循环结束

until循环适用于这种情况：如果想让循环不停的进行，直到某件事发生为止

##### 7.3  for循环

- 算数表示方式

  格式： 

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\32.jpg)

​    说明： 三个条件表达式任何一个都可以缺少，但是分号不能缺少

- 值表方式

格式： 

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\33.jpg)

说明： variable：是循环变量

​    list – of – values：是参数列表

​    in后列出n个参数，则循环体执行n次

​    for循环语句中的参数列表允许使用通配符

​    如果在for语句中没有参数列表，则循环变量的取值范围是全部参数

```shell
打印给定行数的*
#!/bin/bash
for i in `seq 1 $1`
do
        for j in `seq 1 $i`
        do
                echo -n "*"
        done
        echo ''
done
```

### 8.continue和break

==break命令==

功能： 使程序跳出while、for、until循环

格式： break n

说明： n表示要跳出的循环的层数，默认值为1

==continue命令===

功能： 跳到下一次循环继续执行

格式：  continue n

说明： n表示要跳出的循环的层数，默认值为1

