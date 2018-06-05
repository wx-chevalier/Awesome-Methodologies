# Egg.js 源码探秘

## 代码加载与依赖注入

## Cluster 模式如何启动

```js
// egg-scripts/start.js
// 封装参数
this.serverBin = path.join(__dirname, '../start-cluster');
...
const eggArgs = [
  ...(execArgv || []),
  this.serverBin,
  clusterOptions,
  `--title=${argv.title}`
];
...
// 启动执行进程
spawn('node', eggArgs, options);

// start-cluster
require(options.framework).startCluster(options);
```
