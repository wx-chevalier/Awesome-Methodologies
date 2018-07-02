[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# SQL 语法速览与备忘清单

SQL 是 Structrued Query Language 的缩写，即结构化查询语言。它是负责与 ANSI(美国国家标准学会)维护的数据库交互的标准。作为关系数据库的标准语言，它已被众多商用 DBMS 产品所采用，使得它已成为关系数据库领域中一个主流语言，不仅包含数据查询功能，还包括插入、删除、更新和数据定义功能。最为重要的 SQL92 版本的详细标准可以查看[这里](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)，或者在 [Wiki](https://en.wikipedia.org/wiki/SQL) 上查看 SQL 标准的变化。

- T-SQL 是 SQL 语言的一种版本，且只能在 SQL SERVER 上使用。它是 ANSI SQL 的加强版语言、提供了标准的 SQL 命令。另外，T-SQL 还对 SQL 做了许多补允，提供了数据库脚本语言，即类似 C、Basic 和 Pascal 的基本功能，如变量说明、流控制语言、功能函数等。

- PL-SQL(Procedural Language-SQL)是一种增加了过程化概念的 SQL 语言，是 Oracle 对 SQL 的扩充。与标准 SQL 语言相同，PL-SQL 也是 Oracle 客户端工具(如 SQL\*Plus、Developer/2000 等)访问服务器的操作语言。它有标准 SQL 所没有的特征：变量(包括预先定义的和自定义的)；控制结构(如 IF-THEN-ELSE 等流控制语句)；自定义的存储过程和函数 ；对象类型等。由于 P/L-SQL 融合了 SQL 语言的灵活性和过程化的概念，使得 P/L-SQL 成为了一种功能强大的结构化语言，可以设计复杂的应用。

本文默认使用 [test_db](https://github.com/datacharmer/test_db) 作为标准表结构。

# DML | 数据库管理

# DQL | 数据查询

## Where | 条件查询

```sql
expression IS NOT NULL

SELECT *
FROM contacts
WHERE last_name IS NOT NULL;
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

## Full Join

Full Join 相当于把 Left 和 Right 联结到一起，告诉 SQL Server 要全部包含左右两侧所有的行，相当于做集合中的并集操作。

## Cross Join

与其它的 JOIN 不同在于，它没有 ON 操作符，它将 JOIN 一侧的表中每一条记录与另一侧表中的所有记录联结起来，得到的是两侧表中所有记录的笛卡儿积。

## 统计查询

```sql
SELECT a.distributor_id,
          (SELECT COUNT(*) FROM myTable WHERE level='personal' and distributor_id = a.distributor_id) as PersonalCount,
          (SELECT COUNT(*) FROM myTable WHERE level='exec' and distributor_id = a.distributor_id) as ExecCount,
          (SELECT COUNT(*) FROM myTable WHERE distributor_id = a.distributor_id) as TotalCount
       FROM myTable a ;
```

## 子查询

子查询本质上是嵌套进其他 SELECT,UPDATE,INSERT,DELETE 语句的一个被限制的 SELECT 语句,在子查询中，只有下面几个子句可以使用：

1.  SELECT 子句(必须)
2.  FROM 子句(必选)
3.  WHERE 子句(可选)
4.  GROUP BY(可选)
5.  HAVING(可选)
6.  ORDER BY(只有在 TOP 关键字被使用时才可用)

```
子查询也可以嵌套在其他子查询中,这个嵌套最多可达32层。子查询也叫内部查询(Inner query)或者内部选择(Inner Select),而包含子查询的查询语句也叫做外部查询(Outter)或者外部选择(Outer Select),子查询的概念可以简单用下图阐述:
```

![](http://images.cnblogs.com/cnblogs_com/CareySon/201107/201107181306039322.png)

### 子查询作为数据源使用

```
当子查询在外部查询的**FROM**子句之后使用时,子查询被当作一个**数据源**使用,即使这时子查询只返回一个单一值(Scalar)或是一列值(Column)，在这里依然可以看作一个特殊的**数据源**,即一个二维数据表(Table).作为数据源使用的子查询很像一个**View(视图),**只是这个子查询只是临时存在，并不包含在数据库中。比如：
```

```sql
SELECT     P.ProductID, P.Name, P.ProductNumber, M.Name AS ProductModelName
FROM         Production.Product AS P INNER JOIN
(SELECT     Name, ProductModelID
 FROM          Production.ProductModel) AS M
ON P.ProductModelID = M.ProductModelID
```

```
上述子查询语句将ProductModel表中的**子集**M,作为数据源(表)和Product表进行内连接。结果如下:
```

![2](http://images.cnblogs.com/cnblogs_com/CareySon/201107/201107181306074258.png)

```
作为**数据源**使用也是子查询最简单的应用。当然，当子查询作为数据源使用时，也分为**相关子查询**和**无关子查询。
```

### 子查询作为选择条件使用

作为选择条件的子查询也是子查询相对最复杂的应用。作为选择条件的子查询是那些只返回**一列(Column)**的子查询，如果作为选择条件使用，即使只返回**单个值**，也可以看作是只有**一行**的**一列.**比如:    在 AdventureWorks 中：我想取得总共请病假天数大于 68 小时的员工:

```sql
SELECT [FirstName]
      ,[MiddleName]
      ,[LastName]
  FROM [AdventureWorks].[Person].[Contact]
  WHERE ContactID IN
  (SELECT EmployeeID
  FROM [AdventureWorks].[HumanResources].[Employee]
  WHERE SickLeaveHours>68)
```

结果如下：

![3](http://images.cnblogs.com/cnblogs_com/CareySon/201107/201107181306101910.png)

```
上面的查询中，在IN关键字后面的子查询返回一列值作为**外部查询**的**选择条件**使用.同样的，与IN关键字的逻辑取反的NOT IN关键字，这里就不再阐述了   但是要强调的是，不要用IN和NOT IN关键字，这会引起很多潜在的问题，[这篇文章](http://wiki.lessthandot.com/index.php/Subquery_typo_with_using_in)对这个问题有着很好的阐述:。这篇文章的观点是永远不要再用IN和NOT IN关键字，我的观点是存在即合理，我认为只有在IN里面是固定值的时候才可以用IN和NOT IN，比如:
```

```
SELECT [FirstName]
      ,[MiddleName]
      ,[LastName]
  FROM [AdventureWorks].[Person].[Contact]
  WHERE ContactID  IN (25,33)
```

```
只有在上面这种情况下，使用IN和NOT IN关键字才是安全的，其他情况下，最好使用EXISTS,NOT EXISTS,JOIN关键字来进行替代. 除了IN之外，用于选择条件的关键字还有**ANY**和**ALL**,这两个关键字和其字面意思一样. 和"<",">",”="连接使用，比如上面用IN的那个子查询：我想取得总共请病假天数大于68小时的员工。 用ANY关键字进行等效的查询为:
```

```sql
SELECT [FirstName]
      ,[MiddleName]
      ,[LastName]
  FROM [AdventureWorks].[Person].[Contact]
  WHERE ContactID =ANY

  (SELECT EmployeeID
  FROM [AdventureWorks].[HumanResources].[Employee]
  WHERE SickLeaveHours>68)
```

### 子查询作为计算列使用

```
当**子查询**作为**计算列**使用时，只返回单个值(Scalar) 。用在SELECT语句之后，作为计算列使用。同样分为**相关子查询**和**无关子查询**。**相关子查询**的例子比如：我想取得每件产品的名称和总共的销量：
```

```sql
SELECT [Name],
      (SELECT COUNT(*) FROM AdventureWorks.Sales.SalesOrderDetail S
      WHERE S.ProductID=P.ProductID) AS SalesAmount
FROM [AdventureWorks].[Production].[Product] P
```

部分结果如下：

![5](http://images.cnblogs.com/cnblogs_com/CareySon/201107/201107181306165951.png)

# DDL | 数据定义

## Update | 更新

### 存在性更新

我们经常需要处理某个唯一索引时存在则更新，不存在则插入的情况，其基本形式如下：

```sql
INSERT INTO ON ... DUPLICATE KEY UPDATE ...
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

# 事务管理

| 隔离级别                    | 脏读 / Dirty Read | 不可重复读 / Non Repeatable Read | 幻读 / Phantom Read | 锁             |
| --------------------------- | ----------------- | -------------------------------- | ------------------- | -------------- |
| 未提交读 / Read Uncommitted | 可能              | 可能                             | 可能                | 无事务         |
| 提交读 / Read Committed     | 不可能            | 可能                             | 可能                | 事务保护       |
| 可重复读 / Repeatable Read  | 不可能            | 不可能                           | 可能                | 行锁，顺序写   |
| 串行化 / Serializable       | 不可能            | 不可能                           | 不可能              | 表锁，顺序读写 |

不同级别的现象描述如下：

- 脏读: 当一个事务读取另一个事务尚未提交的修改时，产生脏读。

- 不可重复读: 同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生不可重复读。

- 幻读: 同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，此时发生幻读。

# 存储过程
