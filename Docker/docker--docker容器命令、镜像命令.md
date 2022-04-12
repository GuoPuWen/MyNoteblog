# 配置阿里云镜像加速器

在阿里云官网，找到镜像服务，根据说明配置镜像加速器

![image-20210705072949133](/Users/guopuwen/Documents/笔记/docker/docker--docker容器命令、镜像命令.assets/image-20210705072949133.png)

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

# Docker常用命令

### 帮助命令

```shell
docker version			#显示docker的版本信息
docker info					#显示docker的系统信息
docker 命令 --help	 #帮助命令
```

帮助文档：https://docs.docker.com/engine/reference/commandline/build/

### 镜像命令

**`docker images`查看所有本地主机的镜像**

```shell
[root@tdsql_1 ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         latest    5c62e459e087   11 days ago    556MB
hello-world   latest    d1165f221234   4 months ago   13.3kB
```

一些可选项：

```shell
Options:
  -a, --all             Show all images
  -q, --quiet           Only show image IDs
```

```shell
[root@tdsql_1 ~]# docker images -aq		#显示所有的镜像id
5c62e459e087
d1165f221234
```

**`docker search搜素镜像	`**

![image-20210705073802667](/Users/guopuwen/Documents/笔记/docker/docker--docker容器命令、镜像命令.assets/008i3skNgy1gs5un7ys6pj31be0rg17a.jpg)

一些可选项

```shell
[root@tdsql_1 ~]# docker search --help

Usage:  docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions 
```

**`docker pull`下载镜像**

```shell
[root@tdsql_1 ~]# docker pull tomcat:8
8: Pulling from library/tomcat	#默认最新版
0bc3020d05f1: Pull complete			#分层下载
a110e5871660: Pull complete
83d3c0fa203a: Pull complete
a8fd09c11b02: Pull complete
96ebf1506065: Pull complete
26b72ffca293: Pull complete
0bffa2ea17aa: Pull complete
d880cebcc7a6: Pull complete
d607223b73ca: Pull complete
27a5deacef56: Pull complete
Digest: sha256:a266dd222864de2fe72e0464e6d91c406a687c861bb72a07218e6d7c89fe1d3e
Status: Downloaded newer image for tomcat:8
docker.io/library/tomcat:8
```

**`docker rmi`删除镜像**

```shell
docker rmi -f 镜像id	#删除指定的镜像
docker rmi -f 镜像id 镜像id 镜像id		#批量删除
docker rmi -f $(docker images -aq)	#删除所有镜像
```

### 容器命令

```shell
docker run 镜像id			#新建容器并启动
dovker ps 					 #列出所有运行的容器
docker rm 容器id			#删除指定的容器
docker start 容器id		#启动容器
docker restart 容器id	#重启容器
docker stop 容器id		#停止容器
docker kill 容器id		#强制停止当前正在运行的容器
```

**docker run 新建容器并运行**

首先拉取一个centos最新版的镜像

```shell
docker pull centos
```

```shell
docker run [可选参数] image | docker container run [可选参数] image 
#说明
--name="Name"		容器名字 tomcat01 tomcat02 用来区分容器
-d					后台方式运行
-it 				使用交互方式运行，进入容器查看内容
-p					指定容器的端口 -p 8080(宿主机):8080(容器)
		-p ip:主机端口:容器端口
		-p 主机端口:容器端口(常用)
		-p 容器端口
		容器端口
-P(大写) 				随机指定端口

```

启动centos镜像

```shell
docker run -it centos /bin/bash
```

![image-20210705075128833](/Users/guopuwen/Documents/笔记/docker/docker--docker容器命令、镜像命令.assets/image-20210705075128833.png)

从这里也可以看出，docker中的镜像比一整个操作系统更加轻量的多

**docker -ps 列出所有正在运行的容器**

```shell
[root@tdsql_1 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
e0b2193fb0e3   centos    "/bin/bash"   24 seconds ago   Up 22 seconds             musing_rhodes
[root@tdsql_1 ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                          PORTS     NAMES
e0b2193fb0e3   centos         "/bin/bash"   29 seconds ago   Up 28 seconds                             musing_rhodes
c76d1b60f4ec   centos         "/bin/bash"   2 minutes ago    Exited (0) About a minute ago             adoring_goodall
9860d2f2f9bb   d1165f221234   "/hello"      37 hours ago     Exited (0) 37 hours ago                   recursing_jang
5ef10e91fd30   d1165f221234   "/hello"      37 hours ago     Exited (0) 37 hours ago                   sharp_cartwright
[root@tdsql_1 ~]# docker ps -aq
e0b2193fb0e3
c76d1b60f4ec
9860d2f2f9bb
5ef10e91fd30
```

