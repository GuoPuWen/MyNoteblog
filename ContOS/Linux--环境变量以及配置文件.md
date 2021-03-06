在变量一节以及记录了变量的增删改查的方式，这一节介绍重要的环境变量，以及环境变量配置文件

## 一、重要的环境变量

##### 1.PATH

脚本在 Linux 中运行时，需要使用绝对路径或相对路径指定这个脚本所在的位置。PATH 环境变量中指定了系统命令的绝对路径，PATH 变量的值是用":"分隔的路径 。如果输入了一个命令，系统就会到 PATH 变量定义的路径中去寻找是否有可以执行的程序，如果找到则执行，否则会报"命令没有发现"的错误。也可以把用户自己写的脚本复制到 PATH 变量定义的路径中，通过脚本名来直接运行，

```
[root@localhost ~]# echo $PATH
/usr/lib/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
```

通过将脚本对应的全路径追加到环境变量中，然后直接再通过脚本名来直接运行脚本程序

##### 2.PS1

PS1 是用来定义命令行的提示符的，可以按照我们自己的需求来定义自己喜欢的提示符。例如当前系统下是[root@localhost ~]#这个提示符，查看PS1变量

```
[root@localhost ~]# echo $PS1
[\u@\h \W]\$
```

PS1可以支持下面选项：

| 选项 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| \d   | 显示曰期，格式为"星期 月 日"。                               |
| \H   | 显示完整的主机名。如默认主机名"localhost.localdomain"        |
| \h   | 显示简写的主机名。如默认主机名"localhost"                    |
| \t   | 显示 24 小时制时间，格式为"HH:MM:SS"                         |
| \T   | 显示 12 小时制时间，格式为"HH:MM:SS"                         |
| \A   | 显示 24 小时制时间，格式为"HH:MM"                            |
| \@   | 显示 12 小时制时间，格式为"HH:MM am/pm"                      |
| \u   | 显示当前用户名                                               |
| \w   | 示当前所在目录的完整名称                                     |
| \W   | 显示当前所在目录的最后一个目录                               |
| \\$  | 提示符。如果是 root 用户，则会显示提示符为"#"；如果是普通用户，则会显示提示符为"$"。 |

可以将默认的更改：

```
[root@localhost ~]# PS1='[\u@\H \t]\$  '
[root@localhost.localdomain 11:03:44]#  
```

注意：在PS1 变量的值要用单引号包含，否则设置不生效。而且这些提示符的修改同样是临时生效的，一旦注销或重启系统就会消失。要想永久生效，必须写入环境变量配置文件。

##### 3.LANG

LANG变量：定义系统的主语系环境

```
[root@localhost ~]# echo $LANG
zh_CN.UTF-8
```

使用 locale 命令查询当前系统支持什么语言。LANG 是定义系统语言的变量，更改这个变量只是临时生效，需要改配置文件：/etc/sysconfig/il8n永久生效

## 二、环境变量配置文件

环境变量配置文件主要是有5个

```
系统层次：
/etc/profile 
/etc/profile.d/*.sh 
/etc/bashrc
用户层次
~/.bash_profile 
~/.bashrc
```

-  /etc/profile

在/etc/profile文件中修改环境变量，在这里修改的内容是对所有用户起作用的，这里主要定义了

​		PATH：决定了shell将到哪些目录中寻找命令或程序

　　HOME：当前用户主目录

　　MAIL：是指当前用户的邮件存放目录。

　　SHELL：是指当前用户用的是哪种Shell。

　　HISTSIZE：是指保存历史命令记录的条数。

　　LOGNAME：是指当前用户的登录名。

　　HOSTNAME：是指主机的名称，许多应用程序如果要用到主机名的话，通常是从这个环境变量中来取得的。

　　LANG/LANGUGE：是和语言相关的环境变量，使用多种语言的用户可以修改此环境变量。

　　umask：定义umask默认权限

- /etc/profile.d/*.sh 

  这里指的是/etc/profile.d/目录下的所有以.sh文件结尾的脚本文件

![](F:\笔记\ContOS\环境变量\1.png)



- /etc/bashrc

PS1

