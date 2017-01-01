---

layout: post
title: "使用Service worker让页面支持离线访问"
date: 2016-12-30 16:42
comments: true

---
Google最近推行的[Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/)是个很酷的东西，而且采用的技术也是w3c的标准，不像微信小程序那样自己搞了一套封闭的东西。其实里边相关的一项技术`service worker`早在去年就有了，目前只有高版本的Chrome和Firefox支持，这个东西就可以把网页做成支持离线访问。

首先要在入口页面注册`service worker`：

```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/serviceworker.js').then(function(registration) {
    console.log('ServiceWorker registration successful with scope: ', registration.scope);
  }).catch(function(err) {
    console.log('ServiceWorker registration failed: ', err);
  });
}
```

注意这个`serviceworker.js`必须放在根目录下。

然后就是编辑这个`serviceworker.js`文件，控制哪些文件需要在本地缓存：

```
// 缓存名称
var ONLINE_CACHE_NAME = '2016-12-30 16:35';
// 缓存页面
var onlineCacheUrl = [
  '/',
  '/foo/bar/xxx.js',
  '/foo/bar/xxx.css',
  '/foo/bar/xxx.html',
  '/foo/bar/xxx.png',
];
```

删除过期的缓存

```
this.addEventListener('activate', function(event) {
  var cacheWhitelist = [ONLINE_CACHE_NAME];

  event.waitUntil(
    caches.keys().then(function(keyList) {
      return Promise.all(keyList.map(function(key) {
        if (cacheWhitelist.indexOf(key) === -1) {
          console.log('delete ' + key);
          return caches.delete(key);
        }
      }));
    })
  );
});
```

```
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open(ONLINE_CACHE_NAME)
    .then(function(cache) {
      console.log('Opened cache');
      return cache.addAll(onlineCacheUrl);
    })
  );
});
```

下面这个`fetch`事件是最重要的，这相当于一个本地的代理服务器，所有的请求都会经过这里，然后把缓存里存在的东西原样返回，缓存里没有的东西从服务器获取再返回。

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
    .then(function(response) {
        if (response) {
            return response;
        }
        return fetch(event.request);
    })
  );
});
```

最后还有一点，除了本地调试的`localhost`以外，只有https的页面才能采用`service worker`。可以参考这个例子：

[https://ooxx.gyteng.com](https://ooxx.gyteng.com)

首次打开页面以后，把网络关掉再次打开，就不会出现Chrome的小恐龙了。

![serviceworker0](/media/pic/serviceworker0.png)

打开Chrome的调试工具，在`Application -> ServiceWorker`一栏可以看到已经成功启用。

![serviceworker1](/media/pic/serviceworker1.png)
