本篇文章是《UNIX网络编程卷1》第三、四、五章阅读笔记，首先给出该书第五章的代码示例，然后依次剖析其中的代码深入理解在Linux下的网络编程，同时结合《TCP/IP协议详解卷》更加深入的理解TCP三次握手、四次挥手的作用机理，配合对应的代码，理解其中的过程。当然由于是使用c++编写，对于除Socket编程除外的代码与c会有比较大的出入，不是完全按照书本实现，但是Socket编程是一样的，并不影响我们理解Socket代码

# 完整的TCP程序

service.cpp

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

typedef void sigfunc(int);
void str_echo(int sockfd);
void sig_chld(int signo);
sigfunc *signal(int signo,sigfunc *func);


int main() {
    int     listenfd, connfd;
    char	buff[MAXLINE];
    socklen_t clilen;
    struct sockaddr_in clientaddr, servaddr;
    char clientIP[INET_ADDRSTRLEN] = "";

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
    signal(SIGCHLD,sig_chld);
    pid_t child;
    clilen = sizeof(clientaddr);
    for(;;) {
        cout << "...listening" << endl;
        connfd = accept(listenfd, (SA *) &clientaddr, &clilen);     //堵塞
        if(connfd < 0) {
            cout << "Error : accept" << endl;
            continue;
        }
        inet_ntop(AF_INET, &clientaddr.sin_addr, clientIP, INET_ADDRSTRLEN);
        cout << "...connect" << clientIP << ":" << ntohs(clientaddr.sin_port) << endl;  
        if((child = fork()) == 0){
            cout << "childPid" << getpid() << endl;
            close(listenfd);
            while(true) {
                memset(buff, 0, sizeof(buff));
                int len = recv(connfd, buff, sizeof(buff), 0);      //堵塞
                buff[len] = '\0';
                if(strcmp(buff, "exit") == 0) {
                    cout << "...disconnect " << clientIP << ":" << ntohs(clientaddr.sin_port) <<endl;
                    break;
                }
                cout << buff << endl;
                send(connfd, buff, len, 0);
            }
            exit(0);
            
        }


        close(connfd);
    }
    close(listenfd);
}


void sig_chld(int signo)
{
	pid_t pid;
	int stat;
    while((pid = waitpid(-1, &stat, WNOHANG)) > 0) {
        printf("child %d terminated.\n",pid);
    } 
	
	return;
}
sigfunc* signal(int signo,sigfunc *func)
{
	struct sigaction act,oact;
	act.sa_handler=func;
	sigemptyset(&act.sa_mask);
	act.sa_flags=0;
	if(signo==SIGALRM)
	{
#ifdef SA_INTERRUPT
		act.sa_flags|=SA_INTERRUPT;
#endif
	}else{
#ifdef SA_RESTART
		act.sa_flags|=SA_RESTART;
#endif
	}
	if(sigaction(signo,&act,&oact)<0)
		return(SIG_ERR);
	return(oact.sa_handler);
}
```

client.cpp

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
    int                 sockfd, n;
    char                recvline[MAXLINE + 1];
    struct sockaddr_in  servaddr;
    
    for(int i = 1;i <= 5; i++){
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

            char data[MAXLINE] = "zhang";
            char buf[MAXLINE];
            while(true){
                //cin >> data;
                send(sockfd, data, strlen(data), 0);
                if(strcmp(data, "exit") == 0) {
                    cout << "..disconnect" << endl;
                    break;
                }
                memset(buf, 0, sizeof(buf));
                int len = recv(sockfd, buf, sizeof(buf), 0);
                buf[len] = '\0';
                cout << buf << endl;
            }
            close(sockfd);
    }
}
```

# 套接字地址结构

不管是服务端程序还是客户端程序，要做Socket做TCP编程第一步要做的便是定义TCP套接字地址结构，TCP套接字地址结构以sockaddr_in命名，在`<netint/in.h>`头文件中定义

