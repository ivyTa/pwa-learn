# pwa-study

## 1. 基本介绍

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
