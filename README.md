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

通过监听 fetch 事件的触发来拦截任何 HTTP 请求

```js
self.addEventListener('fetch', function (event) {                                        
  if (/\.jpg$/.test(event.request.url)) {                                                
    event.respondWith(
      new Response('<p>This is a response that comes from your service worker!</p>', {
        headers: { 'Content-Type': 'text/html' }                                         
      });
    );
  }
});
```



### 3.3 save-data

一些现代浏览器可以“选择性加入”功能以允许用户节省数据。如果启用此功能，浏览器会为每个 HTTP 请求添加一个新的首部，使用 Service Workers 可以进入 fetch 事件并决定是否返回网站的“轻量级”版本

```js
this.addEventListener('fetch', function (event) {

  if(event.request.headers.get('save-data')){
    // 我们想要节省数据，所以限制了图标和字体
    if (event.request.url.includes('fonts.googleapis.com')) {
        // 不返回任何内容
        event.respondWith(new Response('', {status: 417, statusText: 'Ignore fonts to save data.' }));
    }
  }
});

```

## 4. 添加至首屏

### 4.1 manifest.json

#### 4.1.1 概述

PWA 添加至桌面的功能实现依赖于 manifest.json。PWA 添加到主屏幕的不仅仅是一个网页快捷方式，它将提供更多的功能，让 PWA 具有更加原生的体验。

为了实现 PWA 应用添加至桌面的功能，除了要求站点支持 HTTPS 之外，还需要准备 manifest.json 文件去配置应用的图标、名称等信息。

```json
{
    "short_name": "短名称",
    "name": "这是一个完整名称",
    "icon": [
        {
            "src": "icon.png",
            "type": "image/png",
            "sizes": "48x48"
        }
    ],
    "start_url": "index.html"
}
```


使用 link 标签将 manifest.json 部署到 PWA 站点 HTML 页面的头部，如下所示：

```html
<link rel="manifest" href="path-to-manifest/manifest.json">
```

#### 4.1.2. 基本功能

##### 4.1.2.1 自定义名称

PWA在通过应用安装横幅引导用户安装 app，以及被添加到主屏幕时，需要显示应用名称以便用户将其与其他应用区分开来。对应的属性为：

- name: {string} 应用名称，用于安装横幅、启动画面显示
- short_name: {string} 应用短名称，用于主屏幕显示

所以用两个字段做区分，是由于显示在主屏幕的应用名称长度有限，超长部分会被截断并显示省略号，需要设置较短的应用名称优化显示；而安装横幅没有长度限制，可以将完整的应用名称显示出来。

##### 4.1.2.2 自定义图标

当用户将 PWA 添加至主屏幕时，会如同原生应用一样显示应用名和图标。我们可以通过 icons 属性定义一组不同大小的图标供浏览器进行选择。

- icons: {Array.<ImageObject>} 应用图标列表

其中 ImageObject 的属性值包括：

- src: {string} 图标 url
- type {string=} 图标的 mime 类型，非必填项，该字段可让浏览器快速忽略掉不支持的图标类型
- sizes {string} 图标尺寸，格式为widthxheight，宽高数值以 css 的 px 为单位。如果需要填写多个尺寸，则使用空格进行间隔，如"48x48 96x96 128x128"

当PWA添加到主屏幕时，浏览器会根据有效图标的 sizes 字段进行选择。首先寻找与显示密度相匹配并且尺寸调整到 48dp 屏幕密度的图标；如果未找到任何图标，则会查找与设备特性匹配度最高的图标；如果匹配到的图标路径错误，将会显示浏览器默认 icon。


##### 4.1.2.3 设置启动网址

当PWA添加到主屏幕后，需要通过 start_url 去指定应用打开的网址。

- start_url: {string=} 应用启动地址
如果该属性为空，则默认使用当前页面，这可能不是用户想要的内容，因此建议配置 start_url；如果 start_url 配置的相对地址，则基地址与 manifest.json 相同。

##### 4.1.2.4 设置作用域

对于一些大型网站而言，有时仅仅对站点的某些模块进行 PWA 改造，其余部分仍为普通的网页。因此需要通过 scope 属性去限定作用域，超出范围的部分会以浏览器的方式显示。

- scope: {string} 作用域

scope 应遵循如下规则：

- 如果没有在 manifest 中设置 scope，则默认的作用域为 manifest.json 所在文件夹；
- scope 可以设置为 ../ 或者更高层级的路径来扩大PWA的作用域；
- start_url 必须在作用域范围内；
- 如果 start_url 为相对地址，其根路径受 scope 所影响；
- 如果 start_url 为绝对地址（以 / 开头），则该地址将永远以 / 作为根地址；


