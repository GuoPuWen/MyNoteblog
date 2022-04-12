# 1.什么是SQL

​		Structured Query Language ：结构化查询语言

​		定义了操作所有关系型数据库的规则，每一种数据库存在不一样的地方

# 2. SQL通用语法

​		①SQL 语句可以单行或者多行书写，**以分号结尾**

​		② MySQL 数据库的SQL语句不区分大小写，建议使用大写

​		③注释方法： 单行注释： -- 注释内容(注意这里要有空格)    # 注释内容(MySQL独有的)

​								多行注释： /*      注释内容         */

# 3. SQL分类



![数据库](C:\Users\VSUS\Desktop\新建文件夹\数据库.jpg)



## 3.1 **DDL操作数据库、表**

### 3.1.1 操作数据库： CRUD

* C(Create) : 创建

  1. 创建数据库:  

     ​		```create database [数据库名称];```

  2. 创建数据库，并判断是否存在:  

     ​		```create database if not exists [数据库名称];```

  3. 创建数据库，并指定字符集:

     ​		```create database [数据库名称] character set [字符编码];```

  4. ④综合②③：

     ​		```create database if not exists [数据库名称] character set [字符编码]；```


* R(Retrieve):查询

  1. 查询所有数据库： 
     ```SHOW DATABASES;``

  2. 查询某个数据库的字符集，查询某个数据库的创建语句

     ```show create database [要查询的数据库];```

 * (Update):修改

     1. 修改数据库的字符集：

        		```alter database [数据库名称] character set [字符集];```                               

 * D(Delete) :删除

     1. 删除数据库：

        			```drop database [数据库名称];```

     2. 判断是否存在，然后删除数据库

        			```drop database if exists [数据库名称]```

* 使用数据库：
  1. 选择数据库
     ```use  [数据库名称];```
  2. 查看当前选择的数据库
      ```select database();```

### 3.1.2操作表

* C(Create) : 创建
  
  1. 语法：
  
     ```java
     create table 表名(
     	列名1 数据类型1,
     	列名2 数据类型2,
         列名1 数据类型3,
             ······
         列名n 数据类型n 		# 注意这里没有逗号
     );
     ```
  
  2. 数据表类型
  
    1.  int ：整数类型
  
    2.  double : 小数类型
  
    3.  date ：日期类型，包含年月日， yyyy-MM-dd
  
    4.  datatime :  日期，包含年月日时分秒  yyyy-MM-dd-  HH-mm-dd
  
    5.  timestamp : 时间戳类型，包含年月日，不给这个字段赋值，默认当前系统时间
  
    6.  varchar： 字符串，要指定字符串的最大字符数
  
      	name varchar(20);
      zhangsan 8个字符   张三 2个字符	
       
    7. null值：具有NULL值的字段是没有值的字段	
    
       注意：NULL值与零值或包含空格的字段不同，具有NULL值的字段是在记录创建期间留空的字段！
  
  ```sql
  create table Student(
  	id int,
  	name varchar(20),
  	age int,
  	score double(4, 1),
      birthday date,
      insert_time timestamp
  );
  ```
  
  ![创建表后查看效果](C:\Users\VSUS\Desktop\捕获.PNG) 
  
  3. 复制表
     create table [表名] like [被复制的表];
  
* R(Retrieve):查询
  
  1. 查询某个数据库的所有的表名称：
     ```show tables;```
  
  2. 查询表结果：
     ```desc [表名称];```
  
  3. 查询某个表的字符集
  
     show create table [表名];
  
* (Update):修改

  1. 修改表名
     			
  ```
     			alter table [表名] rename to [新表名];
  ```
  
  2. 修改表字符集

     ```
     alter table [表名] character set [字符集]; 
     ```

  3. 添加列

     ```
     alter table [表名] add 列名 数据类型;
     ```

  4. 修改列名称，类型
    
  ```
     alter table [表名] change [列名] [新列名] [数据类型];

     alter table [表名] modify [列名] [数据类型];      # 只能修改列的数据类型 
  ```
  
  5. 删除列
    
     ```
     alter table [表名] drop [列名];
     ```

* D(Delete) :删除

  1. drop table [表名];
  2. drop table if exists  [表名]; 

## **3.2 DML** 增删改表中的数据

- 添加数据

  1. 语法：

     ``````sql
         insert into [表名]([列名1], [列名2], ..., [列名3]) values([值1], [值1], [值1]);
     ``````
     
  2. 注意：

     1. 表名和值是一一对应的关系

     2. 如果表名后面不定义列名，则默认给所有值添加，多写或者小写均会报错

        ```sql
        insert into values([值1], [值1], [值1]);
        ```

     3. 除了数字类型外，其他类型需要用引号

- 删除数据

  1. 语法

     ```sql
     delete from [表名] where [条件]
     ```

  2. 注意

     1. 如果不加条件会把整表进行删除，这样删除整表效率低下，是一条一条记录的删除，可以使用下面的代码

        ```sql
        truncate table [表名];
        ```

        这样的话是先删除整表，然后在创建一个一模一样的空表。

- 修改数据

  1. 语法

     ```sql
     update [表名] set 列名1 = 值1, 列名2 = 值2 ... where [条件]
     ```

  2. 注意：
    
     1. 如果不加where那么将会将所有的数据记录进行修改

## 3.3 DQL 查看表中数据

- 整体语法
  
```sql
  select 
  	字段列表
  
  from
  	表名类表
  where
  	条件列表
  group by
  	分组字段
  having
  
  ​	分组之后的操作
  order by
  ​	排序
  limit
  ​	分页限定
