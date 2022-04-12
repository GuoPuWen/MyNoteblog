[C++Scoket编程--堵塞式IO相关函数介绍](https://blog.csdn.net/weixin_44706647/article/details/123946905)

[C++Socket编程--探讨一些边界条件](https://blog.csdn.net/weixin_44706647/article/details/123980129)

本篇文章是阅读《UNIX网络编程卷1》的第六章的学习笔记，首先是介绍select模型，为什么需要select来处理网络编程？接着就是介绍select函数的用法和poll函数，在最后结合网上一些文章博客去探讨select和poll函数的底层原理

# 概述

在文章[C++Socket编程--探讨一些边界条件](https://blog.csdn.net/weixin_44706647/article/details/123980129)中提到过，由于TCP客户端需要同时处理两个输入，标准输入和套接字，当客户堵塞于标准输入输出时，将服务端进行杀死，服务器TCP虽然看起来正确的给客户TCP发送一个FIN，但是客户整堵塞与标准输入过程中，所以无法收到这个，直到重新从套接字读取（可能已经过了很长时间）。

需要解决这个问题，那么我们的客户端进程就需要一种预先告知内核的能力，一旦内核发现进程指定的一个或者多个IO条件准备就绪，就通知进程，这就叫做IO复用，IO复用的典型场景有：

- 当客户处理多个描述符（通常是交互式输入和网络套接字）时，必须使用IO复用
- 一个客户同时处理多个套接字是可能的，不过比较少见
- 如果一个TCP服务器既要处理监听套接字，又要处理已连接套接字
- 如果一个服务器即要处理TCP，又要处理UDP，一般就要使用IO复用
- 如果一个服务器要处理多个服务或者多个协议

IO复用模型如下图所示：

![image-20220407141512920](http://cdn.noteblogs.cn/image-20220407141512920.png)

使用IO复用，我们就可以调用select和poll，阻塞在这两个系统调用上的某一个之上，而不是之上阻塞在IO上

# select函数

select函数运行进程指示内核等待多个事件中的任何一个发生，并在有一个或多个事件发生或经历一段指定的时间后才唤醒它。例如，我们可以在下列情况发生时，select调用返回：

- 集合{1，4， 5}中的任何描述符已经准备好读
- 集合{2，7}中的任何描述符已经准备好写
- 集合{1，4}中的任何描述符有异常条件待处理
- 已经经历了10.2秒

```c++
#include <sys/select.h>
#include <sys/time.h>
int select(int maxfdpl,fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout)
```

`timeout`

timeout参数告知内核等待所指定描述符中的任何一个就绪可花多长时间，其中timeval结构用于指定这段时间的秒数和微秒数

```c++
struct timeval {
	long tv_sec;
    long tv_usec;
}
```

timeout参数具备三种可能性：

- 永远等待下去：仅在有一个描述符准备好IO时才返回，则该参数设置为空参数
- 等待一段固定时间：在有一个描述符准备好IO时返回，但是不超过由该参数锁所指向的timeval结构中指定的秒数和微秒数
- 根本不等待：检查描述符后立即返回，称为轮询，该参数必须指向一个timeval结构，而且其中的定时器值必须为0

timeout参数被设置为const值，使得在函数返回时不会被select修改，比如我们指定一个10s的超时值，不过在定时器到时之前select就返回了，那么timeout参数所指向的timeval结构不会被更新成该函数返回时剩余的秒数

`中间的三个参数`

readset、writeset、exceptset指定我们要让内核测试读、写和异常条件的描述符，对于异常条件就两个：

- 某个套接字的带外数据的到达
- 某个已置为分组模式的伪终端存在可从其主端读取的控制状态信息

> 带外数据（Out-of-band data）是一些通信协议所支持的可选特征，允许更高优先级的数据比普通数据优先传输。即使传输队列已经有数据，带外数据先行传输**。**TCP支持带外数据，但是UDP不支持。套接字接口对带外数据的支持，很大程度受TCP带外数据具体实现的影响

select使用描述符集，通常是一个整数数组，对应于fd_set的数据类型和以下四个宏中

```c++
void FD_ZERO(fd_set *fdset)			//清除fdset的所有位
void FD_SET(int fd, fd_set *fdset)	//设置一个位到fdset
void FD_CLR(int fd, fd_set *fdset)	//关掉fdset所有的位
void FD_ISSET(int fd, fd_set *fdset)	//判断该尾是否准备好
```

select函数中间的三个参数readset、writeset、exceptset，如果我们对某一个条件不兴趣，就可以设置为空指针，如果这三个指针均为空，那么就有一个比Unix的sleep函数更为精确的定时器

maxfdpl参数指定待测试的描述符个数，它的值是待测试的最大描述符加1，描述符0,1,2,3...移植到maxfdp-1均被测试

当select函数返回后，使用`FD_ISSET`宏来测试fd_set数据类型中得描述符，描述符内任何与为就绪描述符对应的位返回时均返回0

### 描述符就绪条件

这个小节将更加明确的界定套接字在什么条件下“就绪”：

（1）满足下列四个条件中的任何一个时，一个套接字准备好读：

- 该套接字接收缓冲区中的数据字节数大于等于套接字接收缓冲区低水位标记的当前大小，对这样的套接字执行读操作不会堵塞并将返回一个大于0的值，其实就是已经准备好读入数据。可以使用SO_RCVLOWAT套接字选项设置该套接字的低水位标记
- 该连接的读半部分关闭，也就是接收了FIN的TCP连接，对这样的套接字的读操作将不阻塞并返回0
- 该套接字是一个监听套接字且已完成的连接数不为0，对这样的套接字accept通常不会阻塞
- 其上有一个套接字错误待处理，对这样的套接字的读操作将不阻塞并返回-1，同时把errno设置为确切的错误条件

（2）满足下列四个条件中的任何一个时，一个套接字准备好写：

- 该套接字发送缓冲区中的可用空间字节数大于等于套接字发送缓冲区低水位标记的当前大小，并且该套接字已经连接，或者该套接字不需要连接（UDP套接字）。可以使用SO_SNDLOWAT套接字选项设置该套接字的低水位标记
- 该连接的写半部关闭，对这样的套接字写操作将产生SIGPIPE信号
- 使用非堵塞式connect的套接字已建立连接，或者connect已经以失败告终
- 其上有一个套接字错误待处理对这样的套接字写操作将不堵塞并返回-1

# TCP客户端程序

```c++
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cerrno>
#include <cstring>

#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <arpa/inet.h>

using namespace std;

#define MAXLINE     255
#define	LISTENQ		1024
#define	SA	struct sockaddr

int main() {
    int                 sockfd, len;
    char                recvline[MAXLINE + 1];
    struct sockaddr_in  servaddr;
    
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        cout << "Error : socket" << endl;
    }

    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(8899);
    servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");   

    if(connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0) {
        cout << "Error : connect" << endl;
        return 0;
    }

    char data[MAXLINE] ;
    char buf[MAXLINE];

    //使用select 模型
    int maxfdp1, stdineof = 0;
    fd_set rset;
    FD_ZERO(&rset);

    while(true){
        FD_SET(fileno(stdin), &rset);
        FD_SET(sockfd, &rset);
        maxfdp1 = max(fileno(stdin), sockfd) + 1;
        select(maxfdp1, &rset, NULL, NULL, NULL);
        if(FD_ISSET(sockfd, &rset)) {
            if((len = recv(sockfd, buf, sizeof(buf), 0)) > 0) {
                buf[len] = '\0';
                cout << "recv form service: " <<  buf << endl;
            }
        }

        if(FD_ISSET(fileno(stdin), &rset)) {
            if(cin >> data) {
                send(sockfd, data, strlen(data), 0);
            }
            if(!strcmp(data, "exit")) {
                cout << "..disconnect" << endl;
                break;
            }
        }
    }
    close(sockfd);
    exit(0);
}
```



# shutdown函数

上面的程序仍然存在问题，当我们在客户端输入exit的时候，标准输入检测到了，开始退出循环，并且立马执行close关闭套接字连接，在大批量的输入下标准输入的EOF并不代表套接字输入的完成，因为仍然有可能有请求在去往服务器的路上，或者仍然有应答在客户的路上。所以我们需要的是一种关闭TCP连接其中一半的方法，也就是说当数据发送完成的时候给服务器一个FIN，但是仍然把套接字打开以便服务器读取，这就需要shutdown函数，shutdown函数有两个好处：

- close把描述符的引用计数减1，仅在该计数变为0的时候关闭这个套接字，这在堵塞式IO中为了处理一个客户连接fork一个子进程中说明过，而shutdown函数可以不用管引用计数就激发TCP的正常终止序列
- close终止两个方向读和写的数据传输，但是TCP是全双工通信的，有时候我们对需要告知对端我们数据完成传输了，但是对端还是有数据需要发送给我们

![image-20220411085732271](http://cdn.noteblogs.cn/image-20220411085732271.png)

```c++
#include <sys/socket.h>
int shutdown(int sockfd, int howto)
```

shutdown函数依赖于howto参数：

- SHUT_RD：关闭连接中读的这一半，套接字不再有数据可接受，而且套接字接受缓冲区中的任何数据都要被丢弃，由该套接字接受的来自对端的任何数据都被确认，然后被丢弃，进程不能再对这样的套接字调用任何的读函数
- SHUT_WR：关闭连接中的写的一半，当前留在套接字发送缓冲区中的数据，将被发送掉，然后跟TCP连接正常终止序列，进程不能在跟这样的函数调用任何的写函数
- SHUT_RDWR：关闭连接的读和写

# 改造服务器程序



在前面，对于服务器程序，我们是使用fork出子进程来对付一个客户端的链接的，这是可取的。这里我们将使用select来处理每一个客户端的请求，也就是使用但进程的方式，使用select单进程来处理监听套接字、客户连接，需要一个数组来存下当前的客户连接并且记下连接客户的TCP连接描述符。下面具体介绍：

首先，服务器有单个监听描述符，使用一个原点来表示

![image-20220411144103723](http://cdn.noteblogs.cn/image-20220411144103723.png)

假设服务器是在前台启动的，那么描述符0、1、2将被设置为标准输入、标准输出和标准错误，可见第一个监听套接字的描述符应该是4，下图表示了一个client的数组，这个时候还没有客户端连接那么client数组全部被设置为-1，rset描述符集中存在一个监听描述符

![image-20220411144408616](http://cdn.noteblogs.cn/image-20220411144408616.png)

当第一个客户与服务器建立连接的时候，监听套接字变为可读，于是服务器调用accept，假设accept返回的已连接描述符是4，那么client[0] = 4，其中数据结构更新变化如下：

![image-20220411144559729](http://cdn.noteblogs.cn/image-20220411144559729.png)

当第二个客户建立连接时，假设返回的描述符是5，则相应的数据结构变化如下：

![image-20220411144658233](http://cdn.noteblogs.cn/image-20220411144658233.png)

当第一个客户发送FIN，使得服务器中的描述符中4变为可读，当服务器读取到这个套接字的时候，read将返回0，于是关闭套接字并更新数据结构如下：

![image-20220411144840698](http://cdn.noteblogs.cn/image-20220411144840698.png)

完整的程序如下：

```c++
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cerrno>
#include <cstring>

#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/wait.h>

#define MAXLINE     4096
#define	LISTENQ		1024
#define	SA	struct sockaddr
#define SERV_PORT 9788
using namespace std;


int main() {
    int   listenfd, connfd, maxi, i, maxfd, nready, sockfd, len;
    char	buff[MAXLINE];
    socklen_t clilen;
    struct sockaddr_in clientaddr, servaddr;
    char clientIP[INET_ADDRSTRLEN] = "";

    // select模型
    fd_set allset, rset; 
    int client[FD_SETSIZE];

    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        cout << "Error : socket" << endl;
        return 0;
    }
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(8899);  

    if(bind(listenfd, (SA *) &servaddr, sizeof(servaddr)) == -1) {
        cout << "Error : bind" << endl;
        return 0;
    }
    if(listen(listenfd, 5) == -1) {
        cout << "Error : listenfd" << endl;
        return 0;
    }

    //初始化    
    for(int i = 0;i < FD_SETSIZE;i++){
        client[i] = -1;
    }
    maxfd = listenfd;
    maxi = -1;
    FD_ZERO(&allset);
    FD_SET(listenfd, &allset);

    clilen = sizeof(clientaddr);

    for(;;) {
        cout << "...listening" << endl;
        rset = allset;

        nready = select(maxfd + 1, &rset, NULL, NULL, NULL);
        if(FD_ISSET(listenfd, &rset)) {
            if((connfd = accept(listenfd, (SA *) &clientaddr, &clilen)) < 0) {  //堵塞
                cout << "Error : accept" << endl;
                continue;
            }
            inet_ntop(AF_INET, &clientaddr.sin_addr, clientIP, INET_ADDRSTRLEN);
            cout << "...connect" << clientIP << ":" << ntohs(clientaddr.sin_port) << endl; 
            for(i = 0;i < FD_SETSIZE;i++) {
                if(client[i] == -1) {
                    client[i] = connfd;
                    cout << "add client array" << endl;
                    break;
                }
            }
            if(i == FD_SETSIZE) {
                cout << "too many clients" << endl;
            }
            //放入描述符集中
            FD_SET(connfd, &allset);
            maxfd = max(maxfd, connfd);
            maxi = max(i, maxi);
            if(--nready <= 0) continue;

        }

        for(i = 0;i <= maxi; i++) {
            cout << "recv from client" << endl;
            if((sockfd = client[i]) < 0){
                cout << "sockfd error" << endl;
                continue;
            } 
            if(FD_ISSET(sockfd, &rset)) {
                memset(buff, 0, sizeof(buff));
                if((len = recv(sockfd, buff, sizeof(buff), 0)) < 0) {
                    cout << "recv success" << endl;
                    buff[len] = '\0';
                }
                if(strcmp(buff, "exit") == 0) {
                    cout << "...disconnect " << clientIP << ":" << ntohs(clientaddr.sin_port) <<endl;
                    close(sockfd);
                    client[i] = -1;
                    FD_CLR(sockfd, &allset);
                    break;
                }
                cout << buff << endl;
                send(sockfd, buff, len, 0);
                if(--nready <= 0) break; 
            }
        } 
    }
    close(listenfd);
}
```

这个程序还存在一个问题，叫做拒绝服务式攻击，基本概念是：当一个服务器在处理多个客户请求时，它绝不能堵塞于单个客户相关的函数调用，如果客户发送完数据之后进入到睡眠状态，那么服务端将一直堵塞于该read调用尝试读取来自客户的输入，拒绝再为其他客户提供服务，解决办法有：

- 使用非堵塞IO
- 让每个客户有单独的线程来处理提供服务
- 对IO操作设置一个超时

# poll函数

poll的机制与select类似，与select在本质上没有多大差别，也是需要将多个描述符进行轮训，根据描述符的状态进行处理，但是poll没有最大连接数量的限制，同样的poll和select有缺点：包含大量描述符的的数组被复制与用户态和内核太之间，存在着不小的开销

```c++
int poll(struct pollfd *fdaddr, unsigned long nfds, int timeout);
```

第一个参数是一个指向pollfd数组的第一个元素的指针，每一个元素都是pollfd结构：

```c
struct pollfd {
  int fd;
  short events;
  short revents;
}
```

要测试的条件有events成员决定，函数在相应的revents成员中返回该描述符的状态，下面列出一些常用的值：

- POLLIN ：有数据可读
- POLLRDNORM ：有普通数据可读
- POLLRDBAND：有优先数据可读。
- POLLPRI：有紧迫数据可读。
- POLLOUT：写数据不会导致阻塞。
- POLLWRNORM：写普通数据不会导致阻塞。
- POLLWRBAND：写优先数据不会导致阻塞。
- POLLMSGSIGPOLL：消息可用

此外，revents域中还可能返回下列事件：

- POLLER：指定的文件描述符发生错误
- POLLHUP：指定的文件描述符挂起事件。
- POLLNVAL：指定的文件描述符非法。

poll函数大体使用和select差不多，这里直接给出程序：

```c
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cerrno>
#include <cstring>

#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/wait.h>
#include <poll.h>

#define MAXLINE     4096
#define	LISTENQ		1024
#define	SA	struct sockaddr
#define OPEN_MAX 1024
#define INFTIM -1
using namespace std;


int main() {
    int     listenfd, connfd, maxi, i, maxfd, nready, sockfd, len;
    char	buff[MAXLINE];
    socklen_t clilen;
    struct sockaddr_in clientaddr, servaddr;
    char clientIP[INET_ADDRSTRLEN] = "";

    // poll模型
    struct pollfd client[OPEN_MAX];


    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        cout << "Error : socket" << endl;
        return 0;
    }
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(8899);  

    if(bind(listenfd, (SA *) &servaddr, sizeof(servaddr)) == -1) {
        cout << "Error : bind" << endl;
        return 0;
    }
    if(listen(listenfd, 5) == -1) {
        cout << "Error : listenfd" << endl;
        return 0;
    }

    //初始化    
    client[0].fd = listenfd;
    client[0].events = POLLRDNORM;
    for(i = 1;i < OPEN_MAX;i++) {
        client[i].fd = -1;
    }
    maxi = 0;

    clilen = sizeof(clientaddr);
    for(;;) {
        cout << "...listening" << endl;
        nready = poll(client, maxi + 1, INFTIM);
        if(client[0].revents & POLLRDNORM) {
            if((connfd = accept(listenfd, (SA *) &clientaddr, &clilen)) < 0) {  //堵塞
                cout << "Error : accept" << endl;
                continue;
            }
            inet_ntop(AF_INET, &clientaddr.sin_addr, clientIP, INET_ADDRSTRLEN);
            cout << "...connect" << clientIP << ":" << ntohs(clientaddr.sin_port) << endl; 
            for(i = 0;i < OPEN_MAX;i++) {
                if(client[i].fd == -1) {
                    client[i].fd = connfd;
                    cout << "add client array" << endl;
                    break;
                }
            }
            if(i == OPEN_MAX) {
                cout << "too many clients" << endl;
            }
            //放入描述符集中
            client[i].events = POLLRDNORM;
            maxfd = max(maxfd, connfd);
            maxi = max(i, maxi);
            if(--nready <= 0) continue;
        }


        for(i = 1;i <= maxi; i++) {
            cout << "recv from client" << endl;
            if((sockfd = client[i].fd) < 0){
                cout << "sockfd error" << endl;
                continue;
            } 
            if(client[i].revents & (POLLRDNORM | POLLERR)) {
                memset(buff, 0, sizeof(buff));
                if((len = recv(sockfd, buff, sizeof(buff), 0)) < 0) {
                    cout << "recv success" << endl;
                    buff[len] = '\0';
                }
                if(strcmp(buff, "exit") == 0 || len == 0 ) {
                    cout << "...disconnect " << clientIP << ":" << ntohs(clientaddr.sin_port) <<endl;
                    close(sockfd);
                    client[i].fd = -1;
                    break;
                }
                cout << buff << endl;
                send(sockfd, buff, len, 0);
                
                if(--nready <= 0) break; 
            }
        } 
    }
    close(listenfd);
}
```
