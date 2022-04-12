# 第五章 linux内核介绍

### 1. 进程管理

##### 1.1 进程概念

- 处于执行期的程序及其所包含代码的总称

- 程序：可执行代码

- 资源：打开文件、信号、地址空间、数据段等

![image-20201107123248461](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\21.png)

![image-20201107123302901](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\22.png)



##### 1.2  进程的结构

获取进程标识符的系统调用

==函数原型==

```c
#include<unistd.h>
#include<sys/types.h>
pid_t getpid(void)	//返回当前进程的pid
pid_t getppid(void)	//返回父进程的pid
uid_t getuid(void)	//返回当前进程的用户id
gid_t getgid(void)	//放回当前进程用户组的id
```

```c
#include<stdio.h>
#include<unistd.h>
int main(){
        printf("process id = %d\n",getpid());
        printf("parent process id = %d\n",getppid());
        printf("process's user id = %d\n",getuid());
        printf("process's group id = %d\n",getgid());
        return 0;
}
```

![image-20201107124649163](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\23.jpg)

==进程状态==

- 可运行态(TASK_RUNNING)：正在运行或处于就绪状态的进程
- 睡眠状态
  - 浅度睡眠状态(TASK_INTERRUPTIBLE)：等待资源被满足时被唤醒，可被信号或中断唤醒
  - 深度睡眠状态(TASK_UNINTERRUPTIBLE)：只有等待资源有效时唤醒，不可被信号唤醒

- 暂停状态(TASK_STOPPED)：进程由于收到一个信号致使进程停止
- 僵死状态(TASK_ZOMBIE)：进程结束但是尚未消亡的状态，等待父进程回收

```c
vim /usr/src/kernels/3.10.0-957.el7.x86_64/include/linux/sched.h 
    
    
#define TASK_RUNNING            0
#define TASK_INTERRUPTIBLE      1
#define TASK_UNINTERRUPTIBLE    2
```

##### 1.3 进程的创建

```c
#include<unistd.h>
pid_t fork(void)	
pid_t vfork(void)
```

==fork函数==

功能：

- 子进程完全复制父进程的资源

- 子进程的执行独行于父进程

返回值：

- 调用成功
  -  对于父进程，返回值为新创建的子进程的PID
  - 对于子进程，返回值为0

- 调用失败 返回值为-1

例子：

```c

#include<stdio.h>
#include<unistd.h>
int main(){
        pid_t val;
        printf("PID before fork(): %d\n",getpid());
        val = fork();
        if(val == 0){
                printf("I am the child process ,PID is %d\n",getpid());
        }
        else if(val > 0)
                printf("I am the parent process,PID is %d\n",getpid());
        else
                printf("error");
        return 0;
}
```

==vfork()==

- vfork()创建的子进程与父进程共享地址空间，调用vfork创建子进程后，父进程被挂起，直到子进程结束。因此，总是子进程先返回

- 调用vfork创建子进程后，父进程被挂起，直到子进程结束。因此，总是子进程先返回

### 2. 文件系统

##### 2.1 文件系统常用操作

==create()==

```c
int create(const char *pathname,mode_t mode)
```

创建一个普通文件

参数

- pathname： 要创建文件的路径（绝对路径/相对路径）
- mode：  创建文件的访问权限（八进制数/宏定义）

返回值：

- 成功，返回文件调用符

- 失败，返回-1

==open()==

```java
int open(const char *pathname,int flags);
int open(const char *pathname,int flags,mode_t mode);
```

功能：打开或创建一个文件

参数：

pathname： 要打开的文件路径（绝对路径/相对路径）

flags：  打开文件的方式

mode：  可选参数，只在创建文件时有效。用于定义新建文件的访问权限

参数flags的说明：

![img](C:\Users\VSUS\Desktop\笔记\ContOS\操作系统\24.jpg)

- 成功，返回文件调用符

- 失败，返回-1

==close()==

```c
int close(int fd)
```

功能：关闭一个可以打开的文件

参数：

fd：要关闭的文件的描述符

- 成功，返回0

- 失败，返回-1

==read()==

```c
ssize_t read(int fd,const void *buf,size_t count);
```



参数：

- fd：  文件描述符

- buf：  缓冲区指针，用于缓存从文件中读取的数据

- count： 请求读取的字节数

返回值：

- 成功，返回本次实际读取的字节数

- 失败，返回-1

==write()==

```c
ssize_t write(int fd,const void *buf,size_t count);
```

功能：向一个打开的文件写入数据

参数：

- fd：  文件描述符

- buf：  缓冲区指针，用于缓存从文件中读取的数据

- count： 要写入文件的字节数

返回值：

- 成功，返回实际写入的字节数

- 失败，返回-1

==lseek()==

```c
off_t lseek(int fd, off_t )
```

##### 2.2 文件系统

- 传统UNIX
  - UFS

- BSD文件系统
  - FFS

- windows文件系统
  - FAT16,FAT32,NTFS

- linux文件系统
  - ext2,ext3,ext4,XFS

### 3.进程通信

##### 3.1 进程通信概述

- 传统进程通信
  - 信号(signal)
  - 管道(pipe)

- System VIPC进程通信
  - 消息队列(message)
  - 共享主存(shared memory)
  - 信号量( semaphore )

- 远程通信
  - 套接字(socket)