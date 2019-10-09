## UNIX里启动服务器程序

在类`UNIX`系统中用来启动`MySQL`服务器程序的可执行文件有很多，大多在`MySQL`安装目录的`bin`目录下

#### mysqld

`mysqld`这个可执行文件就代表着`MySQL`服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。

#### mysqld\_safe

`mysqld_safe`是一个启动脚本，它会间接的调用`mysqld`，而且还顺便启动了另外一个监控进程，这个监控进程在服务器进程挂了的时候，可以帮助重启它。另外，使用`mysqld_safe`启动服务器程序时，它会将服务器程序的出错信息和其他诊断信息重定向到某个文件中，产生出错日志，这样可以方便我们找出发生错误的原因。

## 启动MySQL客户端程序

```shell
mysql -h主机名  -u用户名 -p密码
```

`-h`

表示服务器进程所在计算机的域名或者IP地址，如果服务器进程就运行在本机的话，可以省略这个参数，或者填`localhost`或者`127.0.0.1`。也可以写作 `--host=主机名`的形式。

`-u`

表示用户名。也可以写作 `--user=用户名`的形式。

`-p`

表示密码。也可以写作 `--password=密码`的形式。

`-P`

端口号





## 启动选项和配置文件

### 在命令行上使用选项

不使用网络通信(TCP/IP)

```mysql
mysqld --skip_networking
```

改变默认存储引擎

```
mysqld --default-storage-engine=MyISAM
```

### 配置文件

可以在配置文件配置

my.cnf查找路径

```java
`/etc/my.cnf`

`/etc/mysql/my.cnf`

`SYSCONFDIR/my.cnf`

`$MYSQL_HOME/my.cnf`
```

配置重叠的话以命令行优先



## mysql系统变量

`MySQL`服务器程序运行过程中会用到许多影响程序行为的变量，它们被称为MySQL系统变量，比如允许同时连入的客户端数量用系统变量`max_connections`表示，表的默认存储引擎用系统变量`default_storage_engine`表示，查询缓存的大小用系统变量`query_cache_size`表示，

MySQL服务器程序的系统变量有好几百条，我们就不一一列举了。

**每个系统变量都有一个默认值，我们可以使用命令行或者配置文件中的选项在启动服务器时改变一些系统变量的值。大多数的系统变量的值也可以在程序运行过程中修改，而无需停止并重新启动它。**



### 查看系统变量

```
SHOW VARIABLES [LIKE 匹配的模式];
```



```mysql
mysql> SHOW VARIABLES LIKE 'default_storage_engine';
+------------------------+--------+
| Variable_name          | Value  |
+------------------------+--------+
| default_storage_engine | InnoDB |
+------------------------+--------+
1 row in set (0.01 sec)

mysql> SHOW VARIABLES like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.00 sec)

```

也可以加通配符`SHOW VARIABLES LIKE 'default%';`



### 设置系统变量

大部分的`系统变量`都可以通过启动服务器时传送启动选项的方式来进行设置

- 命令行

- 配置文件

  

**服务器程序运行过程中设置**

对于大部分系统变量来说，它们的值可以在服务器程序运行过程中进行动态修改而无需停止并重启服务器。不过系统变量有作用范围之分.



**设置不同作用范围的系统变量**

我们前边说过，多个客户端程序可以同时连接到一个服务器程序。对于同一个系统变量，我们有时想让不同的客户端有不同的值。比方说狗哥使用客户端A，他想让当前客户端对应的默认存储引擎为`InnoDB`，所以他可以把系统变量`default_storage_engine`的值设置为`InnoDB`；猫爷使用客户端B，他想让当前客户端对应的默认存储引擎为`MyISAM`，所以他可以把系统变量`default_storage_engine`的值设置为`MyISAM`。这样可以使狗哥和猫爷的的客户端拥有不同的默认存储引擎，使用时互不影响，十分方便。但是这样各个客户端都私有一份系统变量会产生这么两个问题：

- 有一些系统变量并不是针对单个客户端的，比如允许同时连接到服务器的客户端数量`max_connections`，查询缓存的大小`query_cache_size`，这些公有的系统变量让某个客户端私有显然不合适。
- 一个新连接到服务器的客户端对应的系统变量的值该怎么设置？

为了解决这两个问题，设计`MySQL`的大叔提出了系统变量的`作用范围`的概念，具体来说`作用范围`分为这两种：

- `GLOBAL`：全局变量，影响服务器的整体操作。
- `SESSION`：会话变量，影响某个客户端连接的操作。（注：`SESSION`有个别名叫`LOCAL`）

在服务器启动时，会将每个全局变量初始化为其默认值（可以通过命令行或选项文件中指定的选项更改这些默认值）。然后服务器还为每个连接的客户端维护一组会话变量，客户端的会话变量在连接时使用相应全局变量的当前值初始化。

