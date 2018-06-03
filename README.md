# pwa-study

## 1. pwa基本介绍

### 1.1 概念

Progressive Web App, 简称 PWA，是提升 Web App 的体验的一种新方法，能给用户原生应用的体验。

PWA 能做到原生应用的体验不是靠特指某一项技术，而是经过应用一些新技术进行改进，在安全、性能和体验三个方面都有很大提升，PWA 本质上是 Web App，借助一些新技术也具备了 Native App 的一些特性，兼具 Web App 和 Native App 的优点。


### 1.2 特性

1. 渐进式 - 适用于所有浏览器，因为它是以渐进式增强作为宗旨开发的
2. 连接无关性 - 能够借助 Service Worker 在离线或者网络较差的情况下正常访问
3. 类似应用 - 由于是在 App Shell 模型基础上开发，因为应具有 Native App 的交互和导航，给用户 Native App 的体验
4. 持续更新 - 始终是最新的，无版本和更新问题
5. 安全 - 通过 HTTPS 协议提供服务，防止窥探和确保内容不被篡改
6. 可索引 - 应用清单文件和 Service Worker 可以让搜索引擎索引到，从而将其识别为『应用』
7. 粘性 - 通过推送离线通知等，可以让用户回流
8. 可安装 - 用户可以添加常用的 webapp 到桌面，免去去应用商店下载的麻烦
9. 可链接 - 通过链接即可分享内容，无需下载安装



## 2. Service Workers

### 2.1 概念

W3C 组织早在 2014 年 5 月就提出过 Service Worker 这样的一个 HTML5 API ，主要用来做持久的离线缓存。


### 2.2 特性

1. 一个独立的 worker 线程，独立于当前网页进程，有自己独立的 worker context。

2. 一旦被 install，就永远存在，除非被 uninstall

3. 需要的时候可以直接唤醒，不需要的时候自动睡眠（有效利用资源，此处有坑）

4. 可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）

5. 离线内容开发者可控

6. 能向客户端推送消息

7. 不能直接操作 DOM

8. 出于安全的考虑，必须在 HTTPS 环境下才能工作

9. 异步实现，内部大都是通过 Promise 实现

### 2.3 Service Worker 生命周期

当用户首次导航至 URL 时，服务器会返回响应的网页。在图中，你可以看到在第1步中，当你调用 register() 函数时， Service Worker 开始下载。在注册过程中，浏览器会下载、解析并执行 Service Worker (第2步)。如果在此步骤中出现任何错误，register() 返回的 promise 都会执行 reject 操作，并且 Service Worker 会被废弃。

一旦 Service Worker 成功执行了，install 事件就会激活 (第3步)。Service Workers 很棒的一点就是它们是基于事件的，这意味着你可以进入这些事件中的任意一个。我们将在本书的第3章中使用这些不同的事件来实现超快速缓存技术。

一旦安装这步完成，Service Worker 便会激活 (第4步) 并控制在其范围内的一切。如果生命周期中的所有事件都成功了，Service Worker 便已准备就绪，随时可以使用了！

![Service Worker生命周期1](/images/sw1.png)
![Service Worker生命周期2](/images/sw2.png)


### 2.4 注册 Service Worker

```js
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function () {
      navigator.serviceWorker.register('./sw.js')
        .then(function (registration) {

          // 注册成功
          console.log('ServiceWorker registration successful with scope: ', registration.scope);
        })
        .catch(function (err) {

          // 注册失败:(
          console.log('ServiceWorker registration failed: ', err);
        });
    });
  }
```


### 2.5  Service Workers 缓存

#### 2.5.1 在 Service Worker 安装过程中预缓存

当用户首次访问网站时，Service Worker 会开始下载并安装自身。在安装阶段中，我们可以进入这个事件，并准备缓存 Web 应用所需的所有重要资源。

在 Service Worker 安装阶段，我们可以获取资源并为下次访问准备好缓存;

代码进入了 install 事件，并在此阶段将 JavaScript 文件和 hello 图片添加到缓存中。在上面的清单中，我还引用了一个叫做 cacheName 的变量。这是一个字符串，我用它来设置缓存的名称。你可以为每个缓存取不同的名称，甚至可以拥有一个缓存的多个不同的副本，因为每个新的字符串使其唯一。当看到本章后面的版本控制和缓存清除时，你将会感受到它所带来的便利。

