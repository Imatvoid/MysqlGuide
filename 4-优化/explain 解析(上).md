一条查询语句在经过`MySQL`查询优化器的各种基于成本和规则的优化会后生成一个所谓的`执行计划`，这个执行计划展示了接下来具体执行查询的方式，比如多表连接的顺序是什么，对于每个表采用什么访问方法来具体执行查询等等。



如果我们想看看某个查询的执行计划的话，可以在具体的查询语句前边加一个`EXPLAIN`，就像这样：

![1567341256080](assets/explain 解析/1567341256080.png)



## 概览

|      列名       | 描述                                                       |
| :-------------: | :--------------------------------------------------------- |
|      `id`       | 在一个大的查询语句中每个`SELECT`关键字都对应一个唯一的`id` |
|  `select_type`  | `SELECT`关键字对应的那个查询的类型                         |
|     `table`     | 表名                                                       |
|  `partitions`   | 匹配的分区信息                                             |
|     `type`      | 针对单表的访问方法                                         |
| `possible_keys` | 可能用到的索引                                             |
|      `key`      | 实际上使用的索引                                           |
|    `key_len`    | 实际使用到的索引长度                                       |
|      `ref`      | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息     |
|     `rows`      | 预估的需要读取的记录条数                                   |
|   `filtered`    | 某个表经过搜索条件过滤后剩余记录条数的百分比               |
|     `Extra`     | 一些额外的信息                                             |





```mysql
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```



## 执行计划输出中各列详解

### table

不论我们的查询语句有多复杂，里边儿包含了多少个表，到最后也是需要对每个表进行单表访问的，所以设计`MySQL`的大叔规定<span style="color:red">EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名</span>。所以我们看一条比较简单的查询语句：

![1567341444387](assets/explain 解析/1567341444387.png)

这个查询语句只涉及对`s1`表的单表查询，所以`EXPLAIN`输出中只有一条记录，其中的`table`列的值是`s1`，表明这条记录是用来说明对`s1`表的单表访问方法的。

下边我们看一下一个连接查询的执行计划：

![1567341466811](assets/explain 解析/1567341466811.png)

可以看到这个连接查询的执行计划中有两条记录，这两条记录的`table`列分别是`s1`和`s2`，这两条记录用来分别说明对`s1`表和`s2`表的访问方法是什么。





### id

我们知道我们写的查询语句一般都以`SELECT`关键字开头，比较简单的查询语句里只有一个`SELECT`关键字，比如下边这个查询语句：

```mysql
SELECT * FROM s1 WHERE key1 = 'a';
```

稍微复杂一点的连接查询中也只有一个`SELECT`关键字，比如：

```mysql
SELECT * FROM s1 INNER JOIN s2
    ON s1.key1 = s2.key1
    WHERE s1.common_field = 'a';
```

但是下边两种情况下在一条查询语句中会出现多个`SELECT`关键字：

- 查询中包含子查询的情况

  比如下边这个查询语句中就包含2个`SELECT`关键字：

  ```mysql
  SELECT * FROM s1 
      WHERE key1 IN (SELECT * FROM s2);
  ```

- 查询中包含`UNION`语句的情况

  比如下边这个查询语句中也包含2个`SELECT`关键字：

  ```mysql
  SELECT * FROM s1  UNION SELECT * FROM s2;
  ```

查询语句中每出现一个`SELECT`关键字，设计`MySQL`的大叔就会为它分配一个唯一的`id`值。这个`id`值就是`EXPLAIN`语句的第一个列，比如下边这个查询中只有一个`SELECT`关键字，所以`EXPLAIN`的结果中也就只有一条`id`列为`1`的记录：

![1567341608001](assets/explain 解析/1567341608001.png)

对于连接查询来说，一个`SELECT`关键字后边的`FROM`子句中可以跟随多个表，所以在连接查询的执行计划中，<span style="color:red">每个表都会对应一条记录，但是这些记录的id值都是相同的</span>，比如：

![1567341644492](assets/explain 解析/1567341644492.png)

可以看到，上述连接查询中参与连接的`s1`和`s2`表分别对应一条记录，但是这两条记录对应的`id`值都是`1`。这里需要大家记住的是，<span style="color:red">在连接查询的执行计划中，每个表都会对应一条记录，这些记录的id列的值是相同的，出现在前边的表表示驱动表，出现在后边的表表示被驱动表</span>。所以从上边的`EXPLAIN`输出中我们可以看出，查询优化器准备让`s1`表作为驱动表，让`s2`表作为被驱动表来执行查询。

