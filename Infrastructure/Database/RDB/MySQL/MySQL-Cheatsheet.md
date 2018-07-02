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
