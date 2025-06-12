---
tags:
  - 知识笔记/经验
---
> [!Info] 背景
> 可以理解为存在三个应用
> - 主应用A
> - 子应用B
> - 子应用C: C是lowcode引擎应用
> 
> 加载顺序是A会在特定路由通过**MicroApp**加载B,B又通过**MicroApp**加载C,C内部又会在需要的时候通过**MF**加载B暴露出来的MF组件。

>[!Danger] 前期分析结论
>1. 整体链路过长，且链路上存在两层**MicroApp**嵌套，第二层**MicroApp**又会可能动态加载MF组件
>2. lowcode引擎内部绘制画布时会引入大量静态资源，这些静态资源请求时间都很长
>3. 部分接口响应很慢

>[!Info] 处理理论分析
>1. 先尝试了microApp.preFetch和先在主应用中先在视口外渲染B，都在细节上存在不少Bug，大部分由于microApp的js沙箱处理多层嵌套存在问题，小部分由于C加载B的MF时，数据传递问题，最终放弃预渲染的方式
>2. 最后考虑了一下，选择了ServiceWorker来处理，理由是：
>	- 目前慢最大的影响是静态资源
>	- 二层嵌套暂时无法取消
>	- 接口目前甚至连一点可以用于缓存更新的能力都没有，所以接口没发缓存，ETag和Last-Modified响应头都没有提供
>3. 所以目前能做的其实也就是去缓存静态资源，所以采用ServiceWorker在应用启动时直接缓存需要的静态资源的方式处理

>[!Success] 关键ServiceWorker代码
>// 主线程注册代码
> ```typescript
> if ('serviceWorker' in navigator) {
> // window 加载完成之后开始注册
>   window.addEventListener('load', () => {
>     navigator.serviceWorker
>     // 设置注册范围，''/'' 表示注册到根域
>       .register('./service-worker.js', { scope: '/' })
>       .then((registration) => {
>         console.log('[ServiceWorker] ServiceWorker registered with scope:', registration.scope);
> 
>         // 检查更新
>         registration.onupdatefound = () => {
>           const installingWorker = registration.installing;
>           console.log('[ServiceWorker] ServiceWorker update found.');
>           installingWorker.onstatechange = () => {
>             if (installingWorker.state === 'installed') {
>               if (navigator.serviceWorker.controller) {
>                 // 新内容可用
>                 console.log('[ServiceWorker] New content available, please refresh.');
>               } else {
>                 // 内容已缓存
>                 console.log('[ServiceWorker] Content cached for offline use.');
>               }
>             }
>           };
>         };
>       })
>       .catch((error) => {
>         console.error('[ServiceWorker] ServiceWorker registration failed:', error);
>       });
>   });
> }
> ```
> serviceWorker线程代码:
> ```javascript
> const CACHE_NAME = `athena-web-cache-${Date.now()}`; // 使用当前时间戳生成唯一名称;
> // 需要去缓存的静态资源
> const URLS_TO_CACHE = [
>   'https://alifd.alicdn.com/npm/@alifd/theme-lowcode-light@0.2.1/variables.css',
>   '/相对路径'
> ];
> 
> const cacheAssets = async (cache) => {
>   for (const asset of URLS_TO_CACHE) {
>     try {
>       if (asset.startsWith('http') || asset.startsWith('https')) {
>         const response = await fetch(asset);
>         if (response.ok) {
>           await cache.put(asset, response);
>           console.log(`[ServiceWorker] Cached full path: ${asset}, response = `, response);
>         }
>       } else {
>         const baseUrl = self.location.origin;
>         const fullUrl = new URL(asset, baseUrl).href;
>         const response = await fetch(fullUrl);
>         if (response.ok) {
>           await cache.put(fullUrl, response);
>           console.log(`[ServiceWorker] Cached relative path: ${fullUrl}, response = `, response);
>         } else {
>           console.log(`[ServiceWorker] Cache relative path: ${fullUrl} failure`);
>         }
>       }
>     } catch (error) {
>       console.error(`[ServiceWorker] Failed to cache ${asset}:`, error);
>     }
>   }
> };
> 
> self.addEventListener('install', (event) => {
>   console.log('[ServiceWorker] Installing..., CACHE_NAME = ', CACHE_NAME);
>   event.waitUntil(
>     caches
>       .open(CACHE_NAME)
>       .then(cacheAssets)
>       .then(() => {
>         console.log('[ServiceWorker] ServiceWorker take control of the page...');
>         return self.skipWaiting(); // 返回 Promise
>       }),
>   );
> });
> 
> self.addEventListener('activate', (event) => {
>   console.log('[ServiceWorker] Activating...');
>   event.waitUntil(
>     caches
>       .keys()
>       .then((cacheNames) => {
>         console.log('[ServiceWorker] previous deleting old cache, CACHE_NAME = ', CACHE_NAME);
>         return Promise.all(
>           cacheNames.map((cacheName) => {
>             if (cacheName !== CACHE_NAME) {
>               console.log('[ServiceWorker] Deleting old cache, cacheName = ', cacheName);
>               return caches.delete(cacheName);
>             }
>             return Promise.resolve(); // 避免 map 中返回 undefined
>           }),
>         );
>       })
>       .then(() => {
>         console.log('[ServiceWorker] ServiceWorker take control of the page...');
>         return self.clients.claim(); // 返回 Promise
>       })
>       .catch((error) => {
>         console.error('[ServiceWorker] Activation failed:', error);
>       }),
>   );
> });
> 
> const updateCache = (request, cachedResponse) => {
>   console.log('[ServiceWorker] try to update cache...');
>   const cachedETag = cachedResponse.headers.get('ETag');
>   if (cachedETag) {
>     const fetchOptions = {
>       headers: {
>         'If-None-Match': cachedETag,
>       },
>     };
>     fetch(request, fetchOptions)
>       .then((networkResponse) => {
>         if (networkResponse.ok) {
>           console.log('[ServiceWorker] exist update, update cache...');
>           caches.keys().then((cacheNames) => {
>             cacheNames.map((cacheName) => {
>               caches.open(cacheName).then((cache) => {
>                 cache.put(request, networkResponse.clone());
>               });
>             });
>           });
>         } else {
>           console.log('[ServiceWorker] cache still freshes...');
>         }
>       })
>       .catch((error) => {
>         console.log('[ServiceWorker] update cache error...', error);
>       });
>   }
> };
> 
> // 拦截网络请求，优先返回缓存数据
> self.addEventListener('fetch', (event) => {
>   event.respondWith(
>     caches.match(event.request, { ignoreVary: true }).then((response) => {
>       if (response) {
>         console.log(`[ServiceWorker]: get data from ServiceWorker, url = `, event.request.url);
>         updateCache(event.request, response.clone());
>         return response;
>       }
>       console.log(`[ServiceWorker]: get data from network, url = `, event.request.url);
>       return fetch(event.request);
>     }),
>   );
> });
> ```