对于包含子查询的查询语句来说，就可能涉及多个`SELECT`关键字，所以在包含子查询的查询语句的执行计划中，每个`SELECT`关键字都会对应一个唯一的`id`值，比如这样：

![1567341695511](assets/explain 解析/1567341695511.png)

从输出结果中我们可以看到，`s1`表在外层查询中，外层查询有一个独立的`SELECT`关键字，所以第一条记录的`id`值就是`1`，`s2`表在子查询中，子查询有一个独立的`SELECT`关键字，所以第二条记录的`id`值就是`2`。

但是这里大家需要特别注意，<span style="color:red">查询优化器可能对涉及子查询的查询语句进行重写，从而转换为连接查询</span>。所以如果我们想知道查询优化器对某个包含子查询的语句是否进行了重写，直接查看执行计划就好了，比如说：

![1567341730213](assets/explain 解析/1567341730213.png)

可以看到，虽然我们的查询语句是一个子查询，但是执行计划中`s1`和`s2`表对应的记录的`id`值全部是`1`，这就表明了<span style="color:red">查询优化器将子查询转换为了连接查询</span>。

对于包含`UNION`子句的查询语句来说，每个`SELECT`关键字对应一个`id`值也是没错的，不过还是有点儿特别的东西，比方说下边这个查询：

![1567341779525](assets/explain 解析/1567341779525.png)

这个语句的执行计划的第三条记录是个什么鬼？为毛`id`值是`NULL`，而且`table`列长的也怪怪的？大家别忘了`UNION`子句是干嘛用的，它会把多个查询的结果集合并起来并对结果集中的记录进行去重，怎么去重呢？`MySQL`使用的是内部的临时表。正如上边的查询计划中所示，`UNION`子句是为了把`id`为`1`的查询和`id`为`2`的查询的结果集合并起来并去重，所以在内部创建了一个名为`<union1, 2>`的临时表（就是执行计划第三条记录的`table`列的名称），`id`为`NULL`表明这个临时表是为了合并两个查询的结果集而创建的。

跟`UNION`对比起来，`UNION ALL`就不需要为最终的结果集进行去重，它只是单纯的把多个查询的结果集中的记录合并成一个并返回给用户，所以也就不需要使用临时表。所以在包含`UNION ALL`子句的查询的执行计划中，就没有那个`id`为`NULL`的记录，如下所示：

![1567341791403](assets/explain 解析/1567341791403.png)

### select_type

通过上边的内容我们知道，一条大的查询语句里边可以包含若干个`SELECT`关键字，每个`SELECT`关键字代表着一个小的查询语句，而每个`SELECT`关键字的`FROM`子句中都可以包含若干张表（这些表用来做连接查询），每一张表都对应着执行计划输出中的一条记录，对于在同一个`SELECT`关键字中的表来说，它们的`id`值是相同的。

设计`MySQL`的大叔为每一个`SELECT`关键字代表的小查询都定义了一个称之为`select_type`的属性，意思是我们只要知道了某个小查询的`select_type`属性，就知道了这个小查询在整个大查询中扮演了一个什么角色，口说无凭，我们还是先来见识见识这个`select_type`都能取哪些值（为了精确起见，我们直接使用文档中的英文做简要描述，随后会进行详细解释的）：



|          名称          | 描述                                                         |
| :--------------------: | :----------------------------------------------------------- |
|        `SIMPLE`        | Simple SELECT (not using UNION or subqueries)                |
|       `PRIMARY`        | Outermost SELECT                                             |
|        `UNION`         | Second or later SELECT statement in a UNION                  |
|     `UNION RESULT`     | Result of a UNION                                            |
|       `SUBQUERY`       | First SELECT in subquery                                     |
|  `DEPENDENT SUBQUERY`  | First SELECT in subquery, dependent on outer query           |
|   `DEPENDENT UNION`    | Second or later SELECT statement in a UNION, dependent on outer query |
|       `DERIVED`        | Derived table                                                |
|     `MATERIALIZED`     | Materialized subquery                                        |
| `UNCACHEABLE SUBQUERY` | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
|  `UNCACHEABLE UNION`   | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |



#### SIMPLE

查询语句中不包含`UNION`或者子查询的查询都算作是`SIMPLE`类型，比方说下边这个单表查询的`select_type`的值就是`SIMPLE`：

![1567341949596](assets/explain 解析/1567341949596.png)

当然，连接查询也算是`SIMPLE`类型，比如：

![1567341963508](assets/explain 解析/1567341963508.png)

#### PRIMARY

对于包含`UNION`、`UNION ALL`或者子查询的大查询来说，它是由几个小查询组成的，其中最左边的那个查询的`select_type`值就是`PRIMARY`，比方说：

![1567341995877](assets/explain 解析/1567341995877.png)

从结果中可以看到，最左边的小查询`SELECT * FROM s1`对应的是执行计划中的第一条记录，它的`select_type`值就是`PRIMARY`。



#### UNION

- 对于包含`UNION`或者`UNION ALL`的大查询来说，它是由几个小查询组成的，其中除了最左边的那个小查询以外，其余的小查询的`select_type`值就是`UNION`，可以对比上一个例子的效果，这就不多举例子了。



#### UNION RESULT

`MySQL`选择使用临时表来完成`UNION`查询的去重工作，针对该临时表的查询的`select_type`就是`UNION RESULT`，例子上边有，就不赘述了。



#### SUBQUERY

如果包含子查询的查询语句不能够转为对应的`semi-join`的形式，并且该子查询是不相关子查询，并且查询优化器决定采用将该子查询物化的方案来执行该子查询时，该子查询的第一个`SELECT`关键字代表的那个查询的`select_type`就是`SUBQUERY`，比如下边这个查询：

![1567342128554](assets/explain 解析/1567342128554.png)

可以看到，外层查询的`select_type`就是`PRIMARY`，子查询的`select_type`就是`SUBQUERY`。需要大家注意的是，由于select_type为SUBQUERY的子查询由于会被物化，所以只需要执行一遍。



#### DEPENDENT SUBQUERY

如果包含子查询的查询语句不能够转为对应的`semi-join`的形式，并且该子查询是相关子查询，则该子查询的第一个`SELECT`关键字代表的那个查询的`select_type`就是`DEPENDENT SUBQUERY`，比如下边这个查询：

![1567342170741](assets/explain 解析/1567342170741.png)

需要大家注意的是，select_type为DEPENDENT SUBQUERY的查询可能会被执行多次。

#### DEPENDENT UNION

在包含`UNION`或者`UNION ALL`的大查询中，如果各个小查询都依赖于外层查询的话，那除了最左边的那个小查询之外，其余的小查询的`select_type`的值就是`DEPENDENT UNION`。说的有些绕哈，比方说下边这个查询：

![1567342227410](assets/explain 解析/1567342227410.png)

这个查询比较复杂啊，大查询里包含了一个子查询，子查询里又是由`UNION`连起来的两个小查询。从执行计划中可以看出来，`SELECT key1 FROM s2 WHERE key1 = 'a'`这个小查询由于是子查询中第一个查询，所以它的`select_type`是`DEPENDENT SUBQUERY`，而`SELECT key1 FROM s1 WHERE key1 = 'b'`这个查询的`select_type`就是`DEPENDENT UNION`。

#### DERIVED

对于采用物化的方式执行的包含派生表的查询，该派生表对应的子查询的`select_type`就是`DERIVED`，比方说下边这个查询：

![1567342285201](assets/explain 解析/1567342285201.png)

从执行计划中可以看出，`id`为`2`的记录就代表子查询的执行方式，它的`select_type`是`DERIVED`，说明该子查询是以物化的方式执行的。`id`为`1`的记录代表外层查询，大家注意看它的`table`列显示的是`<derived2>`，表示该查询是针对将派生表物化之后的表进行查询的。

#### MATERIALIZED

当查询优化器在执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询对应的`select_type`属性就是`MATERIALIZED`，比如下边这个查询：

![1567342344722](assets/explain 解析/1567342344722.png)