#### 4.2 改善应用体验

#### 4.2.1 添加启动画面

当 PWA 从主屏幕点击打开时，幕后执行了若干操作：

1. 启动浏览器
2. 启动显示页面的渲染器
3. 加载资源

在这个过程中，由于页面未加载完毕，因此屏幕将显示空白并且看似停滞。如果是从网络加载的页面资源，白屏过程将会变得更加明显。因此 PWA 提供了启动画面功能，用标题、颜色和图像组成的画面来替代白屏，提升用户体验。

#### 4.2.2 设置图像和标题

浏览器会从 icons 中选择最接近 128dp 的图片作为启动画面图像。标题则直接取自 name。

#### 4.2.3 设置启动背景颜色

通过设置 background_color 属性可以指定启动画面的背景颜色。

- background_color: {Color} css色值


#### 4.2.4 设置启动显示类型

仅当显示类型 display 设置为 standalone 或 fullscreen 时，PWA 启动的时候才会显示启动画面。


#### 4.2.5 设置显示类型

可以通过设置 display 属性去指定 PWA 从主屏幕点击启动后的显示类型。

- display: {string} 显示类型
显示类型的值包括以下四种：

|显示类型	|描述|降级显示类型|
|-------|----------|-------|
|fullscreen|	应用的显示界面将占满整个屏幕 |standalone|
|standalone	|浏览器相关UI（如导航栏、工具栏等）将会被隐藏	 |minimal-ui|
|minimal-ui	|显示形式与standalone类似，浏览器相关UI会最小化为一个按钮，不同浏览器在实现上略有不同 |browser|
|browser	|浏览器模式，与普通网页在浏览器中打开的显示一致	|(None)|


> CSS中可以通过 display-mode 这个媒体查询条件去指定在不同的显示类型下不同的显示样式，如：

```css
@media all and (display-mode: fullscreen) {
    body {
        margin: 0;
    }
}

@media all and (display-mode: standalone) {
    body {
        margin: 1px;
    }
}

@media all and (display-mode: minimal-ui) {
    body {
        margin: 2px;
    }
}

@media all and (display-mode: browser) {
    body {
        margin: 3px;
    }
}
```

#### 4.2.6 指定页面显示方向

PWA允许应用通过设置 orientation 属性的值，强制指定显示方向。

- orientation: string 应用显示方向

orientation属性的值有以下几种：

- landscape-primary
- landscape-secondary
- landscape
- portrait-primary
- portrait-secondary
- portrait
- natural
- any

由于不同的设备的宽高比不同，因此对于“横屏”、“竖屏”不能简单地通过屏幕旋转角去定义。如对于手机来说，90° 和 270° 为横屏，而在某些平板电脑中，0° 和 180° 才是横屏。因此需要通过应用视窗去定义。

- 当视窗宽度大于高度时，当前应用处于“横屏”状态。横屏分为两种角度，两者相位差为 180°，分别为 landscape-primary 和 landscape-secondary。
- 当视窗宽度小于等于高度时，当前应用处于“竖屏”状态。同样，竖屏分为两种，两者相位差为 180°，分别为 portrait-primary 和 portrait-secondary。

有了 landscape-primary、landscape-secondary、portrait-primary、portrait-secondary 的定义，我们就可以用它们来定义其他的属性值了。

- landscape: 根据不同平台的规则，该值可等效于 landscape-primary 或 landscape-secondary，或者根据当前屏幕旋转角不同，去自由切换 landscape-primary 或 landscape-secondary；
- portrait: 根据不同平台的规则，该值可等效于 portrait-primary 或 portrait-secondary，或者根据当前屏幕旋转角不同，去自由切换 portrait-primary 或 portrait-secondary；
- natural: 根据不同平台的规则，该值可等效于 portrait-primary 或 landscape-primary，即当前屏幕旋转角为 0° 时所对应的显示方向；
- any: 根据屏幕旋转角自由切换 landscape-primary、landscape-secondary、portrait-primary、portrait-secondary。


#### 4.2.7 设置主题颜色


通过设置 theme_color 属性可以指定 PWA 的主题颜色。可以通过该属性来控制浏览器 UI 的颜色。比如 PWA 启动画面上状态栏、内容页中状态栏、地址栏的颜色，会被 theme_color 所影响。

- theme_color: {Color} css色值
对于当前版本的 Chrome 浏览器，在 browser 显示类型下，内容页的状态栏、地址栏并不会显示成 theme_color 所指定的颜色。