```

- 查询所有
  select * from [表名];

- 排序查询

  1. 语法

     ```sql
     select * from [表名] order by 排序字段1 排序方式1,  排序字段2 排序方式2 .....;
     ```

  2. 排序方式

     1. ASC :升序
     2. DESC :降序

  3. 注意：
     1. 不指明排序方式则默认升序
     2. 只有在排序字段1的值相等时，才会对排序字段2进行排序

- 聚合函数

  1. 用法：
     sleect   [函数名称] from [表名];

  2. 聚合函数：

     1. count：计算个数

        **注意：一般选择非空的主键或者是count(*)**

     2. min：最小值

     3. max：最大值

     4. sum：求和

     5. avg：计算平均值

  3. **注意：聚合函数的计算不包含null值，解决方法是①选择不包含null值的列；②IFNULL函数**

- 分组查询

  1. 语法
       where 条件 group by 分组字段 having 条件

  2. 注意: 

     1. 查询字段是分组字段或者是聚合函数，其他字段没有意义

     2. where和having的区别是？

        ①where是在分组前进行限定，如果不满足条件，则不参与分组。having在分组之后进行限定，如果不满足结果，则不会查询出来
        
        ②where 后不可以更聚合函数，having后面可以更聚合函数的判断。

- 分页查询
  1. 语法
    
     ```
     limit 开始的索引, 每页查询的条数
     ```
     
     注意
     
     1. 开始的索引 = （页数 - 1）* 每页查询的条数
     2. limit的操作是一个“方言”，即只有mysql可用

# 4.约束

### **1.概念**

### 2.非空约束  not null

1. 创建表时进行非空约束

   ```sql
   create table 表名(
   	列名1 数据类型1 not null,
   	列名2 数据类型2 not null,
       列名1 数据类型3 ........,
           ······
       列名n 数据类型n 		# 注意这里没有逗号
   );
   ```

   如果不对有非空约束的值添加值，那么会报
   ```Field '[]' doesn't have a default value```

2. 创建表之后，进行非空约束，就是使用操作表中的更改列名的方式

   ```sql
   table [表名] change [列名] [新列名] [数据类型] not null;
   
   alter table [表名] modify [列名] [数据类型] not null;  
   ```