执行计划的第三条记录的`id`值为`2`，说明该条记录对应的是一个单表查询，从它的`select_type`值为`MATERIALIZED`可以看出，查询优化器是要把子查询先转换成物化表。然后看执行计划的前两条记录的`id`值都为`1`，说明这两条记录对应的表进行连接查询，需要注意的是第二条记录的`table`列的值是`<subquery2>`，说明该表其实就是`id`为`2`对应的子查询执行之后产生的物化表，然后将`s1`和该物化表进行连接查询。





#### 其他

- `UNCACHEABLE SUBQUERY`

  不常用，就不多唠叨了。

- `UNCACHEABLE UNION`

  不常用，就不多唠叨了。



### partitions

由于我们压根儿就没唠叨过分区是个啥，所以这个输出列我们也就不说了哈，一般情况下我们的查询语句的执行计划的`partitions`列的值都是`NULL`。





### Type

我们前边说过执行计划的一条记录就代表着`MySQL`对某个表的执行查询时的访问方法，其中的`type`列就表明了这个访问方法是个啥，比方说下边这个查询：

![1567342402246](assets/explain 解析/1567342402246.png)

可以看到`type`列的值是`ref`，表明`MySQL`即将使用`ref`访问方法来执行对`s1`表的查询。但是我们之前只唠叨过对使用`InnoDB`存储引擎的表进行单表访问的一些访问方法，完整的访问方法如下：`system`，`const`，`eq_ref`，`ref`，`fulltext`，`ref_or_null`，`index_merge`，`unique_subquery`，`index_subquery`，`range`，`index`，`ALL`。当然我们还要详细唠叨一下哈：





#### system

当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如MyISAM、Memory，那么对该表的访问方法就是`system`。比方说我们新建一个`MyISAM`表，并为其插入一条记录：

```mysql
mysql> CREATE TABLE t(i int) Engine=MyISAM;
Query OK, 0 rows affected (0.05 sec)

mysql> INSERT INTO t VALUES(1);
Query OK, 1 row affected (0.01 sec)
```

#### const

这个我们前边唠叨过，就是当我们根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是`const`，比如：

![1567342476828](assets/explain 解析/1567342476828.png)



#### eq_ref

在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的（如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较），则对该被驱动表的访问方法就是`eq_ref`，比方说：

![1567342505630](assets/explain 解析/1567342505630.png)

从执行计划的结果中可以看出，`MySQL`打算将`s1`作为驱动表，`s2`作为被驱动表，重点关注`s2`的访问方法是`eq_ref`，表明在访问`s2`表的时候可以通过主键的等值匹配来进行访问。

#### ref

当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是`ref`，最开始举过例子了，就不重复举例了。

#### fulltext

全文索引

#### ref_or_null

当对普通二级索引进行等值匹配查询，该索引列的值也可以是`NULL`值时，那么对该表的访问方法就可能是`ref_or_null`，比如说：

![1567342576585](assets/explain 解析/1567342576585.png)



#### index_merge

一般情况下对于某个表的查询只能使用到一个索引，但我们唠叨单表访问方法时特意强调了在某些场景下可以使用`Intersection`、`Union`、`Sort-Union`这三种索引合并的方式来执行查询，忘掉的回去补一下哈，我们看一下执行计划中是怎么体现`MySQL`使用索引合并的方式来对某个表执行查询的：

![1567342612399](assets/explain 解析/1567342612399.png)

从执行计划的`type`列的值是`index_merge`就可以看出，`MySQL`打算使用索引合并的方式来执行对`s1`表的查询。



#### unique_subquery

类似于两表连接中被驱动表的`eq_ref`访问方法，`unique_subquery`是针对在一些包含`IN`子查询的查询语句中，如果查询优化器决定将`IN`子查询转换为`EXISTS`子查询，而且子查询可以使用到主键进行等值匹配的话，那么该子查询执行计划的`type`列的值就是`unique_subquery`，比如下边的这个查询语句：

![1567342641834](assets/explain 解析/1567342641834.png)

可以看到执行计划的第二条记录的`type`值就是`unique_subquery`，说明在执行子查询时会使用到`id`列的索引。

#### index_subquery

`index_subquery`与`unique_subquery`类似，只不过访问子查询中的表时使用的是普通的索引，比如这样：

![1567342666995](assets/explain 解析/1567342666995.png)

#### range

如果使用索引获取某些`范围区间`的记录，那么就可能使用到`range`访问方法，比如下边的这个查询：

