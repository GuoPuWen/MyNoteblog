## 1.概述

-  JavaEE：Java语言在企业级开发中使用的技术规范的总和，一共规定了13项大的规范
- Tomcat：Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范

## 2.使用

1. 下载：http://tomcat.apache.org/
2. 安装：解压压缩包即可。
  * 注意：安装目录建议不要有中文和空格

3. 卸载：删除目录就行了
4.  启动：bin/startup.bat ,双击运行该文件即可
5. 访问：htt://ip:端口(默认是8080)
6. 关闭：
   1.  bin/shutdown.bat
   2. ctrl+c

## 3.目录结构

![](Tomcat\1.png)

- 注意：这里是Tomcat8.5.51版本
- bin：存放启动和关闭Tomcat脚本
- conf：存放不同的配置文件（server.xml和web.xml）
- lib：存放Tomcat运行需要的库文件（jar包）
- logs：存放Tomcat执行时的日志文件
- webapps：Tomcat的主要Web发布目录（包括应用程序示例）
- work：存放jsp编译后产生的class文件；

## 4.部署web项目

tomcat有2种发布模式：

- 一种是把项目代码拷到tomcat里面去 tomcat就能跑起来这个项目。
- 另一种就是虚拟目录，不用把代码拷进去，让tomcat跑tomcat之外的目录里面代码。



1. 直接将项目放到webapps目录下即可。

![](Tomcat\2.png)



例如将testweb项目放在webapps下，其中test为虚拟目录，这样访问的时候就直接输入：

http:localhost:8080/test/文件



2. 配置conf/server.xml文件
   - 在<Host>标签体中配置
   - ![](Tomcat\3.png)
        * docBase:项目存放的路径
        * path：虚拟目录

3. 在conf\Catalina\localhost创建任意名称的xml文件。在文件中编写
   			![](Tomcat\4.png)

      				* 虚拟目录：xml文件的名称

   - docBase:项目存放的路径

## 5.动态项目

1. 目录结构：

   ----项目的根目录

   ​			----- WEB-INF目录：

   ​							-- web.xml：web项目的核心配置文件
   ​							-- classes目录：放置字节码文件的目录
   ​							-- lib目录：放置依赖的jar包

2. 将Tomcat集成到IDEA中，并且创建JavaEE的项目，部署项目。

## 6.IDEA与Tomcat

- IDEA会为每一个tomcat部署的项目单独建立一份配置文件，可以查看控制台的输出看到该配置文件在哪。查看控制台的log：Using CATALINA_BASE

![](C:\Users\four and ten\Desktop\笔记\JAVA\Tomcat\5.png)

- 工作空间项目    和     tomcat部署的web项目

tomcat真正访问的是“tomcat部署的web项目”，"tomcat部署的web项目"对应着"工作空间项目" 的web目录下的所有资源。工作空间的文件对应着WEB-INF/classes

