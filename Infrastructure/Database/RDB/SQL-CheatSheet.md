[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# SQL 语法速览与备忘清单

SQL 是 Structrued Query Language 的缩写，即结构化查询语言。它是负责与 ANSI(美国国家标准学会)维护的数据库交互的标准。作为关系数据库的标准语言，它已被众多商用 DBMS 产品所采用，使得它已成为关系数据库领域中一个主流语言，不仅包含数据查询功能，还包括插入、删除、更新和数据定义功能。最为重要的 SQL92 版本的详细标准可以查看[这里](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)，或者在 [Wiki](https://en.wikipedia.org/wiki/SQL) 上查看 SQL 标准的变化。

- T-SQL 是 SQL 语言的一种版本，且只能在 SQL SERVER 上使用。它是 ANSI SQL 的加强版语言、提供了标准的 SQL 命令。另外，T-SQL 还对 SQL 做了许多补允，提供了数据库脚本语言，即类似 C、Basic 和 Pascal 的基本功能，如变量说明、流控制语言、功能函数等。

- PL-SQL(Procedural Language-SQL)是一种增加了过程化概念的 SQL 语言，是 Oracle 对 SQL 的扩充。与标准 SQL 语言相同，PL-SQL 也是 Oracle 客户端工具(如 SQL\*Plus、Developer/2000 等)访问服务器的操作语言。它有标准 SQL 所没有的特征：变量(包括预先定义的和自定义的)；控制结构(如 IF-THEN-ELSE 等流控制语句)；自定义的存储过程和函数 ；对象类型等。由于 P/L-SQL 融合了 SQL 语言的灵活性和过程化的概念，使得 P/L-SQL 成为了一种功能强大的结构化语言，可以设计复杂的应用。

作为查询语言，与普通编程语言相比，它还处于业务上层；SQL 最终会转化为关系代数执行，但关系代数会遵循一些等价的转换规律，比如交换律、结合律、过滤条件拆分等等，通过预估每一步的时间开销，将 SQL 执行顺序重新组合，可以提高执行效率。如果有多个 SQL 同时执行，还可以整合成一个或多个新的 SQL，合并重复的查询请求；在数据驱动商业的今天，SQL 依然是数据查询最通用的解决方案。

# Data Definition Language | 数据定义

DDL 包含 CREATE, ALTER, DROP 等常见的数据定义语句

[完整的表结构 SQL](https://gist.github.com/wxyyxc1992/ebd1ceb919a68e428e7901f7fc766f02)

```sql
CREATE TABLE `product` (
  `_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '_ID，内部自增编号',
  `code` varchar(6) DEFAULT NULL,
  `name` varchar(15) DEFAULT NULL,
  `category` varchar(15) DEFAULT NULL,
  `price` decimal(4,2) DEFAULT NULL,
  `created_at` datetime DEFAULT NULL,
  `updated_at` datetime DEFAULT NULL,
  `deleted_at` datetime DEFAULT NULL,
  PRIMARY KEY (`_id`),
  UNIQUE KEY `unique_product_in_category` (`name`,`category`) USING BTREE,
  KEY `code` (`code`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4
```

## 表与索引规约

参考阿里的 [p3c](https://github.com/alibaba/p3c) 规范。

### 命名

表名、字段名必须使用小写字母或数字，禁止出现数字开头，禁止两个下划线中间只出现数字。数据库字段名的修改代价很大，因为无法进行预发布，所以字段名称需要慎重考虑。 MySQL在Windows下不区分大小写，但在Linux下默认是区分大小写。因此，数据库名、表名、字段名，都不允许出现任何大写字母，避免节外生枝。 

表名应该仅仅表示表里面的实体内容，不应该表示实体数量，对应于DO类名也是单数形式，不使用复数名词，符合表达习惯。

表达是与否概念的字段，必须使用is_xxx的方式命名，数据类型是unsigned tinyint（ 1表示是，0表示否）。 

表必备三字段：id, gmt_create, gmt_modified。 
说明：其中id必为主键，类型为unsigned bigint、单表时自增、步长为1。gmt_create, gmt_modified的类型均为datetime类型，前者现在时表示主动创建，后者过去分词表示被动更新。

单表行数超过500万行或者单表容量超过2GB，才推荐进行分库分表。

### 字段

任何字段如果为非负数，必须是unsigned。 

小数类型为decimal，禁止使用float和double。 float和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。

### 索引

主键索引名为pk_字段名；唯一索引名为uk_字段名；普通索引名则为idx_字段名。 

# Data Manipulation Language | 数据操作

DML 包含了 INSERT, UPDATE, DELETE 等常见的数据操作语句。

## Update | 更新

### 存在性更新

我们经常需要处理某个唯一索引时存在则更新，不存在则插入的情况，其基本形式如下：

```sql
INSERT INTO ... ON DUPLICATE KEY UPDATE ...
```

对于多属性索引的更新方式如下：

```sql
/* 创建语句中添加索引描述 */
UNIQUE INDEX `index_var` (`var1`, `var2`, `var3`)

/* 同时更新索引包含的多属性域值 */
INSERT INTO `test_table`
(`var1`, `var2`, `var3`, `value1`, `value2`, `value3`) VALUES
('abcd', 0, 'xyz', 1, 2, 3)
ON DUPLICATE KEY UPDATE `value1` = `value1` + 1 AND
`value2` = `value2` + 2 AND `value3` = `value3` + 3;
```

# DQL | 数据查询

## Column | 查询列

```sql
CASE expression
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
   ...
    WHEN conditionN THEN resultN
    ELSE result
END as field_name
```

## Where | 条件查询

```sql
expression IS NOT NULL

SELECT *
FROM contacts
WHERE last_name IS NOT NULL;
```

### 分页查询

```sql
SELECT column1, column2, ...
FROM table_name
LIMIT offset, count;

SELECT *
FROM yourtable
ORDER BY id
LIMIT 100, 20
```

```sql
SELECT *
FROM yourtable
WHERE id > 234374
ORDER BY id
LIMIT 20
```

## 统计查询

不要使用count(列名)或count(常量)来替代count()，count()是SQL92定义的标准统计行数的语法，跟数据库无关，跟NULL和非NULL无关。count(*)会统计值为NULL的行，而count(列名)不会统计此列为NULL值的行。

count(distinct col) 计算该列除NULL之外的不重复行数，注意 count(distinct col1, col2) 如果其中一列全为NULL，那么即使另一列有不同的值，也返回为0。

## Join | 表联接

表联接最常见的即是出现在查询模型中，但是实际的用法绝不会局限在查询模型中。较常见的联接查询包括了以下几种类型：Inner Join  / Outer Join / Full Join / Cross Join 。

### Inner Join | 内联查询

Inner Join 是最常用的 Join 类型，基于一个或多个公共字段把记录匹配到一起。Inner Join 只返回进行联结字段上匹配的记录。  如：

```sql
select * from Products inner join Categories on Products.categoryID=Categories.CategoryID
```

以上语句，只返回物品表中的种类 ID 与种类表中的 ID 相匹配的记录数。这样的语句就相当于：

```sql
select * from Products, Categories where Products.CategoryID=Categories.CategoryID
```

Inner Join 是在做排除操作，任一行在两个表中不匹配，注定将从结果集中除掉。(我想，相当于两个集合中取其两者的交集，这个交集的条件就是 on 后面的限定)还要注意的是，不仅能对两个表作联结，可以把一个表与其自身进行联结。

## Outer Join | 外联接

Outer Join 包含了 Left Outer Join 与 Right Outer Join. 其实简写可以写成 Left Join 与 Right Join。left join，right join 要理解并区分左表和右表的概念，A 可以看成左表,B 可以看成右表。left join 是以左表为准的.,左表(A)的记录将会全部表示出来,而右表(B)只会显示符合搜索条件的记录(例子中为: A.aID = B.bID).B 表记录不足的地方均为 NULL。 right join 和 left join 的结果刚好相反,这次是以右表(B)为基础的,A 表不足的地方用 NULL 填充。

## Full Join | 全连接

Full Join 相当于把 Left 和 Right 联结到一起，告诉数据库要全部包含左右两侧所有的行，相当于做集合中的并集操作。

## Cross Join | 笛卡尔积

与其它的 JOIN 不同在于，它没有 ON 操作符，它将 JOIN 一侧的表中每一条记录与另一侧表中的所有记录联结起来，得到的是两侧表中所有记录的笛卡儿积。

## 子查询

子查询本质上是嵌套进其他 SELECT, UPDATE, INSERT, DELETE 语句的一个被限制的 SELECT 语句,在子查询中，只有下面几个子句可以使用：

- SELECT 子句(必须)
- FROM 子句(必选)
- WHERE 子句(可选)
- GROUP BY(可选)
- HAVING(可选)

子查询也可以嵌套在其他子查询中，子查询也叫内部查询(Inner query)或者内部选择(Inner Select),而包含子查询的查询语句也叫做外部查询(Outter)或者外部选择(Outer Select)。

### 子查询作为数据源使用

当子查询在外部查询的 FROM 子句之后使用时,子查询被当作一个数据源使用,即使这时子查询只返回一个单一值(Scalar)或是一列值(Column)，在这里依然可以看作一个特殊的数据源,即一个二维数据表(Table)。作为数据源使用的子查询很像一个视图(View),只是这个子查询只是临时存在，并不包含在数据库中。

### 子查询作为选择条件使用

作为选择条件的子查询也是子查询相对最复杂的应用。作为选择条件的子查询是那些只返回一列(Column)的子查询，如果作为选择条件使用，即使只返回单个值，也可以看作是只有一行的一列。譬如我们需要查询价格高于某个指定产品的所有其余产品信息：

```sql
SELECT
	*
FROM
	product
WHERE
	price > (
		SELECT
			price
		FROM
			product
		WHERE
			NAME = "产品一"
	)
```

### 子查询作为计算列使用

当子查询作为计算列使用时，只返回单个值(Scalar)，其用在 SELECT 语句之后，作为计算列使用，同样分为相关子查询和无关子查询。

```sql
--- 查询每个类别中价格大于某个值的产品数目
SELECT
	p1.category,
	(
		SELECT
			count(*)
		FROM
			product p2
		WHERE
			p2.category = p1.category
		AND p2.price > 30
	) AS 'Expensive'
FROM
	product p1
GROUP BY
	p1.category;
```

```sql
--- 自连接查询不同等级的数目
SELECT a.distributor_id,
      (SELECT COUNT(*) FROM my_table WHERE level='personal' and distributor_id = a.distributor_id) as personal_count,
      (SELECT COUNT(*) FROM my_table WHERE level='exec' and distributor_id = a.distributor_id) as exec_count,
      (SELECT COUNT(*) FROM my_table WHERE distributor_id = a.distributor_id) as total_count
FROM my_table a ;
```

# Database Design | 数据库设计

![image](https://user-images.githubusercontent.com/5803001/46092422-70455c00-c1e7-11e8-80b8-2b1c8520c4ff.png)

# Max Compute/ODPS

MaxCompute SQL 适用于海量数据（GB、TB、EB 级别），离线批量计算的场合。MaxCompute 作业提交后会有几十秒到数分钟不等的排队调度，所以适合处理跑批作业，一次作业批量处理海量数据，不适合直接对接需要每秒处理几千至数万笔事务的前台业务系统。

## DDL

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[STORED BY StorageHandler] -- 仅限外部表
[WITH SERDEPROPERTIES (Options)] -- 仅限外部表
[LOCATION OSSLocation];-- 仅限外部表
[LIFECYCLE days]
[AS select_statement]

create table if not exists sale_detail
(
	shop_name     string,
	customer_id   string,
	total_price   double
)
partitioned by (sale_date string,region string);
-- 创建一张分区表sale_detail

CREATE TABLE [IF NOT EXISTS] table_name
LIKE existing_table_name

create table sale_detail_ctas1 as
select * from sale_detail;
```

Partitioned by 指定表的分区字段，目前支持 Tinyint、Smallint、 Int、 Bigint、Varchar 和 String 类型。分区值不允许有双字节字符（如中文），必须是以英文字母 a-z，A-Z 开始后可跟字母数字，名称的长度不超过 128 字节。当利用分区字段对表进行分区时，新增分区、更新分区内数据和读取分区数据均不需要做全表扫描，可以提高处理效率。

在 create table…as select…语句中，如果在 select 子句中使用常量作为列的值，建议指定列的名字；否则创建的表 sale_detail_ctas3 的第四、五列类似于\_c5、\_c6。

```sql
--- 删除表
DROP TABLE [IF EXISTS] table_name;

--- 重命名表
ALTER TABLE table_name RENAME TO new_table_name;
```

## Select | 查询

### Join

```sql
--- 左连接
select a.shop_name as ashop, b.shop_name as bshop from shop a
        left outer join sale_detail b on a.shop_name=b.shop_name;
    -- 由于表shop及sale_detail中都有shop_name列，因此需要在select子句中使用别名进行区分。

--- 右连接
select a.shop_name as ashop, b.shop_name as bshop from shop a
        right outer join sale_detail b on a.shop_name=b.shop_name;

--- 全连接
select a.shop_name as ashop, b.shop_name as bshop from shop a
        full outer join sale_detail b on a.shop_name=b.shop_name;
```

连接条件，只允许 and 连接的等值条件。只有在 MAPJOIN 中，可以使用不等值连接或者使用 or 连接多个条件。

### Map Join

当一个大表和一个或多个小表做 Join 时，可以使用 MapJoin，性能比普通的 Join 要快很多。MapJoin 的基本原理为：在小数据量情况下，SQL 会将您指定的小表全部加载到执行 Join 操作的程序的内存中，从而加快 Join 的执行速度。

![image](https://user-images.githubusercontent.com/5803001/47965355-15721080-e081-11e8-8e33-ad18258c6d9f.png)

MapJoin 简单说就是在 Map 阶段将小表读入内存，顺序扫描大表完成 Join；以 Hive MapJoin 的原理图为例，可以看出 MapJoin 分为两个阶段：

- 通过 MapReduce Local Task，将小表读入内存，生成 HashTableFiles 上传至 Distributed Cache 中，这里会对 HashTableFiles 进行压缩。

- MapReduce Job 在 Map 阶段，每个 Mapper 从 Distributed Cache 读取 HashTableFiles 到内存中，顺序扫描大表，在 Map 阶段直接进行 Join，将数据传递给下一个 MapReduce 任务。

```sql
select /* + mapjoin(a) */
        a.shop_name,
        b.customer_id,
        b.total_price
    from shop a join sale_detail b
    on a.shop_name = b.shop_name;
```

left outer join 的左表必须是大表，right outer join 的右表必须是大表，inner join 左表或右表均可以作为大表，full outer join 不能使用 MapJoin。

### Subquery | 子查询

在 from 子句中，子查询可以当作一张表来使用，与其它的表或子查询进行 Join 操作，子查询必须要有别名。

```sql
create table shop as select * from sale_detail;

--- 子查询作为表
select a.shop_name, a.customer_id, a.total_price from
(select * from shop) a join sale_detail on a.shop_name = sale_detail.shop_name;

--- IN SUBQUERY / NOT IN SUBQUERY
SELECT * from mytable1 where id in (select id from mytable2);
--- 等效于
SELECT * from mytable1 a LEFT SEMI JOIN mytable2 b on a.id=b.id;

--- EXISTS SUBQUERY/NOT EXISTS SUBQUERY
SELECT * from mytable1 where not exists (select * from mytable2 where id = mytable1.id);
--- 等效于
SELECT * from mytable1 a LEFT ANTI JOIN mytable2 b on a.id=b.id;

--- SCALAR SUBQUERY
select * from t1 where (select count(*)  from t2 where t1.a = t2.a) > 1;
-- 等效于
select t1.* from t1 left semi join (select a, count(*) from t2 group by a having count(*) > 1) t2 on t1 .a = t2.a;
```

## UDF

```java
package org.alidata.odps.udf.examples;
  import com.aliyun.odps.udf.UDF;

public final class Lower extends UDF {
  public String evaluate(String s) {
    if (s == null) {
        return null;
    }
        return s.toLowerCase();
  }
}
```

# 数据建模

每个 DBMS，无论是 NoSQL 还是 SQL，最终，都是把无意义的物理状态（高电压和低电压，或者开和关）和有意义的事物建立映射关系，从而表示数据。我们把这个映射称为物理表示。在更高的层次上，我们使用表、图形和文档等结构来表示关系。理解的关键是逻辑数据模型应该完全忽略这些物理映射问题。逻辑数据模型应该把重点完全放在数据的含义上以及数据如何按照逻辑表示问题域内的数据。但是，在从逻辑模型转移到物理模型时，保留从物理模型到逻辑模型的映射关系以及物理表示设计都变得至关重要了。
