![awesome coder](https://user-images.githubusercontent.com/5803001/43364904-59f5bda6-9356-11e8-9ab3-ae073d08bb9e.png)

[中文版本](./README.md) | [English Version](./README.en.md)

# [Awesome CheatSheet: 语法速览，实践备忘，从零到一，上手即用 🚀](https://github.com/wx-chevalier/Awesome-CheatSheets)

`Copyright © 王下邀月熊`

[Awesome CheatSheet](https://github.com/wx-chevalier/Awesome-CheatSheets) 是对某项技术/领域的语法速览与实践备忘清单集锦。它包含了 JavaScript，Java，Go，Python，Rust 等常见的编程语言，Web，数据库，信息安全等 [ITCS 知识图谱与技术路线](https://wx-chevalier.github.io/home/#/perspective)中归档的知识技能点，其致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。

需要注意的是，Awesome CheatSheets 中很多的文章仅有目录而无内容，这并不意味着这些文章是空白的，而是作者尚未将[具体的文档仓库](https://github.com/topics/wx-doc)中的相关主题下内容整合进来，您可以前往[感兴趣的分类仓库](https://github.com/topics/wx-doc)再行浏览。

# Nav | 导航

您可以通过以下任一方式阅读笔者的系列文章，涵盖了技术资料归纳、编程语言与理论、Web 与大前端、服务端开发与基础架构、云计算与大数据、数据科学与人工智能、产品设计等多个领域：

- 在 Gitbook 中在线浏览，每个系列对应各自的 Gitbook 仓库。

| [Awesome Lists](https://ngte-al.gitbook.io/i/) | [Awesome CheatSheets](https://ngte-ac.gitbook.io/i/) | [Awesome Interviews](https://github.com/wx-chevalier/Developer-Zero-To-Mastery/tree/master/Interview) | [Awesome RoadMaps](https://github.com/wx-chevalier/Developer-Zero-To-Mastery/tree/master/RoadMap) | [Awesome-CS-Books-Warehouse](https://github.com/wx-chevalier/Awesome-CS-Books-Warehouse) |
| ---------------------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |


| [编程语言理论与实践](https://ngte-pl.gitbook.io/i/) | [软件工程、数据结构与算法、设计模式、软件架构](https://ngte-se.gitbook.io/i/) | [现代 Web 开发基础与工程实践](https://ngte-web.gitbook.io/i/) | [大前端混合开发与数据可视化](https://ngte-fe.gitbook.io/i/) | [服务端开发实践与工程架构](https://ngte-be.gitbook.io/i/) | [分布式基础架构](https://ngte-infras.gitbook.io/i/) | [数据科学，人工智能与深度学习](https://ngte-aidl.gitbook.io/i/) | [产品设计与用户体验](https://ngte-pd.gitbook.io/i/) |
| --------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------- |


- 前往 [xCompass https://wx-chevalier.github.io](https://wx-chevalier.github.io/home/#/search) 交互式地检索、查找需要的文章/链接/书籍/课程。

![](https://i.postimg.cc/3RVYtbsv/image.png)

- 在下文的 [MATRIX 文章与代码矩阵 https://github.com/wx-chevalier/Developer-Zero-To-Mastery](https://github.com/wx-chevalier/Developer-Zero-To-Mastery) 中查看文章与项目的源代码。

# Preface | 前言

`让知识在该在的地方 @ 王下邀月熊`

笔记系统，或者说知识管理的首个难点，在于知识的检索，与更新；直观地来说，就是当我们想到某个知识时，应该如何去找到对应的笔记，或者说当我们想去记录某些心得体会时，应该把它们放在什么地方。一般来说，目前的笔记组织结构，可以是树形层级目录式，就像思维脑图一样，从某个点展开，延伸到各个具体的技术领域；也可以是 Tag 式，即为每个笔记进行人工地主题词提取，然后依赖于搜索功能进行检索。笔者上车伊始，即主张：知识应该放在它应该在的地方，因此数年来一直以近乎强迫的方式，去构建 [ITCS 技术体系与知识图谱](https://github.com/wx-chevalier/Awesome-Coder/blob/master/MindMap/README.md)，并且将其作为泛笔记系统的目录规范。这种细致的划分方式，往往会随着自身对于技术世界的认知变化而不断衍化，因此也是建立在广泛的阅读、涉猎的基础上；不过磨刀不误砍材工，若能坚持下来，浇灌培育出属于自己的知识体系树，也是别有一番滋味在心头。

有了遨游星海的星图指引，我们就要开始逐个探索美丽的知识星球了。根据知识本身的特点，使用的频次，检索与更新的方式，我们又可以将笔记分为索引式、清单式、书籍式与代码式；下面我会结合自身的实践认知，来阐述这几个不同类型的笔记的构建与使用。

- 索引式笔记，典型的代表是 [Awesome Lists](https://github.com/wx-chevalier/Awesome-Lists)，其按照知识图谱来将各个领域的有效的链接沉淀下来。与 Google 这样搜索引擎搜出的结果相比，其含金量会更高。一方面，纳入到 AwesomeList 当中的链接，往往都是自身阅读、筛选过的，尽可能去芜存菁，去重留一，无论是分享某个技术领域的文章给他人，还是自己学习阅读，都能够避免冗余阅读。另一方面，很多优秀的文章并不一定会出现在搜索引擎的前几页，而是需要依靠自己日常浏览中主动发现、归纳。[Awesome Links](https://github.com/wx-chevalier/Awesome-Lists) 中还包含了很重要的一个部分，\*-OpenSource-List，是对于各个领域的常见开源项目的归档；即可以方便快速查找所需的框架与库、开发工具等，也能够提供一些优秀的，可以借鉴的开源项目来学习。

- 清单式笔记，典型的代表是 [Awesome CheatSheet](https://github.com/wx-chevalier/Awesome-CheatSheets)，即是对于某个领域、方向的精华，以及日常工作中常用知识点的归档。无论是快速学习，还是作为日常开发中的工具手册，都是极好的。

- 书籍式笔记，即可以是 [Awesome-CS-Books-Warehouse](https://github.com/wx-chevalier/Awesome-CS-Books-Warehouse) 这样对于优秀书籍的搜集，也可以是 [现代 Web 开发](https://github.com/wx-chevalier/Web-Series)，[深入浅出分布式基础架构](https://github.com/wx-chevalier/Distributed-Infrastructure-Series) 等这样子各个领域的自己的笔记的编排。值得一提的是，书籍式笔记，并不强调一定要遵循知识图谱的结构，而是赋予其一定灵活性，以方便记录与交流。

- 代码式笔记，典型的代表是 [coding-snippets](https://github.com/wx-chevalier/coding-snippets), 对于程序员这个角色而言，代码也是我们笔记系统的重要组成。在这个之上，我们又可以构建出一系列小的项目。

对于每个领域，Awesome CheatSheet 可能包含了以下主题的文件：

- {Something}-CheatSheet.md: 对于 {Something} 的语法特性，实践技巧，技术栈进行速览与盘点。

- {Something}-Snippets-CheatSheet.md: 有用的/令人吃惊的 {Something} 代码片，帮助开发者迅速理解并且应用到工作中。

- {Something}-OpenSource-CheatSheet.md: 推荐的 {Something} 相关的开源库或者工具。

# About | 关于

## 参考

- [2018-Awesome Cheatsheets](https://github.com/LeCoupa/awesome-cheatsheets): Useful cheatsheets with everything you should know in one single-file. 🚀

## 版权

![](https://parg.co/bDY) ![](https://parg.co/bDm)

笔者所有文章遵循 [知识共享 署名 - 非商业性使用 - 禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)，欢迎转载，尊重版权。如果觉得本系列对你有所帮助，欢迎给我家布丁买点狗粮(支付宝扫码)~

[![default](https://i.postimg.cc/y1QXgJ6f/image.png)](https://github.com/wx-chevalier)
