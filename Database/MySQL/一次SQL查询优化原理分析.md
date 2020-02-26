### 一次SQL查询优化原理分析

#### 现象

有一张财务流水表，未分库分表，目前的数据量为9555695，分页查询使用到了limit，优化之前的查询耗时**16 s 938 ms** (execution: 16 s 831 ms, fetching: 107 ms)，按照下文的方式调整SQL后，耗时**347 ms** (execution: 163 ms, fetching: 184 ms)；

#### 操作

查询条件放到子查询中，子查询只查主键ID，然后使用子查询中确定的主键关联查询其他的属性字段；

```sql
-- 优化前SQL
SELECT  各种字段
FROM `table_name`
WHERE 各种条件
LIMIT 0,10;
```

```sql
-- 优化后SQL
SELECT  各种字段
FROM `table_name` main_tale
RIGHT JOIN 
(
SELECT  子查询只查主键
FROM `table_name`
WHERE 各种条件
LIMIT 0,10;
) temp_table ON temp_table.主键 = main_table.主键
```

#### 原理

减少回表操作；

#### 详细

**MySQL的版本**：

```sql
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.17    |
+-----------+
1 row in set (0.00 sec)
```

**表结构**：

```sql
mysql> desc test;
+--------+---------------------+------+-----+---------+----------------+
| Field  | Type                | Null | Key | Default | Extra          |
+--------+---------------------+------+-----+---------+----------------+
| id     | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| val    | int(10) unsigned    | NO   | MUL | 0       |                |
| source | int(10) unsigned    | NO   |     | 0       |                |
+--------+---------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```

其中，id为自增主键，val为非唯一索引。

**灌入大量数据，共500万**：

```sql
mysql> select count(*) from test;
+----------+
| count(*) |
+----------+
|  5242882 |
+----------+
1 row in set (4.25 sec)
```

众所周知，当limit offset rows中的offset很大时，会出现效率问题，从下面可以看出，当offset取300000时候，查询时间消耗为15.98：

```sql
mysql> select * from test where val=4 limit 300000,5;
+---------+-----+--------+
| id      | val | source |
+---------+-----+--------+
| 3327622 |   4 |      4 |
| 3327632 |   4 |      4 |
| 3327642 |   4 |      4 |
| 3327652 |   4 |      4 |
| 3327662 |   4 |      4 |
+---------+-----+--------+
5 rows in set (15.98 sec)
```

下面的查询会得到和上面查询相同的结果，但是时间消耗很小，和上面的时间消耗比起来差距非常大：

```sql
mysql> select * from test a inner join (select id from test where val=4 limit 300000,5) b on a.id=b.id;
+---------+-----+--------+---------+
| id      | val | source | id      |
+---------+-----+--------+---------+
| 3327622 |   4 |      4 | 3327622 |
| 3327632 |   4 |      4 | 3327632 |
| 3327642 |   4 |      4 | 3327642 |
| 3327652 |   4 |      4 | 3327652 |
| 3327662 |   4 |      4 | 3327662 |
+---------+-----+--------+---------+
5 rows in set (0.38 sec)
```

**分析**：

