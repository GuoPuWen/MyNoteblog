# MySQL的安装启动停止 

 配置好path路径后，cmd命令窗口管理员方式

​	启动：net start mysql(服务的名称)

​	停止：net stop mysql(服务名称)

​	登陆：mysql -u root -p(密码)

​	卸载MySQL:	①卸载MySQL软件----控制面板

​							   ②关闭MySQL服务和删除MySQL服务  sc delect mysql[服务名称]

​	更改服务密码：https://blog.csdn.net/huang1600301017/article/details/84866007

# MySQL的目录结构



* Mysql的安装目录

  basedir=D:/mysql-8.0.18-winx64

* MySQL的数据目录：
  datadir=D:/mysql-8.0.18-winx64/Data

  * 数据库：文件夹
  * 表：文件
  * 数据：数据

  