这话有点儿绕，还是以`default_storage_engine`举例，在服务器启动时会初始化一个名为`default_storage_engine`，作用范围为`GLOBAL`的系统变量。之后每当有一个客户端连接到该服务器时，服务器都会单独为该客户端分配一个名为`default_storage_engine`，作用范围为`SESSION`的系统变量，该作用范围为`SESSION`的系统变量值按照当前作用范围为`GLOBAL`的同名系统变量值进行初始化。

很显然，通过启动选项设置的系统变量的作用范围都是`GLOBAL`的，也就是对所有客户端都有效的，因为在系统启动的时候还没有客户端程序连接进来呢。了解了系统变量的`GLOBAL`和`SESSION`作用范围之后，我们再看一下在服务器程序运行期间通过客户端程序设置系统变量的语法：

```
SET [GLOBAL|SESSION] 系统变量名 = 值;

```

或者写成这样也行：

```
SET [@@(GLOBAL|SESSION).]var_name = XXX;

```

比如我们想在服务器运行过程中把作用范围为`GLOBAL`的系统变量`default_storage_engine`的值修改为`MyISAM`，也就是想让之后新连接到服务器的客户端都用`MyISAM`作为默认的存储引擎，那我们可以选择下边两条语句中的任意一条来进行设置：

```
语句一：SET GLOBAL default_storage_engine = MyISAM;
语句二:SET @@GLOBAL.default_storage_engine =
MyISAM;
```

如果只想对本客户端生效，也可以选择下边三条语句中的任意一条来进行设置：

```
语句一：SET SESSION default_storage_engine = MyISAM;
语句二:SET @@SESSION.default_storage_engine =
MyISAM;
语句三:SET default_storage_engine = MyISAM;
```

如果在设置系统变量的语句中省略了作用范围，默认的作用范围就是`SESSION`。也就是说`SET 系统变量名 = 值`和`SET SESSION 系统变量名 = 值`是等价的。





### 查看不同作用范围的系统变量

既然`系统变量`有`作用范围`之分，那我们的`SHOW VARIABLES`语句查看的是什么`作用范围`的`系统变量`呢？

答：默认查看的是`SESSION`作用范围的系统变量。

当然我们也可以在查看系统变量的语句上加上要查看哪个`作用范围`的系统变量，就像这样：

```
SHOW [GLOBAL|SESSION] VARIABLES [LIKE 匹配的模式];
```



### 注意

- 并不是所有系统变量都具有`GLOBAL`和`SESSION`的作用范围。

  - 有一些系统变量只具有`GLOBAL`作用范围，比方说`max_connections`，表示服务器程序支持同时最多有多少个客户端程序进行连接。
  - 有一些系统变量只具有`SESSION`作用范围，比如`insert_id`，表示插入值时使用的`AUTO_INCREMENT`修饰的列的值。
  - 有一些系统变量的值既具有`GLOBAL`作用范围，也具有`SESSION`作用范围，比如我们前边用到的`default_storage_engine`，而且其实大部分的系统变量都是这样的，

- 有些系统变量是只读的，并不能设置值。

  比方说`version`，表示当前`MySQL`的版本，我们客户端是不能设置它的值的，只能在`SHOW VARIABLES`语句里查看。



## 启动选项和系统变量的区别

`启动选项`是在程序启动时我们程序员传递的一些参数，而`系统变量`是影响服务器程序运行行为的变量，它们之间的关系如下：

- 大部分的系统变量都可以被当作启动选项传入。
- 有些系统变量是在程序运行过程中自动生成的，是不可以当作启动选项来设置，比如`auto_increment_offset`、`character_set_client`啥的。
- 有些启动选项也不是系统变量，比如`defaults-file`。

  



## 状态变量

为了让我们更好的了解服务器程序的运行情况，`MySQL`服务器程序中维护了好多关于程序运行状态的变量，它们被称为`状态变量`。比方说`Threads_connected`表示当前有多少客户端与服务器建立了连接，`Handler_update`表示已经更新了多少行记录吧啦吧啦，像这样显示服务器程序状态信息的`状态变量`还有好几百个，我们就不一一唠叨了，等遇到了会详细说它们的作用的。

由于`状态变量`是用来显示服务器程序运行状况的，所以它们的值只能由服务器程序自己来设置，我们程序员是不能设置的。与`系统变量`类似，`状态变量`也有`GLOBAL`和`SESSION`两个作用范围的，所以查看`状态变量`的语句可以这么写：

```
SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

类似的，如果我们不写明作用范围，默认的作用范围是`SESSION`，比方说这样：

类似的，如果我们不写明作用范围，默认的作用范围是`SESSION`，比方说这样：

```
mysql> SHOW STATUS LIKE 'thread%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 0     |
| Threads_connected | 1     |
| Threads_created   | 1     |
| Threads_running   | 1     |
+-------------------+-------+
4 rows in set (0.00 sec)

mysql>

```

所有以`Thread`开头的`SESSION`作用范围的状态变量就都被展示出来了。