```c++
struct sockaddr_in clientaddr, servaddr;
```

结构如下：

```c++
struct sockaddr_in {
   sa_family_t		sin_family;		//套接字地址结构的协议族
   in_port_t 		sin_port;		//16位TCP或者UDP Port	
   struct in_addr 	sin_addr;		/* Internet address.  */
   unsigned char sin_zero[8];		//未使用
};

struct in_addr {
    in_addr_t 		s_addr;		// 32位IP地址
};
```

sockaddr_in是IPV4的套接字结构，但是对于套接字函数来说，应该要被定义为能够处理任何协议族的套接字地址结构，在1982年采用的办法是在`<sys/socket.h>`头文件中定义一个通用的套接字地址结构

```c++
struct sockaddr {
    sa_family_t		sin_family;		//套接字地址结构的协议族
    char			sa_data[14];
}
```

那么这样在将这些套接字结构体传入套接字函数的时候，需要强制转换，变成指向某个通用套接字地址结构的指针，正如一开始代码所示：

```c++
#define	SA	struct sockaddr
bind(listenfd, (SA *) &servaddr, sizeof(servaddr)
```

bind的函数原型为：

```c++
int bind(int, struct sockaddr *, socklen_t);
```

# htons、htonl、ntohs、ntohl

考虑一个16bit的整数，由两个字节组成，在内存中存储这两个字节有两种方法：

- 将低字节存储在低地址，这称为小端字节序
- 将高字节存储在高地址，这称为大端字节序

