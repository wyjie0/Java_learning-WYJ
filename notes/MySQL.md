此文是我学习《MySQL技术内幕：SQL编程》的读书笔记，后续还会学习《MySQL技术内幕：InnoDB存储引擎》、《高性能MySQL》（第三版）和一些关于数据库的博客。

* [一、查询操作](#一查询操作)
* [二、MySQL体系结构和存储引擎](#二MySQL体系结构和存储引擎)
  * [MySQL体系结构](#MySQL体系结构)
  * [MySQL存储引擎](#MySQL存储引擎)
  * [连接MySQL](#连接MySQL)
* [三、InnoDB存储引擎](#三InnoDB存储引擎)
  * [InnoDB体系架构](#InnoDB体系架构)

# 一、查询操作
对于查询处理，可分为逻辑查询处理以及物理查询处理。逻辑查询处理表示执行查询应该产生什么样的结果，而物理查询代表MySQL数据库是如何得到该结果的。
## 逻辑查询处理
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

## 物理查询处理
数据库可能并不会完全按照逻辑查询处理的方式来进行查询。在MySQL数据库层有Parser和Optimizer两个组件。Parser的工作是分析SQL语句，Optimizer的工作是对SQL语句进行优化，选择一条最优的路径来选取数据，但是必须保证最终结果与逻辑查询处理是相等的。物理查询可以利用表上的索引来缩短SQL语句运行的时间，一次来提高数据库的整体性能。

# 二、MySQL体系结构和存储引擎
## MySQL体系结构
![image](https://user-images.githubusercontent.com/25001763/71666991-2e1da980-2d9e-11ea-9992-04c402213105.png)
MySQL由以下几部分组成：

* 连接池组件

* 管理服务和工具组件

* SQL接口组件

* 查询分析器组件

* 优化器组件

* 缓冲组件

* 插件式存储引擎

* 物理文件

## MySQL存储引擎
每个存储引擎都有各自的特点，能够根据具体的应用建立不同存储引擎表。

### InnoDB存储引擎
InnoDB存储引擎支持事务，其设计目标是面向在线事务处理（OLTP）的应用。其特点是行锁设计、支持外键，支持类似于Oracle的非锁定读，即默认读操作不会产生锁。InnoDB通过使用多版本并发控制（MVCC）来获得高并发性，并且实现了SQL标准的4中隔离级别（READ-UNCOMMITTED(读取未提交)、READ-COMMITTED(读取已提交)、REPEATABLE-READ(可重复读)、SERIALIZABLE(可串行化)），默认为REPEATABLE级别。同时被称为一种next-key locking的策略来避免幻读。对于表中数据的存储采用**聚集**的方式，因此每张表的存储都是按主键的顺序进行存放。如果没有显示地在表定义时指定主键，InnoDB存储引擎会为每一行生成一个6字节的ROWID，以此作为主键。

### MyISAM存储引擎
不支持事务、锁为表锁，支持全文检索，主要面向一些OLAP数据库应用，它的缓冲池只缓冲索引文件，而不缓冲数据文件

## 连接MySQL
### TCP/IP
TCP/IP套接字方式是MySQL数据库在任何平台下都提供的连接方式，也是网络中使用得最多的一种方式。这种方式在TCP/IP连接上建立了一个基于网络的连接请求，一般情况下客户端在一台服务器上，而MySQL实例在另一台服务器上，这两台机器通过一个TCP/IP网络连接。

# 三、InnoDB存储引擎
## InnoDB体系架构
![image](https://user-images.githubusercontent.com/25001763/71668881-a8056100-2da5-11ea-92dd-e78799b85720.png)

内存池的作用：
 
 * 维护所有进程/线程需要访问的多个内部数据结构
 
 * 缓存磁盘上的数据，方便快速地读取，同时在对磁盘文件的数据修改之前在这里缓存
 
 * 重做日志缓冲
 
后台线程的主要作用是：负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。将已修改的数据文件刷新到磁盘文件，保证在数据库发生异常的情况下，InnoDB能恢复到正常运行状态。

### 后台线程
1. Master Thread

主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，保证脏页的刷新、合并插入缓冲、Undo页的回收等

2. IO Thread

InnoDB存储引擎中大量使用了AIO（Async IO）来处理写IO请求，IO Thread负责IO请求的回调处理。

3. Purge Thread

事务被提交后，其使用的undolog可能不再需要，因此需要此类线程来回收已经使用并分配的undo页。

4. Page Cleaner Thread

将之前版本的脏页的刷新操作都放入到单独的线程来完成，以减轻原Master Thread的工作及对于用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能

### 内存
1. 缓冲池

缓冲池是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页“Fix”到缓冲池中。对于缓冲池中页的修改，首先修改在缓冲池中的页，然后再以一定的频率刷新到缓冲池中。页从缓冲池刷新回磁盘的操作**不是**每次页发生更新时触发，而是通过一种称为**CheckPoint**的机制刷新回磁盘。

2. LRU List、Free List和FlushList

通常来，数据库中的缓冲池是通过LRU（最近最少使用）算法来进行整理的。即最频繁使用的页在LRU List的前端，最少使用的页在LRU列表的后端。当缓冲池不能存储新读取到的页时，将首先释放LRU列表中尾端的页。

InnoDB对传统的LRU算法进行了优化。在InnoDB引擎中，LRU列表还加入了midpoint位置，新读取到的页并不直接放入到LRU列表首部，而是放入到midpoint位置。这种优化的目的是：某些SQL操作（索引、数据的扫描）可能会使缓冲池中的页被刷新出，而这些操作可能会读取很多页甚至所有的页，而读取到的页可能只会在这一次使用到，并不是活跃的热点数据。如果直接放在首部可能会使真正的活跃数据页从LRU列表中移除，而在下次读取这些页时，需要重新访问磁盘。

LRU列表用来管理已经读取的页，但到数据库刚启动时，LRU列表是空的，即没有任何的页。这时页都存放在Free列表中。当需要从缓冲池中分页的时候，首先从Free列表中查找是否有可用的空闲页，若有则将该页从Free列表中删除，放入到LRU列表中。

在LRU列表中的页被修改后，该页被称为脏页，即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过CheckPoint机制将脏页刷新回磁盘，而Flush列表中的页即为脏页列表。脏页既存在于LRU列表中，又存在于Flush列表中。LRU列表用来管理缓冲池中页的可用性，Flush列表用来管理将页刷新回磁盘。

## CheckPoint技术
该技术的目的是为了解决以下几个问题：

* 缩短数据库的恢复时间

* 缓冲池不够用时，将脏页刷新到磁盘

* 重做日志不可用时，刷新脏页

当数据库发生宕机时，数据库不需要重做所有的日志，因为CheckPoint之前的页都已经刷新回磁盘。故数据库只需对CheckPoint后的重做日志进行恢复。当缓冲池不够用时，根据LRU算法会溢出最近最少使用的页，若此页为脏页，那么需要强制执行CheckPoint，将脏页刷回磁盘。

## InnoDB关键特性
* 插入缓冲

* 两次写

* 自适应哈希缓冲

* 异步IO

* 刷新邻近页

### 插入缓冲
1. Insert Buffer

Insert Buffer是物理页的一个组成部分。对于非聚集索引的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在，则直接插入；若不在，则先放入到一个Insert Buffer对象中，然后再以一定的频率和情况进行Insert Buffer和辅助索引页子结点的合并操作，这时，通常能将多个插入合并到一个操作中。Insert Buffer的使用需要同时满足以下两个条件：索引是辅助索引；索引不是唯一的。

2. 两次写

doublewrite结构如图所示
![image](https://user-images.githubusercontent.com/25001763/71705804-0836ea00-2e1c-11ea-824c-b22ff6ac4c47.png)

它由两部分组成，一部分是内存中的doublewrite buffer，大小为2MB，另一部分是物理磁盘上共享表空间中连续的128个页，即2个区，大小也为2MB。在对缓冲池中的脏页进行刷新时，并不直接写磁盘，而是会通过memcpy函数将脏页先复制到内存中的doublewrite buffer，之后通过doublewrite buffer再分两次，每次1MB顺序地写入共享表空间的物理磁盘上，然后马上调用fsync函数，同步磁盘，避免缓冲写带来的问题。在完成doublewrite页的写入后，再将doublewrite buffer中的页写入各个表空间文件中。

3. 自适应哈希索引

InnoDB会监控对表上各索引页的查询，如果观察到建立哈希索引会带来速度提升，则建立哈希索引，称之为自适应哈希索引（AHI）。AHI是通过缓冲池的B+树页构造而来，因此建立的速度很快，而且不需要对整张表构建哈希索引。InnoDB存储引擎会自动根据访问的频率和模式来自动地为某些热点页建立哈希索引。

4. 刷新邻接页

当刷新一个脏页时，InnoDB存储引擎会检测该页所在区的所有页，如果是脏页，那么一起进行刷新。

# 四、表
## 索引组织表
在InnoDB存储引擎中，表都是根据主键顺序组织存放的，这种存储方式的表为索引组织表。在InnoDB存储引擎表中，每张表都有个主键，如果在创建表时没有显示地定义主键，则InnoDB存储引擎会按如下方式选择或创建主键：

* 首先判断表中是否有非空的唯一索引，如果有，则该列为主键。当表中有多个非空的唯一索引，则选择第一个定义的为主键。主键的选择是根据定义非空唯一索引的顺序，而不是表中列的顺序。

* 如果不符合上述条件，InnoDB存储引擎自动创建一个6字节大小的指针

## InnoDB逻辑存储结构
从InnoDB存储引擎的逻辑存储结构看，所有数据都被逻辑地存放在一个空间中，称之为表空间。表空间又由段、区、页组成。
![image](https://user-images.githubusercontent.com/25001763/71791329-9c4cbf80-306f-11ea-890f-544a9fb7602c.png)

### 表空间
表空间可以看做InnoDB存储引擎逻辑结构的最高层，所有的数据都存放在表空间中。
### 段
表空间是由各个段组成的，常见的段有数据段、索引段、回滚段等。数据段即为B+树的叶子节点，索引段即为B+树的非索引节点。
### 区
区是由连续页组成的空间，在任何情况下每个区的大小都为1MB。在默认情况下，InnoDB存储引擎页的大小为16KB

## InnoDB行记录格式
InnoDB存储引擎提供了Compact和Redundant两种格式来存放行记录数据，

### Compact行记录格式
![image](https://user-images.githubusercontent.com/25001763/71791882-8cce7600-3071-11ea-800b-d92e78d4118d.png)

变长字段长度：最大不能超过2字节，这是因为MySQL数据库中的VARCHAR类型的最大长度限制为65535。

Null标志位：指示该行数据中是否有Null值

### Redundant行记录格式
![image](https://user-images.githubusercontent.com/25001763/71792226-f13e0500-3072-11ea-8d37-d05514c780ff.png)







# 一些文章
* [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

* [数据库两大神器--索引和锁](https://juejin.im/post/5b55b842f265da0f9e589e79)

* [MySQL的索引](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483701&idx=1&sn=bd229dd584f51ef4fe545d44ad8cdbf9&chksm=979688c7a0e101d1b5c752094013b78f5bd50ab905257ba82149d85d35ea07aba1a15b0e52b4&mpshare=1&scene=1&srcid=0409Tn66UYWSWvqEVlOpwGtR&key=6cd553e86912686a47d76f2d900b1b5b388c90b29708f016db3a6e1bcebe032220ba63626095c4298f32cda7d0d7bd11bded2365f05c32e522584dd149b98db0bb8549ef144cdca694665d31d35cfeef&ascene=0&uin=MzAzMjU4NDM3Nw%3D%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.12.4+build(16E195)&version=12020810&nettype=WIFI&lang=zh_CN&fontScale=100&pass_ticket=YHEmqDDX8hHkj5FiSVpQvjYqIMBDHHDS2po4mfJe%2BqIXlqwJI%2Bg7aJUZq0%2BDwGJ0)

# 参考资料
[1]姜承尧.《MySQL技术内幕：SQL编程》[M].机械工业出版社

[2]姜承尧.《MySQL技术内幕：InnoDB存储引擎》[M].机械工业出版社
