## 执行计划输出中各列详解

### Extra

顾名思义，`Extra`列是用来说明一些额外信息的，我们可以通过这些额外信息来更准确的理解`MySQL`到底将如何执行给定的查询语句。`MySQL`提供的额外信息有好几十个，我们就不一个一个介绍了（都介绍了感觉我们的文章就跟文档差不多了～），所以我们只挑一些平时常见的或者比较重要的额外信息介绍给大家哈。



#### No tables used

当查询语句的没有`FROM`子句时将会提示该额外信息，比如：

![1567343811125](assets/explain 解析(下)/1567343811125.png)

#### Impossible WHERE

查询语句的`WHERE`子句永远为`FALSE`时将会提示该额外信息，比方说：

![1567343845085](assets/explain 解析(下)/1567343845085.png)

#### No matching min/max row

当查询列表处有`MIN`或者`MAX`聚集函数，但是并没有符合`WHERE`子句中的搜索条件的记录时，将会提示该额外信息，比方说：

![1567343874226](assets/explain 解析(下)/1567343874226.png)



#### Using index

当我们的查询列表以及搜索条件中只包含属于某个索引的列，也就是在可以使用索引覆盖的情况下，在`Extra`列将会提示该额外信息。比方说下边这个查询中只需要用到`idx_key1`而不需要回表操作：

![1567343898977](assets/explain 解析(下)/1567343898977.png)



#### Using index condition

有些搜索条件中虽然出现了索引列，但却不能使用到索引，比如下边这个查询：

```
SELECT * FROM s1 WHERE key1 > 'z' AND key1 LIKE '%a';
```

其中的`key1 > 'z'`可以使用到索引，但是`key1 LIKE '%a'`却无法使用到索引，在以前版本的`MySQL`中，是按照下边步骤来执行这个查询的：

- 先根据`key1 > 'z'`这个条件，从二级索引`idx_key1`中获取到对应的二级索引记录。
- 根据上一步骤得到的二级索引记录中的主键值进行回表，找到完整的用户记录再检测该记录是否符合`key1 LIKE '%a'`这个条件，将符合条件的记录加入到最后的结果集。

但是虽然`key1 LIKE '%a'`不能组成范围区间参与`range`访问方法的执行，但这个条件毕竟只涉及到了`key1`列，所以设计`MySQL`的大叔把上边的步骤改进了一下：

- 先根据`key1 > 'z'`这个条件，定位到二级索引`idx_key1`中对应的二级索引记录。
- 对于指定的二级索引记录，先不着急回表，而是先检测一下该记录是否满足`key1 LIKE '%a'`这个条件，如果这个条件不满足，则该二级索引记录压根儿就没必要回表。
- 对于满足`key1 LIKE '%a'`这个条件的二级索引记录执行回表操作。

我们说回表操作其实是一个随机`IO`，比较耗时，所以上述修改虽然只改进了一点点，但是可以省去好多回表操作的成本。设计`MySQL`的大叔们把他们的这个改进称之为`索引条件下推`（英文名：`Index Condition Pushdown`）。

如果在查询语句的执行过程中将要使用`索引条件下推`这个特性，在`Extra`列中将会显示`Using index condition`，比如这样：

![1567343967561](assets/explain 解析(下)/1567343967561.png)

#### Using where

当我们使用全表扫描来执行对某个表的查询，并且该语句的`WHERE`子句中有针对该表的搜索条件时，在`Extra`列中会提示上述额外信息。比如下边这个查询：



![1567343998846](assets/explain 解析(下)/1567343998846.png)

当使用索引访问来执行对某个表的查询，并且该语句的`WHERE`子句中有除了该索引包含的列之外的其他搜索条件时，在`Extra`列中也会提示上述额外信息。比如下边这个查询虽然使用`idx_key1`索引执行查询，但是搜索条件中除了包含`key1`的搜索条件`key1 = 'a'`，还有包含`common_field`的搜索条件，所以`Extra`列会显示`Using where`的提示：



![1567344021786](assets/explain 解析(下)/1567344021786.png)





#### Using join buffer (Block Nested Loop)