在指定了 theme_color 的值之后，地址栏依然呈白色。针对这种情况，可以在页面 HTML 里设置 name 为 theme-color 的 meta 标签，例如：

```html
<meta name="theme-color" content="green">
```

## 4.3. 应用添加横幅

PWA 提供两种添加应用横幅的形式，分别实现引导用户添加 PWA 至桌面和引导用户下载原生应用的功能。

### 4.3.1. 引导用户添加应用至主屏幕

打开浏览器菜单，会看到添加到主屏幕的功能，用户可以点击该选项手动将 PWA 站点添加至主屏幕。

#### 4.3.1.1 显示应用安装横幅的条件

- 站点部署 manifest.json，该文件需配置如下属性：

    + short_name （用于主屏幕显示）
    + name （用于安装横幅显示）
    + icons （其中必须包含一个 mime 类型为 image/png 的图标声明）
    + start_url （应用启动地址）
    + display （必须为 standalone 或 fullscreen）

- 站点注册 Service Worker。
- 站点支持 HTTPS 访问。
- 站点在同一浏览器中被访问至少两次，两次访问间隔至少为 5 分钟。


#### 4.3.1.2 应用安装横幅事件

1. 判断用户是否安装此应用

beforeinstallprompt 事件返回一个名为 userChoice 的 Promise 对象，并在当用户对横幅进行操作时进行解析。promise 会返回属性 outcome，该属性的值为 dismissed 或 accepted，如果用户将网页添加到主屏幕，则返回 accepted。

```js
window.addEventListener('beforeinstallprompt', function (e) {
    // beforeinstallprompt event fired

    e.userChoice.then(function (choiceResult) {
        if (choiceResult.outcome === 'dismissed') {
            console.log('用户取消安装应用');
        }
        else {
            console.log('用户安装了应用');
        }
    });
});
```

2. 取消或延迟安装横幅的触发事件

网站虽然不能主动触发安装横幅的显示事件，但是当该事件被浏览器触发之后，可以对其进行取消或者延迟。

通过阻止 beforeinstallprompt 事件的默认行为，即可取消横幅弹出：

```js
window.addEventListener('beforeinstallprompt', function (e) {
    e.preventDefault();
    return false;
});
```

beforeinstallprompt 事件返回一个名为 prompt 的方法，通过执行该方法可以触发安装横幅的显示。为了实现显示事件的延迟操作，可以将 beforeinstallprompt 事件的返回值给存储起来，再异步地调用 prompt()。


```js
var deferredPrompt = null;

window.addEventListener('beforeinstallprompt', function (e) {
    // 将事件返回存储起来
    deferredPrompt = e;

    // 取消默认事件
    e.preventDefault();
    return false;
});

// 当按钮点击事件触发的时候，再去触发安装横幅的显示
button.addEventListener('click', function () {
    if (deferredPrompt != null) {
        // 异步触发横幅显示
        deferredPrompt.prompt();
        deferredPrompt = null;
    }
});
```


通过 prompt() 触发显示的横幅，同样可以通过 userChoice 去监测用户的安装行为：

```js
button.addEventListener('click', function () {
    if (deferredPrompt != null) {
        // 异步触发横幅显示
        deferredPrompt.prompt();
        // 检测用户的安装行为
        deferredPrompt.userChoice.then(function (choiceResult) {
            console.log(choiceResult.outcome);
        });

        deferredPrompt = null;
    }
});
```


### 4.3.2. 引导用户安装原生应用

#### 4.3.2.1 显示原生应用安装横幅的条件

- 站点部署 manifest.json，该文件需配置如下属性：

    + short_name （用于主屏幕显示）
    + name （用于安装横幅显示）
    + icons （其中必须包含一个 192x192 且 mime 类型为 image/png 的图标声明）
    + 包含原生应用相关信息的 related_applications 对象

- 站点注册 Service Worker。
- 站点支持 HTTPS 访问。
= 站点在同一浏览器中被访问至少两次，两次访问间隔至少为 2 天。

其中 related_applications 的定义如下：

- related_applications: Array.<AppInfo> 关联应用列表
AppInfo 的属性值包括：

- platform: {string} 应用平台
- id: {string} 应用id

例如：

```json
"related_applications": [
    {
        "platform": "play",
        "id": "com.baidu.samples.apps.iosched"
    }
]
```

如果只希望用户安装原生应用，而不需要弹出横幅引导用户安装 PWA，那么可以在 manifest.json 设置：

```json
"prefer_related_applications": true
```


## 5. 推送通知
