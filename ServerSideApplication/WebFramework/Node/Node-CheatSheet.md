[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Node.js CheatSheet | Node.js 语法基础、框架使用与实践技巧

Node.js 的包管理，或者说依赖管理使用了语义化版本的规范，版本的发布分为四个不同的层次：使用 1.0.0 表示新发布，使用 1.0.1 这样第三位数字表示错误修复等小版本更新；使用 1.1.0 这样的第二位数字表示新特性等兼容性更新；使用 2.0.0 这样第一位数字表示大版本的更新。相对应地，在 package.json 声明依赖版本时，我们也可以指定不同的兼容范围：

- Patch releases：1.0 或者 1.0.x 或者 ~1.0.4

- Minor releases: 1 或者 1.x 或者 ^1.0.4

- Major releases: \* 或者 x

# 基础使用

## 调试

在 VSCode 中，我们也能够方便地进行断点调试，首先我们需要在 package.json 中添加 debug 指令：

```json
  "scripts": {
    "debug": "node --nolazy --inspect-brk=9229 myProgram.js"
  },
```

然后在 `./vscode/launch.json` 中添加启动配置项：

```json
{
  "name": "Launch via NPM",
  "type": "node",
  "request": "launch",
  "cwd": "${workspaceFolder}",
  "runtimeExecutable": "npm",
  "runtimeArgs": ["run-script", "debug"],
  "port": 9229
}
```

如果我们希望包含较多的环境变量，还可以指定环境变量：

```json
   //...
   "envFile": "${workspaceFolder}/.env",
   "env": { "USER": "john doe" }
   //...
```

或者指定加载外部的环境变量文件：

```env
USER=doe
PASSWORD=abc123
```

# IO

## Stream

就像数组或者映射，Stream 也是数据的集合，只不过其代表了不一定正在内存中的数据。Stream 是 Node.js 中的基础概念，类似于 EventEmitter，专注于 IO 管道中事件驱动的数据处理方式。Node.js 的 Stream 分为以下类型：

- Readable Stream: 可读流，数据的产生者，譬如 process.stdin
- Writable Stream: 可写流，数据的消费者，譬如 process.stdout 或者 process.stderr
- Duplex Stream: 双向流，即可读也可写
- Transform Stream: 转化流，数据的转化者

Stream 本身提供了一套接口规范，很多 Node.js 中的内建模块都遵循了该规范，譬如著名的 `fs` 模块，即是使用 Stream 接口来进行文件读写；同样的，每个 HTTP 请求是可读流，而 HTTP 响应则是可写流。

![](https://cdn-images-1.medium.com/max/1160/1*4Xv-enWqwdy_AlMYYl4JVg.png)

### Readable Stream

```js
const stream = require('stream');
const fs = require('fs');

const readableStream = fs.createReadStream(process.argv[2], {
  encoding: 'utf8'
});

// 手动设置流数据编码
// readableStream.setEncoding('utf8');

let wordCount = 0;

readableStream.on('data', function(data) {
  wordCount += data.split(/\s{1,}/).length;
});

readableStream.on('end', function() {
  // Don't count the end of the file.
  console.log('%d %s', --wordCount, process.argv[2]);
});
```

当我们创建某个可读流时，其还并未开始进行数据流动；添加了 data 的事件监听器，它才会变成流动态的。在这之后，它就会读取一小块数据，然后传到我们的回调函数里面。 `data` 事件的触发频次同样是由实现者决定，譬如在进行文件读取时，可能每行都会触发一次；而在 HTTP 请求处理时，可能数 KB 的数据才会触发一次。可以参考 [nodejs/readable-stream/\_stream_readable](https://github.com/nodejs/readable-stream/blob/master/lib/_stream_readable.js) 中的相关实现，发现 on 函数会触发 resume 方法，该方法又会调用 flow 函数进行流读取：

```js
// function on
if (ev === 'data') {
  // Start flowing on next tick if stream isn't explicitly paused
  if (this._readableState.flowing !== false) this.resume();
}
...
// function flow
while (state.flowing && stream.read() !== null) {}
```

我们还可以监听 `readable` 事件，然后手动地进行数据读取：

```js
let data = '';
let chunk;
readableStream.on('readable', function() {
  while ((chunk = readableStream.read()) != null) {
    data += chunk;
  }
});
readableStream.on('end', function() {
  console.log(data);
});
```

Readable Stream 还包括如下常用的方法：

- Readable.pause(): 这个方法会暂停流的流动。换句话说就是它不会再触发 data 事件。
- Readable.resume(): 这个方法和上面的相反，会让暂停流恢复流动。
- Readable.unpipe(): 这个方法会把目的地移除。如果有参数传入，它会让可读流停止流向某个特定的目的地，否则，它会移除所有目的地。

在日常开发中，我们可以用 [stream-wormhole](https://github.com/node-modules/stream-wormhole) 来模拟消耗可读流：

```js
sendToWormhole(readStream, true);
```

### Writable Stream

```js
readableStream.on('data', function(chunk) {
  writableStream.write(chunk);
});

writableStream.end();
```

当 `end()` 被调用时，所有数据会被写入，然后流会触发一个 `finish` 事件。注意在调用 `end()` 之后，你就不能再往可写流中写入数据了。

```js
const { Writable } = require('stream');

const outStream = new Writable({
  write(chunk, encoding, callback) {
    console.log(chunk.toString());
    callback();
  }
});

process.stdin.pipe(outStream);
```

Writable Stream 中同样包含一些与 Readable Stream 相关的重要事件：

- error: 在写入或链接发生错误时触发
- pipe: 当可读流链接到可写流时，这个事件会触发
- unpipe: 在可读流调用 unpipe 时会触发

### Pipe | 管道

```js
const fs = require('fs');

const inputFile = fs.createReadStream('REALLY_BIG_FILE.x');
const outputFile = fs.createWriteStream('REALLY_BIG_FILE_DEST.x');

// 当建立管道时，才发生了流的流动
inputFile.pipe(outputFile);
```

多个管道顺序调用，即是构建了链接(Chaining):

```js
const fs = require('fs');
const zlib = require('zlib');
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('output.txt'));
```

参照[分布式系统导论](https://parg.co/Uxo)，可知在典型的流处理场景中，我们不可以避免地要处理所谓的背压(Backpressure)问题。无论是 Writable Stream 还是 Readable Stream，实际上都是将数据存储在内部的 Buffer 中，可以通过 `writable.writableBuffer` 或者 `readable.readableBuffer` 来读取。当要处理的数据存储超过了 `highWaterMark` 或者当前写入流处于繁忙状态时，write 函数都会返回 `false`。`pipe` 函数即会自动地帮我们启用背压机制：

![](https://www.transitions-now.com/wp-content/uploads/2015/12/constying-flow-main1.png)

当 Node.js 的流机制监测到 write 函数返回了 `false`，背压系统会自动介入；其会暂停当前 Readable Stream 的数据传递操作，直到消费者准备完毕。

```
+===============+
|   Your_Data   |
+=======+=======+
        |
+-------v-----------+          +-------------------+         +=================+
|  Readable Stream  |          |  Writable Stream  +--------->  .write(chunk)  |
+-------+-----------+          +---------^---------+         +=======+=========+
        |                                |                           |
        |     +======================+   |        +------------------v---------+
        +----->  .pipe(destination)  >---+        |    Is this chunk too big?  |
              +==^=======^========^==+            |    Is the queue busy?      |
                 ^       ^        ^               +----------+-------------+---+
                 |       |        |                          |             |
                 |       |        |  > if (!chunk)           |             |
                 ^       |        |      emit .end();        |             |
                 ^       ^        |  > else                  |             |
                 |       ^        |      emit .write();  +---v---+     +---v---+
                 |       |        ^----^-----------------<  No   |     |  Yes  |
                 ^       |                               +-------+     +---v---+
                 ^       |                                                 |
                 |       ^   emit .pause();        +=================+     |
                 |       ^---^---------------------+  return false;  <-----+---+
                 |                                 +=================+         |
                 |                                                             |
                 ^   when queue is empty   +============+                      |
                 ^---^-----------------^---<  Buffering |                      |
                     |                     |============|                      |
                     +> emit .drain();     |  <Buffer>  |                      |
                     +> emit .resume();    +------------+                      |
                                           |  <Buffer>  |                      |
                                           +------------+  add chunk to queue  |
                                           |            <--^-------------------<
                                           +============+
```

### Duplex Stream

```js
const { PassThrough } = require('stream');
const fs = require('fs');

const duplexStream = new PassThrough();

// can be piped from reaable stream
fs.createReadStream('tmp.md').pipe(duplexStream);

// can pipe to writable stream
duplexStream.pipe(process.stdout);

// 监听数据，这里直接输出的是 Buffer<Buffer 60 60  ... >
duplexStream.on('data', console.log);
```

### Transform Stream

```js
const util = require('util');
const Transform = require('stream').Transform;

function Base64Encoder(options) {
  Transform.call(this, options);
}

util.inherits(Base64Encoder, Transform);

Base64Encoder.prototype._transform = function(data, encoding, callback) {
  callback(null, data.toString('base64'));
};

process.stdin.pipe(new Base64Encoder()).pipe(process.stdout);
```

## 文件读写

`__dirname` 指向当前文件所在目录。我们可以使用 join 与 resolve 两个辅助函数来进行路径构造，区别在于 join 只是进行简单连接，而 resolve 会进行根路径重置：

```js
path.join('/a', '/b'); // Outputs '/a/b'

path.resolve('/a', '/b'); // Outputs '/b'
```

```js
const { promisify } = require('util');
const fs = require('fs');
const readFileAsync = promisify(fs.readFile); // (A)
const filePath = process.argv[2];

readFileAsync(filePath, { encoding: 'utf8' })
  .then(text => {
    console.log('CONTENT:', text);
  })
  .catch(err => {
    console.log('ERROR:', err);
  });
```

# HTTP Server

## WebSocket

## Production | 部署

我们也可以将 Node.js 应用部署为 Systemd 服务，首先创建 `/etc/systemd/system/node-sample.service` 文件：

```yaml
[Unit]
Description=node-sample - making your environment variables rad
Documentation=https://example.com
After=network.target

[Service]
ExecStart=[node binary] /home/srv-node-sample/[main file]
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=node-sample
User=srv-node-sample
Group=srv-node-sample
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

然后使用 systemctl 命令启动与控制服务：

```sh
$ systemctl enable node-sample
$ systemctl start node-sample

# 查看应用状态
$ systemctl status node-sample

# 查看日志
$ journalctl -u node-sample
```

## Cluster | 集群模式

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end('hello world\n');
    })
    .listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

# Express

# Koa
