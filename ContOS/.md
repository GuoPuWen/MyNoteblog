# 一、软件包分类

1. 源码包
   1. 开源
2. 二进制包
   1. DPKG包：Dbian系列的包管理机制
   2. RPM：Red Hat系列的包管理机制

# 二、RPM包依赖

1. 什么是依赖：

   有时我们会发现需要安装软件包a时需要先安装b和c，而安装b时需要安装d和e。这是需要先安装d和e，再安装b和C,最后才能安装a包。

2. 树形依赖
3. 环形依赖
4. 函数库依赖(模块依赖)。例如需要以.so.2为结尾的文件是函数库文件，而函数库文件没有单独成包，是在一个软件包里的，需要查询在那个软件包里。可以通过网站查询

# 三、rpm安装

## 1.rpm包全名命名规则

> httpd-2.2.15-45.el6.centos.i686.rpm

httpd：软件包名
		2.2.15：软件版本
		15：软件发布的次数
		e16：软件发行商。

i686：适合的硬件平台 
		rpm：rpm包的扩展

## 2.rpm包安装

1. 安装命令

```
[root@localhost ~]#rpm -ivh 包全名(绝对路径)
选项：
	-i 安装(install)
	-v 显示详细信息(verbose)
	-h 打印安装进度(hash)
	--force 强制安装，不管是否已经安装好，都强制安装
	--prefix指定安装路径(一般不使用)
```

## 3.rpm包升级

```
[root@localhost ~]#rpm -Uvh 包全名(绝对路径)
选项：
	-U 升级安装，如果没有安装过，系统直接安装
	-F 升级安装，系统必须有较旧版本
```

## 4.查询

```
[root@localhost ~]#rpm -q 包名
选项：
	-q 查询(query)
	-a:所有(all)
	-i:查询软件信息(information)
	-p:查询没有安装的软件包信息(package)
	-l:列出软件包中所有的文件列表和软件安装的目录(list)
	-f:查询系统文件属于那个软件包(file)
	-R:查询软件包的依赖性(requires)
```

1. **查询软件包是否安装**

   ```
   [root@localhost ~]#rpm -q 包名
   ```

   

注意：这里可以使用管道符进行包含匹配查询所需要的内容

2. **查询软件包的详细信息，这个软件包可以是已经安装也可以是没有安装。不过已经安装的软件包要写包名，没有安装的软件包要写包全名。**

   ```
   #查询已经安装
   [root@localhost ~]#rpm -qi 包名
   #查询没有安装
   [root@localhost ~]#rpm -qip 包全名
   ```

![](F:\笔记\ContOS\软件包管理\1.png)

3. **查询软件包中的文件列表**

   ```
   #查询已经安装的软件包
   [rootelocalhost]#rpm -ql 包名
   #查询没有安装的软件包
   [rootelocalhost]#rpm -qlp 包名选项：
   ```

4. 查询系统文件属于那个rpm包

   ```
   [rootelocalhost]#rpm -qf 系统文件
   #注意;这里只能查询系统文件，手工建立的文件不能查询
   ```

5. 查询软件包所依赖的软件包

   ```
   #查询已经安装的软件包所依赖的软件包
   [rootelocalhost]#rpm -qR 包名
   #查询没有安装的软件包所依赖的软件包
   [rootelocalhost]#rpm -qRp 包名
   ```

## 5.验证

### 1.验证

```
[root@localhost ~]#rpm -V 已经安装的包名
[root@localhost ~]#rpm -Vf 系统文件名
选项：
	-Vf校验某个系统文件是否被修改
```

### 2.数字证书

上面的验证方法可以对已经安装过的rpm包中的文件进行校验，判断文件是否被修改过，但是如果rpm包本身就被修改过，那么就校验就不准，这时可以使用数字证书验证

1. 数字证书的位置：①在第一张光盘中的RPM-GPG-KEY-CentOS-6 文件

![](F:\笔记\ContOS\软件包管理\2.png)

②系统中也有：/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

2. 导入数字证书

```
[root@localhost ~]#rpm --import 数字证书的位置
[root@localhost ~]# rpm -qa | grep gpg-pubkey #查看系统中安装好的数字证书
```

# 四、rpm包中文件的提取

情景：当一不小心把系统文件删除了，就在对应的rpm包中提取相应文件，拷贝到相应位置即可。例如当删除bin\ls时。便可通过查询系统文件对应软件包的命令查询到对应软件包，在使用提取命令复制到相应路径。

1. 提取命令

```
[root@localhost ~]# rpm2cpio 包全名(绝对路径) | cpio -idv .文件绝对路径
#这里的.不可省略
```

# 五、rpm在线安装(yum安装)

## 1.yum源文件解析

yum源配置文件保存在/etc/yum.repos.d/目录中，文件的扩展名一定是 *.repo

![](F:\笔记\ContOS\软件包管理\3.png)

## 2.yum配置文件解析

使用vim打开默认的/etc/yum.repos.d/ContOS-Base.repo  yum源配置文件

下面是部分配置文件的截图

![](F:\笔记\ContOS\软件包管理\4.png)

在CentOS-Base.repo文件中有5个yum源容器，这里只列出了base 容器，其他容器和base容器类似。

- [base]：容器名称，一定要放在]中。
- name：容器说明，可以自己随便写。
- mirrorlist：镜像站点，这个可以注释掉。
- baseurl：我们的yum源服务器的地址。默认是CentOS官方的yum源服务器，是可以使用的。
  如果你觉得慢，则可以改成你喜欢的yum源地址。
- enabled：此容器是否生效，如果不写或写成enabled=1则表示此容器生效，写成enabled=0则表示此容器不生效。
- gpgcheck：如果为1则表示RPM的数字证书生效；如果为0则表示RPM的数字证书不生效。
- gpgkey：数字证书的公钥文件保存位置。不用修改。

## 3.搭建网易163yum源

1. 下载网易yun源名字为CentOS-Base.repo_new默认下载目录为家目录

```
wget -O CentOS-Base.repo_new \ http://mirrors.163.com/.help/CentOS7-Base-163.repo
```

2. 将yum源配置文件下的其他yum源更改名字，将.repo文件全部改为.repo.bak这样方便以后要使用默认源时改回
3. 将/root/下的CentOS-Base.repo_new文件剪切到/etc/yum.repos.d/目录下，并将其后缀改为.repo

# 六、yum命令

## 1.查询

1. 查询yum源服务器上所有可用软件包

   ```
   [root@localhost ~]#yum list
   ```

2. 查询yum源服务器上是否含有莫个软件包

   ```
   [root@localhost ~]#yum list 包名
   ```

3. 查询yum源服务器上所有和关键字祥光的软件包

   ```
   [root@localhost ~]#yum list 关键字
   ```

## 2.安装

```
[root@localhost ~]#yum -y install 包名
选项：
	-y 自动回答yes
	install 安装
```

## 3.升级

```
[root@localhost ~]#yum -y update 包名
选项
	upadte 升级
	-y 自动回答yes
```

