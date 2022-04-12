# Kubernetes介绍

Kubernetes简称K8s，前面已经介绍过docker，docker的出现使得部署应用程序变得更为简单，部署应用程序一共经历了三个时代：

- 传统部署：直接将应用程序部署到物理机上，优点是操作简单，但是缺点也很明显，无法为应用程序设置资源边界，很难的合理分配资源，程序之间也容易影响
- 虚拟化部署：在一台物理机上运行多个虚拟机，虚拟化让应用程序之间隔离，应用程序之间不会相互影响，但是每一个VM都是一个操作系统，为了部署一个应用需要安装一个操作系统，资源浪费太大
- 容器部署时代：可以在应用程序之间共享操作系统，容器是比较轻量的，容器化的应用程序可以跨操作系统进行部署，资源之间隔离并且可以合理利用资源

![image-20210712104002369](http://cdn.noteblogs.cn/image-20210712104002369.png)

docker便是支持容器部署的技术，但是仅仅具有docker，也存在一些问题：

- 一个容器故障停机了，怎么让另外一个容器启动并工作呢？
- 如果访问量增大了，如何扩容？访问量变小，如何缩呢

而这些就是容器编排问题，就是要出现新的技术去管理docker，容器编排软件目前一共有三款：

- Swarm：dock儿自己的容器编排工具
- Mesos：Apache的一个资源管理统一管控的工具
- Kubernets：Google开源的容器编排工具

目前，K8s市场占有率是最大的

# 组件

一个K8s集群主要是由控制节点（masger）、工作节点（node）构成的，每个节点上会安装不同的组件

![image-20210712104753355](http://cdn.noteblogs.cn/image-20210712104753355.png)

master节点：集群的控制平面，负责集群的管理和决策：

- APIServer：资源操作的唯一入口，接收用户输入的命令，提供认证，授权，API注册和发现等机制
- Scheduler：负责集群的资源调度，按照一定的策略将pod调度到相应的node节点上
- ContrllerManager：负责维护集群的状态，例如程序部署安排、故障检测、自动扩容等等
- Etcd：负责存储集群中的各种资源对象的信息

node：集群的数据平面，负责为容器提供运行环境

- Kubelet：维护容器的生命周期、通过控制docker来创建、更新、销毁容器
- KubeProxy：负责集群内部的负载均衡
- Docker：负责节点上容器的各种操作

例如，部署一个Nginx应用，各种组件的调用关系如下：

1. 首先，容器启动过的时候，会将master和node节点的信息存储到etcd数据库里面
2. 按照nginx的服务请求会首先被发送到master的ApiServer组件
3. ApiServer组件会调用Scheduler决定将这个服务安装到那一个节点上，会从etcd总读取节点信息，然后在按照算法进行选择，将结果告知ApiServer
4. ApiServer调用ContrllerManager去调度Node节点来安装nginx
5. kubelet接收到指令后，会通知docker，然后由docker启动一个nginx的pod，pod是k8s的最小单元的概念，容器必须在pod中运行
6. 外部如果要访问nginx，需要通过kubeProxy来对pod产生访问的代理

# 集群环境搭建

搭建K8s的测试环境，采取一主两从的规划部署，主机规划如下：

| 作用   | IP地址    | 操作系统                    | 配置                     |
| ------ | --------- | --------------------------- | ------------------------ |
| Master | 10.0.0.7  | Centos7.8    基础设施服务器 | 2颗CPU  4G内存   50G硬盘 |
| Node1  | 10.0.0.16 | Centos7.8    基础设施服务器 | 2颗CPU  4G内存   50G硬盘 |
| Node2  | 10.0.0.6  | Centos7.8    基础设施服务器 | 2颗CPU 4G内存   50G硬盘  |

> 1. 配置主主机名解析，在/etc/hosts文件下，对三台机器同时操作

```
10.0.0.7 master
10.0.0.16 node1
10.0.0.6 node2
```

> 2. 时间同步，因为使用的是腾讯云的服务器，所以时间是同步的

```shell
[root@VM-0-7-centos ~]# systemctl start chronyd
[root@VM-0-7-centos ~]# systemctl enable chronyd
[root@VM-0-7-centos ~]# systemctl status chronyd
```

> 3. 禁用iptables和防火墙规则，因为是测试环境，所以直接删除

```shell
[root@VM-0-7-centos ~]# systemctl stop firewalld
[root@VM-0-7-centos ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@VM-0-7-centos ~]# systemctl disable firewalld
```

> 4. 禁用selinux

```shell
# 编辑 /etc/selinux/config 文件，修改SELINUX的值为disabled
# 注意修改完毕之后需要重启linux服务
SELINUX=disabled
```

> 5. 禁用swap分区
>
>    swap分区指的是虚拟内存分区，它的作用是在物理内存使用完之后，将磁盘空间虚拟成内存来使用
>
>    启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备
>
>    但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明

```shell
# 编辑分区配置文件/etc/fstab，注释掉swap分区一行
# 注意修改完毕之后需要重启linux服务
 UUID=455cc753-7a60-4c17-a424-7741728c44a1 /boot    xfs     defaults        0 0
 /dev/mapper/centos-home /home                      xfs     defaults        0 0
# /dev/mapper/centos-swap swap                      swap    defaults        0 0
```

> 6. 修改linux的内核参数

```shell
# 修改linux的内核参数，添加网桥过滤和地址转发功能
# 编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置:
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

# 重新加载配置
[root@VM-0-7-centos ~] sysctl -p

# 加载网桥过滤模块
[root@VM-0-7-centos ~] modprobe br_netfilter

# 查看网桥过滤模块是否加载成功
[root@VM-0-7-centos ~] lsmod | grep br_netfilter
```

> 7.配置ipvs

```shell
# 1 安装ipset和ipvsadm
[root@VM-0-7-centos ~]# yum install ipset ipvsadmin -y

# 2 添加需要加载的模块写入脚本文件
[root@VM-0-7-centos ~]# cat <<EOF >  /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

# 3 为脚本文件添加执行权限
[root@VM-0-7-centos ~]# chmod +x /etc/sysconfig/modules/ipvs.modules

# 4 执行脚本文件
[root@VM-0-7-centos ~]# /bin/bash /etc/sysconfig/modules/ipvs.modules

# 5 查看对应的模块是否加载成功
[root@VM-0-7-centos ~]# lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

> 9. 重启linux

```shell
[root@VM-0-7-centos ~]# reboot
```

> 10. 安装docker

```shell
# 1 切换镜像源
[root@ks8_master ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 2 查看当前镜像源中支持的docker版本
[root@ks8_master ~]# yum list docker-ce --showduplicates

# 3 安装特定版本的docker-ce
# 必须指定--setopt=obsoletes=0，否则yum会自动安装更高版本
[root@ks8_master ~]# yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4 添加一个配置文件
# Docker在默认情况下使用的Cgroup Driver为cgroupfs，而kubernetes推荐使用systemd来代替cgroupfs
[root@ks8_master ~]# mkdir /etc/docker
[root@ks8_master ~]# cat <<EOF >  /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF

# 5 启动docker
[root@ks8_master ~]# systemctl restart docker
[root@ks8_master ~]# systemctl enable docker

# 6 检查docker状态和版本
[root@ks8_master ~]# docker version
```

> 11. 安装kubernetes组件

```java
# 由于kubernetes的镜像源在国外，速度比较慢，这里切换成国内的镜像源
# 编辑/etc/yum.repos.d/kubernetes.repo，添加下面的配置 
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 安装kubeadm、kubelet和kubectl
[root@master ~]# yum install --setopt=obsoletes=0 kubeadm-1.17.4-0 kubelet-1.17.4-0 kubectl-1.17.4-0 -y

# 配置kubelet的cgroup
# 编辑/etc/sysconfig/kubelet，添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"

# 4 设置kubelet开机自启
[root@master ~]# systemctl enable kubelet
```

> 12.新建一个脚本文件，为1.sh

```shell
images=(
    kube-apiserver:v1.17.4
    kube-controller-manager:v1.17.4
    kube-scheduler:v1.17.4
    kube-proxy:v1.17.4
    pause:3.1
    etcd:3.4.3-0
    coredns:1.6.5
)

for imageName in ${images[@]} ; do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName 		k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

```shell
[root@master ~]# sh 1.sh
```

> 13. 集群初始化

```shell
# 创建集群
[root@master ~]# kubeadm init \
	--kubernetes-version=v1.17.4 \
    --pod-network-cidr=10.244.0.0/16 \
    --service-cidr=10.96.0.0/12 \
    --apiserver-advertise-address=10.0.0.7

# 创建必要文件
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

注意：下面的操作只需要在`node`节点上执行即可

当做到这一步的时候，会提示怎么讲slave节点加入进来，这里忘记截图了，只需要按照提示操作即可

```shell
[root@node1 ~]# kubeadm join 10.0.0.7:6443 --token gyh0bu.5q6z10ns7t6q4vxt     --discovery-token-ca-cert-hash sha256:d409e049d02490021903bac41edf86f29b86e62b6f9a775dfb87b5679ea382b3

# 查看集群状态 此时的集群状态为NotReady，这是因为还没有配置网络插件
[root@master ~]# kubectl get nodes
```

> 14. 安装网络插件

下面操作值只需要在master上执行即可

```shell
# 获取fannel的配置文件
[root@master ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 修改文件中quay.io仓库为quay-mirror.qiniu.com

# 使用配置文件启动fannel
[root@master ~]# kubectl apply -f kube-flannel.yml

# 稍等片刻，再次查看集群节点的状态
[root@master ~]# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   15m     v1.17.4
node1    Ready    <none>   8m53s   v1.17.4
node2    Ready    <none>   8m50s   v1.17.4
```

# 测试：搭建一个Nginx

```shell
# 部署nginx
[root@master ~]# kubectl create deployment nginx --image=nginx:1.14-alpine

# 暴露端口
[root@master ~]# kubectl expose deployment nginx --port=80 --type=NodePort

[root@master ~]# kubectl get pods,service
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6867cdf567-lt9wz   1/1     Running   0          29m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        5h52m
service/nginx        NodePort    10.107.160.22   <none>        80:30909/TCP   29m
```

![image-20210712201016598](http://cdn.noteblogs.cn/image-20210712201016598.png)