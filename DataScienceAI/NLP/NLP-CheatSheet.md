# Introduction

自然语言处理经历了从规则的方法到基于统计的方法。基于统计的自然语言处理方法，在数学模型上和通信就是相同的，甚至相同的。但是科学家们也是用了几十年才认识到这个问题。统计语言模型的初衷是为了解决语音识别问题，在语音识别中，计算机需要知道一个文字序列能否构成一个有意义的句子。

## Reference

### Practices & Resources

* [DL4NLP-Deep Learning for NLP resources](https://github.com/andrewt3000/DL4NLP)

### Books & Tools

* [snownlp](https://github.com/isnowfy/snownlp?utm_source=tuicool):一个基于 Python 的中文处理

- [数学之美]()

# Language Model(语言模型)

语言模型其实就是看一句话是不是正常人说出来的。这玩意很有用，比如机器翻译、语音识别得到若干候选之后，可以利用语言模型挑一个尽量靠谱的结果。在 NLP 的其它任务里也都能用到。语言模型形式化的描述就是给定一个字符串，看它是自然语言的概率 $P(w_1, w_2, …, w_t)$。$w_1$ 到 $w_t$ 依次表示这句话中的各个词。有个很简单的推论是：

$P(w*1, w_2, …, w_t) = P(w_1) \times P(w_2 | w_1) \times P(w_3 | w_1, w_2) \times … \times P(w_t | w_1, w_2, …, w*{t-1}) $

常用的语言模型都是在近似地求 $P(w*t | w_1, w_2, …, w*{t-1})$。比如 n-gram 模型就是用 $P(w*t | w*{t-n+1}, …, w\_{t-1})$ 近似表示前者。

## 词表示(Word Representation)

自然语言理解的问题要转化为机器学习的问题，第一步肯定是要找一种方法把这些符号数学化。

### One-hot Representation

NLP 中最直观，也是到目前为止最常用的词表示方法是 One-hot Representation，这种方法把每个词表示为一个很长的向量。这个向量的维度是词表大小，其中绝大多数元素为 0，只有一个维度的值为 1，这个维度就代表了当前的词。

举个栗子，

“话筒”表示为 [0 0 0 **1** 0 0 0 0 0 0 0 0 0 0 0 0 …]

“麦克”表示为 [0 0 0 0 0 0 0 0 **1** 0 0 0 0 0 0 0 …]

每个词都是茫茫 0 海中的一个 1。

这种 One-hot Representation 如果采用稀疏方式存储，会是非常的简洁：也就是给每个词分配一个数字

ID。比如刚才的例子中，话筒记为 3，麦克记为 8（假设从 0 开始记）。如果要编程实现的话，用 Hash

表给每个词分配一个编号就可以了。这么简洁的表示方法配合上最大熵、SVM、CRF 等等算法已经很好地完成了 NLP 领域的各种主流任务。

当然这种表示方法也存在一个重要的问题就是“词汇鸿沟”现象：任意两个词之间都是孤立的。光从这两个向量中看不出两个词是否有关系，哪怕是话筒和麦克这样的同义词也不能幸免于难。

### 词向量(Distributed Representation)

而是用 **Distributed Representation**（不知道这个应该怎么翻译，因为还存在一种叫“Distributional Representation”的表示方法，又是另一个不同的概念）表示的一种低维实数向量。这种向量一般长成这个样子：[0.792, −0.177,−0.107, 0.109, −0.542, …]。维度以 50 维和 100 维比较常见。这种向量的表示不是唯一的，后文会提到目前计算出这种向量的主流方法。（个人认为）Distributed representation

最大的贡献就是让相关或者相似的词，在距离上更接近了。向量的距离可以用最传统的欧氏距离来衡量，也可以用 cos 夹角来衡量。用这种方式表示的向量，“麦克”和“话筒”的距离会远远小于“麦克”和“天气”。可能理想情况下“麦克”和“话筒”的表示应该是完全一样的，但是由于有些人会把英文名“迈克”也写成“麦克”，导致“麦克”一词带上了一些人名的语义，因此不会和“话筒”完全一致。

## 统计语言模型

传统的统计语言模型是表示语言基本单位（一般为句子）的概率分布函数， 这个概率分布也就是该语言的生成模型。一般语言模型可以使用各个词语条件概率的形式表示：

$p(s)=p(w*1^T)=p(w_1,w_2,\dots,w_T)=\Pi^T*{t=1}p(w_t|Context) $

目标也可以是采用极大似然估计来求取最大化的 Log 概率的平均值，公式为$\frac{1}{T}\sum^T*{t=1}\sum*{-c \le j\le c,j \ne0}log p(w\_{t+j}|w_t)$。

其中：

* $c$是训练上下文的大小。譬如$c$取值为 5 的情况下，一次就拿 5 个连续的词语进行训练。一般来说$c$越大，效果越好，但是花费的时间也会越多。
* $p(w*{t+j}|w_t)$表示$w_t$条件下出现$w*{t+j}$的概率。

其中 Context 即为上下文，根据对 Context 不同的划分方法，可以分为五大类。

### 上下文无关模型（Context=NULL)

该模型仅仅考虑当前词本身的概率，不考虑该词所对应的上下文环境。这是一种最简单，易于实现，但没有多大实际应用价值的统计语言模型。

$p(w*t|Context)=p(w_t)=\frac{N*{w_t}}{N}$

这个模型不考虑任何上下文信息，仅仅依赖于训练文本中的词频统计。它是 n-gram 模型中当 n=1 的特殊情形，所以有时也称作 Unigram Model (—元文法统计模型）。实际应用中，常被应用到一些商用语音识别系统中。

### N-Gram 模型(Context=$w*{t-n+1},w*{t-n+2},\dots,w\_{t-1}$)

N-Gram 模型时大词汇连续语音识别中常用的一种语言模型，对中文而言，我们称之为汉语语言模型（CLM, Chinese Language Model）。汉语语言模型利用上下文中相邻词间的搭配信息，在需要把连续无空格的拼音、笔画，或代表字母或笔画的数字，转换成汉字串（即句子）时，可以计算出最大概率的句子，从而实现从到汉字的自动转换，无需用户手动选择，避开了许多汉字对应一个相同的拼音（或笔画串、数字串）的重码问题。该模型基于这样一种假设，第 n 个词的出现只与前面 n-1 个词相关，而与其它任何词都不相关，整句的概率就是各个词出现的概率的乘积。这些概率可以通过直接从语料中统计 n 个词同时出现的次数得到。常用的是二元的 Bi-Gram 和三元的 Tri-Gram。

n=1 时，就是上面所说的上下文无关模型，这里 N-Gram—般认为是$N\ge2$是 的上下文相关模型。当 n=2 时，也称为 Bigram 语言模型，直观的想，在自然语 言中“白色汽车”的概率比“白色飞翔”的概率要大很多，也就是 P(汽车|白色)> P(飞翔|白色)。n>2 也类似，只是往前看 n-1 个词而不是一个词。一般 N-Gram 模型优化的目标是最大 log 似然，即：

$\Pi^T*{t=1}p_t(w_t|w*{t-n+1},w*t|w*{t-n+2},\dots,w*t|w*{t-1})$

N-Gram 模型的优点包含了前 N-1 个词所能提供的全部信息，这些信息对当前 词出现具有很强的约束力。同时因为只看 N-1 个词而不是所有词也使得模型的效率较高。这里以 Bi-Gram 做一个实例，假设语料库总的词数为 13748：

![image](http://images.cnitblog.com/blog/407700/201310/18171638-c87b895734e748ff9a188265bccbe6bb.png)

![](http://images.cnitblog.com/blog/407700/201310/18171638-c325ffe1717e4763838913964e3971fc.png)

N-Gram 语言模型也存在一些问题：

* n-gram 语言模型无法建模更远的关系，语料的不足使得无法训练更高阶的语言模型。大部分研究或工作都是使用 Trigram，就算使用高阶的模型，其统计 到的概率可信度就大打折扣，还有一些比较小的问题采用 Bigram。

- 这种模型无法建模出词之间的相似度，有时候两个具有某种相似性的词，如果一个词经常出现在某段词之后，那么也许另一个词出现在这段词后面的概率也比较大。比如“白色的汽车”经常出现，那完全可以认为“白色的轿车”也可能经常出现。

* 训练语料里面有些 n 元组没有出现过，其对应的条件概率就是 0,导致计算一整句话的概率为 0。

#### 平滑法

方法一为平滑法。最简单的方法是把每个 n 元组的出现次数加 1，那么原来出现 k 次的某个 n 元组就会记为 k+1 次，原来出现 0 次的 n 元组就会记为出现 1 次。这种也称为 Laplace 平滑。当然还有很多更复杂的其他平滑方法，其本质都 是将模型变为贝叶斯模型，通过引入先验分布打破似然一统天下的局面。而引入 先验方法的不同也就产生了很多不同的平滑方法。

#### 回退法

方法二是回退法。有点像决策树中的后剪枝方法，即如果 n 元的概率不到， 那就往上回退一步，用 n-1 元的概率乘上一个权重来模拟。

### N-Pos 模型(Context = $c(w*{t-n+1}),c(w*{t-n+2}),\dots,c(w\_{t-1})$)

严格来说 N-Pos 只是 N-Gram 的一种衍生模型。N-Gram 模型假定第 t 个词出现概率条件依赖它前 N-1 个词，而现实中很多词出现的概率是条件依赖于它前面词的语法功能的。N-Pos 模型就是基于这种假设的模型，它将词按照其语法功能进行分类，由这些词类决定下一个词出现的概率。这样的词类称为词性 (Part-of-Speech，简称为 POS)。N-Pos 模型中的每个词的条件概率表示为：

$p(s)=p(w^T*1)=p(w_1,w_2,\dots,w_T)= \\ \Pi^T*{t=1}p(w*t|c(w*{t-n+1}),c(w*{t-n+2}),\dots,c(w*{t-1}))$

$c$为类别映射函数，即把$T$个词映射到$K$个类别($1 \le K \le T$)，实际上 N-Pos 使用了一种聚类的思想，使得 N-Gram 中$w*{t-n+1},w*{t-n+2},\dots,w*{t-1}$中的可能为$T^{n-1}$减少为$c(w*{t-n+1}),c(w*{t-n+2}),\dots,c(w*{t-1})$中的$K^{N-1}$，同时这种减少还采用了语义有意义的类别。

### 基于决策树的语言模型

上面提到的上下文无关语言模型、n-gram 语言模型、n-pos 语言模型等等，都可以以统计决策树的形式表示出来。而统计决策树中每个结点的决策规则是一 个上下文相关的问题。这些问题可以是“前一个词时 w 吗？ ”“前一个词属于类别 C,吗？”。当然基于决策树的语言模型还可以更灵活一些，可以是一些“前一个词是动词?”，“后面有介词吗?”之类的复杂语法语义问题。基于决策树的语言模型优点是：分布数不是预先固定好的，而是根据训练预 料库中的实际情况确定，更为灵活。缺点是：构造统计决策树的问题很困难，且时空开销很大。

### 最大熵模型

最大熵原理是 E.T. Jayness 于上世纪 50 年代提出的，其基本思想是：对一个 随机事件的概率分布进行预测时，在满足全部已知的条件下对未知的情况不做任何主观假设。从信息论的角度来说就是：在只掌握关于未知分布的部分知识时,应当选取符合这些知识但又能使得熵最大的概率分布。

$p(w|Context)=\frac{e^{\Sigma_i \lambda_i f_i(context,w)}}{Z(Context)}$

### 自适应语言模型

前面的模型概率分布都是预先从训练语料库中估算好的，属于静态语言模型。 而自适应语言模型类似是 Online Learning 的过程，即根据少量新数据动态调整模型，属于动态模型。在自然语言中，经常出现这样现象：某些在文本中通常很少出现的词，在某一局部文本中突然大量地出现。能够根据词在局部文本中出现的 情况动态地调整语言模型中的概率分布数据的语言模型成为动态、自适应或者基于缓存的语言模型。通常的做法是将静态模型与动态模型通过参数融合到一起， 这种混合模型可以有效地避免数据稀疏的问题。还有一种主题相关的自适应语言模型，直观的例子为：专门针对体育相关内 容训练一个语言模型，同时保留所有语料训练的整体语言模型，当新来的数据属 于体育类别时，其应该使用的模型就是体育相关主题模型和整体语言模型相融合 的混合模型。

### Skip-Gram

> [A CloserLook at Skip-gram Modelling](http://homepages.inf.ed.ac.uk/ballison/pdf/lrec_skipgrams.pdf)

根据论文中的定义可知道，常说的`k-skip-n-grams`在句子$w_1 \dots w_m$可以表示为：

$\{ w*{i_1},w*{i*2}, \dots w*{i*n} | \sum*{j=1}^{n}i*j - i*{j-1} < k \}$

Skip-gram 實際上的定義很簡單，就是允许跳几个字的意思… 依照原論文裡的定義，這個句子：

> Insurgents killed in ongoing fighting.

​ 在 bi-grams 的時候是拆成：{ `insurgents killed, killed in, in ongoing, ongoing fighting` }。

​ 在 2-skip-bi-grams 的時候拆成：{ `insurgents killed, insurgents in, insurgents ongoing, killed in, killed ongoing, killed fighting, in ongoing, in fighting, ongoing fighting` }。

​ 在 tri-grams 的時候是：{ `insurgents killed in, killed in ongoing, in ongoing fighting` }。

​ 在 2-skip-tri-grams 的時候是：{ `insurgents killed in, insurgents killed ongoing, insurgents killed fighting, insurgentsin ongoing, insurgents in fighting, insurgents ongoing fighting, killed in ongoing, killed in fighting, killed ongoing fighting, in ongoing fighting` }。

对于上文的语言模型的目标公式而言，Skip-Gram 模型中的$p(w\_{t+j} | w_t)$公式采用的是 Softmax 函数：

$p(w*o | w_I) = \frac{exp(v'^T*{w*o}v*{w*I})}{\sum^W*{w=1}exp(v'^T*wv*{w_I})}$

其中$p(w*o | w_I)$表示在词语$w_I$条件下出现$w_o$的概率，$v*{w_o}$表示$w_o$代表的词向量，而$v_w$代表词汇表中所有词语的向量。$W$是词汇表的长度。不过该公式不太切实际，因为$W$太大了，通常是$10^5–10^7$。

#### Hierarchical Softmax

这种是原始 skip-gram 模型的变形。我们假设有这么一棵二叉树，每个叶子节点对应词汇表的词语，一一对应。所以我们可以通过这棵树来找到一条路径来找到某个词语。比如我们可以对词汇表，根据词频，建立一棵 huffman 树。每个词语都会对应一个 huffman 编码，huffman 编码就反映了这个词语在 huffman 树的路径。对于每个节点，都会定义孩子节点概率，左节点跟右节点的概率不同的，具体跟输入有关。譬如，待训练的词组中存在一句：“我爱中国”。

输入：爱

预测：我

假设，“我”的 Huffman 编码是 1101，那么就在 Huffman 树上从根节点沿着往下走，每次走的时候，我们会根据当前节点和“爱”的向量算出（具体怎么算先不管），走到下一个节点的概率是多少。于是，我们得到一连串的概率，我们的目标就是使得这些概率的连乘值（联合概率）最大。

$p(w|w*I)=\Pi*{j=1}^{L(w)-1}\sigma([n(w,j+1)=ch(n(w,j))]\*v'^T*{n(w,j)}v*{w_I})$

* $L(w)$为词语$w$在二叉树路径中的长度
* $\sigma(\*)$即为 Sigmoid 函数
* $n(w,j+1)$即指$w$在二叉树的第$j+1$个节点
* $ch(n(w,j))$表示定义了任意一个固定的节点，要么是左，要么是右。合起来的意思是左右节点的正负号是不一致的，可以是左负右正，可以是左正右负。

而对于单一的选择左右节点的概率：

$\sigma(x)=\frac{1}{1+e^{-x}} \\ \sigma(-x)=\frac{1}{1+e^{x}} \\ \sigma(x) + \sigma(-x) = 1 $

显然，我们计算这个联合概率的复杂度取决了词语在 huffman 树的路径长度，显然她比 W 小得多了。另外，由于按词频建立的 huffman 树，词频高的，huffman 编码短，计算起来就比较快。词频高的需要计算概率的次数肯定多，而 huffman 让高频词计算概率的速度比低频词的快。这也是很犀利的一个设计。

## NNLM

NNLM 是 Neural Network Language Model 的缩写，即神经网络语言模型。神经网络语言模型方面最值得阅读的文章是 Deep Learning 二号任人物 Bengio 的《A Neural Probabilistic Language Model》，JMLR 2003。NNLM 米用的是 Distributed Representation，即每个词被表示为一个浮点向量。其模型图如下：

![](http://7xlgth.com1.z0.glb.clouddn.com/2288BF90-FD22-493A-B703-C5AB32726FF2.png)

目标是要学到一个好的模型：

$f(w*t,w*{t-1},\dots,w*{t-n+2},w*{t-n+1})=p(w_t|w_1^{t-1})$

需要满足的约束为：

$f(w*t,w*{t-1},\dots,w*{t-n+2},w*{t-n+1}) > 0$

$\Sigma*{i=1}^{|V|} f(i,w*{t-1},\dots,w*{t-n+2},w*{t-n+1}) = 1$

上图中，每个是输入词都被映射为一个向量，该映射用$C$表示，所以$C(w*{t-1})$即为$w*{t-1}$的词向量。$g$为一个前馈或者递归神级网络，其输出是一个向量，向量中的第$i$个元素表示概率$p(w_t=i|w_1^{t-1})$。训练的目标依然是最大似然加正则项，即：

$Max Likelihood = max \frac{1}{T}\sum*tlogf(w_t,w*{t-1},\dots,w*{t-n+2},w*{t-n+1};\theta) \\ + R(\theta)$

其中$\theta$为参数，$R(\theta)$为正则项，输出层采用 sofamax 函数：

$p(w*t|w*{t-1},\dots,w*{t-n+2},w*{t-n+1})=\frac{e^{y\_{w_t}}}{\sum_ie^{y_i}}$

其中$y_i$是每个输出词$i$的未归一化$log$概率，计算如下：

$y=b+Wx+Utanh(d+Hx)$

其中$b,W,U,d,H$都是参数，$x$为输入，需要注意的是，一般的神级网络输入是不需要优化，而在这里，$x=(C(w*{t-1}),C(w*{t-2}),\dots,C(w\_{t-n+1}))$，也是需要优化的参数。在图中，如果下层原始输入$x$不直接连到输出的话，可以令$b=0$，$W=0$。如果采用随机梯度算法的话，梯度的更新规则为：

$ \theta + \epsilon \frac{\partial log p(w*t | w*{t-1},\dots,w*{t-n+2},w*{t-n+1})}{\partial \theta} \to \theta $

其中$\epsilon$为学习速率，需要注意的是，一般神级网络的输入层只是一个输入值，而在这里，输入层$x$也是参数(存在$C$中)，也是需要优化的。优化结束之后，词向量有了，语言模型也有了。这个 Softmax 模型使得概率取值为(0,1)，因此不会出现概率为 0 的情况，也就是自带平滑，无需传统 N-Gram 模型中那些复杂的平滑算法。Bengio 在 APNews 数据集上做的对比实验也表明他的模型效果比精心设计平滑算法的普 通 N-Gram 算法要好 10%到 20%。

# Chinese Word Segment(CWS 中文分词)

> [Which is More Suitable for Chinese Word Segmentation, the Generative Model or the Discriminative One? ](http://aclweb.org/anthology//Y/Y09/Y09-2047.pdf)

## Java

### [ansj_seg](https://github.com/NLPchina/ansj_seg)

# Sentiment Analysis(情感分析)

> [Twitter 情感分析技术](http://www.infoq.com/cn/news/2015/12/Twitter-api-notion)

# 拼音识别

> [使用 HMM 实现简单拼音输入法](http://sobuhu.com/ml/2013/03/07/hmm-pinyin-input-method.html?utm_source=tuicool&utm_medium=referral)