```js
var cacheName = 'helloWorld';
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(cacheName)
      .then(cache => cache.addAll([
        '../images/sw3.png'
      ]))
  );
});

```

如果所有的文件都成功缓存了，那么 Service Worker 便会安装完成。如果任何文件下载失败了，那么安装过程也会随之失败。这点非常重要，因为它意味着你需要依赖的所有资源都存在于服务器中，并且你需要注意决定在安装步骤中缓存的文件列表。定义一个很长的文件列表便会增加缓存失败的几率，多一个文件便多一份风险，从而导致你的 Servicer Worker 无法安装。


现在我们的缓存已经准备好了，我们能够开始从中读取资源。我们需要添加代码，让 Service Worker 开始监听 fetch 事件。

```js
self.addEventListener('fetch', function (event) {  
  event.respondWith(
    caches.match(event.request)                    
    .then(function (response) {
      if (response) {                              
        return response;                           
      }
      return fetch(event.request);                 
    })
  );
});
```

使用 caches.match() 函数来检查传入的请求 URL 是否匹配当前缓存中存在的任何内容。如果存在的话，我们就简单地返回缓存的资源。但是，如果资源并不存在于缓存当中，我们就如往常一样继续，通过网络来获取资源

#### 2.5.2  拦截并缓存

在 Service Worker 安装期间缓存任何重要的资源，这被称之为“预缓存”。

因为 Service Workers 能够拦截 HTTP 请求，对于我们来说，这是发起请求然后将响应存储在缓存中的绝佳机会。这意味着我们改为先请求资源，然后立即缓存起来。这样一来，对于同样资源的发起的下一次 HTTP 请求，我们可以立即将其从 Service Worker 缓存中取出。


![Service Worker缓存](/images/sw3.png)

**对于发起的任何 HTTP 请求，我们可以检查资源是否在缓存中已经存在，如果没有的话再通过网络来获取**

先通过添加事件监听器来进入 fetch 事件。我们首先要做的就是检查请求的资源是否存在于缓存之中。如果存在，我们可以就此返回缓存并不再继续执行代码。

```js
var cacheName = 'helloWorld';                                
self.addEventListener('fetch', function (event) {            
  event.respondWith(
    caches.match(event.request)                              
    .then(function (response) {
      if (response) {                                        
        return response;
      }
      var requestToCache = event.request.clone();            
      return fetch(requestToCache).then(                     
        function (response) {
          if (!response || response.status !== 200) {        
            return response;
          }
          var responseToCache = response.clone();            
          caches.open(cacheName)                             
            .then(function (cache) {
              cache.put(requestToCache, responseToCache);    
            });
          return response;
        }
      );
    })
  );
});
```

然而，如果请求的资源于缓存之中没有的话，我们就按原计划发起网络请求。在代码更进一步之前，我们需要克隆请求。需要这么做是因为请求是一个流，它只能消耗一次。因为我们已经通过缓存消耗了一次，然后发起 HTTP 请求还要再消耗一次，所以我们需要在此时克隆请求。然后，我们需要检查 HTTP 响应，确保服务器返回的是成功响应并且没有任何问题。我们绝不想缓存一个错误的结果！

如果成功响应，我们会再次克隆响应。你可能会疑惑我们为什么需要再次克隆响应，请记住响应是一个流，它只能消耗一次。因为我们想要浏览器和缓存都能够消耗响应，所以我们需要克隆它，这样就有了两个流。

最后，代码中使用这个响应并将其添加至缓存中，以便下次再使用它。如果用户刷新页面或访问网站另一个请求了这些资源的页面，它会立即从缓存中获取资源，而不再是通过网络。


#### 2.5.3 对文件进行版本控制

更新缓存时可以使用两种方式。第一种方式，可以更新用来存储缓存的名称。第二种方式，就是实际上对文件进行版本控制。这种技术被称为“缓存破坏”，而且已经存在很多年了。当静态文件被缓存时，它可以存储很长一段时间，然后才能到期。如果期间你对网站进行更新，这可能会造成困扰，因为文件的缓存版本存储在访问者的浏览器中，它们可能无法看到所做的更改。通过使用一个唯一的文件版本标识符来告诉浏览器该文件有新版本可用，缓存破坏解决了这个问题。

