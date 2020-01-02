此文是我学习《MySQL技术内幕：SQL编程》的读书笔记，后续还会学习《MySQL技术内幕：InnoDB存储引擎》、《高性能MySQL》（第三版）和一些关于数据库的博客。

* [一、查询操作](#一查询操作)

## 一、查询操作
对于查询处理，可分为逻辑查询处理以及物理查询处理。逻辑查询处理表示执行查询应该产生什么样的结果，而物理查询代表MySQL数据库是如何得到该结果的。
### 逻辑查询处理
在SQL语言中，第一个被处理的子句总是FROM子句。下代码块显示了逻辑查询处理的顺序以及步骤的序号

```
(8) select (9) distinct
(1) from<left_table>
(3) <join_type>join<right_table>
(2) on<join_condition>
(4) where<where_condition>
(5) group by<group_by_list>
(6) with {cube|rollup}
(7) having<having_condition>
(10) order by<order_by_list>
(11) limit <limit_number>
```
每个操作都会产生一张虚拟表，该虚拟表作为一个处理的输入。这些虚拟表对用户是透明的，只有最后一步生成的虚拟表才会返回给用户。具体分析每个阶段：

（1） from：对from子句中的坐标<left_table>和右表<right_table>执行笛卡尔积，产生虚拟表VT1

（2） on：对虚拟表VT1应用on筛选，只有那些符合<join_condition>的行才被插入虚拟表VT2。对于OUTER JOIN中的过滤，在ON过滤器过滤完之后还会添加保留表中被
ON
条件过滤掉的记录

（3） join：如果制定了outer join，那么保留表中为匹配的行作为外部行添加到虚拟表VT2中，产生VT3。如果from子句包含两个以上表，则对上一个连接生成的结果
表VT3和下一个表重复执行步骤1~步骤3，知道处理完所有表为止

（4） where：对VT3应用where过滤条件，只有符合<where_condition>的记录才被插入虚拟表VT4中。与on过滤器不同，where条件中被过滤掉的记录被永久过滤

（5） group by：根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5

（6） cube|rollup：对表VT5进行cube或rollup操作，产生表VT6

（7） having：对VT6应用having过滤器，只有符合<having_condition>的记录才被插入VT7中

（8） select：选择指定的列，插入到虚拟表VT8中。虽然select是查询中最先被指定的部分，但是知道现在才真正进行处理。有一点容易被忽视的是，**列的别名不能
在select中的其他别名表达式中使用**

（9） distinct：去除重复数据，产生虚拟表VT9

（10） order by：将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生VT10。没有order by子句的查询只代表从**集合**中查询数据，而集合是没有顺序
概念的，所以就算有主键按序递增，但是查询的时候，查出的列没有顺序，并不是按主键来取出数据。

（11） limit：取出指定行的记录，产生VT11，并返回给查询用户

### 物理查询处理
数据库可能并不会完全按照逻辑查询处理的方式来进行查询。在MySQL数据库层有Parser和Optimizer两个组件。Parser的工作是分析SQL语句，Optimizer的工作是对SQL语句进行优化，选择一条最优的路径来选取数据，但是必须保证最终结果与逻辑查询处理是相等的。物理查询可以利用表上的索引来缩短SQL语句运行的时间，一次来提高数据库的整体性能。

# 一些好的文章
* [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

* [数据库两大神器--索引和锁](https://juejin.im/post/5b55b842f265da0f9e589e79)
