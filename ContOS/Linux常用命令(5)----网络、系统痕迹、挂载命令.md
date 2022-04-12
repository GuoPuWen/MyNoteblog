# 一、网络命令

## 1.ifconfig

- 英文原意：configure a network interface

- 所在路径：/sbin/ifconfig

- 执行权限：超级用户

- 功能描述：配置网络接口

  ifconfig命令最主要的作用就是查看IP地址的信息，直接输入ifconfig命令即 可

  这个命令简单主要是这个命令的输出

  

## 2.ping

通过ICMP协议进行网络探测，测试网络中主机的通信情况

```
[root@localhost ~]# ping [选项] ip
选项：
	-b 后面加入广播地址，对整个网络进行探测
	-c次数 指定次数
```

## 3.netstat 

netstat是网络查看命令，即可以看到本机开启的端口，也可以看到那些客服端链接。

```
[root@localhost ~]netstat [选项]
选项
	-a 列出所有网络状态
	-t 使用TCP协议端口的连接情况
	-u 使用UCP协议端口的连接情况
	-l 仅显示监听状态的连接
	-r 显示路由表
```

例子：

1. 查看本机开启的端口

```
netstat -tuln
```

2. 查看所有连接

```
netstat -an
```

# 二、系统痕迹命令

## 1.简述

系统中有一些重要的痕迹日志文件，这些文件不能用vim打开，只能使用命令打开，如/var/log

/下的日志文件

## 2.w

w命令是显示系统中正在登陆的用户信息的命令，这个命令查看的痕迹日志是/var/run/utmp

```
[root@localhost ~]# w
 05:33:14 up  2:22,  3 users,  load average: 0.00, 0.00, 0.00
 系统当前时间		持续开机时间	登录的用户	系统在1分钟，5分钟，15分钟之内的平均负载
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     -                05:45   31.00s  0.06s  0.06s -bash
root     pts/0    192.168.18.1     05:46    0.00s  0.11s  0.07s w
```

第二行内容：

- USER：当前登录的用户

- TTY：登陆的终端： tty1-6本地字符终端(alt + F1~F6切换)

  ​									tty：本地图形终端

  ​									pts/0-255：远程终端，这里使用了远程工具

- FROM 登陆的ip
- LOGIN@：登录时间
- IDLE：用户闲置时间
- JCPU：所有进程占用的CPU时间
- PUPU：当前进程占用的CPU时间
- WHAT：用户正在进行的操作（W表示当前正在使用w命令的终端）

## 3. who

也是查看正在登陆的用户，但是比w命令更简单/var/run/utmp

## 4.last

查看系统所有登陆过的用户信息，包括正在登陆的用户和之前登陆的用户，查看的是/var/log/wtmp痕迹文件

## 5.lastlog

查看所有用户最后一次登陆的时间的命令/var/log/lastlog/

## 6.lastb

查看错误登陆信息/var/log/btmp

# 三、挂载命令

## 1.概念

挂载是指将一个设备（通常是存储设备）挂接到一个已存在的空目录上。这个目录可以不为空但是之前的内容将变为不可以访问，这个内容还是存在，只是这个内容的Inode节点号指向了存储设备，当取消挂载时，该内容可以访问。

## 2.mount umount

```
[root@localhost ~]# mount
#查看已经挂载的设备
[root@localhost ~]# mount [-t 文件系统] [-o 文件选项] 设备文件名 挂载点
[root@localhost ~]# umount 挂载点或者设备文件名
```

## 3.光盘挂载

- 主要是找到设备文件名，不同版本的Linux，设备文件名不一样
  - CentOS 5.x 以前的系统，光盘的设备文件名是/dev/hdc (c是指第三块设备，依次延顺)
  - CentOS 6.x以后的系统，设备文件名是/dev/sr0

- 其次是挂载点，一般挂载在/mnt/cdrom

- 用完之后一定要记得取消挂载，记得取消挂载时一定要推出光盘目录

  ```
  [root@localhost ~]# umount 挂载点或者设备文件名
  ```

  

## 4.挂载u盘

- 设备文件名：U盘会和硬盘共用设备文件名，所以需要手工查询

  ``````
  [root@localhost ~]#fdisk
  ``````

- 挂在点 /mnt/usb

- 如果U盘有中文，那么需要在挂载的时候指定中文编码

  ![](Linux\Snipaste_2020-02-04_20-22-30.png)

  

```
[root@localhost ~]# mount -o iocharset=utf8 /dev/sdb /mnt/usb
```

