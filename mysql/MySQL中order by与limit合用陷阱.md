# MySQL中order by与limit合用陷阱

在Mysql中我们常用order by来进行排序，使用limit来进行分页，当需要先排序后分页时，写法如下：

```
select * from table_a order by cloumn1 limit M,N;
```

但是这种写法隐藏着一个较深的陷阱。在排序字段值有数据重复的情况下，会很容易出现排序结果与预期不一致的问题。

现有如下一张表：

```
CREATE TABLE `student` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(12) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

数据内容如下：

```
mysql> select * from student;
+----+----------+---------------------+
| id | username | create_time         |
+----+----------+---------------------+
|  1 | xiaoming | 2018-12-07 11:01:44 |
|  2 | chen     | 2018-11-07 11:01:44 |
|  3 | xiao     | 2018-12-07 11:01:44 |
|  4 | li       | 2018-12-08 11:01:44 |
|  5 | liming   | 2018-12-07 11:01:44 |
|  6 | haha     | 2018-12-07 11:01:44 |
+----+----------+---------------------+
```

只用order by语句查询情况如下：

```
mysql> select * from student order by create_time;
+----+----------+---------------------+
| id | username | create_time         |
+----+----------+---------------------+
|  2 | chen     | 2018-11-07 11:01:44 |
|  1 | xiaoming | 2018-12-07 11:01:44 |
|  3 | xiao     | 2018-12-07 11:01:44 |
|  5 | liming   | 2018-12-07 11:01:44 |
|  6 | haha     | 2018-12-07 11:01:44 |
|  4 | li       | 2018-12-08 11:01:44 |
+----+----------+---------------------+
```

按照create_time排序，每次查询两条，SQL如下所示：

```
select * from student order by create_time limit pageNo, 2;

```

当查询第二页数据时，数据结果如下：

```
mysql> select * from student order by create_time limit 2,2;
+----+----------+---------------------+
| id | username | create_time         |
+----+----------+---------------------+
|  3 | xiao     | 2018-12-07 11:01:44 |
|  6 | haha     | 2018-12-07 11:01:44 |
+----+----------+---------------------+
```

当查询第三页数据时，数据结果如下：

```
mysql> select * from student order by create_time limit 4,2;
+----+----------+---------------------+
| id | username | create_time         |
+----+----------+---------------------+
|  6 | haha     | 2018-12-07 11:01:44 |
|  4 | li       | 2018-12-08 11:01:44 |
+----+----------+---------------------+
```

奇怪的事情出现了，第三页第一条数据和第二页第二条数据相同，这两个的执行结果已经证明现实与想象的是有出入的，实际SQL执行时并不是按照上述方法执行的。这里其实是Mysql会对Limit做优化，具体优化方式见官方文档，[LIMIT Query Optimization](https://dev.mysql.com/doc/refman/5.7/en/limit-optimization.html)

提取相关的点做下说明：

> If you combine LIMIT ```row_count``` with ORDER BY, MySQL stops sorting as soon as it has found the first row_count rows of the sorted result, rather than sorting the entire result. If ordering is done by using an index, this is very fast. If a filesort must be done, all rows that match the query without the LIMIT clause are selected, and most or all of them are sorted, before the first ```row_count``` are found. After the initial rows have been found, MySQL does not sort any remainder of the result set.

> One manifestation of this behavior is that an ORDER BY query with and without LIMIT may return rows in different order, as described later in this section.

上面提到如果将LIMIT row_count与ORDER BY结合使用，MySQL会在找到排序结果的第一个row_count行后立即停止排序，而不是对整个结果进行排序。如果使用索引完成排序，则速度非常快。如果必须完成文件排序，则在找到第一个row_count之前，将选择与没有LIMIT子句的查询匹配的所有行，并对其中的大部分或全部进行排序。在找到初始行之后，MySQL不会对结果集的任何剩余部分进行排序。

我们来看一下对应SQL的执行计划：

```
mysql> explain select * from student order by create_time limit 4,2;
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | student | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    6 |   100.00 | Using filesort |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+----------------+
``` 

可以确认是用的文件排序，表确实也没有加额外的索引。所以我们可以确定这个SQL执行时是会找到limit要求的行后立马返回查询结果的。

不过就算它立马返回，为什么分页会不准呢？

官方文档里面做了如下说明： 

> If multiple rows have identical values in the ORDER BY columns, the server is free to return those rows in any order, and may do so differently depending on the overall execution plan. In other words, the sort order of those rows is nondeterministic with respect to the nonordered columns.

上面提到如果ORDER BY列中的多个行具有相同的值，则服务器可以按任何顺序自由返回这些行，并且可能会根据整体执行计划的不同而不同。换句话说，这些行的排序顺序相对于无序列是不确定的。

基于这个我们就基本知道为什么分页会不准了，因为我们排序的字段是create_time，正好又有几个相同的值的行，在实际执行时返回结果对应的行的顺序是不确定的。

那这种情况应该怎么解决呢？

官方给出了解决方案： 

> If it is important to ensure the same row order with and without LIMIT, include additional columns in the ORDER BY clause to make the order deterministic.

如果想在Limit存在或不存在的情况下，都保证排序结果相同，可以额外加一个排序条件。例如id字段是唯一的，可以考虑在排序字段中额外加个id排序去确保顺序稳定

所以上面的情况下可以在SQL再添加个排序字段，比如student的id字段，这样分页的问题就解决了。修改后的SQL可以像下面这样： 

select * from student order by create_time,id limit 4,2