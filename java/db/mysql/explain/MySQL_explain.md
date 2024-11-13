

# 1.id：每个select都有一个对应的id号，并且是从1开始自增的

- 如果id序号相同，从上往下执行
- 如果id序号不同，序号大先执行
- 如果两种都存在，先执行序号大，在同级从上往下执行
- 如果显示为null，最后执行。表示结果集，并且不需要使用它来进行查询

# 2.select type：查询语句执行的查询操作类型

## 2.1.simple

> 简单select，不包括union与子查询	

```sql
explain select * from user;
explain select * from user inner join order on user.id = order.userId
```



## 2.2.promary

> 复杂查询中最外层查询，比如使用union或union all时，id为1的记录select_type通常时primary

```sql
explain select * from user union select * from order;
```



## 2.3.subquery

> 是指select语句中出现的子查询语句，结果不依赖与外部查询（不在from语句中）

```sql
explain select order.*, (select name from product where id = 1) from order;
```

## 2.4.dependent subqurey

> 指在select语句中出现的查询语句，结果依赖于外部查询

```sql
explain select order.*, (select name from product where product.id = order.productId) from order;
# 依赖于订单表的productId
```

## 2.5.derived

> 派生表，在from子句的查询语句，表示从外部数据源中推导出来的，而不是从select语句中的其他列中选择出来的

```sql
set session optimizer_switch = 'derived_merge=off'; #关闭MySQL5.7队衍生表合并优化

explain
select * from (select userId from user where id = 1) as temp

set session optimizer_switch = 'derived_merge=on';
```

## 2.6.union

> 分为union与union all两种，若第二个select出现咋union之后，则被标记为union；如果union被from子句的子查询包含，那么第一个select会被标记为derived；union会针对相同的结果集进行去重，union all不会进行去重处理。

```sql
explain
select * from(
	select id from product where price = 10
    union
    select id from order where userId in(1,2)
    union
    select id from user where name = '张三'
) as temp
```

## 2.7.dependent union

> 当union作为子查询时，其中第一个union为dependent subquery，第二个为dependent union。

```sql
explain
select * from orders where id in(
	select id from product where price = 10
    union 
    select id from order where userId = 2
    union
    select id from user where name = '张三
)
```

## 2.8.union result

> 如果两个查询有相同的列，则会对这些列进行重复删除，只保留一个表中的列

```sql
explain
select id from user
union
select id from product
```

# 3.table ：表名

# 4.PARTITIONS：表分区情况

# 5.type：查询所用的访问类型

> **效率**从高到低分别为：system > const >  eq_ref > ref > fulltext > ref_or_null > range > index，一般来说保证range级别，最好能达到ref级别

## 5.1.system

> const类型的一种特殊场景，查询的表只有一行记录的情况，并且该表使用的存储引擎的统计数据是精确的

```sql
drop table explain_type;
create table explain_type(i int) engine=InnoDB;
insert into explain_type values(1);
explain select * from explain_type;


drop table explain_type;
create table explain_type(i int) engine=memory;
insert into explain_type values(1);
explain select * from explain_type;
```



> InnoDB存储引擎的统计数据不是精确的，虽然只有一条数据但是type类型为ALL；
>
> Memmory存储引擎的统计数据是精确的，所以当只有一条记录的时候type类型为system

## 5.2.CONST

> 基于**主键**或**唯一索引**查看一行，当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问转换常量查询，效率高

```sql
explain
select * from order where id = 1;
```

## 5.3.eq_ref

> 基于主键或唯一索引连接两个表，对于每个索引键值，只有一条匹配记录，被驱动表的类型为'eq_ref'

```sql
explain
select * from metoo_vlan inner join metoo_domain on metoo_vlan.id = metoo_domain.id
```

## 5.4.ref

> 基于非唯一索引连接两个表或通过**二级索引列**与常量进行等值匹配，可能会存在多条匹配记录

1，关联查询，使用非唯一索引进行匹配

```sql
explain
select * from metoo_domain inner join metoo_vlan on metoo_domain.id = metoo_vlan.domain_id 
```

2，简单查询，使用二级索引匹配

```sql
explain
select * from metoo_vlan where domain_id = 9
```

## 5.5.range

> 使用非唯一索引扫描部分索引，比如使用索引获取某些范围区间的记录

