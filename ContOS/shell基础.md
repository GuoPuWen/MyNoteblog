

## 1.概念

- 在计算机科学中，Shell俗称壳（用来区别于核），Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

- 分类：
  - Bourne Shell（/usr/bin/sh或/bin/sh）

  - Bourne Again Shell（/bin/bash）

  - C Shell（/usr/bin/csh）

  - K Shell（/usr/bin/ksh）

  - Shell for Root（/sbin/sh）

    .........

- /etc/shells可以查看支持的shell类型

## 2.shell的执行方式

1. 赋予执行权限
2. 相对路径或者绝对路径执行

## 3.shell数值运算

在shell变量一节中知道shell变量默认类型都是字符串格式的，那么如果要进行数值运算比较麻烦。

```
[root@localhost ~]# a=1
[root@localhost ~]# b=2
[root@localhost ~]# echo $a+$b
1+2
```

1. 第1种方法是使用declare，这种方法使用起来比较复杂，了解一下

```
declare [+/-][选项] 变量名
选项：
   -：给变量舍得类型属性
   +：取消变量的类型属性
  -a：将变量声明为数组型
  -i：将变量声明为整型
  -x：将变量声明为环境变量
  -r：将变量声明为只读变量
  -p：查看变量的被声明的类型
```

2. 第2种方法是使用expr或let数值运算工具

```
#使用expr
[root@localhost ~]# a=1
[root@localhost ~]# b=2
[root@localhost ~]# dd=$(expr $a + $b)
[root@localhost ~]# echo $dd
3
#使用let
[root@localhost ~]# let c=$a+$b
[root@localhost ~]# echo $c
3
```

使用expr +号两边必须要有空格，使用let就没那么多要求了

3. 使用$((运算式)) 或$[运算式]

   ```
   [root@localhost ~]# aa=1
   [root@localhost ~]# bb=2
   [root@localhost ~]# echo $(($aa + $bb ))
   3
   ```

## 4.变量测试与内容置换

| 变量置换方式 | 变量y没有设置                  | 变量y为空值            | 变量y设置值 |
| ------------ | ------------------------------ | ---------------------- | ----------- |
| x=${y-新值}  | x= 新值                        | x 为空                 | x=$y        |
| x=${y:-新值} | x= 新值                        | x= 新值                | x=$y        |
| x=${y+新值}  | x 为空                         | x= 新值                | x=新值      |
| x=${y:+新值} | x 为空                         | x 为空                 | x=新值      |
| x=${y=新值}  | x= 新值                        | x 为空                 | x=$y        |
| y= 新值      | y 值不变                       | y值不变                |             |
| x=${y:=新值} | x= 新值                        | X= 新值                | x=$y        |
| y= 新值      | y= 新值                        | y值不变                |             |
| x=${y?新值}  | 新值输出到标准错误输出（屏幕） | x 为空                 | x=$y        |
| x=${y:?新值} | 新值输出到标准错误输出         | 新值输出到标准错误输出 | x=$y        |

变量测试与内容置换只是为了检测一个变量是否是空值还是没有设置，直接使用echo是无法看出来的

```
[user1@localhost root]$ echo $a

[user1@localhost root]$ s=""
[user1@localhost root]$ echo $s

[user1@localhost root]$ 
```

都输出的是空。所以使用x=${y-新值}可以用来判断一个变量是否存在还是为空