- 缓存破坏背后的理念是每次更改文件时创建一个全新的文件名，这样以确保浏览器可以获取最新的内容。

```js
<script type="text/javascript" src="/js/main-xtvbas65.js"></script>
```

#### 2.5.4 处理额外的查询参数

当 Service Worker 检查已缓存的响应时，它使用请求 URL 作为键。默认情况下，请求 URL 必须与用于存储已缓存响应的 URL 完全匹配，包括 URL 查询部分的任何字符串。

如果对文件发起的 HTTP 请求附带了任意查询字符串，并且查询字符串会更改，这可能会导致一些问题。例如，如果你对一个先前匹配的 URL 发起了请求，则可能会发现由于查询字符串略有不同而导致该 URL 找不到。当检查缓存时想要忽略查询字符串，使用 ignoreSearch 属性并设置为 true 。


```js
self.addEventListener('fetch', function (event) {
  event.respondWith(
    caches.match(event.request, {
      ignoreSearch: true
    }).then(function (response) {
      return response || fetch(event.request);
    })
  );
});

```

ignoreSearch 选项来忽略请求参数和缓存请求的 URL 的查询部分。你可以通过使用其他忽略选项 (如 ignoreMethod 和 ignoreVary) 进一步扩展。例如，ignoreMethod 选项会忽略请求参数的方法，所以 POST 请求可以匹配缓存中的 GET 项。ignoreVary 选项会忽略已缓存响应中的 vary 首部。


#### 2.5.5 Servicer Worker toolbox

[Service Worker toolbox](https://github.com/GoogleChrome/sw-toolbox)。它是由 Google 团队编写的，它是一个辅助库，以使你快速开始创建自己的 Service Workers，内置的处理方法能够涵盖最常见的网络策略。只需短短几行代码，你就可以决定是否只是要从缓存中提供指定资源，或者从缓存中提供资源并提供备用方案，或者只能从网络返回资源并且永不缓存。这个库可以让你完全控制缓存策略。

```js
importScripts('/sw-toolbox/sw-toolbox.js');          

toolbox.router.get('/css/(.*)', toolbox.cacheFirst);
```

先使用 importScripts 函数导入 Service Worker toolbox 库。Service Workers 可以访问一个叫做 importScripts() 的全局函数，它可以将同一域名下脚本导入至它们的作用域。这是将另一个脚本加载到现有脚本中的一种非常方便的方法。它保持代码整洁，也意味着你只需要在需要时加载文件。

一旦脚本导入后，我们就可以开始定义想要缓存的路由。定义了一个路由，它匹配路径是 /css/，并且永远使用缓存优选的方式。这意味着资源会永远从缓存中提供，如果不存在的话再回退成通过网络获取。Toolbox 还提供了一些其他内置缓存策略，比如只通过缓存获取、只通过网络获取、网络优先、缓存 优先或者尝试从缓存或网络中找到最快的响应。每种策略都可以应用于不同的场景，甚至你可以混用不同的路由来匹配不同的策略，以达到最佳效果。

Service Worker toolbox 还为你提供了预缓存资源的功能。在 Service Worker 安装期间预缓存了资源，我们可以使用 Service Worker toolbox 以同样的方式来实现，并且只需要一行代码。

```js
toolbox.precache(['/js/script.js', '/images/hello.png']);
```

在 Service Worker 安装步骤中应该被缓存的 URL 数组。这行代码会确保在 Service Worker 安装阶段资源被缓存。


## 3. Fetch 

### 3.1 Fetch API 

1. get请求

```js
fetch('/some/url', {           
  method: 'GET'
}).then(function (response) {  
  // 成功
}).catch(function (err) {      
  // 出问题了
});
```

2. post请求

```js
fetch('/some/url', {                            
    method: 'POST',
    headers: {
      'auth': '1234'                            
    },
    body: JSON.stringify({                      
      name: 'dean',
      login: 'dean123',
    })
  })
  .then(function (data) {                       
    console.log('Request success: ', data);
  })
  .catch(function (error) {                     
    console.log('Request failure: ', error);
  });
```


想在不支持的浏览器上使用的话，你可能要考虑使用 polyfill 。


### 3.2 Fetch 事件












