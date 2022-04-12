今天使用docker安装elasticsearch ，发现报出了磁盘空间不足的提示，就是说我虚拟机一开始安装的时候分盘分的有点小，才给了8个G，一些hadoop集群搭建下来，空间不够了，所以记录一些给Centos7进行磁盘扩容的操作，方便下次查阅

![image-20210310175645678](http://cdn.noteblogs.cn/image-20210310175645678.png)

首先查看磁盘空间使用情况df -h

![image-20210310175924225](http://cdn.noteblogs.cn/image-20210310175924225.png)

由于之前误以为磁盘空间不足，在挂载一下