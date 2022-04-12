本篇文章将会理解docker的网络模式

# Docker的网络模式

当docker进程一运行，会自动的在主机上创建一个docker0的虚拟网桥，相当于一个物理交换机，能够交换网络信息实现网段之间的主机进行通信，同时分配一个未被使用过的私有网段中的一个地址给docker0接口

![image-20210708101535856](http://cdn.noteblogs.cn/image-20210708101535856.png)

同时，当创建Docker容器的时候，会创建一对veth pair接口，使用的是veth pair技术，veth pair是一对虚拟设备接口，它是成对出现的，一端连着协议栈，一端彼此相连，所以veth pair被充当做一个网桥，链接着各种虚拟设备

下面例子创建一个tomcat01的容器

![image-20210708102054404](http://cdn.noteblogs.cn/image-20210708102054404.png)

当我们宿主机尝试联通docker中的tomcat01容器，发现是可以连通的

![image-20210708102425707](http://cdn.noteblogs.cn/image-20210708102425707.png)

接着做测试，再次创建一个tomcat02的容器，尝试两个容器之间互相ping，发现是可以通的

![image-20210708104407879](http://cdn.noteblogs.cn/image-20210708104407879.png)

同时在宿主机上，发现又多了一对网卡

![image-20210708104530846](http://cdn.noteblogs.cn/image-20210708104530846.png)

对于docker来说，所有的网络接口都是虚拟的，虚拟的转发效率很高，同时使用的是veth pair技术



# 自定义网络

前面，我们已经知道了docker会自动的创建docker0这个虚拟网卡，同时每一个容器都会使用veth pair技术创建一对网卡，做到容器之间的互通，但是存在的问题是：

- 两个容器之间通信，只能通过ip吗，能不能做一个映射，例如ping tomcat01可以实现？这样即使ip发生变化，两者之间还是可以通信的
- 不能做到容器之间的隔离，经过上面的测试可以发现两两容器之间可以通信，但是实际上是我Redis集群和MySQL集群之间可能不需要通信，也就是要做到容器之间的隔离

这就需要使用到自定义网络了，使用`docker network`可以查看当前网络中的所有信息

```shell
[root@tdsql_1 /]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

例如查看当前的所有网络

```shell
[root@tdsql_1 /]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
4eda75cb17cb   bridge    bridge    local
3367ec36fa31   host      host      local
8df0f89d4c48   none      null      local

[root@tdsql_1 /]# docker network inspect 4eda75cb17cb
```

![image-20210708110927315](http://cdn.noteblogs.cn/image-20210708110927315.png)

可以发现这个网卡包含了三个容器！

而自定义网络只需要使用`docker network create`命令

```shell
[root@tdsql_1 /]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
a50aadf24c2339636c3ba4eb9e6905fd5c8bb9384986bcd53a4156cd1c07218d
```

- --driver bridge：表示创建网络的链接模式，一般有四种链接模式：
  - bridge ：桥接 docker（默认，自己创建也是用bridge模式）
  - none ：不配置网络，一般不用
  - host ：和所主机共享网络
  - container ：容器网络连通（用得少！局限很大）

- --subnet：指定网段
- --gateway：指定网关

这样便创建好了一个自定义的网络，接着创建tomcat使用这个自定义的网络，使用--net指定创建的网络mynet

```shell
[root@tdsql_1 /]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
c6c4a6180a6592170a3f1e7122f473ece22f6fc37121047143918ed7e086e867
[root@tdsql_1 /]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
395bee1d4fa68812afac4b2d44845b02038589daff5ff84b3a46e1eb5c331ab2
```

查看mynet网络的基本信息

![image-20210708111812556](http://cdn.noteblogs.cn/image-20210708111812556.png)

![image-20210708111838350](http://cdn.noteblogs.cn/image-20210708111838350.png)

可以发现，创建的两个tomcat容器都被添加到这个网络上了，测试连通信，使用同一个mynet网络之间的容器可以通信

![image-20210708112224166](http://cdn.noteblogs.cn/image-20210708112224166.png)

那么不同网络之间可以通信吗？是不能通信的

![image-20210708113219711](http://cdn.noteblogs.cn/image-20210708113219711.png)

要让两个网段之间通信，得先打通网络，使用`docker connect`命令打通网络，发现可以连通了

![image-20210708113415908](http://cdn.noteblogs.cn/image-20210708113415908.png)