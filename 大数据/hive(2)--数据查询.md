# 七、Hhive查询

### 7.1 基本语法

```sql
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list [HAVING condition]]
[CLUSTER BY col_list
| [DISTRIBUTE BY col_list] [SORT BY| ORDER BY col_list]
]
[LIMIT number]
```

- order by 会对输入做全局排序，因此只有一个reducer，会导致当输入规模较大时，需要
  较长的计算时间。
- sort by不是全局排序，其在数据进入reducer前完成排序。因此，如果用sort by进行排
  序，并且设置mapred.reduce.tasks>1，则sort by只保证每个reducer的输出有序，不保证全
  局有序。
- distribute by(字段)根据指定的字段将数据分到不同的reducer，且分发算法是hash散列。
- cluster by(字段) 除了具有distribute by的功能外，还会对该字段进行排序

因此，如果distribute 和sort字段是同一个时，此时， cluster by = distribute by +
sort by  

### 7.2 查询语法

##### 7.2.1 全表查询

```sql
select * from score;
```

##### 7.2.2 选择特定列

```sql
select s_id from score;
```

##### 7.2.3 列别名

这个作用和sql是一样的

```sql
select s_id as myid ,c_id from score;
```

### 7.3 常用函数

- count()

- max()
- min()
- sum()
- avg()

用法和sql一致

### 7.4 LIMIT语句

```sql
select * from score limit 5;
```

