# AI CheatSheet | AI

所谓 AI，即能够感知（Aware/Sense），然后做出决策（Decision），并且对于环境做出反应（Act）。

人工智能，与机器学习的思维模式，不同于软件工程中以逻辑与数学思维去思考，以断言来证明程序各属性的正确性，并且最终得到确定性产品的开发思维模式；它的关注点从数学科学转移到自然科学，观察不确定的未知世界，开展实验，并使用统计信息 而非逻辑来分析实验结果。

## 意义与应用

首先，它会为您提供一个可缩短编程时间的工具。假设我想编写一个程序来纠正拼写错误。我可以通过大量示例和经验法则 （例如，I 位于 E 之前，但出现在 C 之后时例外），取得一定的进展，然后经过数周的努力，编写出一个合理的程序。

其次，借助机器学习，您可以自定义自己的产品，使其更适合特定的用户群体。假设我手动编写了一个英文拼写纠正程序，这个程序很成功，因此我打算 针对 100 种最常用语言提供相应的版本。这样一来，每种语言版本几乎都需要从头开始，这将需要付出数年的努力。但如果我使用机器学习技术构建该程序， 然后迁移到其他语言，基本上就相当于，我只需收集该特定语言的数据， 并将这些数据提供给完全一样的机器学习模型即可。

第三，借助机器学习，您可以解决自己作为编程人员不知道如何用人工方法解决的问题。作为人类，我可以认出朋友的面孔， 理解他们所说的话，但所有这些都是在潜意识下完成的， 如果让我编写一个程序来做这些事，我会完全不知所措。但是，机器学习算法对此却很擅长；我不需要告诉算法应该怎么做，只需向其展示大量样本， 问题就可以迎刃而解。

## 发展与变迁

人工智能发展有三个阶段：计算智能、感知智能和认知智能。

第一阶段的计算智能即快速计算和记忆存储，像机器人战胜围棋大师，靠的就是超强的记忆能力和运算速度。人脑的逻辑能力再强大，也敌不过人工智能每天和自己下几百盘棋，通过强大的计算能力对十几步后的结果做出预测，从这一角度来说，人工智能多次战败世界级围棋选手，足以证明这一领域发展之成熟。

第二阶段的感知智能，即让机器拥有视觉、听觉、触觉等感知能力。自动驾驶汽车做的就是这一方面的研究，使机器通过传感器对周围的环境进行感知和处理，从而实现自动驾驶。感知智能方面的技术目前发展比较成熟的领域有语音识别和图像识别，比如做安全领域人脸识别技术的 Face++，以成熟的计算机视觉技术深耕电商、短视频等领域的 Yi+，能够对多种语言进行准确识别翻译的科大讯飞等。

第三阶段的认知智能与前面在人工智能的 3 大分支里提到的认知 AI 类似，就是让机器拥有自己的认知，能理解会思考。认知智能是目前机器和人差距最大的领域，因为这不仅涉及逻辑和技术，还涉及心理学、哲学和语言学等学科。

# Machine Learning | 机器学习

Maximum Objective Function

常见机器学习的任务可以分解为以下七个步骤：

Data Collection

Data Preparation

Build Model

Train Model

Evaluation

Tune

Predict

# Deep Learning | 深度学习

Traditional statistical models do very well on structured data, i.e. tabular data, but have notoriously struggled with unstructured data like images, audio, and natural language. Neural networks that contain many layers of neurons embody the research that is popularly called Deep Learning. The key insight and property of deep neural networks that make them suitable for modeling unstructured data is that complex data, like images, generally have many layers of unique features that are composed to produce the data. As a classic example: images have edges which form the basis for textures, textures form the basis for simple objects, simple objects form the basis for more complex objects, and so on. In deep neural networks we aim to learn these many layers of composable features.

Traditional statistical models do very well on structured data, i.e. tabular data, but have notoriously struggled with unstructured data like images, audio, and natural language. Neural networks that contain many layers of neurons embody the research that is popularly called Deep Learning. The key insight and property of deep neural networks that make them suitable for modeling unstructured data is that complex data, like images, generally have many layers of unique features that are composed to produce the data. As a classic example: images have edges which form the basis for textures, textures form the basis for simple objects, simple objects form the basis for more complex objects, and so on. In deep neural networks we aim to learn these many layers of composable features.