3. 删除非空约束，同样也是用操作表中的更改列名的方式alter ``` 

   ```sql
   table [表名] change [列名] [新列名] [数据类型];
   
   alter table [表名] modify [列名] [数据类型];      # 只能修改列的数
   ```

### 3.唯一约束 unique

- 创建表时添加

  ```sql
  create table 表名(
  	列名1 数据类型1 unique,
  	列名2 数据类型2 unique,
      列名1 数据类型3 ........,
          ······
      列名n 数据类型n 		# 注意这里没有逗号
  );
  ```

  如果添加相同的值，那么会报错

  ​	Duplicate entry '[values]' for key [数据]'

  > 注意：唯一约束可以添加null值

- 删除唯一约束

  ```
  alter table [表名] drop index [列名]; 
  ```

  > 注意：①不能使用非空约束的方式删除唯一约束
  > ② 列名这里不能写数据类型

- 创建表后添加唯一约束

  和非空约束一样：

  ```
  alter table [表名] modify [][] [列名] [数据类型] unique;
  ```

### 4. 主键约束 primary key

- 注意

  > ① 含义; 非空且唯一
  >
  > ② 一张表只能有一个主键字段
  >
  > ③ 主键字段就是记录里的唯一标识
  >
  > ④ primary key 是两个关键字，中间空格不能省略

- 创建表时添加

  ```sql
  create table 表名(
  	列名1 数据类型1 primary key,
  	列名2 数据类型2 primary key,
      列名1 数据类型3 ........,
          ······
      列名n 数据类型n 		# 注意这里没有逗号
  );
  ```

- 删除主键约束

  ```
  alter table [表名] drop primary key 
  ```

  > 注意：①不能使用非空约束的方式删除主键约束
  > ② 这里可以不用写列名，因为主键只能有一个字段

- 创建表后添加唯一约束

  和非空约束一样

### 5.自动增长

- 概念

  如果某一列是数值类型的，使用auto_increment 可以完成值的自动增长，这个值是根据上一个记录的值来加1的。一般和主键配合使用

- 其他操作与非空约束一样

### 6.外键约束   

- 注意：

  1. 外间约束的作用:让表与表产生关系，从而保证数据的正确性

  2. **外键约束所有链接的外表的列必须是主键列，若不是主键，则要报错：**

     ```SQL
     Failed to add the foreign key constraint. Missing index for constraint 'emp_dep_fk' in the referenced table 'dep'
     ```

  3. 外键值可以为null，但是不可以为链接的外表的列中不存在的值

- 语法

  1. 创建表时

     ```SQL
     create table 表名(
     	列名1 数据类型1 primary key,
     	列名2 数据类型2 primary key,
         列名1 数据类型3 ........,
             ······
         列名n 数据类型n 		# 注意这里没有逗号
         constraint [外键名称] foreign key ([外键列名称]) references [主键名称] ([主列表名称])
     );
     ```

     

- 删除外键

  ```
  alter table [] drop foreign key;
  ```

  

- 创建表时添加外键

  ```
  alter table[] add constraint [外键名称] foreign key ([外键列名称]) referances [主键名称] ([主列表名称])
  ```


- 级联操作 
  1. 添加级联操作
     constraint [外键名称] foreign key ([外键列名称]) referances [主键名称] ([主列表名称]) on update cascade on delete casecade
  2. 分类
     1. on update cascade 级联更新
     2. on delete casecade 级联删除

# 5.多表关系

### 1.一对一关系(不常用)

- 例子： 人与身份证
- 实现：在两张表的任意一张表中添加**唯一外键**，指向另一张表的主键

### 2.一对多关系(多对1关系)

- 例子：部门与员工，一个部门可以有多个员工，而一个员工只可以有一个部门
- 实现：在多的一方，建立外键指向另一方的主键

### 3.多对多关系

- 例子：学生与课程，一个学生可以有很多课程，一门课程也可以有很多学生
- 实现：需要借助第三张表来实现，中间表至少包含2个字段，这两个字段作为中间表的外键，分别指向两张表的主键

# 6.数据库设计范式



# 7.数据库的备份与还原

- 命令行
  1. 备份： mysqldump -u用户名 -p密码 数据库名称 > 保存的路径
  2. 还原： 
     1. 登陆数据库
     2. 创建数据库
     3. 使用数据库
     4. 执行文件 source 文件路径

# 8.多表查询

- 语法：

  ```
    select 
    	字段列表
  
    from
    	表名类表
    where
    	条件列表
    group by
    	分组字段
    having
  
    	分组之后的操作
    order by
    	排序
    limit
    	分页限定
  ```

  

- 笛卡儿积： 

   有两个集合A, B，取这两个集合的所有组成情况，要完成多表查询，要消除无用的数据。
   
   **例如**
   
   员工表emp
   
   <img src="F:\笔记\sql图片\QQ截图20200112195047.png" alt="QQ截图20200112194557"  />
   
   部门表dep
   
   ![QQ截图20200112194909](F:\笔记\sql图片\QQ截图20200112194909.png)

### 1.内连接查询

- 隐式内连接：使用where条件 消除无用的数据

  select [查询的字段] from [表] where [条件];

- 显式内连接

  select [查询的字段] from [表名1] inner join 表名2 on [条件]

- 查询的只是交集部分

  使用内连接查询

  <img src="sql图片\QQ截图20200112195222.png" alt="QQ截图20200112195222"  />

### 2.外连接

- 左连接查询

  select [查询的字段] from 表1 left outer join on　条件

  查询的是左表的所有数据以及交集部分

- 右连接查询

  与左连接类似，查询的是右表所有数据以及交集部分

  使用外连接查询

  ![QQ截图20200112195222](sql图片\QQ截图20200112195436.png)

### 3.子查询

- 概念：查询中嵌套查询

  要查找员工中薪水最高的员工信息，使用嵌套查询（使用聚合函数）

  ```
  SELECT * FROM emp WHERE emp.`salary` = (SELECT MAX(emp.`salary`) FROM emp);
  ```

  

- 子查询的不同情况
  
  1. 子查询的结果是单行单列的
  
     > 例如： 查询员工中薪水最高的员工信息
  
  2. 子查询结果是多行单列的
  
     子查询
  
     > 例如：查询员工中属于财务部或者是研发部的员工信息
     >
     > ```sql
     > SELECT * FROM emp WHERE emp.`dep_id` IN (SELECT dep.`dep_num` FROM dep WHERE dep.`dep_name` IN ('市场部', '研发部'));
     > 运算符 IN  代表在这个个区间内都可以
     > ```
  
     
  
  3. 子查询结果是多行多列的
  
     **子查询可以作为2虚拟表**
  
     > 例如： 查询入职时间在2016年以后的员工的信息以及对应的部门信息
     >
     > ```sql
     > // 使用子查询
     >     SELECT * 
     >     FROM 
     >         dep t1 , 	
     >         (SELECT * FROM emp WHERE DATA > 2016-01-01) t2 
     >     WHERE
     >         t1.`dep_num` = t2.dep_id;
     > // 使用内连接查询
     >     SELECT 
     >         *
     >     FROM
     >         dep t1,
     >         emp t2
     >     WHERE
     >         t1.`dep_num` = t2.`dep_id` AND t2.`data` > 2016-01-01;
     > ```
     >
     > 

# 9.事务

### 1.事务的基本概念

1. 概念：如果一个包含多个步骤的业务操作，被事务管理，要么同时成功，要么同时失败。
2. 操作：
   1. 开启事务： start transaction
   2. 回滚：rollback
   3. 提交：commit

3. 例子： A向B账户转账500元

   ```sql
   CREATE TABLE account(
   	id INT PRIMARY KEY AUTO_INCREMENT,
   	NAME VARCHAR(30),
   	balance DOUBLE
   );
   
   INSERT INTO account (NAME, balance) VALUES ('A', 1000), ('B', 1000); 
   
   SELECT * FROM account;
   UPDATE account SET balance = 1000;
   
   -- A向B转账500元
   UPDATE account SET balance = balance - 500 WHERE NAME = 'A';
   。。。。
   UPDATE account SET balance = balance + 500 WHERE NAME = 'B';
   ```

   当转账过程中出现意外时，就会出现下面情况

   

<img src="F:\笔记\sql图片\QQ截图20200115210927.png" style="zoom: 150%;" />

如果使用事务操作，当发现出现意外时，可以使用回滚操作回到之前的状态

```sql
START TRANSACTION;
-- A向B转账500元
UPDATE account SET balance = balance - 500 WHERE NAME = 'A';
。。。。
UPDATE account SET balance = balance + 500 WHERE NAME = 'B';
-- 出现意外时回滚
ROLLBACK;
-- 发现执行没有问题，提交事务
COMMIT;
```

注意：如果执行没有问题，不手动提交事务，临时数据将不被保存。即如果关闭当前窗口，数据不会保存。

​		![](sql图片\QQ截图20200115211946.png)

![QQ截图20200115211923](sql图片\QQ截图20200115211923.png)

4. MySQL数据库是事务默认自动提交

   - 事务提交的两种方式：
     - 自动提交
       - mysql就是默认自动提交的
       - 一条DML(增删改)语句就会自动提交一次事务
     - 手动提交 ： 需要先开启事务，再提交（就像上面讲的例子一样，如果开启了事务，则当前操作是手动提交的，如果不进行手动提交，临时的数据将不会被保存）

   - 修改事务的默认提交方式

     - 查看事务的提交方式

       ```sql
       SELECT @@autocommit; -- 1代表自动提交  0 代表手动提交			      
       ```

       

     - 修改默认提交方式

       如果修改了默认提交方式，如果不手动提交，数据只是临时的

       ```sql
       set @@commit = 0；
       ```

       ![](F:\笔记\sql图片\QQ截图20200115212659.png)

### 2.事务的四大特征

1. 原子性：是不可分割的最小操作单位，要么同时成功，要么同时失败
2. 持久性：当事务提交或回滚之后，数据库会持久化的保存数据
3. 隔离性：多个事务之间，相互独立
4. 一致性：事务操作前后，数据总量不变

### 3.事务的隔离级别