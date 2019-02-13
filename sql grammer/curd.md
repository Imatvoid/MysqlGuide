## CURD

https://forcedotcom.github.io/phoenix/

http://www.w3school.com.cn/sql/sql_delete.asp

### select

![1549985549867](assets/1549985549867.png)

Selects data from a table. `DISTINCT` filters out duplicate results while `ALL`, the default, includes all results. `FROM` identifies the table being queried (single table only currently - no joins or derived tables yet). Dynamic columns not declared at create time may be defined in parenthesis after the table name and then used in the query. `GROUP BY` groups the the result by the given expression(s). `HAVING`filter rows after grouping. `ORDER BY` sorts the result by the given column(s) or expression(s) and is only allowed for aggregate queries or queries with a `LIMIT` clause. `LIMIT` limits the number of rows returned by the query with no limit applied if specified as null or less than zero. The `LIMIT` clause is executed after the `ORDER BY` clause to support TopN type queries. An optional hint overrides the default query plan.

selects从表中选择数据。`distinct`过滤掉重复的结果，而`all`（默认）包含所有结果。

`from`标识符标识正在查询的表（当前只有一个表—还没有联接或派生表）

未在创建时声明的动态列可以在表名后面的括号中定义，然后在查询中使用。

`group by` 根据给定的内容讲查询分组，`HAVING`在group后筛选.

`order by`按给定的列或表达式对结果进行排序，并且仅允许聚合查询或带有LIMIT子句的查询。

`limit`子句在`order by`子句之后执行，以支持`TOPN`类型查询。可选提示将覆盖默认查询计划。

### insert



### update



### delete