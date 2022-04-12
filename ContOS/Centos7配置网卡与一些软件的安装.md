由于之前的虚拟机的磁盘空间不够了，也不是使用lvm逻辑卷组，所以最后选择了冲洗安装了虚拟机，所以这里记录一些一些基本环境的搭建，方便下次直接查询文档，而不需要各种百度各种软件的安装，一些框架使用docker进行安装，本文也会给出一些响应的命令进行参考学习

## 1.配置网卡

安装好虚拟机的第一步便是配置网络，配置静态ip，我一般是选择net模式这样每次在变更网络的时候不需要在进行重新设置

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改为：

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=192.168.18.107
NETMASK=255.255.255.0
GATEWAY=192.168.18.2
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=1097bcbf-a5c8-422f-8917-e5ae2db1ca84
DEVICE=ens33
ONBOOT=yes
```

重启网络：

```shell
systemctl restart netwrok
```

查看ip

![image-20210310194130325](http://cdn.noteblogs.cn/image-20210310194130325.png)



## 2.更换yum源

有一个快速的yum源对于安装软件非常有帮助，那么这里使用163的yum源

```shell
#备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
#更换
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
#刷新 建立缓存
yum makecache
```

## 3.安装docker

1)卸载docker

```shell
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

2)安装docker的依赖

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3)设置 docker repo 的 yum 位置

```shell
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

4)安装 docker， 以及 docker-cli

```shell
 yum install docker-ce docker-ce-cli containerd.io
```

5)启动docker

```shell
systemctl start docker
```

6)设置开机自启

```shell
systemctl enable docker
```

7)设置阿里云镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://263mn3oi.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 4.安装java环境

首先上传jdk-8u144-linux-x64.tar.gz至服务器，解压更改

```shell
tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
vim /etc/profile
```

末尾添加

```shell
export JAVA_HOME=/opt/module/jdk1.8
export JRE_HOME=${JAVA_HOME}/jre
export PATH=${JAVA_HOME}/bin:$PATH
```

使环境变量生效

```shell
source /etc/profile
```