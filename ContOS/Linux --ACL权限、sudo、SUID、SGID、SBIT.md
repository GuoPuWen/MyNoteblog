# 一、概述

1. ACL概念：

ACL的全称是 Access Control List (访问控制列表) ，一个针对文件/目录的访问控制列表。它在UGO权限管理的基础上为文件系统提供一个额外的、更灵活的权限管理机制

> Linux 文件的 ugo 权限把对文件的访问者划分为三个类别：文件的所有者、组和其他人。所谓的 ugo 就是指 user(也称为 owner)、group 和 other 三个单词的首字母组合。

2. 开启ACL权限，一般是开启了的。

```
[root@localhost ~]# dumpe2fs -h /dev/sda5(根分区)
选项：
	-h 仅显示超级块中的信息，而不显示磁盘块组的详细信息
```

如果有如下输出：

![](F:\笔记\ContOS\ACL\1.png)

如果没有默认开启，则需要重新挂载

```
[root@localhost ~]# mount -o remount, acl /
#重新挂载根分区，并加入ACL权限
```

可以写入配置文件，永久生效 /etc/fstab

# 二、ACL权限设置

## 1.ACL权限管理命令

```
[root@localhost ~]# getfacl 文件名
#查看ACL权限
[root@localhost ~]# setfacl 选项 文件名
#设定ACL权限
选项：
	-m u:用户名:权限 给用户设定ACL权限
	-m g：组名：权限 给组设定权限
```

## 2. 给用户和用户组添加ACL权限

```
#给用户添加ACL权限
[root@localhost ~]# setfacl  -m u:用户名:权限 文件名
#给用户组添加ACL权限
[root@localhost ~]#setfacl -m g：组名：权限 文件名 
```

注意：给某个文件设置了ACL权限之后，使用ll命令可以看到基本权限后面有个＋号

## 3.最大有效权限mask

mask 是用来指定最大有效权限的。mask 的默认权限是 rwx，如果我给 st 用户赋予了 r-x 的 ACL 权限，r-x需要和 mask 的 rwx 权限"相与"才能得到 st 的真正权限，也就是 r-x "相与"rwx 出的值是 r-x，所以 st 用户拥有 r-x 权限。

也就是说用户和用户组所设定的权限必须在 mask 权限设定的范围之内才能生效，mask权限就是最大有效权限。



## 4.默认ACL权限

- 作用：如果给父目录设定了默认 ACL 权限，那么父目录中所有新建的子文件都会继承父目录的 ACL 权限。默认 ACL 权限只对目录生效。

- ```
  [root@localhost ~]# setfacl -m d:u:用户名:权限 目录
  #使用"d:u:用户名：权限"格式设定默认ACL权限
  ```

  

## 5.递归ACL权限

- 作用：递归ACL权限是指父目录在设定 ACL 权限时，所有的子文件和子目录也会拥有相同的 ACL 权限。

- ```
  [root@localhost ~]#setfacl -m u:用户名:权限 -R 目录
  #-R递归
  ```

  

> 注意：默认 ACL 权限指的是针对父目录中新建立的文件和目录会继承父目录的 ACL 权限，格式是"setfacl-m d:u:用户名：权限 文件名"；递归 ACL 权限指的是针对父目录中已经存在的所有子文件和子目录继承父目录的 ACL 权限，格式是"setfacl-m u:用户名： 权限 -R 文件名"。

## 6.删除所有ACL权限

```
#删除指定用户的ACL权限
[root@localhost ~]#setfacl -x u:用户名 文件
#删除所有文件的ACL权限
[root@localhost ~]#setfacl -b 文件
```

# 三、sudo授权

sudo用于为普通用户授权，即允许哪些用户在哪些主机上登录后以管理员身份哪些命令。类似Windows系统中的右键以管理员方式运行。

给用户配置sudo授权

```
[root@localhost ~]# visudo
```

敲这条命令，得到的结果和使用vim编辑器一样。找到，

root    ALL=(ALL)       ALL

用户   ip=(可以使用的身份) 授权命令

一行，在这下面写需要给普通用户授权那些命令，比如，给普通用户授权添加adduser命令。注意：这里的授权命令要写绝对路径，不写绝对路径会报错。所以，先要做的就是使用whereis查看命令在那个文件下，然后再按上面格式添加至visudo文件中。

```
[root@localhost ~]# whereis useradd
useradd: /usr/sbin/useradd /usr/share/man/man8/useradd.8.gz
```

![](F:\笔记\ContOS\ACL\2.png)

这时切换至user1

![](F:\笔记\ContOS\ACL\3.png)

```
#查看用户的sudo权限
[user1@localhost root]$ sudo -l
```

# 四、文件特殊权限

## 1.SetUID

在用户管理这一块其实有一个疑问。用户的用户名、密码等信息是依赖于/etc/shadow这个文件的，也就是说如果用户要修改密码，则必须要修改/etc/shadow这个文件。那么问题来了，查看这个文件的权限发现：

```
[root@localhost ~]# ll /etc/shadow
----------. 1 root root 1305 2月   9 09:52 /etc/shadow
```

这个文件的权限其实是0的，也就是只有root用户才能进行强制修改！那么普通用户是怎么做到的呢。其实就是SUID权限起的作用。

SUID权限的作用：

- SUID权限仅对二进制程序(binary program)有效；
- 执行者对于该程序需要具有x的可执行权限；
- 本权限仅在执行该程序的过程中有效(run-time)；
- 执行者将具有该程序拥有者(owner)的权限。

也就是说，当普通用户使用passwd这个二进制程序时，这个二进制程序是具有SUID权限的，在执行这个权限的过程中，将会拥有所所有者的权限，也就是root权限，所以就可以修改/etc/passwd文件了。



- 设置SUID

```
[root@localhost ~]# chmod u+s 文件名
```

- 标志：当s这个标识出现在文件所有者的x权限上时，则这个文件有SUID权限
- 注意：SUID这个权限是很危险的，因为执行者会拥有root权限是很危险的。

## 2.SetGID

与SUID相似。

我们知道使用locate命令非常快的可以查找到文件是因为，它是按照数据库来查询的。也就是/var/lib/mlocate/mlocate.db这个文件的，查看这个文件的权限：

```
[root@localhost ~]# ll /var/lib/mlocate/mlocate.db 
-rw-r-----. 1 root slocate 1600318 2月   9 10:20 /var/lib/mlocate/mlocate.db
```

这个数据库文件的权限是640，其他人的权限是0，但是以普通用户的身份依旧可以使用这个命令进行查询操作。这是由于locate这个命令具有SGID权限。

SGID权限的作用：这个权限在执行过程中获得该程序所属用户组的权限。上面的例子中也就是slocate这个所属组的权限这样就可以查看了

- 设置

```
[root@localhost ~]# chmod g+s 文件名
```

- SGID主要是针对目录。如果用户在此目录下具有w权限的话，若使用者在此目录下建立新文件，则新文件的群组与此目录的群组相同。

   

## 3.Sticky Bit

SBIT（Sticky Bit）目前只针对目录有效，对于目录的作用是：当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除。
最具有代表的就是/tmp目录，任何人都可以在/tmp内增加、修改文件（因为权限全是rwx），但仅有该文件/目录建立者与 root能够删除自己的目录或文件。

- 设置

```
[root@localhost ~]# chmod o+t 目录
```

