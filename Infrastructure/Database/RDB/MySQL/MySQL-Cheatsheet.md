[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# MySQL

```
SELECT
  TABLE_NAME, table_rows, data_length, index_length,
  round(((data_length + index_length) / 1024 / 1024),2) 'Size in MB'
FROM information_schema.TABLES
WHERE TABLE_NAME = 'r_case_info' and TABLE_TYPE='BASE TABLE'
ORDER BY data_length DESC;
```

# 权限管理

```
SELECT User FROM mysql.user;


SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
```

## Query

* 根据时间日期进行筛选

```
# 使用 DATE 函数提取出时间
SELECT `tag`
FROM `tags`
WHERE DATE(`date`) = '2011-06-07'


# 使用 Between 获取某天内的时间
WHERE `date`
BETWEEN '2011-06-07'
AND '2011-06-07 23:59:59'
```

# Oracle

## Query

* 获取前 N 行

```
SELECT val
FROM   rownum_order_test
ORDER BY val DESC
FETCH FIRST 5 ROWS ONLY;
```

* 获取前 N 行，如果第 N 行有多个相同值，则全部返回

```
SELECT val
FROM   rownum_order_test
ORDER BY val DESC
FETCH FIRST 5 ROWS WITH TIES;
```

* 获取前 `x%` 行：

```
SELECT val
FROM   rownum_order_test
ORDER BY val
FETCH FIRST 20 PERCENT ROWS ONLY;
```

* 根据偏移量查询，适用于需要分页的情况：

```
SELECT val
FROM   rownum_order_test
ORDER BY val
OFFSET 4 ROWS FETCH NEXT 4 ROWS ONLY;
```

* 综合使用偏移量与百分比查询：

```
SELECT val
FROM   rownum_order_test
ORDER BY val
OFFSET 4 ROWS FETCH NEXT 20 PERCENT ROWS ONLY;
```

![](https://cdn-images-1.medium.com/max/800/0*AhVo_3sCq-ft64ki.jpg)

# MySQL 命令速览与实践清单

MySQL is a full-featured open-source relational database management system (RDBMS) that was originally built by MySQL AB and currently owned by Oracle Corporation. It stores data in tables that are grouped into a database, uses Structured Query Language (SQL) to access data and such commands as ‘SELECT’, ‘UPDATE’, ‘INSERT’ and ‘DELETE’ to manage it. Related information can be stored in different tables, but the usage of JOIN operation allows you to correlate it, perform queries across various tables and minimize the chance of data duplication.

MySQL is compatible with nearly all operating systems, namely Windows, Linux, Unix, Apple, FreeBSD and many others. It supports various storage engines, like InnoDB (it is the default one), Federated, MyISAM, Memory, CSV, Archive, Blackhole and Merge.

本清单参考了 [MySQL Command Line Cheatsheet](https://gist.github.com/hofmannsven/9164408)、[MySQL 5.7 Reference Manual](https://parg.co/UpB)，更多 MySQL 的学习资料参考 [MySQL 学习与资料索引]()。此外，本文中涉及的 SQL 语句可以前往 [SQLFiddle](http://sqlfiddle.com/#!9/60f53) 查看完整的 SQL 语句，也可以前往 [SQL Teaching](https://www.sqlteaching.com/) 进行交互式基础语法学习。

# Database Management: 数据库连接与管理

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

# DDL: 数据定义

```sql
CREATE TABLE PRODUCTS
(
       PRODUCT_ID     INTEGER,
       PRODUCT_NAME   VARCHAR(30)
);
CREATE TABLE SALES
(
       SALE_ID        INTEGER,
       PRODUCT_ID     INTEGER,
       DATE           TIMESTAMP,
       Quantity       INTEGER,
       PRICE          INTEGER
);
INSERT INTO PRODUCTS VALUES ( 100, 'Nokia');
...
INSERT INTO SALES VALUES ( 1, 100, '2015-10-01 00:00:00', 25, 5000);
...
COMMIT;
```

# Query: 数据查询

# DML: 数据管理

# Security: 安全

## SQL Injection

