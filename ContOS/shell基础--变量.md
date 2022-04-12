## 一、概述

- 在 Bash shell 中，每一个变量的值=类型都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。

- Shell 变量的命名规范和大部分编程语言都一样：
  * 变量名由数字、字母、下划线组成；
  * 必须以字母或者下划线开头；
  * 不能使用 Shell 里的关键字（通过 help 命令可以查看保留关键字）。

- Bash的变量分类
  - 用户自定义变量：这种变量是最常见的变量，由用户自由定义变量名和变量的值
  - 环境变量：这种变量中主要保存的是和系统操作环境相关的数据，环境变量的变量名可以自由定义，但是一般对系统起作用的环境变量的变量名是系统预先设定好的
  - 位置参数变量：这种变量主要是用来向脚本当中传递参数或数据的，变量名不能自定义，变量作用是固定的
  - 预定义变量：是Bash中已经定义好的变量，变量名不能自定义，变量作用也是固定的

| 分类           | 变量名 | 值     | 作用   |
| -------------- | ------ | ------ | ------ |
| 用户自定义变量 | 自定义 | 自定义 | 自定义 |
| 环境变量       | 自定义 | 自定义 | 自定义 |
| 位置参数变量   | 固定   | 自定义 | 固定   |
| 预定义变量     | 固定   | 自定义 | 固定   |



## 二、变量增删改查

##### 1.设定变量

shell变量不需要声明变量类型，因为每一个变量的值类型都是字符串。所以，bash变量直接声明即可

```
variable=value
variable='value'
variable="value"
```

variable 是变量名，value 是赋给变量的值。如果 value 不包含任何空白符（例如空格、Tab 缩进等），那么可以不使用引号；如果 value 包含了空白符，那么就必须使用引号包围起来。使用单引号和使用双引号也是有区别的

- **注意：**赋值号`=`的周围不能有空格，这和其他编程语言不太一样。这是由于，如果加了空格，bash会认为这是一个命令，所以报-bash: name: command not found 错误

![](F:\笔记\ContOS\变量\1.png)

- **注意：**定义变量时，变量的值可以由单引号`' '`包围，也可以由双引号`" "`包围。要注意它们的区别

```
name=value
test1="name: $name"
test2='name: $name'
echo $test1
echo $test2
```

运行结果：

![](F:\笔记\ContOS\变量\4.png)

以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。

以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

##### 2.使用变量

使用一个定义过的变量，只要在变量名前面加$即可，如：

![](F:\笔记\ContOS\变量\2.png)

##### 3.修改变量

已定义的变量，可以被重新赋值，最后一次赋的值会覆盖之前的值

![](F:\笔记\ContOS\变量\3.png)

##### 4.查看变量

```
[root@localhost ~]# set [选项]
选项
	-u 如果设定此选项，调用未声明变量会报错
```



##### 5.删除变量

```
[root@localhost ~]# unset 变量名                                   
```

## 三、环境变量

用户自定义变量和环境变量的区别就是：用户自定义变量只能在当前 Shell 中有效，而环境变量在当前 Shell 和所有子 Shell 中有效。例如

```
[root@localhost ~]# name1="zhangsan"
[root@localhost ~]# name2="lisi"
#name2为环境变量
[root@localhost ~]# export name2
#在当前shell查看
[root@localhost ~]# set
......
name1=zhangsan
name2=lisi
......
#进入子shell
[root@localhost ~]# bash
#查看变量
[root@localhost ~]# set
......
name2=lisi
......
```

所以说用户自定义变量只能在当前 Shell 中有效，而环境变量在当前 Shell 和所有子 Shell 中有效

##### 1.声明变量

```
[root@localhost ~]# export age=18
```

##### 2.环境变量查询和删除

1. env

env 命令只能查看环境变量。 

```
[root@localhost ~]# env
HOSTNAME=localhost.localdomain		# 主机名字
SELINUX_ROLE_REQUESTED=				#当前shell	
TERM=xterm							#终端环境
SHELL=/bin/bash						#当前shell
HISTSIZE=10000						#历史命令条数
SSH_CLIENT=192.168.124.9 53915 22	#记录客户端ip
SELINUX_USE_CURRENT_RANGE=			
QTDIR=/usr/lib/qt-3.3				
QTINC=/usr/lib/qt-3.3/include
SSH_TTY=/dev/pts/0					#连接的终端
USER=root						#当前登陆的用户
```

2. set

   set 命令可以查看所有变量

##### 3.环境变量删除

```
[root@localhost ~]# unset 变量名    
```



## 四、位置参数变量

主要是向脚本中传递数据，变量名不能自定义，变量作用是固定的

##### 1.变量类型

| 类型 |                             作用                             |
| :--: | :----------------------------------------------------------: |
|  $n  | $0代表命令本身，$1-9代表接受的第1-9个参数，10以上需要用{}括起来，比如${10}代表接收的第10个参数 |
|  $*  |          代表接收所有的参数，将所有参数看作一个整体          |
|  $@  |            代表接收的所有参数，将每个参数区别对待            |
|  $#  |                      代表接收的参数个数                      |

##### 2.$n

示例：编写一个脚本，脚本后面带上2个数，输出两数之和

sum.sh

```shell
#!/bin/bash

sum=$(($1 + $2 ))

echo "Sum is $sum " 
```

```
[root@localhost ~]# chmod 755 sum.sh
[root@localhost ~]# ./sum.sh 1 2
Sum is 3 
```

##### 3.$*、$@、$#

test.sh

```shell
#!/bin/bash

echo "\$* is $*"

echo "\$@ is $@"

echo "\$# is $#"
```

```
[root@localhost ~]# chmod 755 test.sh 
[root@localhost ~]# ./test.sh 1 2
$* is 1 2		#代表接收所有的参数，将所有参数看作一个整体
$@ is 1 2		#代表接收的所有参数，将每个参数区别对待
$# is 2			#两个参数，代表接收的参数个数
```

##### 4.$* 和$@ 的区别

test2.sh

```shell
#!/bin/bash

echo "This is \$*"
for i in "$*"
        do
                echo $i
        done
        
echo "This is \$@"
for i in "$@"
        do
                echo $i
        done
```

```
[root@localhost ~]# chmod 755 test2.sh 
[root@localhost ~]# ./test2.sh 1 2 3 4
This is $*
1 2 3 4
This is $@
1
2
3
4
```

结论：$*中的所有参数看成是一个整体，所以for循环只循环一次。

$@中的每个参数是独立的，所有输入几个参数，就会循环几次。

## 五、预定义变量

主要是Bash中已经定好的变量，名称不能自定义，作用也是固定的

| 类型 | 作用                                           |
| ---- | ---------------------------------------------- |
| $?   | 最后一次执行的命令返回状态，0为成功，非0为失败 |
| $$   | 当前进程的进程号                               |
| $!   | 后台运行的最后一个进程的进程号                 |

更多使用的是$?

```
[root@localhost ~]# echo $?
0
[root@localhost ~]# llll
-bash: llll: command not found
[root@localhost ~]# echo $?
127
```

注意：$?值如果为0证明上一次命令正确执行，如果这个值非0(具体数不确定，依命令为定)，证明这个命令不正确

