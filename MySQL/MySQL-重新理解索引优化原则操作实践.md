前几天字节面试，问我了解MySQL的索引吗，然后直接抛出了几道题给我，发现对这一块的知识还是理解的不深刻，所以重新学习索引相关的知识

# Explain字段分析

环境准备：

```sql
CREATE TABLE t1(id INT(10) AUTO_INCREMENT,content VARCHAR(100) NULL , PRIMARY KEY (id));
CREATE TABLE t2(id INT(10) AUTO_INCREMENT,content VARCHAR(100) NULL , PRIMARY KEY (id));
CREATE TABLE t3(id INT(10) AUTO_INCREMENT,content VARCHAR(100) NULL , PRIMARY KEY (id));
CREATE TABLE t4(id INT(10) AUTO_INCREMENT,content VARCHAR(100) NULL , PRIMARY KEY (id));


INSERT INTO t4(content) VALUES(CONCAT('t4_',FLOOR(1+RAND()*1000)));
INSERT INTO t4(content) VALUES(CONCAT('t3_',FLOOR(1+RAND()*1000)));
INSERT INTO t4(content) VALUES(CONCAT('t2_',FLOOR(1+RAND()*1000)));
INSERT INTO t4(content) VALUES(CONCAT('t1_',FLOOR(1+RAND()*1000)));
```



使用explain可以查看到MySQL是如何处理SQL语句的，explain分析下来一共有七个字段