一些选项

```shell
#docker ps命令 #列出当前正在运行的容器
  -a, --all             Show all containers 
  -n, --last int        Show n last created containers
  -q, --quiet           Only display numeric IDs
```

**exit 退出容器**

```shell
exit #容器直接退出
ctrl +P +Q #容器不停止退出
```

**docker rm 删除容器**

```shell
docker rm 容器id   #删除指定的容器，不能删除正在运行的容器，如果要强制删除 rm -rf
docker rm -f $(docker ps -aq)  #删除指定的容器
docker ps -a -q|xargs docker rm  #删除所有的容器
```

**启动和停止容器**

```shell
docker start 容器id	#启动容器
docker restart 容器id	#重启容器
docker stop 容器id	#停止当前正在运行的容器
docker kill 容器id	#强制停止当前容器
```

### 常用的其他命令

**后台启动命令**

```shell
[root@tdsql_1 ~]# docker run -d centos
5325e7ed9a67eeff61f70fe3dbf1c8ce5934c0349a8650a0ae49fa3e21cd420a
[root@tdsql_1 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

当启动容器之后，使用的是-d参数也就是后台运行，但是docker ps并没有发现该运行的容器，原因是docker使用容器后台运行，发现没有前台进程，则就会自动停止

**查看日志**

```shell
[root@tdsql_1 ~]# docker logs --help

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
  
  -tf		# 显示日志信息
  -tail 10	#显示1o条日志
  docker logs -t -tail n 容器id	#显示n行日志
  docker logs -ft 容器id	#跟着日志
```

**查看容器中进程信息 top**

```shell
[root@tdsql_1 ~]# docker top 9ed2e7062144
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                32653               32632               0                   09:36               ?                   00:00:00            /bin/bash -c while true; do echo 6666;sleep 1;done
root                32708               32653               0                   09:36               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```

**查看镜像的元数据**

![image-20210705093903142](/Users/guopuwen/Documents/笔记/docker/docker--docker容器命令、镜像命令.assets/image-20210705093903142.png)

**进入当前正在运行的容器**

方式一：使用exec

```shell
[root@tdsql_1 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
9ed2e7062144   centos    "/bin/bash -c 'while…"   10 minutes ago   Up 3 minutes             zen_bohr
[root@tdsql_1 ~]# docker exec -it 9ed2e7062144 /bin/bash
[root@9ed2e7062144 /]#
[root@9ed2e7062144 /]#
[root@9ed2e7062144 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
```

方式二：使用attach

```shell
[root@tdsql_1 ~]# docker attach 9ed2e7062144
6666
6666
6666
6666
```

两种进入方式的区别是：

- docker exec 进入当前的容器后开启一个新的终端，在里面进行操作
- docker attach 进入当前容器正在执行的终端，所以卡在while true循环里面

**从容器内拷贝文件到主机上**

```
docker cp 容器id:容器内路径	主机目的路径
docker cp 9ed2e7062144:/test.java /
```

![image-20210705094737142](/Users/guopuwen/Documents/笔记/docker/docker--docker容器命令、镜像命令.assets/image-20210705094737142.png)

# docker部署nginx

步骤：

1. 搜索镜像：docker search 或者docker的hub官网上
2. 拉去镜像：docker pull
3. 运行测试

以上步骤一行命令便可以运行

```shell
[root@tdsql_1 /]# docker run -d --name nginx01 -p  4433:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
b4d181a07f80: Pull complete
edb81c9bc1f5: Pull complete
b21fed559b9f: Pull complete
03e6a2452751: Pull complete
b82f7f888feb: Pull complete
5430e98eba64: Pull complete
Digest: sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
Status: Downloaded newer image for nginx:latest
205de87382eefc21e7edf926b712136aa272a2896304187acb529e32cafc6d45

[root@tdsql_1 /]# curl localhost:4433
```

测试成功，可以进入到nginx内部看看

```shell
[root@tdsql_1 /]# docker exec -it 205de87382ee /bin/bash
root@205de87382ee:/# ls
bin   dev		   docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   media  opt  root  sbin  sys  usr
```

可以看到进入到nginx容器内部之后文件与nginx部署到linux上基本一致，那么是不是每次修改一个nginx的配置文件或者部署一个项目就到进入到容器内部呢，而数据卷的技术变提供了一个路径的映射，也就是说将当前主机的文件路径映射到容器内部的文件路径