![image-20220401151851950](http://cdn.noteblogs.cn/image-20220401151851950.png)

对于这两种字节序并没有特别的标准可言，两种格式都有系统使用，例如如下程序用于输出该主机所使用的的字节序

```c++
#include <iostream>
using namespace std;
int main(void) {
    union {
        short   s;
        char    c[sizeof(s)];
    }un;
    un.s = 0x0102;
    if(sizeof(un.s) == 2) {
        if(un.c[0] == 1 && un.c[1] == 2) cout << "big-endian" << endl;
        if(un.c[0] == 2 && un.c[1] == 1) cout << "little-endian" << endl;
    }else{
        cout << sizeof(short) << endl;
    }
    return 0;
}
```

对于网络编程来说网路协议必须指定一个网路字节序，在网络协议中用大端字节序来接收这些多字节整数，所以引出htons、htonl、ntohs、ntohl这四个函数，在`<netinet/in.h>`中定义

- h表示host
- n代表network
- s表示short，用于2字节，8位整数
- l表示long，用于4字节，32位整数

所以，在上述实例代码中

```c++
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(8899); 
```

# 地址转换函数

地址转换函数用于在ASCII字符串中与网络字节序的二进制值之间转换地址，这五个函数都是做地址转换的，inet_aton、inet_addr、inet_ntoa分为一组，inet_pton、inet_ntop分为一组，下面分别介绍

- `inet_aton`，将所指的字符串转换成一个32位的网络字节序二进制值，并且通过指针addrptr来存储，成功则返回1，否则返回0

```c++
int inet_aton(const char *strptr, struct in_addr *addrptr);
```

- `inet_addr`同样的进行将一个字符串转换为32位的网络字节序，通过返回值的方式，不过并不推荐使用这个函数，因为无法处理255.255.255.255的地址

```c++
inaddr_t inet_addr(const char *strptr);
```

- `inet_nton`函数将一个32位的网络字节序转换为相应的点分十进制数串

上面这一组函数用于IPV4，但是在实际使用中并不推荐，而是推荐使用下面这一组，对于IPV4和IPV6同样适用

- `inet_pton`转换由strptr指针所指的字符串，并且通过adrptr指针存放二进制结果，成功则返回1，其中family参数可以使AF_INET，也可以是AF_INET6

```c++
int inet_pton(int family, const char *strptr, void *addrptr);
```

- `inet_ntop`进行相反的转换，len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区

```c++
const char *inet_ntop(int family, const void *addrptr, char *strptr, size_t len);
```

![image-20220402100037508](http://cdn.noteblogs.cn/image-20220402100037508.png)

# socket函数

一个完成的网络编程通信程序，调用顺序如下图：

![image-20220402133131242](http://cdn.noteblogs.cn/image-20220402133131242.png)

为了执行网络IO，一个进程做的第一件事情就是调用socket函数，指定期望的通信协议类型，使用IPV4或者IPV6，TCP、UDP等等

```c++
int socket(int family, int type, int protocol);
```

family常值的类型有以下几种：

| family   | 说明       |
| -------- | ---------- |
| AF_INET  | IPV4       |
| AF_INET6 | IPV6       |
| AF_LOCAL | Unix域协议 |
| AF_ROUTE | 路由套接字 |
| AF_KEY   | 密钥套接字 |

type类型一共有以下几种：

| type           | 说明           |
| -------------- | -------------- |
| SOCK_STREAM    | 字节流套接字   |
| SOCK_DGRAM     | 数据报套接字   |
| SOCK_SEQPACKET | 有序分组套接字 |
| SOCK_RAM       | 原始套接字     |

socket函数AF_INET或者AF_INET6的protocol常值

| protocol     | 说明     |
| ------------ | -------- |
| IPPPORO_CP   | TCP协议  |
| IPPPORO_UDP  | UDP协议  |
| IPPPORO_SCTP | SCTP协议 |

对于这个函数一般选择的组合如下

![image-20220402134112306](http://cdn.noteblogs.cn/image-20220402134112306.png)

# connect函数

TCP客户使用connect函数来建立与TCP服务器的连接

```c++
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
```

如果使用TCP套接字，调用connect函数将激发TCP的三路握手过程，而且仅在连接建立成功或者出错时才返回，其中出错可能有以下几种情况

![image-20220402134708784](http://cdn.noteblogs.cn/image-20220402134708784.png)

- 若TCP客户没有收到SYN分节的响应，则返回ETIMEDOUT错误，例如，BSD内核发送一个SYN，若无响应则等待6s再发送一个，若无响应则等待24s再发一个，如果总共等待了75s还是没有收到响应则返回本错误
- 一接收到RST就马上返回ECONNREFUSED错误，这种错误有可能是服务器主机在我们指定的端口上没有进程在等待连接

> RST是TCP在发生错误时发送的一种TCP分节，产生RST的三个条件是：
>
> - 目的地为某端口的SYN到达，但是该端口上并没有监听的服务端程序
> - TCP想取消一个TCP链接
> - TCP接收到一个一个根本不存在的链接上的分节

- 如果客户主机发送SYN之后在某个中间的路由器引发了一个“destination unreachable”不可达的ICMP报文错误，则按照第一种规则多次尝试发送SYN报文请求

![image-20220403085653756](http://cdn.noteblogs.cn/image-20220403085653756.png)

按照TCP的状态图，当客户主机发送connet之后，状态从CLOSE转移到SYN_SENT，如果成功则转移到ESTABLISHED状态。当connect失败则套接字不在可用必须关闭，当每次循环调用connect为给定主机尝试各个ip直到有一个成功的时候，在每次connect失败后，都必须关闭当前的套接字，并且重新调用socket

# bind函数

bind函数将一个本地协议赋予给一个套接字

```c++
int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);
```

- 调用bind函数TCP服务端应该绑定一个端口，对于客户端来说内核为套接字随机绑定一个端口是正常的，但是对于服务端来说这并不可取
- bind函数可以将一个一个特定的IP地址绑定在套接字上，对于服务端程序，这就限定该套接字只接收那些目的地为这个ip地址的客户的链接，如果TCP没有将IP地址捆绑到它的套接字上，内核就将收到的SYN报文的目的IP地址作为服务器的源IP地址；对于客户端程序这就为该套接字上发送的IP数据包指定源IP地址

也就是说bind函数可以指定一个IP地址或者端口，也可以不指定，效果如下

![image-20220403091315214](http://cdn.noteblogs.cn/image-20220403091315214.png)

对于IPV4来说，通常指定一个INADDR_ANY，其值一般为0

```c++
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(8899);  
if(bind(listenfd, (SA *) &servaddr, sizeof(servaddr)) == -1) {
  cout << "Error : bind" << endl;
  return 0;
}
```

# listen函数

listen函数发生在服务端，当使用socket函数创建一个套接字的时候，它把一个未链接的套接字转化为一个被动的套接字，服务器TCP状态从CLOSE状态转移到LISTEN状态

```c++
int listen(int sockfd, int backlog)
```

这里需要重点介绍第二个参数backlog，这个参数规定了内核应该为相应套接字排队的最大连接数，那么首先就要理解在TCP程序启动的时候，内核为一个给定的套接字维护两个队列

- 未完成连接队列，每个SYN分节对应其中一项，当客户发出一个SYN分节到达服务器的时候，TCP服务器会将这些SYN请求放入到未完成连接队列中，也叫SYN队列，这些套接字都是处于SYN_RECV状态
- 已完成连接队列，每个已经完成TCP三路握手过程的客户端对应其中一项，这些套接字都是处于ESTABLISHED状态，等待accept调用

![image-20220403092728847](http://cdn.noteblogs.cn/image-20220403092728847.png)

也就是说，当客户端的SYN到达服务端的时候，TCP在未完成队列中创建一个新项目，然后响应三路握手的第二个分节，这一项一直被保留到未完成队列中，直到三路握手的第三个分节到达或者超时了为止，如果三路握手正常，那么TCP就将该项目从未完成队列中移到已完成队列中，等待accept的调用，TCP调用accept的，会将已完成队列中的队头项返回给进程

![image-20220403092800884](http://cdn.noteblogs.cn/image-20220403092800884.png)

- backlog曾被规定为两个队列总和的最大值
- 不要把backlog设置为0
- 在三路握手正常完成的前提下，未完成队列中的任何一项保存的时间都是一个RTT
- 历史上backlog的值是5，这是BSD4.2能支持的最大值，历史上TCP服务器要处理的连接比较少，但是目前繁忙的服务器要处理更多的连接，可以将这个值指定大一些
- backlog这个值并不好预测，并且随着TCP客户数的不断增多，backlog的值也需要随之改变，一个比较好的方法是，设置一个灵活的值，这个值通过命令行或者环境变量指定，这就在改变backlog的值的时候不需要重新编译

```c++
void Listen(int fd, int backlog) {
  char *ptr;
  if((ptr = getenv("LISTENQ")) != NULL) {
		backlog = atoi(ptr);
  }
  if(listen(fd, backlog) < 0) {
		err_sys("listen error");
  }
}
```

- 当一个客户SYN到达的时候，若这些队列是满的，TCP就忽略这些分节，也就是不发送RST，主要是因为对于服务端来说这种情况又可能只是暂时的，客户将重新发送SYN分节，期望不久就能在这些队列中找到可用的空间，要是服务器TCP立马响应一个RST，客户的connect调用就会立马返回一个错误，强制应用程序处理这种情况，但是对于客户来说也无法分辨这个RST是该端口没有服务在监听还是该端口有监听，但是队列满了

# accept函数

accept函数由TCP服务器调用，用于从已完成队列中返回下一个已完成连接，如果已完成连接队列为空，那么进程将投入到睡眠中

```c++
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

如果accept成功，那么其返回值是由内核自动生成的一个全新描述符，代表与客户的TCP连接，需要注意的是accept传入的第一个参数也是套接字但是这是监听套接字，监听套接字是由socket创建，随后用做bind、listen函数的第一个参数，这是全局存在的，它在该生命周期内一直存在，而accept函数的返回值是一个新的套接字，内核为每个客户连接创建一个已连接套接字，也就是说这个套接字表面三次握手已经完成