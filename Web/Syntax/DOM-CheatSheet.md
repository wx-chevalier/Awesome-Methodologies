[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# DOM 语法速览与实践清单

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

相较于 XHR，fetch 优势即在于能够访问到底层的数据流，并且添加自定义的操作以进行局部响应（避免接受全部内容），或者在 ServiceWorker 中进行流转化：

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

* same-origin: 仅针对同域请求有效，拒绝其他域的请求。

* cors: 允许针对同域或者其他访问正确的 CORS 头的请求。

* cors-with-forced-preflight: 每次请求前都会进行预检。

* no-cors: 针对那些未返回 CORS 头的域发起请求，并且返回 opaque 类型的响应，并不常用。

参照上文，

```js
fetch('http://some-site.com/cors-enabled/some.json', { mode: 'cors' });
```
