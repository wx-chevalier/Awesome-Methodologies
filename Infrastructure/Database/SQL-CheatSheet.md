[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# SQL CheatSheet

### 存在性更像

Mysql 处理某个唯一索引时存在则更新，不存在则插入的情况应该是很常见的，网上也有很多类似的文章，我今天就讲讲当这个唯一的索引是多列唯一索引时可能会遇到的问题和方法。

方法一：

使用

INSERT INTO ON ... DUPLICATE KEY UPDATE ...

：

表的创建如下：

```sql
CREATE TABLE `test_table` (  
  `id`  int(11) NOT NULL AUTO_INCREMENT ,  
  `var1`  varchar(100) CHARACTER SET utf8 DEFAULT NULL,  
  `var2`  tinyint(1) NOT NULL DEFAULT '0',  
  `var3`  varchar(100) character set utf8 default NULL,  
  `value1`  int(11) NOT NULL DEFAULT '1',  
  `value2`  int(11) NULL DEFAULT NULL,  
  `value3`  int(5) DEFAULT NULL,  
  PRIMARY KEY (`Id`),  
  UNIQUE INDEX `index_var` (`var1`, `var2`, `var3`)  
) ENGINE=MyISAM DEFAULT CHARACTER SET=latin1 AUTO_INCREMENT=1;  
```

其中该表中 var1、var2 和 var3 完全相同的记录只能有一条，所以建了一个多列唯一索引 index_var，这样一来我们就可以使用 INSERT INTO ON ... DUPLICATE KEY UPDATE ... 来实现插入数据时存在则更新，不存在则插入的功能了，如下：

```sql
INSERT INTO `test_table`
(`var1`, `var2`, `var3`, `value1`, `value2`, `value3`) VALUES
('abcd', 0, 'xyz', 1, 2, 3)
ON DUPLICATE KEY UPDATE `value1` = `value1` + 1 AND
`value2` = `value2` + 2 AND `value3` = `value3` + 3;  
```

该条插入语句的含义是：向 test_table 表中插入，如果不存在 val1 = 'abcd'，val2 = 0, val3 = ‘xyz’的记录，那就插入

val1 = 'abcd'，val2 = 0, val3 = ‘xyz’，value1 = 1, value2 = 2, value3 = 3 的记录，

如果存在，那就更新 value1 的值为 value1+1，更新 value2 的值为 value2+2，更新 value3 的值为 value3+3。

这样，的确是没有问题的，但是，如果表的创建如下：

```sql
CREATE TABLE `test_table` (  
  `id`  int(11) NOT NULL AUTO_INCREMENT ,  
  `var1`  varchar(1024) CHARACTER SET utf8 DEFAULT NULL,  
  `var2`  tinyint(1) NOT NULL DEFAULT '0',  
  `var3`  varchar(1024) character set utf8 default NULL,  
  `value1`  int(11) NOT NULL DEFAULT '1',  
  `value2`  int(11) NULL DEFAULT NULL,  
  `value3`  int(5) DEFAULT NULL,  
  PRIMARY KEY (`Id`),  
  UNIQUE INDEX `index_var` (`var1`, `var2`, `var3`)  
) ENGINE=MyISAM DEFAULT CHARACTER SET=latin1 AUTO_INCREMENT=1;  
```

注意：var1 和 var3 的最大长度由 100 变成了 1024，此时执行该创建语句时会报如下错误：

```
Specified key was too long; max key length is 1000 bytes  
```

这是由于 index*var 索引的为 1024 * 3 + 1 + 1024 \_ 3 > 1000 导致的，如果遇到这种情况怎么办？有两种解决办法。

第一，将数据库的 engine 由 MyISAM 换成 InnoDB 就可以了，那么这两个引擎有什么区别呢？

看这里

不过，这样换有一个缺点，就是 InnoDB 的性能没有 MyISAM 的好，那么如果想要不牺牲性能的话，那就只有用第二个方法了，也就是我们这里说的方法二！

方法二：

使用 dual 虚拟表来实现。

使用 dual 虚拟表来实现的话就不需要创建多列唯一索引了，表的创建如下：

```sql
CREATE TABLE `test_table` (  
  `id`  int(11) NOT NULL AUTO_INCREMENT ,  
  `var1`  varchar(1024) CHARACTER SET utf8 DEFAULT NULL,  
  `var2`  tinyint(1) NOT NULL DEFAULT '0',  
  `var3`  varchar(1024) character set utf8 default NULL,  
  `value1`  int(11) NOT NULL DEFAULT '1',  
  `value2`  int(11) NULL DEFAULT NULL,  
  `value3`  int(5) DEFAULT NULL,  
  PRIMARY KEY (`Id`)  
) ENGINE=MyISAM DEFAULT CHARACTER SET=latin1 AUTO_INCREMENT=1;  
```

插入语句则是形如：

```sql
INSERT INTO table  
(primarykey, field1, field2, ...)  
SELECT key, value1, value2, ...  
FROM dual  
WHERE not exists (select * from table where primarykey = id);  
```

的语句，此时我们可以用以下语句代替：

```sql
INSERT INTO `test_table` SELECT 0, 'abcd', 0, 'xyz', 1, 2, 3  
FROM dual WHERE NOT EXISTS (  
SELECT * FROM `test_table` WHERE
`var1` = 'abcd' AND `var2` = 0 AND `var3` = 'xyz');  
```

此时，如果 val1 = 'abcd'，val2 = 0, val3 = ‘xyz’的记录不存在，那么就会执行该插入语句插入该记录，如果存在，那就需要我们再使用相应的更新语句来更新记录：
​```sql
UPDATE `test_table` SET  
`value1` = `value1` + 1, `value2` = `value2` + 2, `value3` = `value3` + 3  
WHERE `val1` = 'abcd' AND `val2` = 0 AND `val3` = 'xyz';

```

```

# 事务管理

| 隔离级别                    | 脏读 / Dirty Read | 不可重复读 / Non Repeatable Read | 幻读 / Phantom Read | 锁             |
| --------------------------- | ----------------- | -------------------------------- | ------------------- | -------------- |
| 未提交读 / Read Uncommitted | 可能              | 可能                             | 可能                | 无事务         |
| 提交读 / Read Committed     | 不可能            | 可能                             | 可能                | 事务保护       |
| 可重复读 / Repeatable Read  | 不可能            | 不可能                           | 可能                | 行锁，顺序写   |
| 串行化 / Serializable       | 不可能            | 不可能                           | 不可能              | 行锁，顺序读写 |

不同级别的现象描述如下：

* 脏读: 当一个事务读取另一个事务尚未提交的修改时，产生脏读。

* 不可重复读: 同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生不可重复读。

* 幻读: 同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，此时发生幻读。
