# DockerFile

DockerFile就是用来构建docker镜像的文件，其实就是命令参数脚本，本篇文章首先介绍一些基本的命令操作，后面将会以制作一个自带tomcat的centos镜像作为实战！

使用dockerfile创建镜像的基本步骤是：

- 编写一个合格的dockerfile文件
- docker build构建一个镜像
- docker run运行镜像
- 发布到镜像仓库，例如阿里云仓库、DockerHub等等

在官网上查看一些官方的镜像，例如centos的镜像，可以看到其中的DockerFile文件是如何编写的，这对我们编写dockerfile有帮助

```dockerfile
FROM centos:7
ENV container docker
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
```

### 基本命令

需要注意的是：

- 每个dockerfile的关键字都需要大写，从上到下执行
- 根据docker镜像原理，每一个指令都会创建并提交一个新的镜像，最后run的时候是在镜像的最上层叠加一层读写层

DockerFile一共有以下一些指令：

```shell
FROM											# 基础镜像，表示我这个镜像的最底层镜像
MAINTAINER								# 标注镜像的作者，一般是姓名 + 邮箱
RUN												#	镜像构建时需要执行的命令
ADD												# 添加一些文件
WORKDIR										#	工作目录，表示当前的工作目录
VOLUME										# 挂载目录，这个前面用到过
EXPOSE										# 保留端口
CMD												# 指定这个容器启动时候去要执行的命令，只有最后一个生效
ENTRYPOINT								# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD										# 当构建一个被继承 DockerFile 这个时候就会运行ONBUILD的指令，触发指令。
COPY											# 类似ADD，将我们文件拷贝到镜像中
ENV												# 构建的时候设置环境变量！
```

### 创建自己的centos

使用官方镜像的centos，发现是没有vim，ifconfig这些基本命令的，而这次制作centos的目标是制作自己的镜像，然后添加这些命令

```dockerfile
FROM centos

ENV MYPATH /usr/local
WORKDIR	$MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD /bin/bash
```

```shell
docker build -t mycentos . 			#构建docker镜像
docker run -it adf1fe744351			#运行镜像，发现ifconig vim命令都有了
```



### 创建Tomat镜像

创建一个Tomcat镜像，需要准备tomcat的压缩包和jdk的压缩包，直接可以在官网下载，接着就是编写dockerfile文件

```dockerfile
FROM centos
ADD apache-tomcat-9.0.50.tar.gz /usr/local
ADD jdk-8u291-linux-x64.tar.gz /usr/local
RUN yum -y install vim
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENV JAVA_HOME /usr/local/jdk1.8.0_291
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.50
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib
EXPOSE 8080
CMD /usr/local/apache-tomcat-9.0.50/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.50/logs/catalina.out
```