- select * from test where val=4 limit 300000,5;的查询过程：

  - 查询到索引叶子节点数据。
  - 根据叶子节点上的主键值去聚簇索引上查询需要的全部字段值。
  - 示意图：

  ![backtable](http://image.augustrush8.com/images/backtable.jpg)

像上面这样，需要查询300005次索引节点，查询300005次聚簇索引的数据，最后再将结果过滤掉前300000条，取出最后5条。MySQL耗费了大量随机I/O在查询聚簇索引的数据上，而有300000次随机I/O查询到的数据是不会出现在结果集当中的。

**证实**：

为了证实`select * from test where val=4 limit 300000,5`是扫描300005个索引节点和300005个聚簇索引上的数据节点，我们需要知道MySQL有没有办法统计在一个sql中通过索引节点查询数据节点的次数。我先试了Handler_read_*系列，很遗憾没有一个变量能满足条件。

只能通过间接的方式来证实：

InnoDB中有buffer pool。里面存有最近访问过的数据页，包括数据页和索引页。所以我们需要运行两个sql，来比较buffer pool中的数据页的数量。预测结果是运行`select * from test a inner join (select id from test where val=4 limit 300000,5);` 之后，buffer pool中的数据页的数量远远少于`select * from test where val=4 limit 300000,5;`对应的数量，因为前一个sql只访问5次数据页，而后一个sql访问300005次数据页。

```sql
mysql> select index_name,count(*) from information_schema.INNODB_BUFFER_PAGE where INDEX_NAME in('val','primary') and TABLE_NAME like '%test%' group by index_name;
Empty set (0.04 sec)
```

可以看出，目前buffer pool中没有关于test表的数据页。

```sql
mysql> select * from test where val=4 limit 300000,5;
+---------+-----+--------+
| id      | val | source |
+---------+-----+--------+| 
3327622 |   4 |      4 |
| 3327632 |   4 |      4 |
| 3327642 |   4 |      4 |
| 3327652 |   4 |      4 |
| 3327662 |   4 |      4 |
+---------+-----+--------+
5 rows in set (26.19 sec)

mysql> select index_name,count(*) from information_schema.INNODB_BUFFER_PAGE where INDEX_NAME in('val','primary') and TABLE_NAME like '%test%' group by index_name;
+------------+----------+
| index_name | count(*) |
+------------+----------+
| PRIMARY    |     4098 |
| val        |      208 |
+------------+----------+2 rows in set (0.04 sec)
```

可以看出，此时buffer pool中关于test表有4098个数据页，208个索引页。

紧接着，需要测试优化后SQL的情况，为了防止上次试验的影响，我们需要清空buffer pool，重启mysql。

```sql
mysqladmin shutdown
/usr/local/bin/mysqld_safe &
```

```sql
mysql> select index_name,count(*) from information_schema.INNODB_BUFFER_PAGE where INDEX_NAME in('val','primary') and TABLE_NAME like '%test%' group by index_name;

Empty set (0.03 sec)
```

```sql
mysql> select * from test a inner join (select id from test where val=4 limit 300000,5) b on a.id=b.id;
+---------+-----+--------+---------+
| id      | val | source | id      |
+---------+-----+--------+---------+
| 3327622 |   4 |      4 | 3327622 |
| 3327632 |   4 |      4 | 3327632 |
| 3327642 |   4 |      4 | 3327642 |
| 3327652 |   4 |      4 | 3327652 |
| 3327662 |   4 |      4 | 3327662 |
+---------+-----+--------+---------+
5 rows in set (0.09 sec)

mysql> select index_name,count(*) from information_schema.INNODB_BUFFER_PAGE where INDEX_NAME in('val','primary') and TABLE_NAME like '%test%' group by index_name;
+------------+----------+
| index_name | count(*) |
+------------+----------+
| PRIMARY    |        5 |
| val        |      390 |
+------------+----------+
2 rows in set (0.03 sec)
```

可以看明显的看出两者的差别：第一个sql加载了4098个数据页到buffer pool，而第二个sql只加载了5个数据页到buffer pool。符合我们的预测。也证实了为什么第一个sql会慢：读取大量的无用数据行（300000），最后却抛弃掉。
 而且这会造成一个问题：加载了很多热点不是很高的数据页到buffer pool，会造成buffer pool的污染，占用buffer pool的空间。  

为了在每次重启时确保清空buffer pool，我们需要关闭innodb_buffer_pool_dump_at_shutdown和innodb_buffer_pool_load_at_startup，这两个选项能够控制数据库关闭时dump出buffer pool中的数据和在数据库开启时载入在磁盘上备份buffer pool的数据。

