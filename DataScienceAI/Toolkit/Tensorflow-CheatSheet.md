[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# TensorFlow CheatSheet | TensorFlow 基础概念与实践清单

- 构建图
- 初始化 Session
- 填充数据，并且获取结果

```py
# 数据获取

# 数据处理与划分

# 模型/神经网络构建

# 损失函数

# 变量初始化

# 模型训练

# 模型预测

# 模型导出
```

# 模型构建

# 损失函数

## 交叉熵

交叉熵也可以用作损失函数值，其公式定义如下，其中 $q(x)$ 为模型的预估，$p(x)$ 为机器学习中样本的 label:

$$
H(p,q) = -\sum_xp(x)logq(x)
$$

```py
import tensorflow as tf
from random import randint

dims = 8
pos  = randint(0, dims - 1)

logits = tf.random_uniform([dims], maxval=3, dtype=tf.float32)
labels = tf.one_hot(pos, dims)

res1 = tf.nn.softmax_cross_entropy_with_logits(       logits=logits, labels=labels)
res2 = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=logits, labels=tf.constant(pos))

with tf.Session() as sess:
    a, b = sess.run([res1, res2])
    print a, b
    print a == b
```

# 辅助函数

tf.argmax 返回的是 vector 中的最大值的索引号，如果 vector 是一个向量，那就返回一个值，如果是一个矩阵，那就返回一个向量，这个向量的每一个维度都是相对应矩阵行的最大值元素的索引号。

```py
import tensorflow as tf
import numpy as np

A = [[1,3,4,5,6]]
B = [[1,3,4], [2,4,1]]

with tf.Session() as sess:
    print(sess.run(tf.argmax(A, 1)))
    print(sess.run(tf.argmax(B, 1)))
```
