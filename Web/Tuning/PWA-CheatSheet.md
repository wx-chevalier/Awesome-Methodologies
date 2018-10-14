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
    navigator.serviceWorker.register(‘/sw.js’).then(
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

然后在 sw.js 文件中缓存资源：

```js
self.addEventListener('install', e => {
  let timeStamp = Date.now();
  e.waitUntil(
    caches.open('airhorner').then(cache => {
      return cache
        .addAll([
          `/`,
          `/index.html?timestamp=${timeStamp}`,
          `/styles/main.css?timestamp=${timeStamp}`,
          `/scripts/main.min.js?timestamp=${timeStamp}`,
          `/scripts/comlink.global.js?timestamp=${timeStamp}`,
          `/scripts/messagechanneladapter.global.js?timestamp=${timeStamp}`,
          `/sounds/airhorn.mp3?timestamp=${timeStamp}`
        ])
        .then(() => self.skipWaiting());
    })
  );
});

self.addEventListener('activate', event => {
  event.waitUntil(self.clients.claim());
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request, { ignoreSearch: true }).then(response => {
      // 默认不会保护 Cookie，可以使用 fetch(url, {credentials: 'include'}) 来发送 Cookie
      return response || fetch(event.request);
    })
  );
});
```
