# 一、搜索命令

## 1.搜索系统命令的命令

### 1.whereis

whereis是搜索系统命令的命令，whereis不能搜索普通文件，只能搜索系统命令。

### 2.which

which也是搜索系统命令，区别

> whereis命令可以查找二进制命令的同时，查找到帮助文档的位置
>
> which命令 在查找命令的同时，如果这个命令有别名，可以找到别名



## 2.搜索文件的命令

### 1.locate

locate命令可以按照文件名搜索普通文件的命令

优点：按照数据库搜索，搜索速度很快，消耗资源小，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db 。这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。

1. ```
   1. /etc/updatedb.conf  updatedb的配置文件
   2. /var/lib/mlocate/mlocate.db 存放文件信息的文件
   ```

   注意：在updatedb的配置文件里面，有一些目录不能被搜索到，比如/tmp/目录



### 2.find

1. 按照文件名来搜索

   ```
   [root@localhost ~]# find 搜索路径 选项 搜索内容
   选项
   	-name 按照文件名
   	-iname 不区分大小写
   	-inum 按照inode节点号
   ```

   

2. 按照文件大小

   ```
   [root@localhost ~]# find 搜索路径 选项 搜索内容
   选项
   	-size  n[c] 查长度为n块[或n字节]的文件
   	-size n[cwbkMG]
                 File uses n units of space.  The following suffixes can be used:
   
                 ‘b’    for 512-byte blocks (this is the default if no suffix is used)  #默认单位
   
                 ‘c’    for bytes
   				
                 ‘w’    for two-byte words
   
                 ‘k’    for Kilobytes (units of 1024 bytes)
   				#k单位小写
                 ‘M’    for Megabytes (units of 1048576 bytes)
   				#M单位大写
                 ‘G’    for Gigabytes (units of 1073741824 bytes)
   				# 大写
   ```

   注意：find命令是不能写小数大小的，可以使用加减号，代表大于小于。当大小后面不加单位时，默认是以512字节为单位的



3. 按照文件修改时间

   ```
   root@localhost ~]# find 搜索路径 [选项] 搜索内容
   
   选项：
   -mtime   -n +n                   #按文件更改时间来查找文件，-n指n天以内，+n指n天以前
   -atime    -n +n                   #按文件访问时间来查找文件，-n指n天以内，+n指n天以前
   -ctime    -n +n                  #按文件创建时间来查找文件，-n指n天以内，+n指n天以前
   ```

   

![](Linux\QQ截图20200127100239.png)

4. 按照权限

   ```
   [root@localhost ~]# find 搜索路径 [选项] 搜索内容
   
   选项：
   -perm 权限模式：査找文件权限刚好等于"权限模式"的文件
   -perm -权限模式：査找文件权限全部包含"权限模式"的文件（即3个身份的权限都要比这个权限大）
   -perm +权限模式：査找文件权限包含"权限模式"的任意一个权限的文件(即3个身份的权限只要有一个比它大即可)
   ```

   ![](F:\笔记\ContOS\Linux\QQ截图20200127101437.png)

test目录下共有2个文件。123权限是644，abc权限是600。用命令find -perm +444为3个身份中只要有1个比444大即可以查到，所以123、abc；用命令find -perm -444为3个身份的权限都要比444大，所以只有123可以查到

5. 按照所有者和所属组搜索

   ```
   -user 用户名：按照用户名査找所有者是指定用户的文件
   -group 组名：按照组名査找所属组是指定用户组的文件
   -nouser：査找没有所有者的文件(常用)。用于查找垃圾文件（windows下的文件）
   ```

   

6. 按照文件类型搜索

   

   ```
   -type d：查找目录
   -type f：查找普通文件
   -type l：查找软链接文件
   ```

   

7. 逻辑运算符

   ```
   [root@localhost ~]#find 搜索路径 [选项] 搜索内容
   
   选项：
   -a：and逻辑与（两条件都成立）
   -o：or逻辑或（两条件成立一个）
   -not：not逻辑非
   ```

8. 其他选项

   1. -exec选项

      ```
      [root@localhost ~]# find 搜索路径 [选项] 搜索内容 -exec 命令2{}\;
      ```

      这里的"{}"和"\;"是标准格式，只要执行"-exec"选项，这两个符号必须完整输入。这个选项的作用其实是把 find 命令的结果交给由"-exec"调用的命令 2 来处理。"{}"就代表 find 命令的査找结果。

   2. -ok

      -ok"选项和"-exec"选项的作用基本一致，区别在于："-exec"的命令会直接处理，而不询问；"-ok"的命令 2 在处理前会先询问用户是否这样处理，在得到确认命令后，才会执行。

### 3.grep

find是文件查找， grep是文件内容查找。

```
    [root@localhost ~]#grep 搜索内容 文件名
```

完全匹配：又称精确匹配 ，是另一种关键词匹配形式。只有搜索的词和关键词完全一样时，才会显示。find

包含匹配：模糊匹配。grep

## 3.管道符

- ** 使用管道操作符“|”可以把一个命令的标准输出传送到另一个命令的标准输入中，连续的|意味着第一个命令的输出为第二个命令的输入，第二个命令的输入为第一个命令的输出，依次类推。**

注意：管道符输出的是文本流

例子：①分页显示目录下的文件

```
[root@localhost ~]# ll /etc | more
```



## 4.命令的别名

- 命令的小名，照顾管理员的习惯。

  使用命令 alias可以查看已有的别名

  ![](F:\笔记\ContOS\Linux\QQ截图20200128104414.png)

- 设置别名

  ```
  [root@localhost ~]#alias 原名字='别名'
  ```

  

## 5.快捷键

| 快捷键   | 作用                     |
| -------- | ------------------------ |
| Tab      | 命令补全                 |
| ctrl + A | 把光标移动到开头         |
| ctrl + E | 把光标移动到命令行结尾   |
| ctrl + C | 终止当前命令             |
| ctrl + L | 清屏                     |
| ctrl + U | 删除或剪切光标之前的命令 |
| ctrl + Y | 粘贴ctrl+u的内容         |

