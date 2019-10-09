## 数据库/数据表操作

### 建立数据库(CREATE DATABASE)

```mysql
DROP DATABASE  IF EXISTS sampledb;
CREATE DATABASE sampledb DEFAULT CHARACTER SET utf8;
USE sampledb;
```

### 建表(CREATE TABLE )

```mysql
DROP TABLE  IF EXISTS t_user;
CREATE TABLE t_user(
user_id INT auto_increment PRIMARY KEY,
user_name VARCHAR(30),
credits INT,
password VARCHAR(32),
last_visit datetime,
last_ip VARCHAR(23)
)ENGINE = INNODB
```

### 删除表

drop > truncate > delete

#### DROP TABLE (全部删除)

删除表全部数据和表结构（属性以及索引也会被删除）,表的所有数据被删除,立刻释放磁盘空间,不管是 Innodb 和 MyISAM;

```mysql
DROP TABLE 表名称
```

#### TRUNCATE TABLE(结构归零)

删除表全部数据，保留表结构，立刻释放磁盘空间 ，不管是 Innodb 和 MyISAM;

通过释放存储表数据所用的数据页来删除数据，并且**只在事务日志中记录页的释放**。

```sql
TRUNCATE TABLE 表名称
```

TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。索引所占用的空间会恢复到初始大小,新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句。 

#### DELETE(删除数据插事务)

DELETE 语句每次删除一行，并在**事务日志中为所删除的每行记录一项**。

```mysql
delete from 表名称;
```

对于 MyISAM 会立刻释放磁盘空间，InnoDB 不会释放磁盘空间;如果带where条件,那磁盘空间都不会释放,留着以后插数据用.

> optimize table 表名称;



### 修改表

增加字段

```SQL
ALTER TABLE table_name ADD COLUMN column_name type  NOT NULL [FIRST | AFTER column_name]

ALTER TABLE
  so_pack_supplies
ADD
  COLUMN is_open_supplies tinyint(4) DEFAULT NULL COMMENT '是否开放耗材(0:不是 1:是)'
AFTER
  supplies_name,
ADD
  COLUMN supplies_source varchar(100) DEFAULT NULL COMMENT '耗材来源'
AFTER
  supplies_name,
```

添加索引

```mysql
# 普通索引
ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) 
```



## CURD

### select

![1549985549867](../sql%20grammer/assets/1549985549867.png)

selects从表中选择数据。`distinct`过滤掉重复的结果，而`all`（默认）包含所有结果。

`from`标识符标识正在查询的表（当前只有一个表—还没有联接或派生表）

未在创建时声明的动态列可以在表名后面的括号中定义，然后在查询中使用。

`where` 选定筛选条件

`group by` 根据给定的内容讲查询分组，`HAVING`在group后筛选.

`order by`按给定的列或表达式对结果进行排序.

`limit`子句在`order by`子句之后执行，以支持`TOPN`类型查询。`hint`可选提示将覆盖默认查询计划。

#### 执行顺序

```

(1)     FROM <left_table>
(2)     ON <join_condition>
(3)     <join_type> JOIN <right_table>
(4)     WHERE <where_condition>
(5)     GROUP BY <group_by_list>
(6)     HAVING <having_condition>
(7)     SELECT 
(8)     DISTINCT <select_list>
(9)     ORDER BY <order_by_condition> 这里的condition可以不在select_list出现
(10)    LIMIT <limit_number>
```

