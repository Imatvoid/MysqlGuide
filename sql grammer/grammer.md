# grammer



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

#### DROP TABLE 

```sql
--DROP TABLE 语句用于删除表（表的结构、属性以及索引也会被删除）,表的所有数据被删除
DROP TABLE 表名称
```

#### TRUNCATE TABLE

```sql
--当表被TRUNCATE后，这个表和索引所占用的空间会恢复到初始大小,主键记录也会归零。
--DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。 
TRUNCATE TABLE 表名称
```

TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句。 



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

### insert

```sql
INSERT INTO 表名称 VALUES (值1, 值2,....)
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
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

## 表连接







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