```sql
explain
select * from metoo_domain where id >= 9

explain
select * from metoo_vlan where domain_id >= 9
```

## 5.6.index

> 扫描整个索引就能拿到结果，一般是二级索引，这种查询一般为使用覆盖索引（需优化，缩小数据范围）

```sql
explain
select user_id from orders
```

## 5.7.all

> 扫描整个表进行匹配，即扫描聚簇索引树（需优化，添加索引优化）

```sql
explain
select * from metoo_vlan


explain
select min(id) from metoo_vlan
```

# 6.possible_key

> 表示在查询中可能使用到某个索引或多个索引；如果没有选择索引，显示NULL

# 7.key

> 表示在查询中实际使用的索引，如果没有使用索引，显示NULL

# 8.key_len

> 表示当优化器决定使用某个索引执行查询时，该索引记录的最大值（主要使用在联合索引）

> 联合索引可以通过这个值算出具体使用了索引中的哪些列

1. 使用单列索引

   ```sql
   explain
   select * from metoo_domain where id = 9
   ```

2. 使用联合索引

```sql
CREATE INDEX index_username_sex ON metoo_user (username, sex);

explain
select * from metoo_user where  username = 'a' and sex = 1
```

计算规则

> - 字符串
>   - char(n)：n个字节
>   - varchar(n)：如果是utf-8：3n+2字节，加的两个字节存储字符串长度。如果是utf8mb4：4n + 2字节
> - 数值类型
>   - tinyint：1字节
>   - smaillint：2字节
>   - int：4字节
>   - bigint：8字节
> - 时间类型
>   - date：3字节
>   - timestamp：4字节
>   - datetime：8字节
>
> 字段如果为NULL，需要1个字节记录是否为NULL

# 9.ref

> 表示将哪个字段或常量和key列所使用的字段进行比较



> 当使用索引列等值查询时，与索引列进行等值匹配的对象信息

1. 常量

   ```sql
   explain
   select * from metoo_user where  username = 'a' and sex = 1
   ```

   

2. 字段

```sql
explain
select * from metoo_domain inner join metoo_vlan on metoo_domain.id = metoo_vlan.id 

explain
select * from metoo_domain inner join metoo_vlan on metoo_domain.id = trim(metoo_vlan.id)
```



# 10.rows

> 全表扫描时表示需要扫描的行数估计值；
>
> 索引扫描时表示扫描索引的行数估计值；
>
> 值越小越好（不是结果集中的行数）

# 11.filtered

> 表示符合查询条件的数据百分比。可以使用rows * filtered/100 计算出与exlain前一个表进行连接的行数

# 12.Extra

> SQL执行的额外信息

## 12.1.Using Index

> 使用非主键索引树就可以查询所需要的数据。一般是覆盖索引，即查询列都包含在辅助索引树叶子节点中，不需要回表查询

```
explain
select * from 
```

## 12.2.Using where

> 不通过索引查询所需要的数据

```sql
explain
select id from metoo_vlan where domain_id = 8 and name = 'aa'
# name为非索引列
```

## 12.3.Using index condition

> 表示查询列不被索引覆盖，where条件中是是一个索引范围查找，过滤完索引后回表找到所有符合条件的数据行

```sql
explain
select id from metoo_vlan where domain_id = 8 
```

## 12.4.Using temporary

> 表示需要使用临时表来处理查询

1. age列无索引，需要创建一张临时表进行去重

   ```sql
   explain
   select DISTINCT age from metoo_user;
   ```

   

2. username列有联合索引

   ```sql
   explain
   select DISTINCT username from metoo_user;
   ```

# 12.5.Using filesort

> 当查询中包含order by操作而无法利用索引完成的排序操作，数据较少时从内存排序，如果数据较多需要在磁盘中排序。需优化成索引排序

1. age列无索引，无法通过索引进行排序。需要先保存age与对应的主键id，然后在排序age查找数据

   ```sql
   explain
   select age from metoo_user order by age
   ```

2. username列有索引，因索引已经是排好序的所以直接读取就可以了

   ```sql
   explain
   select username from metoo_user order by username
   ```

## 12.6.Select tables optimized away

> 使用某些聚合函数（min，max）来访问某个索引值

```sql
explain
select min(id) from metoo_user

explain
select min(password) from metoo_user

explain
select min(username) from metoo_user
```