在连接查询执行过程过，当被驱动表不能有效的利用索引加快访问速度，`MySQL`一般会为其分配一块名叫`join buffer`的内存块来加快查询速度，也就是我们所讲的`基于块的嵌套循环算法`，比如下边这个查询语句：

![1567344050802](assets/explain 解析(下)/1567344050802.png)

可以在对`s2`表的执行计划的`Extra`列显示了两个提示：

- `Using join buffer (Block Nested Loop)`：这是因为对表`s2`的访问不能有效利用索引，只好退而求其次，使用`join buffer`来减少对`s2`表的访问次数，从而提高性能。
- `Using where`：可以看到查询语句中有一个`s1.common_field = s2.common_field`条件，因为`s1`是驱动表，`s2`是被驱动表，所以在访问`s2`表时，`s1.common_field`的值已经确定下来了，所以实际上查询`s2`表的条件就是`s2.common_field = 一个常数`，所以提示了`Using where`额外信息。





待补充





















## Json格式的执行计划

我们上边介绍的`EXPLAIN`语句输出中缺少了一个衡量执行计划好坏的重要属性 —— 成本。不过设计`MySQL`的大叔贴心的为我们提供了一种查看某个执行计划花费的成本的方式：

- 在`EXPLAIN`单词和真正的查询语句中间加上`FORMAT=JSON`。

这样我们就可以得到一个`json`格式的执行计划，里边儿包含该计划花费的成本，比如这样：

```
buchongmysql> EXPLAIN FORMAT=JSON SELECT * FROM s1 INNER JOIN s2 ON s1.key1 = s2.key2 WHERE s1.common_field = 'a'\G
*************************** 1. row ***************************

EXPLAIN: {
  "query_block": {
    "select_id": 1,     # 整个查询语句只有1个SELECT关键字，该关键字对应的id号为1
    "cost_info": {
      "query_cost": "3197.16"   # 整个查询的执行成本预计为3197.16
    },
    "nested_loop": [    # 几个表之间采用嵌套循环连接算法执行
    
    # 以下是参与嵌套循环连接算法的各个表的信息
      {
        "table": {
          "table_name": "s1",   # s1表是驱动表
          "access_type": "ALL",     # 访问方法为ALL，意味着使用全表扫描访问
          "possible_keys": [    # 可能使用的索引
            "idx_key1"
          ],
          "rows_examined_per_scan": 9688,   # 查询一次s1表大致需要扫描9688条记录
          "rows_produced_per_join": 968,    # 驱动表s1的扇出是968
          "filtered": "10.00",  # condition filtering代表的百分比
          "cost_info": {
            "read_cost": "1840.84",     # 稍后解释
            "eval_cost": "193.76",      # 稍后解释
            "prefix_cost": "2034.60",   # 单次查询s1表总共的成本
            "data_read_per_join": "1M"  # 读取的数据量
          },
          "used_columns": [     # 执行查询中涉及到的列
            "id",
            "key1",
            "key2",
            "key3",
            "key_part1",
            "key_part2",
            "key_part3",
            "common_field"
          ],
          
          # 对s1表访问时针对单表查询的条件
          "attached_condition": "((`xiaohaizi`.`s1`.`common_field` = 'a') and (`xiaohaizi`.`s1`.`key1` is not null))"
        }
      },
      {
        "table": {
          "table_name": "s2",   # s2表是被驱动表
          "access_type": "ref",     # 访问方法为ref，意味着使用索引等值匹配的方式访问
          "possible_keys": [    # 可能使用的索引
            "idx_key2"
          ],
          "key": "idx_key2",    # 实际使用的索引
          "used_key_parts": [   # 使用到的索引列
            "key2"
          ],
          "key_length": "5",    # key_len
          "ref": [      # 与key2列进行等值匹配的对象
            "xiaohaizi.s1.key1"
          ],
          "rows_examined_per_scan": 1,  # 查询一次s2表大致需要扫描1条记录
          "rows_produced_per_join": 968,    # 被驱动表s2的扇出是968（由于后边没有多余的表进行连接，所以这个值也没啥用）
          "filtered": "100.00",     # condition filtering代表的百分比
          
          # s2表使用索引进行查询的搜索条件
          "index_condition": "(`xiaohaizi`.`s1`.`key1` = `xiaohaizi`.`s2`.`key2`)",
          "cost_info": {
            "read_cost": "968.80",      # 稍后解释
            "eval_cost": "193.76",      # 稍后解释
            "prefix_cost": "3197.16",   # 单次查询s1、多次查询s2表总共的成本
            "data_read_per_join": "1M"  # 读取的数据量
          },
          "used_columns": [     # 执行查询中涉及到的列
            "id",
            "key1",
            "key2",
            "key3",
            "key_part1",
            "key_part2",
            "key_part3",
            "common_field"
          ]
        }
      }
    ]
  }
}
1 row in set, 2 warnings (0.00 sec)
```

