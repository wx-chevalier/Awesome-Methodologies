[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Tree CheatSheet

# 哈夫曼

当树中的节点被赋予一个表示某种意义的数值，我们称之为该节点的权。从树的根节点到任意节点的路径长度(经过的边数)与该节点上权值的乘积称为该节点的带权路径长度。树中所有叶节点的带权路径长度之和称为该树的带权路径长度(WPL)。当带权路径长度最小的二叉树被称为哈夫曼树，也成为最优二叉树。

哈夫曼树在构造时每次从备选节点中挑出两个权值最小的节点进行构造，每次构造完成后会生成新的节点，将构造的节点从备选节点中删除并将新产生的节点加入到备选节点中。新产生的节点权值为参与构造的两个节点权值之和。举例如下：

- [](https://images2015.cnblogs.com/blog/872539/201611/872539-20161104194812346-1641453195.png)

- 备选节点为 a,b,c,d，权值分别为 7,5,2,4

- 选出 c 和 d 进行构造(权值最小),生成新节点为 e(权值为 6)，备选节点变为 7,5,6

- 选出 b 和 e 进行构造，生成新节点 f(权值为 11),备选节点为 7,11

- 将最后的 7 和 11 节点进行构造，最后生成如图所示的哈夫曼树

## 哈夫曼树的应用

在处理字符串序列时，如果对每个字符串采用相同的二进制位来表示，则称这种编码方式为定长编码。若允许对不同的字符采用不等长的二进制位进行表示，那么这种方式称为可变长编码。可变长编码其特点是对使用频率高的字符采用短编码，而对使用频率低的字符则采用长编码的方式。这样我们就可以减少数据的存储空间，从而起到压缩数据的效果。而通过哈夫曼树形成的哈夫曼编码是一种的有效的数据压缩编码。

# 多级分类

## 预排序遍历树算法

预排序遍历树算法(modified preorder tree traversal algorithm)是一种为查询而生的遍历树算法，如果业务需求要求查询的场景高于、多于增删改，可以考虑此算法，该算法会牺牲掉一些插入、删除的性能。算法简单概括：类二叉树先序遍历，将树上的每个节点标示 left、right、level、parent 属性（parent 可以更方便画出树结构），查询通过 left 和 right 取值，一般情况可以通过一条 sql 查询和排序出整个需要的树节点。

![image](https://user-images.githubusercontent.com/5803001/45407303-96bdb000-b69b-11e8-8791-b25e69425a74.png)

为了能够像上面的递归函数那样显示整个树状结构，我们还需要对这样的查询进行排序。用节点的左值进行排序：

```sql
SELECT * FROM tree WHERE lft BETWEEN 2 AND 11 ORDER BY lft ASC;
```

添加同一层次的节点的方法如下：

```sql
LOCK TABLE nested_category WRITE;

SELECT @myRight := rgt FROM nested_category WHERE name = 'Cherry';

UPDATE nested_category SET rgt = rgt + 2 WHERE rgt > @myRight;
UPDATE nested_category SET lft = lft + 2 WHERE lft > @myRight;

INSERT INTO nested_category(name, lft, rgt) VALUES('Strawberry', @myRight + 1, @myRight + 2);

UNLOCK TABLES;
```

添加树的子节点的方法如下：

```sql
LOCK TABLE nested_category WRITE;

SELECT @myLeft := lft FROM nested_category WHERE name = 'Beef';

UPDATE nested_category SET rgt = rgt + 2 WHERE rgt > @myLeft;
UPDATE nested_category SET lft = lft + 2 WHERE lft > @myLeft;

INSERT INTO nested_category(name, lft, rgt) VALUES('charqui', @myLeft + 1, @myLeft + 2);

UNLOCK TABLES;
```

每次插入节点之后都可以用以下 SQL 进行查看验证：

```sql
SELECT CONCAT( REPEAT( ' ', (COUNT(parent.name) - 1) ), node.name) AS name
FROM nested_category AS node,
nested_category AS parent
WHERE node.lft BETWEEN parent.lft AND parent.rgt
GROUP BY node.name
ORDER BY node.lft;
```
