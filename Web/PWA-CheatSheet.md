[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# PWA CheatSheet

# Service Worker

Service Worker 是 Web Worker 的一种，其常被当做 Web 应用之间，或者浏览器与网络之间的代理；致力于提供更良好的离线体验，并且能够介入到网络请求中完成缓存与更新等操作，此外还能够被用于通知推送、后台同步接口等。

A service worker is an event-driven worker registered against an origin and a path. It takes the form of a JavaScript file that can control the web page/site it is associated with, intercepting and modifying navigation and resource requests, and caching resources in a very granular fashion to give you complete control over how your app behaves in certain situations, (the most obvious one being when the network is not available.)

A service worker is run in a worker context: it therefore has no DOM access, and runs on a different thread to the main JavaScript that powers your app, so it is not blocking. It is designed to be fully async; as a consequence, APIs such as synchronous XHR and localStorage can't be used inside a service worker.

Service workers only run over HTTPS, for security reasons. Having modified network requests, wide open to man in the middle attacks would be really bad. In Firefox, Service Worker APIs are also hidden and cannot be used when the user is in private browsing mode.

我们不可以在 Service Worker 中使用 Local Storage，并且 Service Worker 不可以使用任何的同步 API，不过可以使用 IndexDB，CacheAPI，或者利用 postMessage() 与界面进行交互。参考 [GoogleChromeLabs/airhorn](https://github.com/GoogleChromeLabs/airhorn)，可以使用如下的代码注册 ServiceWorker，并且使用 Cache API 来进行资源与请求缓存，首先需要注册 ServiceWorker：

```js
if (‘serviceWorker’ in navigator) {
  window.addEventListener(‘load’, function() {
    navigator.serviceWorker.register('/sw.js').then(
      function(registration) {
        // Registration was successful
        console.log(‘ServiceWorker registration successful with scope: ‘, registration.scope); },
      function(err) {
        // registration failed :(
        console.log(‘ServiceWorker registration failed: ‘, err);
      });
  });
}
```

ServiceWorker 的关键生命周期函数包括了 install, activate, fetch 等，在 install 中可以对部分资源进行即时缓存：

```js
// 当浏览器解析完 SW 文件时触发 install 事件
self.addEventListener('install', function(e) {
  // install 事件中一般会将 cacheList 中要换存的内容通过 addAll 方法，请求一遍放入 caches 中
  e.waitUntil(
    caches.open(cacheStorageKey).then(function(cache) {
      return cache.addAll(cacheList);
    })
  );
});
```

然后在 active 中做一些过期资源释放的工作，匹配到就从 caches 中删除：

```js
// 激活时触发 activate 事件
self.addEventListener('activate', function(e) {
  var cacheDeletePromises = caches.keys().then(cacheNames => {
    return Promise.all(
      cacheNames.map(name => {
        if (name !== cacheStorageKey) {
          return caches.delete(name);
        } else {
          return Promise.resolve();
        }
      })
    );
  });

  e.waitUntil(Promise.all([cacheDeletePromises]));
});
```

fetch 事件则是对于网络请求的截获：

```js
self.addEventListener('fetch', function(e) {
  // 在此编写缓存策略
  e.respondWith(
    // 可以通过匹配缓存中的资源返回
    caches.match(e.request)
    // 也可以从远端拉取
    fetch(e.request.url)
    // 也可以自己造
    new Response('自己造')
    // 也可以通过吧 fetch 拿到的响应通过 caches.put 方法放进 caches
  );
});
```

## Workbox

Workbox 极大地简化了 PWA 的构建过程，可以把 Workbox 理解为 Google 官方的 PWA 框架，它解决的就是用底层 API 写 PWA 太过复杂的问题，接管了监听 SW 的 install、active、 fetch 事件做相应逻辑处理等。

```js
// 首先引入 Workbox 框架
importScripts('https://g.alicdn.com/kg/workbox/3.3.0/workbox-sw.js');
workbox.setConfig({
  modulePathPrefix: 'https://g.alicdn.com/kg/workbox/3.3.0/'
});

workbox.precaching([
  // 注册成功后要立即缓存的资源列表
]);

// html的缓存策略
workbox.routing.registerRoute(
  new RegExp(''.*\.html'),
  workbox.strategies.networkFirst()
);

workbox.routing.registerRoute(
  new RegExp('.*\.(?:js|css)'),
  workbox.strategies.cacheFirst()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.cdn\.com/'),
  workbox.strategies.staleWhileRevalidate()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.img\.cdn\.com/'),
  workbox.strategies.cacheFirst({
    cacheName: 'example:img'
  })
);
```

通过 workbox.precaching 中的是 install 以后要塞进 caches 中的内容，workbox.routing.registerRoute 中第一个参数是一个正则，匹配经过 fetch 事件的所有请求，如果匹配上了，就走相应的缓存策略 workbox.strategies 对象为我们提供了几种最常用的策略，如下：

- HTML，如果你想让页面离线可以访问，使用 NetworkFirst，如果不需要离线访问，使用 NetworkOnly，其他策略均不建议对 HTML 使用。

- CSS 和 JS，情况比较复杂，因为一般站点的 CSS，JS 都在 CDN 上，SW 并没有办法判断从 CDN 上请求下来的资源是否正确（HTTP 200），如果缓存了失败的结果，问题就大了。这种我建议使用 Stale-While-Revalidate 策略，既保证了页面速度，即便失败，用户刷新一下就更新了。如果你的 CSS，JS 与站点在同一个域下，并且文件名中带了 Hash 版本号，那可以直接使用 Cache First 策略。

- 图片建议使用 Cache First，并设置一定的失效事件，请求一次就不会再变动了。
