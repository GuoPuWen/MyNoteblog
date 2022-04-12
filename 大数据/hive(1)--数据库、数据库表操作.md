# 一、Hive概述

Hive是基于Hadoop的一个数据仓库工具，可以将**结构化的数据**文件映射为一张数据库表，并提供类SQL查询功能。

其本质是将SQL转换为MapReduce的任务进行运算，底层由HDFS来提供数据的存储，说白了hive可以理解为一个将SQL转换为MapReduce的任务的工具，甚至更进一步可以说hive就是一个MapReduce的客户端

==本质是：将HQL转化成MapReduce程序==

# 二、Hive的架构

![img](C:\Users\VSUS\Desktop\笔记\大数据\img\17.png)

1．用户接口：Client

CLI（hive shell）、JDBC/ODBC(java访问hive)、WEBUI（浏览器访问hive）

2．元数据：Metastore

元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；

默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore

3．Hadoop

使用HDFS进行存储，使用MapReduce进行计算。

4．驱动器：Driver

（1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。

（2）编译器（Physical Plan）：将AST编译生成逻辑执行计划。

（3）优化器（Query Optimizer）：对逻辑执行计划进行优化。

（4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

# 三、安装Hive





# 四、Hive数据类型

### 4.1 基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                                 | 例子                                 |
| ------------ | ------------ | ---------------------------------------------------- | ------------------------------------ |
| TINYINT      | byte         | 1byte有符号整数                                      | 20                                   |
| SMALINT      | short        | 2byte有符号整数                                      | 20                                   |
| INT          | int          | 4byte有符号整数                                      | 20                                   |
| BIGINT       | long         | 8byte有符号整数                                      | 20                                   |
| BOOLEAN      | boolean      | 布尔类型，true或者false                              | TRUE FALSE                           |
| FLOAT        | float        | 单精度浮点数                                         | 3.14159                              |
| DOUBLE       | double       | 双精度浮点数                                         | 3.14159                              |
| STRING       | string       | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | ‘now is the time’ “for all good men” |
| TIMESTAMP    |              | 时间类型                                             |                                      |
| BINARY       |              | 字节数组                                             |                                      |

对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数

### 4.2 集合数据类型



| 数据类型 | 描述                                                         | 语法示例 |
| -------- | ------------------------------------------------------------ | -------- |
| STRUCT   | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last STRING},那么第1个元素可以通过字段.first来引用。 | struct() |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()    |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’,  ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()  |

例如：有如下格式数据：

```json
{
    "name": "songsong",
    "friends": ["bingbing" , "lili"] ,       //列表Array, 
    "children": {                      //键值Map,
        "xiao song": 18 ,
        "xiaoxiao song": 19
    }
    "address": {                      //结构Struct,
        "street": "hui long guan" ,
        "city": "beijing" 
    }
}

```