![1567342691854](assets/explain 解析/1567342691854.png)

或者

![1567342703779](assets/explain 解析/1567342703779.png)



#### index

当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是`index`，比如这样：

![1567342733943](assets/explain 解析/1567342733943.png)

上述查询中的搜索列表中只有`key_part2`一个列，而且搜索条件中也只有`key_part3`一个列，这两个列又恰好包含在`idx_key_part`这个索引中，可是搜索条件`key_part3`不能直接使用该索引进行`ref`或者`range`方式的访问，只能扫描整个`idx_key_part`索引的记录，所以查询计划的`type`列的值就是`index`。

> 再一次强调，对于使用InnoDB存储引擎的表来说，二级索引的记录只包含索引列和主键列的值，而聚簇索引中包含用户定义的全部列以及一些隐藏列，所以扫描二级索引的代价比直接全表扫描，也就是扫描聚簇索引的代价更低一些。



#### ALL

最熟悉的全表扫描，就不多唠叨了，直接看例子：

![1567342772814](assets/explain 解析/1567342772814.png)

一般来说，这些访问方法按照我们介绍它们的顺序性能依次变差。其中除了`All`这个访问方法外，其余的访问方法都能用到索引，除了`index_merge`访问方法外，其余的访问方法都最多只能用到一个索引。





### possible_keys和key

在`EXPLAIN`语句输出的执行计划中，`possible_keys`列表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些，`key`列表示实际用到的索引有哪些，比方说下边这个查询：

![1567342891229](assets/explain 解析/1567342891229.png)

上述执行计划的`possible_keys`列的值是`idx_key1,idx_key3`，表示该查询可能使用到`idx_key1,idx_key3`两个索引，然后`key`列的值是`idx_key3`，表示经过查询优化器计算使用不同索引的成本后，最后决定使用`idx_key3`来执行查询比较划算。

不过有一点比较特别，就是在使用`index`访问方法来查询某个表时，`possible_keys`列是空的，而`key`列展示的是实际使用到的索引，比如这样：

![1567342906603](assets/explain 解析/1567342906603.png)

另外需要注意的一点是，possible_keys列中的值并不是越多越好，可能使用的索引越多，查询优化器计算查询成本时就得花费更长时间，所以如果可以的话，尽量删除那些用不到的索引。





### key_len

`key_len`列表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度，它是由这三个部分构成的：

- 对于使用固定长度类型的索引列来说，它实际占用的存储空间的最大长度就是该固定值，对于指定字符集的变长类型的索引列来说，比如某个索引列的类型是`VARCHAR(100)`，使用的字符集是`utf8`，那么该列实际占用的最大存储空间就是`100 × 3 = 300`个字节。
- 如果该索引列可以存储`NULL`值，则`key_len`比不可以存储`NULL`值时多1个字节。
- 对于变长字段来说，都会有2个字节的空间来存储该变长列的实际长度。

比如下边这个查询：

![1567343004684](assets/explain 解析/1567343004684.png)

由于`id`列的类型是`INT`，并且不可以存储`NULL`值，所以在使用该列的索引时`key_len`大小就是`4`。当索引列可以存储`NULL`值时，比如：

![1567343047743](assets/explain 解析/1567343047743.png)

可以看到`key_len`列就变成了`5`，比使用`id`列的索引时多了`1`。



对于可变长度的索引列来说，比如下边这个查询：

![1567343072269](assets/explain 解析/1567343072269.png)

由于`key1`列的类型是`VARCHAR(100)`，所以该列实际最多占用的存储空间就是`300`字节，又因为该列允许存储`NULL`值，所以`key_len`需要加`1`，又因为该列是可变长度列，所以`key_len`需要加`2`，所以最后`ken_len`的值就是`303`。

有的同学可能有疑问：你在前边唠叨`InnoDB`行格式的时候不是说，存储变长字段的实际长度不是可能占用1个字节或者2个字节么？为什么现在不管三七二十一都用了`2`个字节？这里需要强调的一点是，执行计划的生成是在`MySQL server`层中的功能，并不是针对具体某个存储引擎的功能，设计`MySQL`的大叔在执行计划中输出`key_len`列主要是为了让我们区分某个使用联合索引的查询具体用了几个索引列，而不是为了准确的说明针对某个具体存储引擎存储变长字段的实际长度占用的空间到底是占用1个字节还是2个字节。比方说下边这个使用到联合索引`idx_key_part`的查询：

