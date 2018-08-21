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
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
 FLUSH PRIVILEGES;
```

# 数据库配置与管理

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
```

```sql
SELECT User FROM mysql.user;

SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
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

# Security | 安全

## SQL Injection

# Optimization | 性能调优

 CPU 过载 –- 慢查询， OPTIMZE TABLE
 内存使用问题 –- 内存相关参数配置不合理
 磁盘 I/O -- Buffer pool 命中率；慢查询； redo, undo, data 分开存放
 网络问题 – 非专有网络，网络路由
 表及查询语句问题

 DBT2
– http://osdldbt.sourceforge.net/
– http://samurai-mysql.blogspot.com/2009/03/settingup-dbt-2.html
 mysqlslap MySQL 5.1 +
– http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html
 SysBench
– http://sysbench.sourceforge.net/
 supersmack
– http://vegan.net/tony/supersmack/
 mybench
– http://jeremy.zawodny.com/mysql/mybench/

 innodb_buffer_pool_size
一般设置为机器内存的 50%左右(实践经验)
 innodb_buffer_pool_instances
5.6 及 5.7，可设置为 8-16 个
 innodb_log_file_size
一般设置为 25% 的 buffer pool size 大小
 innodb_flush_log_at_trx_commit
要求高可靠性，设置为 1。不要求高可靠性，可设置为 0 或 2.
 sync_binlog
binlog 的可靠性设置，高可靠性设置为 1，但对于性能影响比较大。如果已经配置了 Slave，这个参数可设置为 0
 innodb_flush_method
Linux 下设置为 O_DIRECT

 innodb_thread_concurrency
(Cores \* 2) + (# Disks)
 skip_name_resolve
使用直接 IP 方式，避免 DNS 解析
 innodb_io_capacity， innodb_io_capacity_max
需要根据你的磁盘的 IOPS 处理能力进行相应设置。
innodb_io_capacity~= IOPS
 query_cache_type
是否使用 Query Cache，对于读/写， 80%+/20%-的应用可考虑打开。写入请求过多的应用，需要关闭，不然反而影响性能。
 tmp_table_size/ max_heap_table_size
通过查看状态变量 Created_tmp_disk_tables 及 Created_tmp_tables，决定是否合适

常用定位问题的方法
 分析状态变量
 MySQL Slow query log
 EXPLAIN 命令查看执行计划
 profiling 查看执行过耗时
 show full processlist
 show engine innodb status
 查看 innodb 系统表

主要的查询性能问题：全表扫描 临时表 排序 FileSort

# Store Engine | 存储引擎

## 锁

InnoDB 行锁是通过给索引上的索引项加锁来实现的，这一点 MySQL 与 Oracle 不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB 这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB 才使用行级锁，否则，InnoDB 将使用表锁！

# Database Design | 数据库设计
