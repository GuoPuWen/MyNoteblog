# 一、压缩和解压缩命令

## 1. .zip格式

.zip是windows最常用的格式，Linux也可以正常识别

```
[root@localhost ~]# zip [选项] 文件名 要压缩的文件
[root@localhost ~]# unzip [压缩的路径] 文件名 
```

## 2. .gz格式

- 英文原意：compress or expand files
- 功能描述：压缩文件或目录
- **缺点：不会打包**

```
[root@localhost ~]# gzip [选项] 文件
选项：
	-c 将压缩数据输出到标准输出中，可以用>> 保留源文件
	-d 解压缩
	-r 压缩(解压)目录
```

## 3. .bz2格式

- 功能描述：.bz2的压缩格式

- **缺点：不能压缩目录**

- ```
  [root@localhost ~]# bzip2 [选项] 文件
  选项：
  	-d 解压缩
  	-k 压缩时保留源文件
  ```

  

## 4. .tar格式

功能描述：只能打包，不能压缩。

```
[root@localhost ~]# tar [选项] [-f 压缩包名] 源文件或目录
选项：
	-c 打包
	-f 指定的压缩文件名
	-v 显示打包文件过程
常用法
[root@localhost ~]# tar -cvf 压缩包名	 源文件或目录
#解打包
选项
	-x 解打包
	-t 测试，不解打包，查看包中文件
	 -C(大) 目录：指定解打包位置(在源文件的后面)
常用法
[root@localhost ~]# tar -xvf 源文件或目录
[root@localhost ~]# tar -tvf 源文件或目录
[root@localhost ~]# tar -xvf 源文件或目录 -C 目录
```

## 5. .tar.gz格式和 .tar.bz2格式

```
使用tar命令直接打包压缩。命令格式如下：
[root@localhost ~]# tar [选项] 压缩包源文件或目录选项：   
	-z：压缩和解压缩“.tar.gz”格式   
	-j：压缩和解压缩“.tar.bz2”格式
常用法
# 打包压缩成.tar.gz格式
[root@localhost ~]#： tar -zcvf 源文件或目录 
# 解压解打包.tar.gz格式
[root@localhost ~]# tar -zxvf 文件 -C目录
# 打包压缩成.tar.bz2格式
[root@localhost ~]#： tar -jcvf 源文件或目录 
# 解压解打包.tar.bz2格式
[root@localhost ~]# tar -jxvf 文件 -C目录
```



几个例子：

```
[root@localhost ~]# tar -zcvf  test.tar.gz  test/ 
#压缩
[root@localhost ~]# tar  -ztvf  test.tar.gz
#只查看，不解压
[root@localhost ~]# tar  -zxvf  test.tar.gz -C /tmp 
#解压缩到指定位置
[root@localhost ~]# tar  -zxvf  test.tar.gz -C /tmp  test/cde
#只解压压缩包中的特定文件，到指定位置
```



# 二、关机和重启命令

## 1. sync

- 英文原意：flush file system buffers

- 所在路径：/bin/sync

- 执行权限：所有用户

- 功能描述：刷新文件系统缓冲区

  

## 2. shutdown

- 英文原意：bring the system down

- 所在路径：/sbin/shutdown

- 执行权限：超级用户

- 功能描述：关机和重启

  ```
  [root@localhost ~]# shutdown [选项] 时间 警告信息
  选项：
  	-c 取消已经执行的shutdown目录
  	-h 关机
  	-r 重启
  ```

  

## 3. reboot

在现在的系统中，reboot命令也是安全的，而且不需要加入过多的选项。