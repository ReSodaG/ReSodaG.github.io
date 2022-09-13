---
title: JavaScript 异步 02 - 微任务和 Web Workers
tags:
  - JavaScript
date: 2022-09-01 23:20:27
---


## 推荐阅读

- [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.cn/post/6844903512845860872)

- [微任务、宏任务与 Event-Loop](https://juejin.cn/post/6844903657264136200)

这两篇文章关于事件循环和运行机制讲的是比较易懂的，我也从这两篇文章中获得了很多的启发，在看本文之前，可以先看看这两篇文章，或许可以解决你的很多疑问。

## 引言

Promise 的出现，使得传统的事件循环机制出现了一些改变，在上一篇文章中，讲了并发模型和事件循环，但是没有涉及到 Promise 和 async/await 这两个分别在 ES6 和 ES8 中新添加的特性，这两个特性又给这个模型带来新的变化。这篇文章主要就基于这两个新添加特性对事件循环的影响来进行补充。

## 宏任务和微任务

在上一篇的文章中，我们知道，JavaScript 会把定时器的回调函数作为消息放入消息队列中进行事件循环，那么针对 Promise，浏览器会对他的回调做怎样的处理呢？答案是对任务去进行精细的区分，分为宏任务（macro-task）和微任务（micro-task）两种，然后放在不同的队列中。宏任务依旧是之前事件模型中定时器的回调函数队列，但是微任务队列中存放的，就是 Promise 的回调函数队列了。

这时候，我们的事件模型有了变化，在每次循环中，除了之前的宏任务队列，还多了一个微任务队列，而 JavaScript 的代码执行顺序就会受到两个队列的调度先后的影响。 **浏览器将会优先处理微任务队列，在微任务队列空的时候再去处理宏任务队列，这一变化带来的结果是，Promise 的回调将会在定时器的回调之前执行。如果一个微任务队列中的回调函数作用是返回一个新的微任务，将会微任务队列一直不为空，就会阻塞宏任务的执行。即便在这个过程中，微任务出队入队很多次任务，但是这依旧算作一次事件循环。** 可以看下图来加深一下理解：

{% asset_img event-loop-microtasks.png 有微任务的模型结构 %}

> [图片来源](https://zh.javascript.info/event-loop)

如上图所示，在事件循环中，宏任务列表的第一个任务是执行网页 script 标签中的代码，然后执行的过程中产生了微任务，用户触发了移动鼠标的监听器，和入队了如图中 setTimeout 一样的宏任务。然后浏览器将会优先执行完微任务的所以任务，就像图中 microtasks 前的循环标记一样，直到微循环结束之后才去渲染网页，执行其中的元素修改，此时一个事件循环才处理完，接下来处理用户鼠标移动触发的宏任务……

## 执行顺序

为了证明这些执行顺序，我写了一个小 demo 大家可以尝试一下。这里是[链接](https://storh.github.io/event-loop-test/)和[仓库](https://github.com/Storh/event-loop-test)。

### 证明微任务的执行是根据微任务的入队顺序

```JavaScript
Promise.resolve()
    .then(() => console.log("Promise 0 1"))
    .then(() => console.log("Promise 0 2"));

Promise.resolve()
    .then(() => console.log("Promise 1 1"))
    .then(() => console.log("Promise 1 2"));

```

结果如下：

```
Promise 0 1
Promise 1 1
Promise 0 2
Promise 1 2
```

可以看到，这两个 Promise 的回调先后进入微任务队列，当第一个回调函数执行的时候，将链式调用的回调函数重新加入队列，根据队列的先进先出特性，接下来执行的就应该是第二个 Promise 的回调函数，而不是链式调用的回调。

### 证明微任务总在宏任务前

```JavaScript
setTimeout(() => {
    console.log("setTimeout 0 start");
    Promise.resolve()
        .then(() => console.log("setTimeout 0 Promise 0"))
        .then(() => console.log("setTimeout 0 Promise 1"));
    console.log("setTimeout 0 end");
})

setTimeout(() => {
    console.log("setTimeout 1 start");
    Promise.resolve()
        .then(() => console.log("setTimeout 1 Promise 0"))
        .then(() => console.log("setTimeout 1 Promise 1"));
    console.log("setTimeout 1 end");
})

```

输出结果：

```
setTimeout 0 start
setTimeout 0 end
setTimeout 0 Promise 0
setTimeout 0 Promise 1
setTimeout 1 start
setTimeout 1 end
setTimeout 1 Promise 0
setTimeout 1 Promise 1

```

### 验证微任务队列的执行会阻塞渲染

这个验证的方法参考了[这个问题的回答](https://stackoverflow.com/questions/62562845/any-example-proving-microtask-is-executed-before-rendering)，做了一些小改进。

```JavaScript
const microtaskTimeout = (rowSecond) => {
    const startTime = Date.now();
    const second = Number(rowSecond);
    if (second) {
        const timeout = () => {
            if (Date.now() - startTime < second * 1000) {
                return Promise.resolve()
                    .then(timeout);
            }
        }
        timeout();
    }
}

const clickButton = () => {
    console.log("clickButton start")
    document.getElementById("intl").textContent = "点击后经过 5s 才修改";
    document.getElementById("line").style.backgroundColor = "red";
    microtaskTimeout(5);
    console.log("clickButton end")
}

```

当进行点击之后就会看到，即便修改 dom 的代码在前，但是页面还是被微任务队列阻塞，直到微任务队列空才修改了 dom 元素（在单步调试的时候不会有这种效果，dom 元素的修改会是实时的）。

## 微任务小结

了解事件循环的机制后，就可以轻松理解 JavaScript 的代码执行顺序了，在使用异步操作的时候就会更加得心应手。我在 demo 地址的最后也给了一个例子来演示执行顺序。文末给到的参考链接中也有很多会给出同样的小示例。 **但是要注意一个要注意的点，一切实现都是基于标准的。不能把目前的顺序或者某一个特定版本下的效果看作“金科玉律”，要去关注标准的变化，这点相信大家看完这个[问答](https://segmentfault.com/q/1010000016147496)之后就会有相同的体会。**

## Web Workers

当你需要在前端页面上运行一个非常花时间的程序的时候，应该怎么办？通过前面的学习我们可以知道，用户的操作是以事件的形式在页面上调度的，而微任务和长时间运行的程序都会阻塞事件循环，使得我们的点击事件等行为长时间无法响应。有一种利用现有的工具，将一个操作拆分成多个计算任务，在每个计算任务结束后再把下一次计算利用 setTimeout 的形式加入宏任务队列中，使得页面得以响应，还能利用这种形式去进行进度条的显示，具体的例子可以参考这个[用例](https://zh.javascript.info/event-loop#yong-li-2-jin-du-zhi-shi)。

这种解决办法增加了编码的复杂度，同时也不是所有的操作都方便更改成这样，所以在 Web API 中提供了这个功能——Web Workers。

Web Workers 可以让你在独立于页面主线程的后台线程中执行函数，来解决线程阻塞的问题。在我先前提到的[小 demo](https://storh.github.io/event-loop-test/) 中我也有做同样的例子，可以用来进行尝试，可以看到在执行 Web Workers 的时候并不影响在页面的点击和内容修改，而结果也可以正常的获取。同时，在 Chrome 的开发者工具的性能选项中，也可以看到，Web Workers 在执行的时候并没有像后面的例子一样阻塞主线程。

{% asset_img web-workers.png 浏览器的性能选项 %}

而关于 Web Workers 的具体介绍和用法，可以参考 mdn web docs 中的 [Web Workers API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API) 和[使用 Web Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers) 这两篇文档。同时，mdn 也提供了一个 [demo](https://github.com/mdn/dom-examples/tree/main/web-workers/simple-web-worker) 可以供了解和学习。

## 参考链接

1. [事件循环：微任务和宏任务](https://zh.javascript.info/event-loop)

2. [微任务（Microtask）](https://zh.javascript.info/microtask-queue)

3. [Any example proving microtask is executed before rendering?](https://stackoverflow.com/questions/62562845/any-example-proving-microtask-is-executed-before-rendering)

4. [Web Workers API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API)

5. [simple-web-worker](https://github.com/mdn/dom-examples/tree/main/web-workers/simple-web-worker)

6. [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

7. [深入：微任务与 Javascript 运行时环境](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth)

8. [在 JavaScript 中通过 queueMicrotask() 使用微任务](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide)

9. [async await 和 promise 微任务执行顺序问题](https://segmentfault.com/q/1010000016147496)

10. [JavaScript 事件循环(Event Loop)](https://segmentfault.com/a/1190000017970432)

11. [理解 javascript 中的事件循环(Event Loop)](https://segmentfault.com/a/1190000015112913)
