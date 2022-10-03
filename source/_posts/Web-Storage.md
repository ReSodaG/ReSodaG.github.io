---
title: Web Storage
categories:
  - [前端, Web API]
tags:
  - Front-end
date: 2022-08-11 09:55:45
---


## 两种 Web Storage

Web Storage 有两种，分别是 sessionStorage 和 localStorage，区别主要在于两者的可用范围。

时间上，sessionStorage 在页面会话期间（浏览器处于打开的情况，页面重新加载或者恢复）可用，关闭浏览器就会清理。页面会话在浏览器打开期间一直保持，并且重新加载或恢复页面仍会保持原来的页面会话。而 localStorage 在浏览器关闭，再重新打开数据仍然存在。

空间上，sessionStorage 只在当前页面会话中存在，并不能跨标签；而 localStorage 只要在同一域名下，是跨浏览器窗口的。

## 详细解释

主要解释关于页面会话的问题，localStorage 并不需要更多的解释。

属于同一个页面会话：

1. 重新加载：刷新页面。

2. 恢复页面：恢复关闭的页面（如，chrome 的重新打开关闭的标签页功能），即便关闭这个页面后又打开了新的页面，只要是恢复这个行为，就还是上一个页面会话。

不属于同一个页面会话的情况：

1. 新的页面也就有新的页面会话：复制链接重新打开等行为并不算同一个页面会话，即“打开多个相同的 URL 的 Tabs 页面，会创建各自的 sessionStorage”。

2. 关闭浏览器才清理：关闭对应浏览器标签或窗口，会清除对应的 sessionStorage。

注意，存储在 sessionStorage 或 localStorage 中的数据特定于页面的协议。也就是说 http://example.com 与 https://example.com 的 sessionStorage 和 localStorage 相互隔离。各浏览器支持的 localStorage 和 sessionStorage 容量上限不同。

## 参考资料

* [Web Storage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API)
* [Window.sessionStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/sessionStorage)
* [Window.localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)
