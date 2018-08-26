[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# DeepLearning CheatSheet | 深度学习概念备忘

# Neural Networks | 神经网络

线性结果加上值相关(Element wise)非线性结果，我们才能去模拟任意的复杂图形。

$X$ 是 $$

# RNN | 循环神经网络

循环神经网络最大的特点就在于循环神经网路的隐含层的每个节点的状态或输出除了与当前时刻的输入有关，还有上一个时刻的状态或者输出有关，隐含层节点之间存在循环连接。这使得循环神经网络具有对时间序列的记忆能力。
实际应用中最有效的序列模型称为门控 RNN（gated RNN）。包括基于长短期记忆（LSTM：long short-term memory）和基于门控循环单元(GRU：gated recurrent unit)的网络。门控 RNN 主要是解决一般 RNN 的梯度消失或者梯度爆炸的问题。

![image](https://user-images.githubusercontent.com/5803001/44517585-b09e4f80-a6fa-11e8-8177-407607d84fd7.png)

## LSTM

不同于普通的 RNN 节点单元，LSTM 引入了遗忘门（总共有 3 个门，输入们、遗忘门和输出们）用来决定我们需要从节点单元中抛弃哪些信息以及保留哪些信息。

![image](https://user-images.githubusercontent.com/5803001/44517720-05da6100-a6fb-11e8-9ffe-c018b9bf5baf.png)
