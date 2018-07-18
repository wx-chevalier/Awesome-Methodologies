[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# DOM CheatSheet | DOM 语法速览与实践清单

```js
const bodyRect = document.body.getBoundingClientRect(),
  elemRect = element.getBoundingClientRect(),
  offset = elemRect.top - bodyRect.top;

console.log('Element is ' + offset + ' vertical pixels from <body>');
```

```js
// 获取 StyleSheet 对象
var sheets = document.styleSheets; // returns an Array-like StyleSheetList
var sheet = document.styleSheets[0];

// 添加样式规则
sheet.insertRule('header { float: left; opacity: 0.8; }', 1);
```

# 界面事件

## 事件监听

```js
function ready(callback) {
  ...
  var state = document.readyState;
  if (state === 'complete' || state === 'interactive') {
    return setTimeout(callback, 0);
  }

  document.addEventListener('DOMContentLoaded', function onLoad() {
    callback();
  });
}
```

The DOM of the referenced element is not part of the DOM of the referencing HTML page. It has isolated style sheets.

## 事件传播与委托

[DOM Events 标准](http://www.w3.org/TR/DOM-Level-3-Events/)中规定了事件传播的三个阶段，分别为：

- Capturing Phase | 捕获阶段：自顶向下传递到目标元素
- Target Phase | 目标阶段：事件达到目标元素
- Bubbling Phase：事件自底向上传递

下图以 table 中的某个 `<td>` 元素为例，指明了事件传播的顺序：

![image](https://user-images.githubusercontent.com/5803001/42418731-decb30bc-82d8-11e8-8fd3-e29230398f85.png)

鉴于捕获阶段很少被提及，我们主要会讨论冒泡阶段；对于常见的 `addEventListener` 设置监听器函数而言，如果其第三个参数设置为 true，则会工作在捕获阶段；否则默认工作在冒泡阶段。

DOM 事件委托即指一种以单一通用父节点上绑定响应函数而不是在每个子元素上绑定响应函数的机制，它主要是依赖于上文提及的事件冒泡(Bubbling)机制。当某个元素上触发了某个事件之后，所有注册到该 Target 上的监听器都会被触发。并且该事件会按照 DOM 树的层级逐步冒泡传递到绑定在父层节点的监听器上，每个监听器都会检查是否是关联的 EventTarget，这一步骤会重复进行直到到达 Document 根节点。

```js
// document.addEventListener("click", delegate(buttonsFilter, buttonHandler));
function delegate(criteria, listener) {
  return function(e) {
    var el = e.target;
    do {
      if (!criteria(el)) {
        continue;
      }
      e.delegateTarget = el;
      listener.call(this, e);
      return;
    } while ((el = el.parentNode));
  };
}
```

# 网络通信

## XMLHTTPRequest

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.responseType = 'json';

xhr.onload = function() {
  console.log(xhr.response);
};

xhr.onerror = function() {
  console.log('Booo');
};

xhr.send();
```

## Fetch

XMLHttpRequest (XHR) 是经典的浏览器中网络请求框架，jQuery 则为我们封装了 jQuery.ajax(), jQuery.get(), jQuery.post() 等辅助方法。而 Fetch API 正逐步成为跨平台的基于 Promise 的异步网络请求标准，其基本用法如下：

```js
fetch('./file.json')
  .then(response => {
    response.json().then(data => {
      console.log(data);
    });
  })
  // 使用 catch 方法容错
  .catch(err => {
    console.error(err);
  });
```

### Request

fetch 函数允许传入 Request 对象，我们可以在封装 Request 对象的时候传入自定义配置:

```js
// Headers 对象用于设置请求头
const headers = new Headers();
headers.append('Content-Type', 'application/json');

const request = new Request('./file.json', {
  headers: new Headers({ 'Content-Type': 'application/json' })
});

fetch(request);
```

简单的 POST 请求构造方式如下：

```js
const options = {
  method: 'post',
  headers: {
    'Content-type': 'application/x-www-form-urlencoded; charset=UTF-8'
  },
  body: 'foo=bar&test=1'
};
fetch(url, options).catch(err => {
  console.error('Request failed', err);
});
```

### Response

fetch 函数会返回 [Stream](https://streams.spec.whatwg.org/) 类型的对象，其包含了关于请求以及响应的信息，可以通过如下方式访问元数据：

```js
fetch('./file.json').then(response => {
  // 返回状态码
  console.log(response.status);
  // 状态描述
  console.log(response.statusText);
  // 101, 204, 205, or 304 is a null body status
  // 200 to 299, inclusive, is an OK status (success)
  // 301, 302, 303, 307, or 308 is a redirect
  console.log(response.headers.get('Content-Type'));
  console.log(response.headers.get('Date'));
});
```

对于响应体，则需要使用内置的 text 或 json 方式访问：

```js
// 根据响应头决定读取方式
fetch(url).then(function(response) {
  if (response.headers.get('Content-Type') === 'application/json') {
    return response.json();
  }
  return response.text();
});

// 内置流读取函数
.arrayBuffer()
.blob()
.formData()
.json()
.text()
```

相较于 XHR，fetch 优势即在于能够访问到底层的数据流，并且添加自定义的操作以进行局部响应(避免接受全部内容)，或者在 ServiceWorker 中进行流转化：

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch('video.unknowncodec').then(function(response) {
      var h264Stream = response.body
        .pipeThrough(codecDecoder)
        .pipeThrough(h264Encoder);

      return new Response(h264Stream, {
        headers: { 'Content-type': 'video/h264' }
      });
    })
  );
});
```

response 作为流对象，往往只可以被读取一次，如果需要多次读取，那么应该使用 clone 方法获取复制体；不过这也意味着原始数据需要保存在内存中，直至所有的副本被读取或者内存回收。

```js
fetch(url).then(function(response) {
  return response.json().catch(function() {
    // This does not work:
    return response.text();
  });
  // 修改为
  // return response.clone().json()...
});
```

### CORS

发起 Fetch 请求后获得的响应体中会包含 `response.type` 属性，其有 basic， cors 以及 opaque 三种类型，用于标识响应结果的来源以及处理方式。当我们发起同域请求时，结果类型会被标识为 `basic`，并且不会对响应的读取有任何限制。当发起的是跨域请求，并且响应头中包含了 [CORS](http://enable-cors.org/) 相关设置，那么响应类型会被标识为 cors。cors 与 basic 还是非常相似的，除了 cors 响应限制了仅可以读取 `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, 以及 `Pragma` 这些响应头字段。最后，opaque 则是针对那些进行跨域访问但是并未返回 CORS 头的请求；标识为该类型的响应并不可以读取其中内容，也无法判断其是否请求成功。

针对响应类型的不同，我们也可以在请求体中预设不同的请求模式，以加以过滤：

- same-origin: 仅针对同域请求有效，拒绝其他域的请求。

- cors: 允许针对同域或者其他访问正确的 CORS 头的请求。

- cors-with-forced-preflight: 每次请求前都会进行预检。

- no-cors: 针对那些未返回 CORS 头的域发起请求，并且返回 opaque 类型的响应，并不常用。

参照上文，

```js
fetch('http://some-site.com/cors-enabled/some.json', { mode: 'cors' });
```

## Blob 与 File

### Blob

### 文件读取与预览

# 数据存储

# Event Loop 与 Worker

## Web Worker

Web Worker 即是运行在后台独立线程中的 JavaScript 脚本，可以用其来执行阻塞性程序以避免影响到页面的性能。Worker 会运行在独立的不同于当前 window 的全局上下文中，因此我们并不能再 Worker 中进行 DOM 操作。直接使用 Worker 构造函数创建的 worker 被称为 dedicated worker, 其运行在所谓的 [DedicatedWorkerGlobalScope](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope) 代表的上下文中，其仅允许创建脚本进行访问；而另一种 shared worker 则运行在 SharedWorkerGlobalScope 代表的上下文中，其允许多个脚本访问。实际上 ServiceWorkers 也是 Web Worker 的一种，其常被用于 Web 应用之间，或者浏览器与网络之间的代理；致力于提供更良好的离线体验，并且能够介入到网络请求中完成缓存与更新等操作。ServiceWorkers 同样能够被用于进行通知推送与后台同步接口，更多关于 ServiceWorkers 与 PWA 相关内容可以参考 [PWA-CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/blob/master/Web/Production/PWA-CheatSheet.md)。

```js
// 判断浏览器是否支持 Worker
typeof Worker !== 'undefined';

// 从脚本中创建 Worker
new Worker('workers.js');

// 使用字符串方式创建 Worker
new Worker('data:text/javascript;charset=US-ASCII,...');
```

创建完毕之后，我们主要依靠 postMessage 与 onmessage 回调来在 Worker 线程与 UI 线程之间进行消息传递：

```js
// worker.js
// 向主线程发送消息
postMessage('event from worker');
// 接收来自主线程的消息
onmessage = function(event) {};

// UI 
worker.onmessage = function(event) {};
worker.postMessage('event from ui');

// 关闭当前 worker
worker.terminate();
```

[worker-loader](https://github.com/webpack-contrib/worker-loader)

[workerize-loader]()

```js
// worker.js
export function expensive(time) {}

// app.js
import worker from 'workerize-loader!./worker';

let instance = worker(); // `new` is optional

instance.expensive(1000).then(count => {
  console.log(`Ran ${count} loops`);
});
```

You cannot use Local Storage in service workers. It was decided that service workers should not have access to any synchronous APIs. You can use IndexedDB instead, or communicate with the controlled page using postMessage().

By default, cookies are not included with fetch requests, but you can include them as follows: fetch(url, {credentials: 'include'}).
