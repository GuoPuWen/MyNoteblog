## 1.echo

echo可以把文本输出到控制台上

```
[root@localhost ~]# echo [选项] 输出内容
选项：
	-e 支持反斜杠控制的字符转换，转义字符
	-n 取消输出后文本的换行符号
	
```

- 转义字符(要加上-e选项)

```
\n    换行符
\r    回车符
\f    换页符
\t    水平制表符
\v    纵向制表符
\b    退格符
\c    禁止尾随的换行符
\\    反斜线
\a	  输出警告声
```

## 2.history

查看历史命令

```
[root@localhost ~]# history [选项] 
选项：
	-c 清空历史命令
	-w 把缓存的历史命令写入到文件中(默认：~/.bash_history)
	
```

- history命令默认保存的条数是1000条，在/etc/profile环境变量文件中的HISESIZE变量中

- 使用history命令查看的历史命令和~/.bash history文件中保存的历史命令是不同的。是因为当前登录操作的命令并没有直接写入~.bash history文件，而是保存在缓存当中的。需要等当前用户注销之后，缓存中的命令才会写入~/，bash_history文件。如果我们需要把内存中的命令直接写入/.bash history文件，而不等用户注销时再写入，就需要使用“-w”选项了。

## 3.alias

命令别名

```
#查询命令别名
[root@localhost ~]# alias
#设定命令别名
[root@localhost ~]# alias 别名='原命令'
```

![](F:\笔记\ContOS\Bash基本功能\1.png)

> 命令执行的顺序：
>
> 1.第一顺位执行用绝对路径或相对路径执行的命令。
> 		2.第二顺位执行别名。
> 		3.第三顺位执行Bash的内部命令。
> 		4.第四顺位执行按照SPATH环境变量定义的目录查找顺序找到的第一个命令。

- 用命令行的方式设定别名是不能永久生效的，需要写入配置文件：~/.bashrc

![](F:\笔记\ContOS\Bash基本功能\2.png)

## 4.输入输出重定向

1. Bash的标准输入输出

|  设备  | 设备文件名  | 文件描述符 |     类型     |
| :----: | :---------: | :--------: | :----------: |
|  键盘  | /dev/stdin  |     0      |   标准输入   |
| 显示器 | /dev/stdout |     1      |   标准输出   |
| 显示器 | /dev/stderr |     2      | 标准错误输出 |

2. 输出重定向

   2.1标准输出重定向

```
命令 > 文件 	以覆盖的方式，把命令的正确输出到文件或设备当中
命令 >> 文件	以追加的方式，把命令的正确输出到文件或设备当中
```

​		2.2 标准错误输出重定向

```
错误命令 2> 文件 	以覆盖的方式，把命令的正确输出到文件或设备当中
错误命令 2>> 文件 	以追加的方式，把命令的正确输出到文件或设备当中
注意：这个2与>>(>)符号之间不能有空格
```

​	2.3 错误输出与正确输出重定向

```
#输出到同一个文件，注意：2>&1之间不能有空格
命令 > 文件 2>&1 	以覆盖的方式，把命令的正确输出和错误输出到同一文件
命令 &>文件 以覆盖的方式，把命令的正确输出和错误输出到同一文件
命令 >> 文件 2>&1 	以追加的方式，把命令的正确输出和错误输出到同一文件
命令 &>> 文件	以追加的方式，把命令的正确输出和错误输出到同一文件
#输出到不同文件
命令>>文件1 2>>文件2 把正确输出到文件1，错误输出到文件2
```

## 5.wc

利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

```
[root@localhost ~]# wc [选项] 文件名
选项：
	-c 统计字节数
	-w 统计单词数
	-l 统计行数
```

注意： wc 可以直接后面跟文件使用，但是会显示文件 ls -l|wc -l 统计行的时候包含了当前目录，所以会多一个所以一般使用管道符

```
# 统计行数
[root@localhost omc]# cat /etc/passwd | wc -l
# 统计最长
[root@localhost omc]# cat /etc/passwd | wc -L
```

## 6.多命令顺序执行

| 多命令执行符 | 格式           | 作用                                       |
| ------------ | -------------- | ------------------------------------------ |
| ;            | 命令1；命令2   | 多个命令顺序执行，命令之间没有任何逻辑联系 |
| &&           | 命令1&&命令2   | 当命令1正确执行，则命令2才会执行           |
| \|\|         | 命令1\|\|命令2 | 当命令1正确执行，则命令2不会执行           |
## 7.grep

行提取命令。用来查找字符串

```
[root@localhost ~]# grep [选项] "搜索内容" 文件名
选项：
	-c 统计找到的符合条件的字符串的次数
	-i 忽略大小写
	n 输出行号
	-v 反向查找
	--color=auto 颜色显示
```

## 8.管道符

命令格式：**命令A|命令B**，即命令1的正确输出作为命令B的操作对象

## 9.其他符号

| 符号 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| ''   | 单引号，在单引号中所有的特殊符号都没有了特殊的意义。$        |
| ""   | 双引号，在双引号中特殊符号都没有特殊含义但是"$"、"`、"\\"例外 |
| ``   | 反引号，反引号括起来的内容是系统命令，在bash中会优先执行，和$()的作用一样，不过推荐使用$() |
| $()  | 和反引号的作用一样，引用系统命令                             |
| ()   | 用于一串命令执行时，()中的命令会在子shell中执行              |
| \#   | 在shell脚本中，以#开头的行代表注释                           |
| $    | 用于调用变量的值，如需要调用变量name的值时，需要用$name的方式得到变量的值 |
| \    | 转义符，跟在\之后的特殊符号将失去特殊含义，变为普通字符。如\$将输出$符号，不当做变量引用 |