![image-20210730222419109](http://cdn.noteblogs.cn/image-20210730222419109.png)

> id：select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

- id相同，执行顺序从上到下，可见加载表的顺序由上到下加载

![image-20210730222706569](http://cdn.noteblogs.cn/image-20210730222706569.png)

- id不同，id的序号会递增，id值越大优先级越高，越先被执行，例如嵌套查询，肯定要先查询最里面的一张表

- id有相同也有不同，id如果相同则认为是同一组的，从上到下执行，在所有组中id值越大，优先级越高

> select_type 代表查询的类型，主要是为了区别普通查询、联合查询、子查询等负责查询，一般来说有以下几个类型

| 类型                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| SIMPLE               | 简单的select查询，查询不包含子查询或者UNION类型              |
| PRIMARY              | 查询中包含任何复杂的子部分，最外层查询则被标记为PRIMARY      |
| SUBQUERY             | 在select或者where列表中包含了子查询                          |
| DERIVED              | 在from的列表里面使用子查询，MySQL会递归执行这些子查询，把结果放在临时表里 |
| UNION                | 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED |
| UNION RESULT         | 从UNION表获取结果的SELECT                                    |
| DEPENDENT SUBQUERY   | 在SELECT或WHERE列表中包含了子查询,子查询基于外层             |
| UNCACHEABLE SUBQUREY | 无法被缓存的子查询                                           |

> table 数据基于那张表的

> Partitions 分区

> type类型，是比较重要的一些指标，有以下取值

| type   | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| NULL   | MySQL不访问任何表、索引直接返回结果                          |
| system | 表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现 |
| const  | 表示通过索引一次就找到了，const 用于比较primary key 或者 unique 索引。因为只匹配一行数 据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常亮。const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较 |
| eq_ref | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常⻅ 于主键或唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某 个单独值的所有行(多个) |
| range  | 只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between ， < , > , in 等操作。 |
| index  | index 与 ALL的区别为 index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件 |
| all    | 将遍历全表以找到匹配的行                                     |

> key类型字段，有三个字段：
>
> - possible_keys：可能用在这张表上的索引
> - key：实际使用的索引吗，null表示没有使用索引
> - key_len：表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，长度越短越好

> rows字段，扫描行的数量

> extra字段，附加的子段

| extra           | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| using filesort  | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称 为 “文件排序”, 效率低 |
| using temporary | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常⻅于 order by 和 group by; 效率低 |
| using index     | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。 |

# 索引优化原则

## 环境准备

docker安装的mysql：5.7

表：tb_seller

字段：

![image-20210730104427432](http://cdn.noteblogs.cn/image-20210730104427432.png)

数据：

![image-20210730104948286](http://cdn.noteblogs.cn/image-20210730104948286.png)

SQL语句

```sql
create table `tb_seller` (
	`sellerid` varchar (100),
	`name` varchar (100),
	`nickname` varchar (50),
	`password` varchar (60),
	`status` varchar (1),
	`address` varchar (100),
	`createtime` datetime,
    primary key(`sellerid`)
)engine=innodb default charset=utf8mb4; 

insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('alibaba','阿里巴巴','阿里小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('baidu','百度科技有限公司','百度小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('huawei','华为科技有限公司','华为小店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itcast','传智播客教育科技有限公司','传智播客','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itheima','黑马程序员','黑马程序员','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('luoji','罗技科技有限公司','罗技小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('oppo','OPPO科技有限公司','OPPO官方旗舰店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('ourpalm','掌趣科技股份有限公司','掌趣小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('qiandu','千度科技','千度小店','e10adc3949ba59abbe56e057f20f883e','2','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('sina','新浪科技有限公司','新浪官方旗舰店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('xiaomi','小米科技','小米官方旗舰店','e10adc3949ba59abbe56e057f20f883e','1','西安市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('yijia','宜家家居','宜家家居旗舰店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');


create index idx_seller_name_sta_addr on tb_seller(name,status,address);//建立联合索引
```

## 规则条例

> 最左匹配规则，MySQL会一直向右匹配知道遇到范围查询（> < between like）就停止匹配

最左匹配规则原理理解起来很轻松，首先MySQL是使用B+树创建索引的，使用B+树比B树一共具备以下两点的好处：

- B+树磁盘IO次数更少，由于非叶子节点不存放数据，所以在相同数据量的情况下，B+树的高度更低
- B+树更适合做范围查询，B+树的叶子节点是有序的并且是有双向链表连接起来

下面通过一个例子来解释为什么会有最左匹配规则，同时为什么遇到了范围查询就会停止，例如创建一个（a,b)的联合索引，那么它的索引树是这样的

![image-20210731170802764](http://cdn.noteblogs.cn/image-20210731170802764.png)

a的值是有顺序的，而b的值是没有顺序的，在a值相等的时候按照b值进行排序，也就是说b是相对于a有序的：

- 对于b=? 这种是使用不到索引的，因为b本身是无序的，这就解释最左匹配
- 对于a>? 这种a的值是不确定的，一旦a的值不确定，自然b就是无序的

下面通过几个例子来演示

1）下面全部走上索引，根据上面的环境准备建立的SQL联合索引是(name,status,address)，所以下面均满足最左匹配原则

```sql
explain select * from tb_seller where name='小米科技';
explain select * from tb_seller where name='小米科技' and status = '1';
explain select * from tb_seller where name='小米科技' and status = '1' and address = '北 京市';
explain select * from tb_seller where address = '北京市' and status = '1' and name = '小米科技';
```

![image-20210731171936808](http://cdn.noteblogs.cn/image-20210731171936808.png)

2）不走索引的例子

```sql
explain select * from tb_seller where status = '1'; 	#违反最左匹配规则
explain select * from tb_seller where tatus = '1' and address = '北京市';	#违反最左匹配规则
explain select * from tb_seller where name = '小米科技' and status > '1' and address = '北京市';	# 范围查询的列
```

![image-20210731172500768](http://cdn.noteblogs.cn/image-20210731172500768.png)

注意：第三个例子包括到了status字段，只有范围查询右边的列才走不上索引

3）跳跃了其中的列，只有使用最左生效

```sql
explain select * from tb_seller where name='小米科技' and address = '北京市';
```

![image-20210731172808668](http://cdn.noteblogs.cn/image-20210731172808668.png)



> 使用运算操作，不使用索引

原因是因为数据库中存放的都是字段的值，这个值是有序的很好比较，但是使用了运算操作，就要将所有元素经过运算才能取出来进行比较

```sql
explain select * from tb_seller where substring(name,3,2) = '科技';
```

![image-20210731172942870](http://cdn.noteblogs.cn/image-20210731172942870.png)



> 字符串不加单引号，不使用索引没，也就是说类型不一致会不走索引

![image-20210731173943307](http://cdn.noteblogs.cn/image-20210731173943307.png)



> 使用or查询的sql语句，如果or前的条件中有索引，但是后面的列中有索引，都不走索引

这个也可以理解，因为or是两者有一个成立即可，但是因为有一个没有索引，那么需要将全部数据取出来进行判断

```sql
explain select name from tb_seller where name = '黑⻢程序员' or createtime = '2018-01-01 12:00:00\G';
```

![image-20210731174441836](http://cdn.noteblogs.cn/image-20210731174441836.png)



> 使用!=会使索引失效

同样的，因为是不等于要进行全表的比较判断才可以

# 总结

对于SQL来说，索引应该是SQL优化的第一前提，使用到了索引对于提升SQL的执行效率是很有帮助的。分析定位一条SQL语句可以查看满查询日志，接着使用explain分析定位

