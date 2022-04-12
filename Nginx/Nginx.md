# 一、Nginx是什么

Nginx是俄罗斯人Igor Sysoev编写的轻量级Web服务器，是一个高性能的HTTP和反向代理服务器，Nginx支持热部署，启动速度特别快，还可以在不间断服务的情况下对软件版本或配置进行升级，即使运行数月也无需重新启动。

# 二、Nginx的作用

### 2.1 反向代理

先来聊一聊正向代理，正向代理是一个位于客户端和目标服务器之间的服务器(代理服务器)，为了从目标服务器取得内容，客户端向代理服务器发送一个请求并指定目标，然后代理服务器向目标服务器转交请求并将获得的内容返回给客户端。

例如，在国内访问不了国外的网址时，而代理服务器可以访问到国外的网址，这个时候客户端可以发请求到代理服务器，请求代理服务器访问国外的网址

![image-20210103202940146](http://cdn.noteblogs.cn/image-20210103202940146.png)

**所以，正向代理，其实是"代理服务器"代理了"客户端"，去和"目标服务器"进行交互**

正向代理具有以下用途：

- **突破访问限制** 
- **提高访问速度**
- **隐藏客户端真实IP**

再来看看反向代理，反向代理（reverse proxy）：是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

![image-20210103203113124](http://cdn.noteblogs.cn/image-20210103203113124.png)

**所以，反向代理，其实是"代理服务器"代理了"目标服务器"，去和"客户端"进行交互。**

综上，可以看出正向代理与反向代理的区别：

- **正向代理其实是客户端的代理**，帮助客户端访问其无法访问的服务器资源。**反向代理则是服务器的代理**，帮助服务器做负载均衡，安全防护等。

- **正向代理中，服务器不知道真正的客户端到底是谁**，以为访问自己的就是真实的客户端。而在**反向代理中，客户端不知道真正的服务器是谁**，以为自己访问的就是真实的服务器。

### 2.2 负载均衡

当服务器的访问量越来越大时，这给服务器带来了负担，而仅仅单纯的增加服务器的数量，从长远看还是不能根本的解决问题，而负载均衡技术可以解决上述问题

负载均衡服务器充当着网络流中“交通指挥官”的角色，“站在”服务器前处理所有服务器端和客户端之间的请求，从而最大程度地提高响应速率和容量利用率，同时确保任何服务器都没有超负荷工作。如果单个服务器出现故障，负载均衡的方法会将流量重定向到其余的集群服务器，以保证服务的稳定性。当新的服务器添加到服务器组后，也可通过负载均衡的方法使其开始自动处理客户端发来的请求。（详情可参考：[What Is Load Balancing?](https://www.nginx.com/resources/glossary/load-balancing/)）

![image-20210103203911038](http://cdn.noteblogs.cn/image-20210103203911038.png)

### 2.3 动静分离

动静分离是将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问。

![image-20210103204055655](http://cdn.noteblogs.cn/image-20210103204055655.png)

# 三、安装Nginx

1. 下载

[Nginx下载地址](http://nginx.org/)，使用的版本是nginx-1.12.2

2. 安装依赖

```shell
#安装pcre依赖
[root@hadoop101 ~]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
[root@hadoop101 ~]# tar –zxvf pcre-8.37.tar.gz
[root@hadoop101 ~]# ./configure
[root@hadoop101 ~]# make && make install
#安装openssl 、 zlib 、 gcc 
[root@hadoop101 ~]#yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

3. 安装nginx

```shell
1. 进入tar.gz目录
2. 解压
3. ./configure
4. make && make install
```

4. 启动nginx

进入目录：cd /usr/local/nginx/

```shell
[root@hadoop101 ~]# cd /usr/local/nginx/sbin
[root@hadoop101 sbin]# ./nginx
```

5. 开放防火墙端口80

6. 查看运行结果

![image-20210103211817522](http://cdn.noteblogs.cn/image-20210103211817522.png)

# 四、常用命令

```shell
[root@hadoop101 sbin]# ./nginx				#启动命令	
[root@hadoop101 sbin]# ./nginx -s stop 		#停止
[root@hadoop101 sbin]# ./nginx -s reload	#重新加载命令
```

# 五、配置文件

nginx各种作用都是要修改配置文件来实现的，配置文件的位置在/usr/local/nginx/conf

![image-20210104112359474](http://cdn.noteblogs.cn/image-20210104112359474.png)

去掉一堆注释之后，nginx.conf的内容如下：

```shell#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
        
        error_page   500 502 503 504  /50x.html;
        
        location = /50x.html {
            root   html;
        }

    }
}
```

nginx的配置文件可以分为3个部分

- 全局块

从配置文件开始到 events 块之间的内容，主要会设置一些影响 Nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数， 进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等  

worker_processes是Nginx服务器的关键配置，它值越大，可以支持的并发处理量也越大，但是不可将该值设置过大，会受到硬件等设备的制约

- events块

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。上述例子就表示每个 work process 支持的最大连接数为 1024

这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。  

- http块

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。需要注意的是： http 块也可以包括 http 全局块、 server 块。  

（1）http全局块

http 全局块配置的指令包括文件引入、 MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。  

（2）server 块  

这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。

而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

1. 全局 server 块
   最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
2. location 块
   一个 server 块可以配置多个 location 块。这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。  

# 六、配置实例

### 6.1 反向代理

需求：使用nginx反向代理，根据访问的路径不同跳转到不同的服务中

访问 http://192.168.18.101:9001/edu/ 直接跳转到 http://192.168.18.104:8080

访问 http://192.168.18.101:9001/vod/ 直接跳转到 http://192.168.18.103:8080

步骤;

1. 找到nginx的配置文件：/usr/local/nginx/conf，进行如下配置

```shell
server {
    listen       9001;
    server_name  192.168.18.101;
    location / {
   	 	proxy_pass http://localhost:8080/;
    }
    location ~ /edu/ {
    	proxy_pass http://192.168.18.104:8080;
    }
    location ~ /vod/ {
    	proxy_pass http://192.168.18.103:8080;
    }
}
```

2. 准备18.103的机器上的tomcat，以及静态资源vod/index.html
3. 准备18.103的机器上的tomcat，以及静态资源edu/index.html
4. 查看结果

![image-20210111193637559](http://cdn.noteblogs.cn/image-20210111193637559.png)

![image-20210111193804901](http://cdn.noteblogs.cn/image-20210111193804901.png)





location指令说明：location指令同于匹配URL，语法格式为：

```shell
location [ = | ~ | ~* | ^~] uri {

}
```

- = ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求  
- ~：用于表示 uri 包含正则表达式，并且区分大小写。  
- ~*：用于表示 uri 包含正则表达式，并且不区分大小写  
- ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 uri 和请求字符串做匹配  

### 6.2 负载均衡

需求：实现访问192.168.18.101/vod/index.html负载均衡到192.168.18.103:8080/vod/index.html和192.168.18.104:8080/vod/index.html

1. 配置文件

```shell
upstream myserver{
	server 192.168.18.103:8080;
	server 192.168.18.104:8080;
}
server {
    listen       80;
    server_name  192.168.18.101;

    location ~ /vod/ {
    	proxy_pass http://myserver;
	}
}
```

2. 查看效果

![image-20210111195336755](http://cdn.noteblogs.cn/image-20210111195336755.png)

![image-20210111195344511](http://cdn.noteblogs.cn/image-20210111195344511.png)

Nginx提供了下面几种负载均衡的分配方式：

- 轮询（默认） ：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

- weight： 代表权，重默认为 1，权重越高被分配的客户端越多
  指定轮询几率， weight 和访问比率成正比，用于后端服务器性能不均的情况。 例如：  

```shell
upstream myserver{
	server 192.168.18.103:8080 weight=1;
	server 192.168.18.104:8080 weight=2;
}
```

-  ip_hash：每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器

```shell
upstream myserver{
	ip_hash;
	server 192.168.18.103:8080 weight=1;
	server 192.168.18.104:8080 weight=2;
}
```

- fair（第三方）: 按后端服务器的响应时间来分配请求，响应时间短的优先分配

### 6.3 动静分离

动静分离是靠location来实现的，通过location指定不同的后缀名实现不同的请求转发

1. 配置文件

```shell
location /software/ {
	root /data/;	#静态资源路径
	autoindex on;
}
```

![image-20210111202516473](http://cdn.noteblogs.cn/image-20210111202516473.png)

2. 查看效果

![image-20210111202540340](http://cdn.noteblogs.cn/image-20210111202540340.png)