[参考](https://blog.csdn.net/qq_26442553/article/details/79467243)

#### distinct

作用于单列根据单列去重,作用于多列根据多列字段拼接后去重.

如果有distinct,必须放在开头.

其他用法:

```mysql
select count(distinct name) from A;	  --表中name去重后的数目， SQL Server支持，而Access不支持
```

[参考](https://www.cnblogs.com/rainman/archive/2013/05/03/3058451.html)

#### group by

“Group By”从字面意义上理解就是根据“By”指定的规则对数据进行分组，所谓的分组就是将一个“数据集”划分成若干个“小区域”，然后针对若干个“小区域”进行数据处理。

1.简单group by

```mysql
select 类别, sum(数量) as 数量之和
from A
group by 类别
```

2.Group By 和 Order By

可以使用group的属性(这里是sum(数量)),对分组进行排序

```mysql
select 类别, sum(数量) AS 数量之和
from A
group by 类别
order by sum(数量) desc
```

3.Group By中Select指定的字段限制

```mysql
select 类别, sum(数量) as 数量之和, 摘要
from A
group by 类别
order by 类别 desc
```

示例3执行后会提示错误,这就是需要注意的一点，**在select指定的字段要么就要包含在Group By语句的后面，作为分组的依据；要么就要被包含在聚合函数中。**

4.Group By与聚合函数

| 函数        | 作用         | 支持性               |
| ----------- | ------------ | -------------------- |
| sum(列名)   | 求和         |                      |
| max(列名)   | 最大值       |                      |
| min(列名)   | 最小值       |                      |
| avg(列名)   | 平均值       |                      |
| first(列名) | 第一条记录   | 仅Access支持         |
| last(列名)  | 最后一条记录 | 仅Access支持         |
| count(列名) | 统计记录数   | 注意和count(*)的区别 |

示例5：求各组平均值

```
select 类别, avg(数量) AS 平均值 from A group by 类别;
```

 示例6：求各组记录数目

```
select 类别, count(*) AS 记录数 from A group by 类别;
```



[参考](https://www.cnblogs.com/rainman/archive/2013/05/01/3053703.html)



####  having

> having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数，使用having 条件**过滤出特定的组**，也可以使用多个分组标准进行分组。

#### 

#### UNION 操作符

> MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。 保留重复可以使用all

```mysql
select posts_id,posts_name,posts_status,'users' as table_name from yy_posts
UNION
select user_id,user_nickname,user_status,'posts' as table_name from yy_user
```

只要相应的字段类型一样就行.

使用order by 必须放在最后,且在属性名在第一条语句中出现.(列的名字都以第一条select语句出现的为准)

[参考](https://segmentfault.com/a/1190000007926959)

#### JOIN(表连接)

下面这个写的真挺好的,没必要再写了

[参考](http://wxb.github.io/2016/12/15/MySQL中的各种join.html)

### insert

```sql
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
INSERT INTO 表名称 VALUES (值1, 值2,....)
```

### update

```sql
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```

### delete(这种删除是比较慢但安全的)

```mysql
# 可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的,索引大小也不会变化，一行一行删除，会写事务日志，方便回滚。
DELETE FROM 表名称 WHERE 列名称 = 值
DELETE FROM table_name
```



### 



##  MySQL NULL 值处理

我们已经知道 MySQL 使用 SQL SELECT 命令及 WHERE 子句来读取数据表中的数据,但是当提供的查询条件字段为 NULL 时，该命令可能就无法正常工作。

为了处理这种情况，MySQL提供了三大运算符:

- **IS NULL:** 当列的值是 NULL,此运算符返回 true。
- **IS NOT NULL:** 当列的值不为 NULL, 运算符返回 true。
- **<=>:** 比较操作符（不同于=运算符），当比较的的两个值为 NULL 时返回 true。

**关于 NULL 的条件比较运算是比较特殊的。你不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。**

在 MySQL 中，NULL 值与任何其它值的比较（即使是 NULL）永远返回 false，即 NULL = NULL 返回false 。

MySQL 中处理 NULL 使用 IS NULL 和 IS NOT NULL 运算符。



## 运算符

|                 |                            |                                                              |
| --------------- | -------------------------- | ------------------------------------------------------------ |
| 符号            | 描述                       | 备注                                                         |
| =               | 等于                       |                                                              |
| <>, !=          | 不等于                     |                                                              |
| >               | 大于                       |                                                              |
| <               | 小于                       |                                                              |
| <=              | 小于等于                   |                                                              |
| >=              | 大于等于                   |                                                              |
| BETWEEN         | 在两值之间                 | >=min&&<=max                                                 |
| NOT BETWEEN     | 不在两值之间               |                                                              |
| IN              | 在集合中                   |                                                              |
| NOT IN          | 不在集合中                 |                                                              |
| <=>             | 严格比较两个NULL值是否相等 | 两个操作码均为NULL时，其所得值为1；而当一个操作码为NULL时，其所得值为0 |
| LIKE            | 模糊匹配                   |                                                              |
| REGEXP 或 RLIKE | 正则式匹配                 |                                                              |
| IS NULL         | 为空                       |                                                              |
| IS NOT NULL     | 不为空                     |                                                              |

[参考](https://www.runoob.com/mysql/mysql-operator.html)



## 索引

### Explain

```mysql
| id | select_type | table | type  | possible_keys | key  | key_len | ref  | rows | Extra |

#id 包含一组数字，表示查询中执行select子句或操作表的顺序（id相同为同组，执行顺序由上至下） 不同组越大先执行

#select_type 表示查询中每个select子句的类型（简单OR复杂）

#type 表示MySQL在表中找到所需行的方式，又称“访问类型”，常见类型如下:
#ALL, index,  range, ref, eq_ref, const, system, NULL
#ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行
#index：Full Index Scan，index与ALL区别为index类型只遍历索引树
#range:索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行。显而易见的索引范围扫描是带有between或者where子句里带有<, >查询。当mysql使用索引去查找一系列值时，例如IN()和OR列表，也会显示range（范围扫描）,当然性能上面是有差异的。
#ref：使用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行
#eq_ref：类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
#const、system：当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量
#NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。



```

<https://www.cnblogs.com/gomysql/p/3720123.html>
