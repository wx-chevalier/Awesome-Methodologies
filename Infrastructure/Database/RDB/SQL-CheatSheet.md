[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# SQL 语法速览与备忘清单

SQL 是 Structrued Query Language 的缩写，即结构化查询语言。它是负责与 ANSI(美国国家标准学会)维护的数据库交互的标准。作为关系数据库的标准语言，它已被众多商用 DBMS 产品所采用，使得它已成为关系数据库领域中一个主流语言，不仅包含数据查询功能，还包括插入、删除、更新和数据定义功能。最为重要的 SQL92 版本的详细标准可以查看[这里](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)，或者在 [Wiki](https://en.wikipedia.org/wiki/SQL) 上查看 SQL 标准的变化。

- T-SQL 是 SQL 语言的一种版本，且只能在 SQL SERVER 上使用。它是 ANSI SQL 的加强版语言、提供了标准的 SQL 命令。另外，T-SQL 还对 SQL 做了许多补允，提供了数据库脚本语言，即类似 C、Basic 和 Pascal 的基本功能，如变量说明、流控制语言、功能函数等。

- PL-SQL(Procedural Language-SQL)是一种增加了过程化概念的 SQL 语言，是 Oracle 对 SQL 的扩充。与标准 SQL 语言相同，PL-SQL 也是 Oracle 客户端工具(如 SQL\*Plus、Developer/2000 等)访问服务器的操作语言。它有标准 SQL 所没有的特征：变量(包括预先定义的和自定义的)；控制结构(如 IF-THEN-ELSE 等流控制语句)；自定义的存储过程和函数 ；对象类型等。由于 P/L-SQL 融合了 SQL 语言的灵活性和过程化的概念，使得 P/L-SQL 成为了一种功能强大的结构化语言，可以设计复杂的应用。

SQL 与其他函数类查询语言不在一个层面上，如果用语法糖、可操纵性抨击 SQL，只能得出看似正确，实则荒谬的结论。

SQL 是一个查询语言，与普通编程语言相比，它还在上层，最终会转化为关系代数执行，但关系代数会遵循一些等价的转换规律，比如交换律、结合律、过滤条件拆分等等，通过预估每一步的时间开销，将 SQL 执行顺序重新组合，可以提高执行效率。

如果有多个 SQL 同时执行，还可以整合成一个或多个新的 SQL，合并重复的查询请求。

在数据驱动商业的今天，SQL 依然是数据查询最通用的解决方案。

# Data Definition Language | 数据定义

DDL 包含 CREATE, ALTER, DROP 等常见的数据定义语句

```sql
CREATE TABLE product
  (
    _id int NOT NULL AUTO_INCREMENT,
    code    VARCHAR (6),
    name    VARCHAR (15),
    price     DECIMAL(4,2),
    created_at DATE,
    updated_at DATE,
    deleted_at DATE,
    PRIMARY KEY (_id)
  );
```

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

当子查询在外部查询的 FROM 子句之后使用时,子查询被当作一个数据源使用,即使这时子查询只返回一个单一值(Scalar)或是一列值(Column)，在这里依然可以看作一个特殊的数据源,即一个二维数据表(Table).作为数据源使用的子查询很像一个 View(视图),只是这个子查询只是临时存在，并不包含在数据库中。比如：

### 子查询作为选择条件使用

作为选择条件的子查询也是子查询相对最复杂的应用。作为选择条件的子查询是那些只返回一列(Column)的子查询，如果作为选择条件使用，即使只返回单个值，也可以看作是只有一行的一列。

### 子查询作为计算列使用

当子查询作为计算列使用时，只返回单个值(Scalar) 。用在 SELECT 语句之后，作为计算列使用。同样分为相关子查询和无关子查询。

```sql
SELECT a.distributor_id,
      (SELECT COUNT(*) FROM my_table WHERE level='personal' and distributor_id = a.distributor_id) as personal_count,
      (SELECT COUNT(*) FROM my_table WHERE level='exec' and distributor_id = a.distributor_id) as exec_count,
      (SELECT COUNT(*) FROM my_table WHERE distributor_id = a.distributor_id) as total_count
FROM my_table a ;
```

# Database Design | 数据库设计
