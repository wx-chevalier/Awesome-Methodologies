[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# DOM 语法速览与实践清单

```js
const bodyRect = document.body.getBoundingClientRect(),
  elemRect = element.getBoundingClientRect(),
  offset = elemRect.top - bodyRect.top;

console.log('Element is ' + offset + ' vertical pixels from <body>');
```

```js
// Add Rules to Stylesheets with JavaScript
// http://davidwalsh.name/add-rules-stylesheets

/* Getting the Stylesheet
Which stylesheet you add the rules to is up to you.  If you have a specific stylesheet in mind, you can add an ID to the LINK or STYLE element within your page HTML and get the CSSStyleSheet object by referencing the element's sheet property.  The stylesheets can be found in the document.styleSheets object:*/

var sheets = document.styleSheets; // returns an Array-like StyleSheetList

/*
Returns:  
StyleSheetList {0: CSSStyleSheet, 1: CSSStyleSheet, 2: CSSStyleSheet, 3: CSSStyleSheet, 4: CSSStyleSheet, 5: CSSStyleSheet, 6: CSSStyleSheet, 7: CSSStyleSheet, 8: CSSStyleSheet, 9: CSSStyleSheet, 10: CSSStyleSheet, 11: CSSStyleSheet, 12: CSSStyleSheet, 13: CSSStyleSheet, 14: CSSStyleSheet, 15: CSSStyleSheet, length: 16, item: function}
*/

// Grab the first sheet, regardless of media
var sheet = document.styleSheets[0];

/* Creating a New Stylesheet
In many cases, it may just be best to create a new STYLE element for your dynamic rules.  This is quite easy:*/
var sheet = (function() {
  // Create the <style> tag
  var style = document.createElement('style');

  // Add a media (and/or media query) here if you'd like!
  // style.setAttribute("media", "screen")
  // style.setAttribute("media", "@media only screen and (max-width : 1024px)")

  // WebKit hack :(
  style.appendChild(document.createTextNode(''));

  // Add the <style> element to the page
  document.head.appendChild(style);

  return style.sheet;
})();

/* Adding Rules
CSSStyleSheet objects have an addRule method which allows you to register CSS rules within the stylesheet.  The addRule method accepts three arguments:  the selector, the second the CSS code for the rule, and the third is the zero-based integer index representing the style position (in relation to styles of the same selector): */
sheet.addRule('#myList li', 'float: left; background: red !important;', 1);

/* Inserting Rules
Stylesheets also have an insertRule method which isn't available in earlier IE's.  The insertRule combines the first two arguments of addRule: */
sheet.insertRule('header { float: left; opacity: 0.8; }', 1);

/* Safely Applying Rules
Since browser support for insertRule isn't as global, it's best to create a wrapping function to do the rule application.  Here's a quick and dirty method: */
function addCSSRule(sheet, selector, rules, index) {
  if (sheet.insertRule) {
    sheet.insertRule(selector + '{' + rules + '}', index);
  } else {
    sheet.addRule(selector, rules, index);
  }
}

// Use it!
addCSSRule(document.styleSheets[0], 'header', 'float: left');

/* Inserting Rules for Media Queries
Media query-specific rules can be added in one of two ways. The first way is through the standard insertRule method: */
sheet.insertRule(
  '@media only screen and (max-width : 1140px) { header { display: none; } }'
);
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

