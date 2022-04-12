# 容器技术的出现

起初，研发人员开发一个软件，在自己的本地环境上开发好了，需要交给测试人员测试人员需要搭建一套测试环境进行测试，接着交给运维人员，运维人员需要搭建一套上线环境，然后发现上线系统崩溃了，因为很多服务器都是linux的！

可以看出上面的流程的一些弊端：

- 需要搭建三套环境，及其浪费了时间与资源
- 上线环境和本地环境系统不一样，维护困难

不过VMware的出现解决了上面的一些问题，研发人员通过虚拟机搭建好虚拟的上线环境然后交给测试人员和运维人员，在没有容器技术之前，这样解决确实是一个好办法

但是新的问题又出现了，虚拟机需要搭建一整个操作系统，而对于我部署一个Java应用来说，可能只需要JDK + MySQL环境即可，也就说有点大才小用，为了部署一个几十兆的Java应用，需要先安装一个几G的操作系统，这样就太占用资源了

为了解决这个问题，更轻量级的东西出现了，就是容器技术！

容器技术的根本思想是集装箱思想，在海洋船舶运输的过程中，集装箱之间相互隔离、反复使用、快速装卸以及标准规格。容器化技术和这很相似，应用程序在运行的时候互不干扰，容器只隔离应用程序的运行时环境但容器之间可以共享同一个操作系统，容器更加的轻量级以及占用的资源更少

使用容器之后，对于部署一个Java环境的流程应该是：Java代码 - jar - 打包项目变成容器镜像 - docker商店 ，其他人想要使用只需要下载这个镜像即可



# Docker是什么

Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级、可移植、自给自足的容器，Docker是基于Go语言开发的开源项目：

- docker官网：https://www.docker.com/
- 文档：https://docs.docker.com/
- 仓库：https://hub.docker.com/

使用docker的好处：

- 应用更快速的交付和部署，使用docker打包镜像发布测试一键运行
- 更便捷的升级和扩缩容，使用了docker一行，部署应用就类似于搭积木
- 更简单的系统运维：在容器化开发之后，测试环境和上线环境都是一致的
- 更高效的计算资源利用：docker是内核级别的虚拟化，可以在一个物理机上运行很多的容器实例

# 安装Docker

环境前提：LInux内核3.0以上

```shell
[root@tdsql_1 ~]# uname -r
3.10.0-1062.18.1.el7.x86_64
[root@tdsql_1 ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

