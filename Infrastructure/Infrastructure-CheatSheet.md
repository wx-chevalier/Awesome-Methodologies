[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Infrastructure CheatSheet

2005 年亚马逊发布了 AWS，算是拉开了云计算的序幕。云计算最重要的技术是分布式计算和分布式存储，分布式计算方面，最开始的技术是虚拟化，也就是所谓的“Software defined xxx”，通过对计算 / 存储和网络资源的虚拟化，同时能够给用户任意分配资源。但这里面一开始做的最好的只有文件存储这一块，AWS S3 及类似的对象存储产品给人们带来了云时代的一些实际的体验，但云服务器则还是走回了卖服务器的老路。

在云计算的发展过程中，有一个分支是 PaaS，最早是 2007 年推出的 Heroku，从形态上来说，它是一个 App Engine，提供应用的运行环境。2014 年 AWS 推出 Lambda 服务，Serverless 开始成为热词，从理论上说，Serverless 可以做到 NoOps、自动扩容和按使用付费，也被视为云计算的未来。但是，Serverless 本身有一些问题，比如难以解决的冷启动性能问题。