![1567343160405](assets/explain 解析/1567343160405.png)

我们可以从执行计划的`key_len`列中看到值是`303`，这意味着`MySQL`在执行上述查询中只能用到`idx_key_part`索引的一个索引列，而下边这个查询：

![1567343173551](assets/explain 解析/1567343173551.png)

这个查询的执行计划的`ken_len`列的值是`606`，说明执行这个查询的时候可以用到联合索引`idx_key_part`的两个索引列。



### ref

当使用索引列等值匹配的条件去执行查询时，也就是在访问方法是`const`、`eq_ref`、`ref`、`ref_or_null`、`unique_subquery`、`index_subquery`其中之一时，`ref`列展示的就是与索引列作等值匹配的东东是个啥，比如只是一个常数或者是某个列。大家看下边这个查询：


![1567343232002](assets/explain 解析/1567343232002.png)

可以看到`ref`列的值是`const`，表明在使用`idx_key1`索引执行查询时，与`key1`列作等值匹配的对象是一个常数，当然有时候更复杂一点：

![1567343276750](assets/explain 解析/1567343276750.png)

可以看到对被驱动表`s2`的访问方法是`eq_ref`，而对应的`ref`列的值是`xiaohaizi.s1.id`，这说明在对被驱动表进行访问时会用到`PRIMARY`索引，也就是聚簇索引与一个列进行等值匹配的条件，于`s2`表的`id`作等值匹配的对象就是`xiaohaizi.s1.id`列（注意这里把数据库名也写出来了）。

有的时候与索引列进行等值匹配的对象是一个函数，比方说下边这个查询：

![1567343326261](assets/explain 解析/1567343326261.png)

我们看执行计划的第二条记录，可以看到对`s2`表采用`ref`访问方法执行查询，然后在查询计划的`ref`列里输出的是`func`，说明与`s2`表的`key1`列进行等值匹配的对象是一个函数。

### rows

如果查询优化器决定使用全表扫描的方式对某个表执行查询时，执行计划的`rows`列就代表预计需要扫描的行数，如果使用索引来执行查询时，执行计划的`rows`列就代表预计扫描的索引记录行数。比如下边这个查询：

![1567343358959](assets/explain 解析/1567343358959.png)

我们看到执行计划的`rows`列的值是`266`，这意味着查询优化器在经过分析使用`idx_key1`进行查询的成本之后，觉得满足`key1 > 'z'`这个条件的记录只有`266`条。



### filtered

之前在分析连接查询的成本时提出过一个`condition filtering`的概念，就是`MySQL`在计算驱动表扇出时采用的一个策略：

- 如果使用的是全表扫描的方式执行的单表查询，那么计算驱动表扇出时需要估计出满足搜索条件的记录到底有多少条。
- 如果使用的是索引执行的单表扫描，那么计算驱动表扇出的时候需要估计出满足除使用到对应索引的搜索条件外的其他搜索条件的记录有多少条。

比方说下边这个查询：

![1567343416359](assets/explain 解析/1567343416359.png)

从执行计划的`key`列中可以看出来，该查询使用`idx_key1`索引来执行查询，从`rows`列可以看出满足`key1 > 'z'`的记录有`266`条。执行计划的`filtered`列就代表查询优化器预测在这`266`条记录中，有多少条记录满足其余的搜索条件，也就是`common_field = 'a'`这个条件的百分比。此处`filtered`列的值是`10.00`，说明查询优化器预测在`266`条记录中有`10.00%`的记录满足`common_field = 'a'`这个条件。

对于单表查询来说，这个`filtered`列的值没什么意义，我们更关注在连接查询中驱动表对应的执行计划记录的`filtered`值，比方说下边这个查询：

![1567343471334](assets/explain 解析/1567343471334.png)

从执行计划中可以看出来，查询优化器打算把`s1`当作驱动表，`s2`当作被驱动表。我们可以看到驱动表`s1`表的执行计划的`rows`列为`9688`， `filtered`列为`10.00`，这意味着驱动表`s1`的扇出值就是`9688 × 10.00% = 968.8`，这说明还要对被驱动表执行大约`968`次查询。