创建对于的数据文件data.txt

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```

注意：MAP，STRUCT和ARRAY里的元素间关系都可以用同一个字符表示，这里用“_”。

hive上创建对应的表：

```sql
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```

- row format delimited fields terminated by ',' -- 列分隔符

- collection items terminated by '_'    --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)

- map keys terminated by ':'             -- MAP中的key与value的分隔符

- lines terminated by '\n';               -- 行分隔符

导入测试数据，访问集合里的元素

```sql
select friends[1],children['xiao song'],address.city from test
where name="songsong";
```











# 五、数据库操作

### 5.1 数据库操作

###### ==创建数据库==

```sql
create database if not exists myhive;
```

hive是基于hadoop的，所以表是存在hdfs上的，默认是放在/user/hive/warehouse上的

![image-20201127201135115](C:\Users\VSUS\Desktop\笔记\大数据\img\18.png)

```xml
<name>hive.metastore.warehouse.dir</name>
<value>/user/hive/warehouse</value>
```

###### ==指定创建的数据库位置==

```sql
hive> create database myhive2 location '/myhive2';
```

###### ==设置数据库键值对信息==

```sql
hive>create database foo with dbproperties('owner' = 'root','data' = '20201127');
```

修改数据库的键值对信息： 

~~~sql
hive>alter database foo set dbproperties ('owner'='itheima');
~~~



###### ==查看数据库信息==

```sql
hive> desc database foo;
OK
foo		hdfs://hadoop101:8020/user/hive/warehouse/foo.db	root	USER	
Time taken: 0.021 seconds, Fetched: 1 row(s)
```

###### ==查看详细的数据库信息==

```sql
hive> desc database extended foo;
OK
foo		hdfs://hadoop101:8020/user/hive/warehouse/foo.db	root	USER	{owner=root, data=20201127}
Time taken: 0.068 seconds, Fetched: 1 row(s)
```

###### ==删除数据库==

删除一个空数据库，如果数据库下面有数据表，那么就会报错

```sql
hive>drop  database  myhive2;
```

强制删除数据库，包含数据库下面的表一起删除

```sql
hive>drop  database  myhive  cascade;   
```

###### ==切换为当前数据库==

```sql
hive>use foo
```

### 5.2 数据库表的操作

##### 5.2.1 创建表的总体语法

```sql
create [external] table [if not exists] table_name (
col_name data_type [comment '字段描述信息']
col_name data_type [comment '字段描述信息'])
[comment '表的描述信息']
[partitioned by (col_name data_type,...)]
[clustered by (col_name,col_name,...)]
[sorted by (col_name [asc|desc],...) into num_buckets buckets]
[row format row_format]
[storted as ....]
[location '指定表的路径']
```

- ==create  table==

创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。

- ==external==

可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），**Hive 创建内部表时，会将数据==移动==到数据仓库指向的路径**；**若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。**

- ==comment==

表示注释,默认不能使用中文 

- ==partitioned by==

表示使用表分区,一个表可以拥有一个或者多个分区，每一个分区单独存在一个目录下 .

- ==clustered by==

对于每一个表分文件， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织。

- ==sorted by==

指定排序字段和排序规则

- ==row format==

指定表文件字段分隔符

- ==storted as==

指定表文件的存储格式,   常用格式：SEQUENCEFILE, TEXTFILE, RCFILE，如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 storted  as SEQUENCEFILE

- ==location==

指定表文件的存储路径

##### 5.2.2 内部表的操作

如果不使用external关键字，则创建的表是内部表，又叫管理表

###### ==创建内部表==

```sql
use myhive;
create table stu(id int,name string);
insert into stu values (1,"zhangsan");  #插入数据
select * from stu;
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\19.png)

从上面可以看到使用insert语句会启动mapreduce，所以执行效率不高，平时一般不使用

###### ==创建表并指定字段之间的分隔符==

之前已经知道了，hive中的表是存在hdfs上的，上面的操作创建了一张stu的表，可以通过hdfs的路径来查看该表的内容，因为没有显式指定表的位置，所以在默认位置；/user/hive/warehouse/myhive.db/stu

![](C:\Users\VSUS\Desktop\笔记\大数据\img\20.png)

下载到本地，通过editplus打开，如果没有指定表中字段的分隔符，hive中默认使用的是 \001

![](C:\Users\VSUS\Desktop\笔记\大数据\img\21.png)

可以通过row format指定表中字段的分隔符

```sql
#创建stu2表，并指定分隔符为"\t"，指定位置为'/user/stu2'
hive> create table if not exists stu2(id int, ame string) row format delimited fields terminated by '\t' location '/user/stu2';	
#插入一条数据
insert into stu2 values(1,'lisi');
```

在指定位置查看该文件

![](C:\Users\VSUS\Desktop\笔记\大数据\img\22.png)

###### ==创建表并指定文件路径==

使用location关键字即可

###### ==根据查询结果创建表==

```sql
hive> create table stu3 as select * from stu2;
```

这个过程也会走mapreduce

###### ==查询表的详细信息==

```sql
hive> desc formatted  stu2;
```



![](C:\Users\VSUS\Desktop\笔记\大数据\img\23.png)

###### ==删除表==

```sql
hive> drop table stu3;
```

###### ==查看所有表==

```sql
hive> show tables;
```

##### 5.2.3 外部表的操作

###### ==什么是外部表==

因为表是外部表，所以Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

###### ==使用场景==

每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。

###### ==创建表==

```sql
#创建教师表
create external table teacher (t_id string,t_name string) row format delimited fields terminated by '\t';
#创建学生表
create external table student (s_id string,s_name string,s_birth string , s_sex string ) row format delimited fields terminated by '\t';
```

###### ==加载数据==

本地有student.csv、teacher.csv文件，现在需求是将这两个文件数据上传到hive中teacher表和student表中

```
#student.csv
01      赵雷    1990-01-01      男
02      钱电    1990-12-21      男
03      孙风    1990-05-20      男
04      李云    1990-08-06      男
05      周梅    1991-12-01      女
06      吴兰    1992-03-01      女
07      郑竹    1989-07-01      女
08      王菊    1990-01-20      女
#teacher.csv
01      张三
02      李四
03      王五
```

