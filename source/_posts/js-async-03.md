---
title: JavaScript 异步 03 - 增补
date: 2022-09-13 14:11:43
tags:
  - JavaScript
---

## 好生生的为什么看起来了标准

正如我在上篇文章里提到的，具体的实现是根据标准来的，我打算在这篇文章里面分享一些之前两篇文章中涉及到的标准，和我的一些理解，并给出一些我看过的相关的博客和参考资料。

## 都有哪些标准

### HTML

我们平常提到的 JavaScript，或者说在前端开发里面用到的 JavaScript，它到底由那些标准定义呢？主要是两个方面来限制的，一个是我们常说的 ES5、ES6 的 ECMA-262 标准，另一个就是 HTML 标准。在第一篇文章中的时候，我们提到了运行时这个概念，那是因为，在 ES5 标准的全文中，根本没有出现异步（async）这个词，此时兼容 ES5 标准的 JavaScript 中的异步功能必然是根据其他标准实现的。同样，在 setTimeout 这个功能归属于 Web API 的时候也可以看出这个功能的标准是遵循 HTML 标准中的要求，具体在[第 8.6 章 Timers](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html) 中。而之前文章里提到的 Event loops 事件循环，也正是在 HTML 标准的[第 8.1.6 章](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)中给出了规范。

在 HTML 的事件循环包括这些内容。事件循环负责了事件、HTML 解析、回调、资源的获取和响应 DOM 操作，要求每个事件循环都要有微任务队列。同时，也对事件循环模型的如何处理进行了规范，像上一篇文章中提到的，微任务的执行在页面渲染之前，也是根据这里的规范进行规定的。标准要求对微任务的检测要优先于更新渲染的行为，而标准对[执行微任务检查点](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint)的要求中，就提到了当微任务队列不为空的时候，去逐个出队执行微任务队列。

### ECMAScript

在 ECMA-262 中，第一次出现异步这个词，是在 [ES6](https://262.ecma-international.org/6.0/) 中描述 Promise 对象的时候。同时，在 ES6 标准的第 8.4 章 [Jobs and Job Queues](https://262.ecma-international.org/6.0/#sec-jobs-and-job-queues) 中，提到了一个先进先出的 Job Queue，并且要求了在每个 ECMAScript 的实现中，都要有 ScriptJobs 和 PromiseJobs 两个 Job Queue。而这里定义的 Job 的执行要求，就是在事件循环模型里经常可以看到的执行栈为空的情况。可以说，正是因为 ES6 引入了 Promise 对象，在 ECMA-262 中才明确有了异步，同时也给出了一个 Job 和 Job Queue 的标准来让开发者可以做出符合标准的引擎去对异步或者延迟事件进行调度。

### Promises/A+

Promises/A+ 是 Promise 的一个最小规范，按照 Promises/A+ 自己的描述，ECMA-262 对于 Promise 对象的添加和规范，主要是来自于 Promises/A+ 社区的努力。也就是说 Promise 对象在 JavaScript 中的实现，就是一个符合 Promises/A+ 标准的具体实现 ^[1]^。

## HTML 还是 ECMAScript

### ES5 时期

因为浏览器除了要有 JavaScript 的引擎外，还有对 DOM 的渲染等内容，所以浏览器作为 HTML 规范中的用户代理的一员 ^[2]^，HTML 规范对浏览器如何实现 ECMA-262 的影响是必然的。和 ECMA-626 不一样，HTML 只保存和发表当前的单一规范版本 ^[3]^，现在的 HTML 标准并不会像之前的 HTML 标准一样有版本号，而将会是一个不断改进的标准，并且覆盖并淘汰了过去所有的 HTML 版本。而我们日常所说的 HTML5 实际上应该指代的并不是某一个固定的 HTML 版本，而是指代现代 Web 技术  ^[4]^。所以要确定当年的 HTML 标准内容，还是需要去查看当时的版本。

在 2017 年的 HTML 标准中的第 8.1.3.7 章 [Integration with the JavaScript job queue](https://web.archive.org/web/20170228232624/https://html.spec.whatwg.org/multipage/webappapis.html#integration-with-the-javascript-job-queue) 中，提到了因为 ECMA-262 中的具体设计并不完善，所以要求用户代理（例如浏览器）遵从他设计的规范去实现 JavaScript 中的 promise job 的操作，从而保证可以把这些设计整合到事件循环中。

### ES13

在现在的 ES13 中，Job 操作和之前的 ES5 中并不一样，同时也在相当多的规范内容中提到了 HTML 规范来举例。具体的规范内容可以在 ECMA-262 的第 9.5 章 [Jobs and Host Operations to Enqueue Jobs](https://262.ecma-international.org/13.0/#sec-jobs) 中查看。而在 HTML 规范中，响应的章节内容在 8.1.5.4 中的 [Job-related host hooks](https://html.spec.whatwg.org/multipage/webappapis.html#integration-with-javascript-jobs) 一节中，具体写了 JavaScript 规范中定义的关于 Jobs 的内容，在用户代理（例如浏览器）中是如何定义的。

## 博客分享

我在研究这个的过程中，看到了一些写的不错的博客，也分享一下。

* [How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

* [How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

* [https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

* [JS Promise (Part 1 - Basics)](https://medium.com/@ramsunvtech/promises-of-promise-part-1-53f769245a53)

* [JS Promise (Part 2 - Q.js, When.js and RSVP.js)](https://medium.com/@ramsunvtech/js-promise-part-2-q-js-when-js-and-rsvp-js-af596232525c)

* [深入探究 eventloop 与浏览器渲染的时序问题](https://github.com/jin5354/404forest/issues/61)

* [Loupe](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

## 参考连接

1. [The ECMAScript Specification](https://promisesaplus.com/implementations#the-ecmascript-specification)

2. [HTML user agents (e.g., web browsers)](https://html.spec.whatwg.org/multipage/introduction.html#abstract:~:text=HTML%20user%20agents%20(e.g.%2C%20web%20browsers))

3. [W3C HTML](https://www.w3.org/html/)

4. [Is this HTML5?](https://html.spec.whatwg.org/multipage/introduction.html#is-this-html5?)

5. [ECMAScript 的 Job Queues 和 Event loop 有什么关系？](https://www.zhihu.com/question/40063533)

6. [HTML Living Standard](https://html.spec.whatwg.org/multipage/)

7. [ECMAScript® 2015 Language Specification](https://262.ecma-international.org/6.0/)

8. [ECMAScript® 2022 Language Specification](https://262.ecma-international.org/13.0/#sec-jobs)
