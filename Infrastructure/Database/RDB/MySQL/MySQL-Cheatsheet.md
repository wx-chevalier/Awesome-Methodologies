[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# MySQL CheatSheet

MySQL is a full-featured open-source relational database management system (RDBMS) that was originally built by MySQL AB and currently owned by Oracle Corporation. It stores data in tables that are grouped into a database, uses Structured Query Language (SQL) to access data and such commands as ‘SELECT’, ‘UPDATE’, ‘INSERT’ and ‘DELETE’ to manage it. Related information can be stored in different tables, but the usage of JOIN operation allows you to correlate it, perform queries across various tables and minimize the chance of data duplication.

MySQL is compatible with nearly all operating systems, namely Windows, Linux, Unix, Apple, FreeBSD and many others. It supports various storage engines, like InnoDB (it is the default one), Federated, MyISAM, Memory, CSV, Archive, Blackhole and Merge.

本清单参考了 [MySQL Command Line Cheatsheet](https://gist.github.com/hofmannsven/9164408)、[MySQL 5.7 Reference Manual](https://parg.co/UpB)，更多 MySQL 的学习资料参考 [MySQL 学习与资料索引]()。此外，本文中涉及的 SQL 语句可以前往 [SQLFiddle](http://sqlfiddle.com/#!9/60f53) 查看完整的 SQL 语句，也可以前往 [SQL Teaching](https://www.sqlteaching.com/) 进行交互式基础语法学习。

对于 SQL 方面的知识，请参阅 [SQL CheatSheet](https://parg.co/mtZ)。

```sh
# 启动本地的 MySQL 实例
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 将 MySQL 端口映射出来
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql:tag

# 允许启动的其他容器中使用 MySQL
$ docker run --name some-app --link some-mysql:mysql -d application-that-uses-mysql

# 启动本地 MySQL 客户端并且连接到目标
$ docker run -it --link some-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

# 将 MySQL 镜像作为客户端使用，连接到远端的数据库
$ docker run -it --rm mysql mysql -hsome.mysql.host -usome-mysql-user -p
```

```sql
--- 老版本中更新 Root 用户信息
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
 FLUSH PRIVILEGES;

--- 5.7 之后版本
update user set authentication_string=password('YOURSUPERSECRETPASSWORD') where user='root';
```

# 配置与管理

## 权限管理

MySQL 的安装过程不在此赘述，其自带了命令行工具进行管理：

```sh
# Prompt for password / 交互式登录
$ mysql -u [username] -p

$ mysql -u [username] -p [database]
```

登录完毕后可以查看并操作数据库：

```sh
# 展示所有数据库
mysql> show databases;

# 创建新的数据库
mysql> create database [database];

# 选择数据库
mysql> use [database];
```

```sh
# 创建用户并且分配权限
mysql> CREATE USER 'finley'@'%' IDENTIFIED BY 'some_pass';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'finley'@'%'
    ->     WITH GRANT OPTION;

# 修改用户密码
use mysql;
update user set password=PASSWORD("mynewpassword") where User='root';
flush privileges;
```

## 信息检索

```sql
SELECT
  TABLE_NAME, table_rows, data_length, index_length,
  round(((data_length + index_length) / 1024 / 1024),2) 'Size in MB'
FROM information_schema.TABLES
WHERE TABLE_NAME = 'r_case_info' and TABLE_TYPE='BASE TABLE'
ORDER BY data_length DESC;
```

# 数据查询与操作

## 数据类型

![group](https://user-images.githubusercontent.com/5803001/44440908-d6ddc580-a5fc-11e8-9701-096e3b5b7be0.jpg)

## 时间与日期

MySQL 中支持 date 和 datetime、timestamp 等时间类型，其中 date 与 datetime 都是时区无关，timestamp 会跟随设置的时区变化而变化，而 datetime 保存的是绝对值不会变化。占用存储空间不同。timestamp 储存占用 4 个字节，datetime 储存占用 8 个字节。

- 可表示的时间范围不同。timestamp 可表示范围:1970-01-01 00:00:00~2038-01-09 03:14:07，datetime 支持的范围更宽 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59

- 索引速度不同。timestamp 更轻量，索引相对 datetime 更快。

MySQL 仅允许插入无时区的时间格式字符串，当我们插入到 DATETIME 类型时，其会保存字面信息了；如果插入到 TIMESTAMP 类型，MySQL 首先会将其根据当前时区转化为 UTC 时间戳存储，在用户查询的时候则会再从 UTC 时区转化为数据库当前时区。DATETIME 与 TIMESTAMP 在使用上并无太大差异，仅当数据库时区发生变化时，其读取值会发生变化。MySQL 也提供了丰富的[时间与日期函数](http://dev.mysql.com/doc/refman/5.1/en/date-and-time-functions.html#function_month)，sysdate() 和 now() 都可以使用当前时间值，其中 sysdate 表示实时的系统时间，不过其有可能导致主库和从库执行时返回值不一样，导致主从数据库不一致。

```sh
mysql> select now(), curdate(), sysdate(), curtime() \G;
*************************** 1. row ***********************
    now(): 2013-01-17 13:07:53
curdate(): 2013-01-17
sysdate(): 2013-01-17 13:07:53
curtime(): 13:07:53
```

```sql
// 获取事件的年与月
GROUP BY YEAR(record_date), MONTH(record_date)

// 根据固定的时间格式排序
GROUP BY DATE_FORMAT(record_date, '%Y%m')
```

我们可以使用 TIMESTAMPDIFF 与 DATEDIFF 计算两个日期时间的差：

```sql
--- FRAC_SECOND、SECOND、 MINUTE、 HOUR、 DAY、 WEEK、 MONTH、 QUARTER或 YEAR
SELECT TIMESTAMPDIFF(DAY,'2012-10-01','2013-01-13');

--- 传入两个日期函数，比较的DAY天数
SELECT DATEDIFF('2013-01-13','2012-10-01');
```

# 数据操作

## 事务与锁

|        | 行锁 | 表锁 | 页锁 |
| ------ | ---- | ---- | ---- |
| MyISAM |      | √    |      |
| BDB    |      | √    | √    |
| InnoDB | √    | √    |      |

- 表锁： 开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低
- 行锁： 开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高
- 页锁： 开销和加锁速度介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般

表锁更适用于以查询为主，只有少量按索引条件更新数据的应用；行锁更适用于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用。

InnoDB 实现了以下两种类型的行锁。
共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。
排他锁（X)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

另外，为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是表锁。

意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的 IS 锁。
意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的 IX 锁。

```sql
mysql> set autocommit = 0;

/* 共享锁 */
-- 当前 session 对 actor_id=178 的记录加 share mode 的共享锁：
mysql> select actor_id,first_name,last_name from actor where actor_id = 178 lock in share mode;

-- 当前session对锁定的记录进行更新操作，等待锁：
mysql> update actor set last_name = 'MONROE T' where actor_id = 178;

-- 其他session仍然可以查询记录，并也可以对该记录加share mode的共享锁：
mysql> select actor_id,first_name,last_name from actor where actor_id = 178lock in share mode;

-- 其他session也对该记录进行更新操作，则会导致死锁退出：
mysql> update actor set last_name = 'MONROE T' where actor_id = 178;
-- ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction

-- 获得锁后，可以成功更新：
-- Query OK, 1 row affected (17.67 sec)

/* 排他锁 */
-- 当前session对actor_id=178的记录加for update的排它锁：
mysql> select actor_id,first_name,last_name from actor where actor_id = 178 for update;

-- 其他session可以查询该记录，但是不能对该记录加共享锁，会等待获得锁：
mysql> select actor_id,first_name,last_name from actor where actor_id = 178 for update;

-- 当前session可以对锁定的记录进行更新操作，更新后释放锁：
mysql> update actor set last_name = 'MONROE T' where actor_id = 178;
mysql> commit;

-- 其他session获得锁，得到其他session提交的记录：
mysql> select actor_id,first_name,last_name from actor where actor_id = 178 for update;
```

对于 InnoDB 表，在绝大部分情况下都应该使用行级锁，因为事务和行锁往往是我们之所以选择 InnoDB 表的理由。但在个别特殊事务中，也可以考虑使用表级锁。
第一种情况是：事务需要更新大部分或全部数据，表又比较大，如果使用默认的行锁，不仅这个事务执行效率低，而且可能造成其他事务长时间锁等待和锁冲突，这种情况下可以考虑使用表锁来提高该事务的执行速度。
第二种情况是：事务涉及多个表，比较复杂，很可能引起死锁，造成大量事务回滚。这种情况也可以考虑一次性锁定事务涉及的表，从而避免死锁、减少数据库因事务回滚带来的开销。

```sql
--- 写表t1并从表t读
SET AUTOCOMMIT=0;
LOCK TABLES t1 WRITE, t2 READ, ...;
[do something with tables t1 and t2 here];
COMMIT;
UNLOCK TABLES;
```

# Store Engine | 存储引擎

## 锁

InnoDB 行锁是通过给索引上的索引项加锁来实现的，这一点 MySQL 与 Oracle 不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB 这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB 才使用行级锁，否则，InnoDB 将使用表锁。当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，另外，不论是使用主键索引、唯一索引或普通索引，InnoDB 都会使用行锁来对数据加锁。

```sql
mysql> select * from tab_no_index where id = 1 for update;

--- 添加索引
mysql> alter table tab_with_index add index id(id);

--- 在没有索引的情况下，InnoDB只能使用表锁，会导致下述查询语句等待
--- 当我们给其增加一个索引后，InnoDB 就只锁定了符合条件的行，不会出现锁等待
mysql> select * from tab_no_index where id = 2 for update;
```

由于 MySQL 的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的。譬如当 id 字段有索引，name 字段没有索引：

```sql
mysql>  select * from tab_with_index where id = 1 and name = '1' for update;

--- 虽然session_2访问的是和session_1不同的记录，但是因为使用了相同的索引，所以需要等待锁
mysql> select * from tab_with_index where id = 1 and name = '4' for update;
```

即便在条件中使用了索引字段，但是否使用索引来检索数据是由 MySQL 通过判断不同执行计划的代价来决定的，如果 MySQL 认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下 InnoDB 将使用表锁，而不是行锁。因此，在分析锁冲突时，别忘了检查 SQL 的执行计划，以确认是否真正使用了索引。

### 间隙锁

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB 会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB 也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key 锁）。
举例来说，假如 emp 表中只有 101 条记录，其 empid 的值分别是 1,2,...,100,101，下面的 SQL：

```sql
mysql> select * from emp where empid > 100 for update;
```

是一个范围条件的检索，InnoDB 不仅会对符合条件的 empid 值为 101 的记录加锁，也会对 empid 大于 101（这些记录并不存在）的“间隙”加锁。
InnoDB 使用间隙锁的目的，一方面是为了防止幻读，以满足相关隔离级别的要求，对于上面的例子，要是不使用间隙锁，如果其他事务插入了 empid 大于 100 的任何记录，那么本事务如果再次执行上述语句，就会发生幻读；另外一方面，是为了满足其恢复和复制的需要。有关其恢复和复制对锁机制的影响，以及不同隔离级别下 InnoDB 使用间隙锁的情况，在后续的章节中会做进一步介绍。
很显然，在使用范围条件检索并锁定记录时，InnoDB 这种加锁机制会阻塞符合条件范围内键值的并发插入，这往往会造成严重的锁等待。因此，在实际应用开发中，尤其是并发插入比较多的应用，我们要尽量优化业务逻辑，尽量使用相等条件来访问更新数据，避免使用范围条件。
还要特别说明的是，InnoDB 除了通过范围条件加锁时使用间隙锁外，如果使用相等条件请求给一个不存在的记录加锁，InnoDB 也会使用间隙锁！

```sql
--- 当前session对不存在的记录加for update的锁：
mysql> select * from emp where empid = 102 for update;

--- 这时，如果其他session插入empid为102的记录（注意：这条记录并不存在），也会出现锁等待：
mysql>insert into emp(empid,...) values(102,...);

--- session 1 执行 rollback：
mysql> rollback;

--- 由于其他session_1回退后释放了Next-Key锁，当前session可以获得锁并成功插入记录：
mysql>insert into emp(empid,...) values(102,...);
```

### 死锁与超时

MyISAM 表锁是 deadlock free 的，这是因为 MyISAM 总是一次获得所需的全部锁，要么全部满足，要么等待，因此不会出现死锁。但在 InnoDB 中，除单个 SQL 组成的事务外，锁是逐步获得的，这就决定了在 InnoDB 中发生死锁是可能的。

```sql
--- session 1 获取 table_1 中排他锁
mysql> set autocommit = 0;
mysql> select * from table_1 where where id=1 for update;

--- session 2 获取 table_2 中排他锁
mysql> select * from table_2 where id=1 for update;

--- session 1 申请 table_2 锁，等待 session_2 完毕
select * from table_2 where id =1 for update;

--- session 2 申请 table_1 锁，等待 session_1 完毕
mysql> select * from table_1 where where id=1 for update;
```

发生死锁后，InnoDB 一般都能自动检测到，并使一个事务释放锁并回退，另一个事务获得锁，继续完成事务。但在涉及外部锁，或涉及表锁的情况下，InnoDB 并不能完全自动检测到死锁，这需要通过设置锁等待超时参数 innodb_lock_wait_timeout 来解决。需要说明的是，这个参数并不是只用来解决死锁问题，在并发访问比较高的情况下，如果大量事务因无法立即获得所需的锁而挂起，会占用大量计算机资源，造成严重性能问题，甚至拖跨数据库。我们通过设置合适的锁等待超时阈值，可以避免这种情况发生。

死锁都是应用设计的问题，通过调整业务流程、数据库对象设计、事务大小，以及访问数据库的 SQL 语句，绝大部分死锁都可以避免。

- 在应用中，如果不同的程序会并发存取多个表，应尽量约定以相同的顺序来访问表，这样可以大大降低产生死锁的机会。
- 在程序以批量方式处理数据的时候，如果事先对数据排序，保证每个线程按固定的顺序来处理记录，也可以大大降低出现死锁的可能。
- 在事务中，如果要更新记录，应该直接申请足够级别的锁，即排他锁，而不应先申请共享锁，更新时再申请排他锁，因为当用户申请排他锁时，其他事务可能又已经获得了相同记录的共享锁，从而造成锁冲突，甚至死锁。

MySQL 5.5 中，information_schema 库中新增了三个关于锁的表，亦即 innodb_trx、innodb_locks 和 innodb_lock_waits。其中 innodb_trx 表记录当前运行的所有事务，innodb_locks 表记录当前出现的锁，innodb_lock_waits 表记录锁等待的对应关系。

# Binlog

MySQL 通过 BINLOG 录执行成功的 INSERT、UPDATE、DELETE 等更新数据的 SQL 语句，并由此实现 MySQL 数据库的恢复和主从复制。

- 一是 MySQL 的恢复是 SQL 语句级的，也就是重新执行 BINLOG 中的 SQL 语句。这与 Oracle 数据库不同，Oracle 是基于数据库文件块的。
- 二是 MySQL 的 Binlog 是按照事务提交的先后顺序记录的，恢复也是按这个顺序进行的。这点也与 Oralce 不同，Oracle 是按照系统更新号（System Change Number，SCN）来恢复数据的，每个事务开始时，Oracle 都会分配一个全局唯一的 SCN，SCN 的顺序与事务开始的时间顺序是一致的。

从上面两点可知，MySQL 的恢复机制要求：在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读，这已经超过了 ISO/ANSI SQL92“可重复读”隔离级别的要求，实际上是要求事务要串行化。这也是许多情况下，InnoDB 要用到间隙锁的原因，比如在用范围条件更新记录时，无论在 Read Commited 或是 Repeatable Read 隔离级别下，InnoDB 都要使用间隙锁，但这并不是隔离级别要求的，有关 InnoDB 在不同隔离级别下加锁的差异在下一小节还会介绍。

# 索引

联合索引中，索引字段的数据必须是有序的，才能实现这种类型的查找，才能利用到索引。

```sql
--- UNIQUE KEY `unique_product_in_category` (`name`,`category`) USING BTREE,
--- key 为 unique_name_in_category
EXPLAIN SELECT * FROM product WHERE name = '产品一'；
--- key 为 null
EXPLAIN SELECT * FROM product WHERE category = '类目一'；
```

- SYSTEM，CONST 的特例，当表上只有一条元组匹配

- CONST，WHERE 条件筛选后表上至多有一条元组匹配时，比如 WHERE ID = 2 （ID 是主键，值为 2 的要么有一条要么没有）

- EQ_REF，参与连接运算的表是内表（在代码实现的算法中，两表连接时作为循环中的内循环遍历的对象，这样的表称为内表）。

基于索引（连接字段上存在唯一索引或者主键索引，且操作符必须是“=”谓词，索引值不能为 NULL）做扫描，使得对外表的一条元组，内表只有唯一一条元组与之对应。

（4）REF
可以用于单表扫描或者连接。参与连接运算的表，是内表。

基于索引（连接字段上的索引是非唯一索引，操作符必须是“=”谓词，连接字段值不可为 NULL）做扫描，使得对外表的一条元组，内表可有若干条元组与之对应。

（5）REF_OR_NULL
类似 REF，只是搜索条件包括：连接字段的值可以为 NULL 的情况，比如 where col = 2 or col is null

（6）RANGE
范围扫描，基于索引做范围扫描，为诸如 BETWEEN，IN，>=，LIKE 类操作提供支持

（7）INDEX_SCAN
索引做扫描，是基于索引在索引的叶子节点上找满足条件的数据（不需要访问数据文件）

（8）ALL
全表扫描或者范围扫描：不使用索引，顺序扫描，直接读取表上的数据（访问数据文件）

（9）UNIQUE_SUBQUERY
在子查询中，基于唯一索引进行扫描，类似于 EQ_REF

（10）INDEX_SUBQUERY
在子查询中，基于除唯一索引之外的索引进行扫描

（11）INDEX_MERGE
多重范围扫描。两表连接的每个表的连接字段上均有索引存在且索引有序，结果合并在一起。适用于作集合的并、交操作。

（12）FT
FULL TEXT，全文检索

# Tuning | 性能调优

MySQL 中常见的性能问题，可能是有如下类型：

- CPU 过载，慢查询，OPTIMZE TABLE
- 内存使用问题，内存相关参数配置不合理
- 磁盘 I/O，Buffer pool 命中率；慢查询； redo, undo, data 分开存放
- 网络问题，非专有网络，网络路由
- 表及查询语句问题，主要的查询性能问题：全表扫描 临时表 排序 FileSort

常用定位问题的方法则有：

- 分析状态变量
- MySQL Slow query log
- EXPLAIN 命令查看执行计划
- profiling 查看执行过耗时
- show full processlist
- show engine innodb status
- 查看 innodb 系统表

## 配置

- innodb_buffer_pool_size, 一般设置为机器内存的 50%左右(实践经验)

- innodb_buffer_pool_instances, 5.6 及 5.7，可设置为 8-16 个

- innodb_log_file_size
  一般设置为 25% 的 buffer pool size 大小
- innodb_flush_log_at_trx_commit
  要求高可靠性，设置为 1。不要求高可靠性，可设置为 0 或 2.
- sync_binlog
  binlog 的可靠性设置，高可靠性设置为 1，但对于性能影响比较大。如果已经配置了 Slave，这个参数可设置为 0
- innodb_flush_method
  Linux 下设置为 O_DIRECT

- innodb_thread_concurrency
  (Cores \* 2) + (# Disks)
- skip_name_resolve
  使用直接 IP 方式，避免 DNS 解析
- innodb_io_capacity， innodb_io_capacity_max
  需要根据你的磁盘的 IOPS 处理能力进行相应设置。
  innodb_io_capacity~= IOPS
- query_cache_type
  是否使用 Query Cache，对于读/写， 80%+/20%-的应用可考虑打开。写入请求过多的应用，需要关闭，不然反而影响性能。
- tmp_table_size/ max_heap_table_size
  通过查看状态变量 Created_tmp_disk_tables 及 Created_tmp_tables，决定是否合适

## explain | 分析

在日常工作中，我们会有时会开慢查询去记录一些执行时间比较久的 SQL 语句，找出这些 SQL 语句并不意味着完事了，些时我们常常用到 explain 这个命令来查看一个这些 SQL 语句的执行计划，查看该 SQL 语句有没有使用上了索引，有没有做全表扫描，这都可以通过 explain 命令来查看。

```sh
mysql> explain select * from (select * from ( select * from t1 where id=2602) a) b;
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
| id | select_type | table      | type   | possible_keys     | key     | key_len | ref  | rows | Extra |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
|  1 | PRIMARY     | <derived2> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  2 | DERIVED     | <derived3> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  3 | DERIVED     | t1         | const  | PRIMARY,idx_t1_id | PRIMARY | 4       |      |    1 |       |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
```

当我们的查询语句包含了多个 Select 语句时，explain 会依次返回每个子查询的信息，其中 select_type 就是表示每个 select 子句的类型：

- SIMPLE: 简单 SELECT,不使用 UNION 或子查询等

- PRIMARY: 查询中若包含任何复杂的子部分,最外层的 select 被标记为 PRIMARY

- UNION: UNION 中的第二个或后面的 SELECT 语句

- DEPENDENT UNION: UNION 中的第二个或后面的 SELECT 语句，取决于外面的查询

- UNION RESULT: UNION 的结果

- SUBQUERY: 子查询中的第一个 SELECT

- DEPENDENT SUBQUERY: 子查询中的第一个 SELECT，取决于外面的查询

- DERIVED: 派生表的 SELECT, FROM 子句的子查询

- UNCACHEABLE SUBQUERY: 一个子查询的结果不能被缓存，必须重新评估外链接的第一行

type 表示 MySQL 在表中找到所需行的方式，又称“访问类型”。常用的类型有： ALL, index, range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好）：

- ALL：Full Table Scan， MySQL 将遍历全表以找到匹配的行

- index: Full Index Scan，index 与 ALL 区别为 index 类型只遍历索引树

- range:只检索给定范围的行，使用一个索引来选择行

- ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

- eq_ref: 类似 ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用 primary key 或者 unique key 作为关联条件

- const, system: 当 MySQL 对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于 where 列表中，MySQL 就能将该查询转换为一个常量,system 是 const 类型的特例，当查询的表只有一行的情况下，使用 system

- NULL: MySQL 在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

possible_keys, 指出 MySQL 能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。key 列显示 MySQL 实际决定使用的键（索引），如果没有选择索引，键是 NULL。要想强制 MySQL 使用或忽视 possible_keys 列中的索引，在查询中使用 FORCE INDEX、USE INDEX 或者 IGNORE INDEX。key_len 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len 显示的值为索引字段的最大可能长度，并非实际使用长度，即 key_len 是根据表定义计算而得，不是通过表内检索出的）；不损失精确性的情况下，长度越短越好。

最后的 Extra 列包含 MySQL 解决查询的详细信息,有以下几种情况：

- Using where: 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示 mysql 服务器将在存储引擎检索行后再进行过滤

- Using temporary: 表示 MySQL 需要使用临时表来存储结果集，常见于排序和分组查询

- Using filesort: MySQL 中无法利用索引完成的排序操作称为“文件排序”

- Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进性能。

- Impossible where：这个值强调了 where 语句会导致没有符合条件的行。

- Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行

## Security | 安全

### SQL Injection

# Todos

- [ ] https://www.jianshu.com/p/486a514b0ded

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/8/1/%25E6%259E%2584%25E5%25BB%25BA%25E9%25AB%2598%25E6%2580%25A7%25E8%2583%25BDMySQL%25E4%25BD%2593%25E7%25B3%25BB.jpg)

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/8/1/MySQL%25E6%2595%25B0%25E6%258D%25AE%25E5%25AE%2589%25E5%2585%25A8%25E6%2580%25A7%25E8%25AE%25A8%25E8%25AE%25BA.jpg)