可以有两种办法：

- 直接从系统本地加载
- 可以先将文件上传值hdfs，然后再从hdfs中加载数据

```sql
#直接从系统本地加载数据
load data local inpath '/root/hivedatas/student.csv' into table student;
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\24.png)

```sql
#从hdfs文件系统中加载数据
hdfs dfs -put techer.csv /hivedatas
load data inpath '/hivedatas/techer.csv' into table teacher;
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\25.png)

****

```sql
# 加载数据并覆盖已有数据
load data local inpath '/export/servers/hivedatas/student.csv' overwrite  into table student;
```

##### 5.2.4 内部表与外部表

- 内部表：删除表时同时删除数据
- 外部表：删除表时只将元数据信息删除，不删除原有数据

上述的student是一张外部表，使用load从hdfs上加载数据的时候，如果这张外部表没有指定location，那么会将数据移动到该表的目录下

![](C:\Users\VSUS\Desktop\笔记\大数据\img\27.png)

![](C:\Users\VSUS\Desktop\笔记\大数据\img\26.png)

但是如果删除一张外部表的话，数据并没有被删除，而是元数据被删除

![](C:\Users\VSUS\Desktop\笔记\大数据\img\28.png)

==存放原数据的文件并没有删除！！==

![](C:\Users\VSUS\Desktop\笔记\大数据\img\29.png)



==也可以在创建表的时候直接指定数据的位置，那么就不需要load就可以加载数据到表中==



内部表与外部表的转换

- 修改内部表student2为外部表

```sql
alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

- 修改外部表student2为内部表

```sql
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

注意：这里'EXTERNAL'='TRUE'是需要大写的，使用小写是不生效的

##### 5.2.5 分区表

==分区表实际上就是对应一个HDFS文件系统上的独立的文件夹==，该文件夹下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

###### ==创建分区表==

使用partitioned by语句

创建一个分区

```sql
create table score2(s_id string,c_id string,s_score int) partitioned by (year string,month string,day string) row formatted delimited fields terminated by '\t';
```

创建多个分区

```sql
create table score2(s_id string,c_id string,s_score int) partitioned by (year string,month string,day string) row format delimited fields terminated by '\t';
```

###### ==加载数据==

```sql
hive> load data local inpath '/root/hivedatas/score.csv' into table score partition(month='20201201');
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\31.png)

使用partition指定分区，会将该分区下的表放在month='20201201'这个文件夹下，例如；

![](C:\Users\VSUS\Desktop\笔记\大数据\img\30.png)

###### ==查看分区==

查看该表对应的分区

```sql
show partitions score;
```

查看某个分区下的数据，可以使用where语句

```sql
select * from score where month=20201201;
```

![](C:\Users\VSUS\Desktop\笔记\大数据\img\32.png)

多个分区联合查询

```sql
select * from score where month = '201806' union all select * from score where month = '201806';
```

###### ==添加分区==

```sql
hive> alter table score add partition(month = '20201202');
```

添加完分区后，会生成对应的目录

![](C:\Users\VSUS\Desktop\笔记\大数据\img\33.png)

###### ==删除分区==

```sql
alter table score drop partition(month=20201202);
```



##### 5.2.6 修改表

###### ==重命名表==

将score修改为score3

```sql
alter table score rename to score3
```

###### ==查询表结构==

```sql
desc table_name
```

###### ==添加列==

```sql
alter table stu add columns(sex int);
```

###### ==更新列==

```sql
alter table stu change column sex sexnew string;
```



### 六、数据操作

##### 6. 1 数据的导入

###### ==向表中加载数据==

基本语法：

```sql
 load data [local] inpath '/opt/module/datas/student.txt' overwrite | into table student [partition (partcol1=val1,…)];
```

- load data：表示加载数据
- local：表示从本地加载数据到hive表；否则从HDFS加载数据到hive表
- inpath：表示加载数据的路径
- overwrite：表示覆盖表中已有数据，否则表示追加

- into table：表示加载到哪张表
- student：表示具体的表
- partition：表示上传到指定分区

##### 6. 2 数据的导出

###### ==将查询结果导出到本地==

```sql
insert overwrite local directory '/root/hivedatas' select * from student;
```

###### ==将查询结果格式化导出到本地==

```sql
insert overwrite local directory '/root/hivedatas' row format delimited fields terminated by '\t'   select * from student;
```

