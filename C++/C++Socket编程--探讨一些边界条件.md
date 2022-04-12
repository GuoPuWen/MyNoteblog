本篇文章接上一节《[C++Scoket编程--堵塞式IO相关函数介绍](https://blog.csdn.net/weixin_44706647/article/details/123946905?spm=1001.2014.3001.5502)》，在上一节中主要介绍了TCP程序中几个出现的必要程序，例如scoket、bind、listen、accept函数，但是并没有介绍在接受发送数据时的写入写出函数，我觉得这些虽然在堵塞IO中时堵塞的关键之一，但是并不打算介绍这些流函数，在书中是对这些函数有介绍的。本节主要是第五章的学习笔记，会探讨一些边界条件：

- 当程序运行的时候，客户和服务器同时启动会发生什么？
- 客户正常终止会发生什么？
- 若服务器进程在客户之前终止，会发生什么？
- 服务器主机崩溃会发生什么？

# 正常启动

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

Client.cpp

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

当程序正常启动时，首先启动服务端，服务器启动之后调用socket、bind、listen和accept，并且堵塞于accept调用，并且使用netstat工具，可以查看到相关的状态

```c++
[root@iz2ze3u71xuet3hjszh72jz MyC++]# netstat -tuln | grep 8899
tcp        0      0 0.0.0.0:8899            0.0.0.0:*               LISTEN
```

正是我们需要的，在服务器端游一个8899端口处于监听状态，当客户调用socket和connect，则发生三次握手，三次握手完成之后，客户中的connect和服务器中的accept均返回，连接于是建立

# 正常终止

当连接建立的时候，服务端和客户端都处于ESTABLISHED状态，使用netstat可以查看相关的状态

![image-20220404111507275](http://cdn.noteblogs.cn/image-20220404111507275.png)

当我们在client端输入exit时，当前连接的客户端进入time_wait状态，目前的端口号是57180，监听服务器仍然还是堵塞在accept调用上

![image-20220404111848096](http://cdn.noteblogs.cn/image-20220404111848096.png)

另外我们需要注意的一点是，我们在服务端程序中使用fork来处理一个客户的请求，当客户端连接输入exit时，该服务器的子进程终止，但是父进程并没有调用wiat或者waitpid获取子进程的状态信息，导致进程中的状态描述符一直保存在系统中，也就是会产生僵尸进程

> - 孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。**
>
> - 僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。**

在服务器子进程终止的时候，会给父进程发送一个SIGCHLD信号，只需要处理该信号，捕获到该信号，然后调用wait或者waitpid，信号就是告知某个进程发送了某个事件的通知，也被称为软件中断，而且信号是异步发生的，也就是说进程预先不知道信号发生的时刻

在本程序中使用signal函数去处理SIGCHLD信号，接收到该信号之后我们有两个选择，使用wait或者waitpid函数：

```c++
	pid_t wait(int *statloc)
  pid_t waitpid(pid_t pid, int *statloc, int options)
```

两个函数的区别是，在一个子进程终止前，wait使其调用者堵塞，waitpid有一个选项，可以选择不堵塞或者堵塞，options参数选项如下：

- WCONTINUED：若实现支持作业控制，那么由pid指定的任一子进程在暂停后已经继续，但是状态没报告，则返回其状态
- WNOHANG：若由pid指定的子进程并不是立即可用的，则waitpid不阻塞，此时返回值为0
- WUNTRACED：若实现支持作业控制，那么由pid指定的任一子进程已经处于暂停状态并没报告过，则返回其状态

wait只是waitpid的简化版，在考虑使用wait或者是waitpid的时候，我们先将客户端程序修改为，也就是客户建立五个与服务器的连接

![image-20220405101653138](http://cdn.noteblogs.cn/image-20220405101653138.png)

当客户终止的时候，所有打开的描述符由内核自动关闭，且五个进程基本在同一时间终止，这就导致客户端发送五个FIN，每个连接一个，使得服务端程序的5个子进程基本在同一个时刻终止，也就是说在这一时刻五个SIGCHLD信号递交给父进程，如下图：

![image-20220405101958308](http://cdn.noteblogs.cn/image-20220405101958308.png)

如果我们使用wait函数，只建立一个信号处理函数并且调用wait是不足以防止出现僵尸进程的，问题是所有的5个信号都在信号处理函数执行之前产生，但是信号处理函数只处理一次，那么就会留下4个僵尸进程

所以正确的做法是使用waitpid，我们在一个循环里面调用waitpid，因为waitpid是不堵塞的，无法在循环中调用wait，因为wait是会堵塞于第一个终止的子进程

```c++
void sig_chld(int signo)
{
	pid_t pid;
	int stat;
    while((pid = waitpid(-1, &stat, WNOHANG)) > 0) {
        printf("child %d terminated.\n",pid);
    } 
	return;
}
```

# 同时启动

这里同时启动的概念是，两个应用程序彼此执行主动打开的情况，也就是说两个主机同时发送SYN请求报文，这种情况是由可能存在的，但是在上面的代码中无法模拟，因为客户端只是作为客户，服务端只是作为服务。

当每一方发送一个SYN，并且SYN必须传递给对方，例如主机A的一个应用程序使用本地端口7777，并与主机B的端口8888执行主动打开，主机B的应用程序则使用8888端口与主机A的7777端口执行主动打开。TCP的特意设计为了可以处理同时打开，对于同时打开它仅建立一条连接，而不是两条连接，当出现同时打开时，状态变迁图如下所示

![image-20220405121920617](http://cdn.noteblogs.cn/image-20220405121920617.png)

两端几乎同时在发送SYN，并且进入到SYN_SENT状态，每一端收到SYN时，状态变为SYN_RECV，同时他们都在发送SYN确认报文，当双方都收到SYN及相应的及相应的ACK时，状态都变为ESTABLISHED，一个同时打开的连接需要交换4个报文，同时我们没有将任何一端称为客户或者服务器，因为他们每一端即是客户又是服务器。

# 同时关闭

TCP协议允许双方都同时执行主动关闭，当执行主动关闭时，状态变迁图如下：

![image-20220405122820067](http://cdn.noteblogs.cn/image-20220405122820067.png)

当应用层发出关闭命令时，两端均从 ESTABLISHED变为FIN_WAIT_1。 这将导致双方各发送一个FIN ，两个 F I N 经过网络传送后分别到达另一端。收到FIN后 ， 状态由FIN_WAIT_1变迁到 CLOSING，并发送最后的ACK。当收到最后的ACK时，状态变化为 TIME_WAIT

# 服务器进程关闭

首先正确启动客户/服务器，然后找到子进程的pid，最后将该子进程kill掉，发生的步骤如下：

1. 在同一个主机上启动客户和服务端程序，并且在客户上输入文本，验证一切正常
2. 找到服务器子进程的进程pid，并且执行kill命令，作为终止进程的部分工作，将导致子进程中所有打开的描述符都关闭，这就导致向客户发送一个FIN，而客户TCP则相应一个ACK，这就是TCP连接终止工作的前半部分
3. SIGCHLD信号被发送给服务器父进程，并得到正确处理
4. 客户TCP接收来自服务器TCP的FIN并响应一个ACK，`但是客户进程堵塞在输入调用上，等待从终端中输入一行文本`
5. 当在客户上输入一行文本，客户TCP可以将数据发送给服务器，TCP是全双工通信，允许在服务端向客户端发送一个FIN报文的情况下，客户端还可以向服务端发送数据
6. 当服务端收到一个数据的时候，由于套接字进程描述符已经全部被关闭，于是响应一个RST
7. 但是客户接受不到这个RST，因为客户一旦将数据发送过去，将会堵塞在readline上

通过上面的步骤，总结为当服务端的FIN到达客户套接字的时候，客户正在堵塞recv读取一行上。所以客户在同时处理两个进程描述符：套接字和用户输入，它不能单纯的堵塞在这两个源的特定源的输入上，而是应该堵塞在其中任何一个源的输入上，这也就是后来由select和poll函数的目的之一

# 服务器主机崩溃

当服务器主机崩溃的时候，已有的网络连接上不发出任何东西，当我们在客户上输入一行文本，它将写入内核，再由TCP作为一个数据分节发出，因为服务器主机崩溃，所以客户TCP持续重传数据分节，并且试图从服务器上获取一个ACK，当客户TCP最后放弃的时候，给客户进程返回一个错误，服务器主机已经崩溃，从而对客户的数据分节根本没有任何响应，那么返回的错误的ETIMEDOUT，假如在某个中间路由器判定服务器主机不可达，那么会响应一个错误的ICMP消息。当然我们不想主动发送数据也想检测到服务器主机的崩溃，那么需要采用另外一个技术，就是在套接字中使用SO_KEEPALIVE字段

# 结语

在这两篇文章中，介绍了TCP的堵塞IO，所谓堵塞主要是指，当用户进行IO请求的时候，内核要去查看相应的缓冲区数据是否已经准备好，如果没有准备好就会一直等待，同时使用了Socket编程的一些重要函数socket、bind、listen、accept等等完成了一个TCP回射程序，通过这个程序来说明一些比较重要的问题，例如正常启动TCP完成三路握手；正常关闭完成四次挥手并且需要等待time_wait时间；当两端同时发送SYN请求时，只是产生一个连接需要发送四次报文；同时关闭也是被TCP标准所允许的；当服务器进程关闭时，由于客户端要同时处理套接字和io进程描述符，可能会接收不到服务器端发送来的FIN，这就需要我们后来介绍的select和poll函数

# 参考

- 《UNIX网络编程卷1：套接字联网API（第3版）》第三、四、五章
- 《TCP-IP详解卷一：协议》
- 《TCP-IP详解卷二：实现》
- https://www.jianshu.com/p/3b233facd6bb

