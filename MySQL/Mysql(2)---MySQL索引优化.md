参考：《MySQL技术内幕  InnoDB存储引擎  第2版》

​			《黑马程序员 高级sql》

使用的mysql版本：5.7.31

###一、使用explain分析执行计划

在分析某条sql语句的性能的时候，可以使用explain去查看mysql底层对这条sql语句的执行计划，执行计划可以告诉我们：

* SQL如何使用索引
* 联接查询的执行顺序
* 查询扫描的数据函数

例如：

```sql
explain select * from tb_item where id = 1 ;
```

![05](http://cdn.noteblogs.cn/05.png)

**其中最重要的字段为：id、type、key、rows、Extra**

> 环境准备：t_user、t_role、user_role，其中t_user中id为主键，t_role为id为主键，user_role中user_id、role_id分别为两表的主键

##### 1.id字段

id 字段是 select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。id 情况有三种 ： 

1）id 相同表示加载表的顺序是从上到下

```sql
 explain select * from t_role r, t_user u, user_role ur where r.id = ur.role_id and u.id = ur.user_id ;
```

![](http://cdn.noteblogs.cn/06.png)

table字段表示加载的表，可见加载的表的id是一样的，所以加载表的顺序是从上到下的，分别是r、u、ur





2） id 不同id值越大，优先级越高，越先被执行。

```sql
 EXPLAIN SELECT * FROM t_role WHERE id = (SELECT role_id FROM user_role WHERE user_id = (SELECT id FROM t_user WHERE username = 'stu1'))
```

上面是一个嵌套查询，那其实很显然要先加载里面的表：

![](http://cdn.noteblogs.cn/07.png)

id越高优先级越高，越先被执行，很显然首先需要被加载的就是最里面的t_user表



3） id 有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。

##### 2.select_type字段

- **SIMPLE**：表示简单查询

```sql
explain select * from t_user;
```

![](http://cdn.noteblogs.cn/08.png)

- **PRIMAR**Y：查询中若包含任何复杂的子查询，最外层查询标记为该标识
- **SUBQUERY**：在SELECT 或 WHERE 列表中包含了子查询

```sql
explain select * from t_user where id = (select id from user_role where role_id = '9')
```

![](http://cdn.noteblogs.cn/09.png)

- **DERIVED**：在FROM 列表中包含的子查询，被标记为 DERIVED（衍生） MYSQL会递归执行这些子查询，把结果放在临时表中

- **UNION**若第二个SELECT出现在UNION之后，则标记为UNION ； 若UNION包含在FROM子句的子查询中，外层SELECT将被标记为 ： DERIVED
- **UNION RESULT**：i从UNION表获取结果的SELECT

##### 3.  type字段

type 显示的是访问类型，是较为重要的一个指标，可取值为： 

| type   | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| NULL   | MySQL不访问任何表，索引，直接返回结果                        |
| system | 表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现 |
| const  | 表示通过索引一次就找到了，const 用于比较primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常亮。const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较 |
| eq_ref | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常见于主键或唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| range  | 只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between ， < , > , in 等操作。 |
| index  | index 与 ALL的区别为  index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件。 |
| all    | 将遍历全表以找到匹配的行                                     |

##### 3. key字段

- possible_keys : 显示可能应用在这张表的索引， 一个或多个。 

- key ： 实际使用的索引， 如果为NULL， 则没有使用索引。

- key_len : 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下， 长度越短越好 。

##### 4. rows字段

扫描行的数量

##### 5. extra字段

| extra            | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| using  filesort  | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为 “文件排序”, 效率低。 |
| using  temporary | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和 group by； 效率低 |
| using  index     | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。 |

### 二、验证索引可以大幅度提升sql语句性能

准备了300万条数据的insert语句，如果有需要的可以私信我，表的结构如下：

```sql

DROP TABLE IF EXISTS `test`;
CREATE TABLE `test` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(200) NOT NULL DEFAULT '' COMMENT '商标名称，sbmc',
  `apply_date` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '申请日期，sqrq',
  `applicant_ch` varchar(200) NOT NULL DEFAULT '' COMMENT '申请人名称中文，sqrmcZw',
  `applicant_en` varchar(200) NOT NULL DEFAULT '' COMMENT '申请人名称英文，sqrmcYw',
  `address_ch` varchar(200) NOT NULL DEFAULT '' COMMENT '申请人地址中文，sqrdzZw',
  `agency` varchar(200) NOT NULL DEFAULT '' COMMENT '代理/办理机构，dlrmc',
  `latest_status` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最新商标状态，newProcess',
  `options_start` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '专用权期限开始时间，zyqqx',
  `options_end` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '专用权期限结束时间，zyqqx',
  `update` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '修改日期',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4709361 DEFAULT CHARSET=utf8 COMMENT='商标数据';
```

导入sql脚本的时候，先将引擎改为MyISAM，这样在导入sql脚本的时候更快一些，但是还是整整用了将近2个小时的时间

修改存储引擎

```sql
alter table mytest ENGINE = InnoDB;
```

![](http://cdn.noteblogs.cn/10.png)

根据id查询某条数据

![](http://cdn.noteblogs.cn/11.png)

根据title来查询某些数据

![](http://cdn.noteblogs.cn/12-1616995578203.png)

可见时间耗费还是挺大的！问题出在哪里呢？可以使用explain来分析执行计划

根据id查询某条数据，使用来主键索引，所以查询速度很快！

![](http://cdn.noteblogs.cn/13-1616995595524.png)



根据title来查询某些数据，没有任何索引，type为All性能很低

![14](http://cdn.noteblogs.cn/14-1616995605054.png)



那么，再对title字段建立索引，

```sql
create index idx_test_title on test(title);
```

再次查询，看看使用时间

![](http://cdn.noteblogs.cn/15-1616995625207.png)

足以可见索引的重要性，再次使用explain分析执行计划吗，使用了刚才创建的索引！

![](http://cdn.noteblogs.cn/16-1616995638856.png)

### 三、索引使用规则

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



#### 3.1 最左前缀法则

如果使用了联合索引，那么要准守最左前缀法则：查询从索引的最左列开始，并且不跳过索引中的列。其实这很好理解其中的原理， 使用联合索引和单个键值的B+树大致相同，看下图

![](http://cdn.noteblogs.cn/17-1616995651527.png)



数据按照(a,b)的顺序进行了存放，如果是对于对于单列的查询a 显然是可以使用联合索引的，但是如果是只对b进行查询，那么b列可以发现是没有顺序的，所以对于b列的查询使用不到(a,b)的索引。

（1）上述的idx_seller_name_sta_addr索引是(name,status,address)，那么对于下面3条sql语句来说都是走索引的：

```sql
explain select * from tb_seller where name='小米科技';
explain select * from tb_seller where name='小米科技' and status = '1';
explain select * from tb_seller where name='小米科技' and status = '1' and address = '北京市';
explain select * from tb_seller where address = '北京市'   and status = '1' and name = '小米科技';//3个索引列都需要查询，那么顺序不影响索引的使用
```



![](http://cdn.noteblogs.cn/18-1616995672875.png)

（2）但是对于下面的sql来说就是不走索引的，因为这违背了最左前缀法则：

```sql
explain select * from tb_seller where status = '1';
explain select * from tb_seller status = '1' and address = '北京市';
```

（3）如果符合最左法则，但是跳跃其中的列，只有最左使用生效

```sql
 explain select * from tb_seller where name='小米科技' and address = '北京市';
```

可以看到索引的长度变小了，只使用了name列的索引，而status因为被跳过所以后面的索引没有用到

![](http://cdn.noteblogs.cn/19-1616995686440.png)

#### 3.2. 范围查询右边的列，不使用索引

```sql
explain select * from tb_seller where name = '小米科技' and status > '1' and  address = '北京市';
```

![](http://cdn.noteblogs.cn/20-1616995700936.png)

可以看到，索引列的长度变为410，说明只有name，status两个列走了索引，单独name走索引长度是403，name、status列走索引是410，name、status、address都走索引是813

#### 3.3 使用运算操作，不使用索引

```sql
explain select * from tb_seller where substring(name,3,2) = '科技';
```

![](http://cdn.noteblogs.cn/21-1616995725000.png)

#### 3.4 字符串不加单引号，不使用索引

```sql
mysql> desc tb_seller;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| sellerid   | varchar(100) | NO   | PRI | NULL    |       |
| name       | varchar(100) | YES  | MUL | NULL    |       |
| nickname   | varchar(50)  | YES  |     | NULL    |       |
| password   | varchar(60)  | YES  |     | NULL    |       |
| status     | varchar(1)   | YES  |     | NULL    |       |
| address    | varchar(100) | YES  |     | NULL    |       |
| createtime | datetime     | YES  |     | NULL    |       |
+------------+--------------+------+-----+---------+-------+
```

对于status字段，是varchar(1)的，但是如果在查询时不使用单引号的话，是可以执行的，

![](http://cdn.noteblogs.cn/22-1616995735509.png)

但是使用explain查询分析计划，是没有使用索引的，mysql的查询优化器自动进行类型转换，造成了索引失效

![](http://cdn.noteblogs.cn/23-1616995744486.png)

#### 3.5 尽量使用索引覆盖

在说索引覆盖之前，必须说说聚集索引和辅助索引（非聚集索引）

- 聚集索引：聚集索引就是按照每张表的主键构造一棵B+树，同时==叶子节点中存放的即为整张表的行记录数据==。

  > 对于Innodb，主键毫无疑问是一个聚集索引。但是当一个表没有主键，或者没有一个索引，Innodb会如何处理呢。请看如下规则:
  >
  > * 如果一个主键被定义了，那么这个主键就是作为聚集索引。
  > * 如果没有主键被定义，那么该表的第一个唯一非空索引被作为聚集索引。
  > * 如果没有主键也没有合适的唯一索引，那么innodb内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个6个字节的列，该列的值会随着数据的插入自增。

- 辅助索引：也叫非聚集索引。和聚集索引相比，==叶子节点中并不包含行记录的全部数据==                                                                                    叶子节点除了包含键值以外，每个叶子节点的索引行还包含了一个书签（bookmark），该书签用来告诉InnoDB哪里可以找到与索引相对应的行数据。

也就是说，如果使用的是辅助索引，那么存在二次查询问题：辅助索引叶节点仍然是索引节点，只是==有一个指针指向对应的数据块==，因此如果使用非聚集索引查询，而查询列中包含了其他该索引没有覆盖的列，那么他还要进行第二次的查询，查询节点上对应的数据行的数据。

那么这个时候，要解决辅助索引的二次查询问题，就要使用覆盖索引，==也就是说mysql直接从辅助索引中就可以拿到想要的值，而不需要在进行第二次的去查询聚集索引中的记录==，举个例子，对于上述索引idx_seller_name_sta_add(name,status,address)，如果我查询的字段小于或者等于<name、status、address>这三个字段(就是除这3个字段外，不查询其他字段)，那么则使用索引覆盖

![](http://cdn.noteblogs.cn/24-1616995754698.png)

上面查询只查询了name字段，Extra中为using index，表示使用了索引覆盖

>     using index ：使用覆盖索引的时候就会出现
>     
>     using where：在查找使用索引的情况下，需要回表去查询所需的数据
>     
>     using index condition：查找使用了索引，但是需要回表查询数据
>     
>     using index ; using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

![](http://cdn.noteblogs.cn/25-1616995764445.png)

但是如果使用select * 查询的字段不完全在索引字段中，那么就还要进行二次查询，效率偏慢。

####  3.6 使用or查询的条件

使用or查询的条件sql语句，如果or前的条件中的列有索引，后面的列没有索引，那么涉及的索引 都不会使用到

```sql
explain select name from tb_seller where name = '黑马程序员' or createtime = '2018-01-01 12:00:00\G';
```

![](http://cdn.noteblogs.cn/26-1616995773975.png)