# PWA CheatSheet

# Service Worker

Service Worker 是 Web Worker 的一种，其常被当做 Web 应用之间，或者浏览器与网络之间的代理；致力于提供更良好的离线体验，并且能够介入到网络请求中完成缓存与更新等操作，此外还能够被用于通知推送、后台同步接口等。

A service worker is an event-driven worker registered against an origin and a path. It takes the form of a JavaScript file that can control the web page/site it is associated with, intercepting and modifying navigation and resource requests, and caching resources in a very granular fashion to give you complete control over how your app behaves in certain situations, (the most obvious one being when the network is not available.)

A service worker is run in a worker context: it therefore has no DOM access, and runs on a different thread to the main JavaScript that powers your app, so it is not blocking. It is designed to be fully async; as a consequence, APIs such as synchronous XHR and localStorage can't be used inside a service worker.

Service workers only run over HTTPS, for security reasons. Having modified network requests, wide open to man in the middle attacks would be really bad. In Firefox, Service Worker APIs are also hidden and cannot be used when the user is in private browsing mode.
