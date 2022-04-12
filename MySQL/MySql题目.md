##### 1. count(1)、count(*)、count(id/其他字段)

1、执行速度上：针对一般情况（SQL语句中没有where条件）执行速度上

**count(\*)=count(1)>count(主键)>count(其他列)，在没有特殊查询要求推荐使用count(\*)来代替其他的count。**

2、执行结果上，count(*)与count(1)以及count(主键)的结果完全相同，即返回表中的所有行数，包含null值；count(其他列)会排除掉该列值为null的记录，返回的值小于或者等于总行数。