我们使用`#`后边跟随注释的形式为大家解释了`EXPLAIN FORMAT=JSON`语句的输出内容，但是大家可能有疑问`"cost_info"`里边的成本看着怪怪的，它们是怎么计算出来的？先看`s1`表的`"cost_info"`部分：

```
"cost_info": {
    "read_cost": "1840.84",
    "eval_cost": "193.76",
    "prefix_cost": "2034.60",
    "data_read_per_join": "1M"
}
```

- `read_cost`是由下边这两部分组成的：

  - `IO`成本
  - 检测`rows × (1 - filter)`条记录的`CPU`成本

  > 小贴士： rows和filter都是我们前边介绍执行计划的输出列，在JSON格式的执行计划中，rows相当于rows_examined_per_scan，filtered名称不变。

- `eval_cost`是这样计算的：

  检测 `rows × filter`条记录的成本。

- `prefix_cost`就是单独查询`s1`表的成本，也就是：

  `read_cost + eval_cost`

- `data_read_per_join`表示在此次查询中需要读取的数据量，我们就不多唠叨这个了。

> 小贴士： 大家其实没必要关注MySQL为啥使用这么古怪的方式计算出read_cost和eval_cost，关注prefix_cost是查询s1表的成本就好了。

对于`s2`表的`"cost_info"`部分是这样的：

```
"cost_info": {
    "read_cost": "968.80",
    "eval_cost": "193.76",
    "prefix_cost": "3197.16",
    "data_read_per_join": "1M"
}
```

由于`s2`表是被驱动表，所以可能被读取多次，这里的`read_cost`和`eval_cost`是访问多次`s2`表后累加起来的值，大家主要关注里边儿的`prefix_cost`的值代表的是整个连接查询预计的成本，也就是单次查询`s1`表和多次查询`s2`表后的成本的和，也就是：

```
968.80 + 193.76 + 2034.60 = 3197.16
```

## Extented EXPLAIN

最后，设计`MySQL`的大叔还为我们留了个彩蛋，在我们使用`EXPLAIN`语句查看了某个查询的执行计划后，紧接着还可以使用`SHOW WARNINGS`语句查看与这个查询的执行计划有关的一些扩展信息，比如这样：

![1567343731938](assets/explain 解析(下)/1567343731938.png)

大家可以看到`SHOW WARNINGS`展示出来的信息有三个字段，分别是`Level`、`Code`、`Message`。我们最常见的就是`Code`为`1003`的信息，当`Code`值为`1003`时，`Message`字段展示的信息类似于查询优化器将我们的查询语句重写后的语句。比如我们上边的查询本来是一个左（外）连接查询，但是有一个`s2.common_field IS NOT NULL`的条件，着就会导致查询优化器把左（外）连接查询优化为内连接查询，从`SHOW WARNINGS`的`Message`字段也可以看出来，原本的`LEFT JOIN`已经变成了`JOIN`。

但是大家一定要注意，我们说`Message`字段展示的信息类似于查询优化器将我们的查询语句重写后的语句，并不是等价于，也就是说`Message`字段展示的信息并不是标准的查询语句，在很多情况下并不能直接拿到黑框框中运行，它只能作为帮助我们理解查`MySQL`将如何执行查询语句的一个参考依据而已。