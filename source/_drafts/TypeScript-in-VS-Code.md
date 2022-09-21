---
title: 在 Visual Studio Code 上搭建一个 TypeScript 开发环境
tags:
  - TypeScript
---

> 由于 TypeScript 和 JavaScript 特殊的关系，谈 TypeScript 绕不开的是 JavaScript，一般来说，TypeScript 代码需要使用 tsc 编译为 JavaScript 代码才能运行。所以我们这里先搭建一个 JavaScript 的运行和调试环境，先利用一下 JavaScript 来熟悉一下 VS Code 的调试和配置方式，后面再针对 TypeScript 来进行设置和配置。

# 从 JavaScript 谈起

在 VS Code 的调试功能介绍中，提到了其有一个内置的调试器，可以对 JavaScript 进行调试。作为调试控制器，它的作用是用来调试代码的，并不能启动代码，你的 JavaScrip 代码必须要在一个运行时上执行，这个调试器才能连接到运行时上，完成在 VS Code 上调试的效果。而普遍的 JavaScript 运行环境，就是浏览器和 nodejs。对于 VS Code 的调试和启动的配置，是通过文件夹 .vscode 中的 launch.json 文件来设置的，虽然打开后有一些默认的配置项，但是如果默认配置项不能实现自己的目的的时候，还是建议自己新建文件去进行配置。

https://code.visualstudio.com/docs/editor/debugging

## 浏览器调试

如果你只是要运行和调试 JS 代码，由于默认的调试器可以方便的和 nodejs 连接，只要你安装 nodejs 作为你的 JavaScript 运行时，你就可以方便的进行代码的执行和调试工作。不过我们这里先使用浏览器来配置一个开发环境。
在浏览器上运行 JS 代码非常的简单，只要打开 DevTools 的 Console 就可以编写和运行 JS 代码。而对于一个网页的 JS 资源文件，可以通过在上面打断点来进行调试。
对于 VS Code 的调试工具，只要告诉它你的浏览器和 url 地址，它就可以完成连接实现使用浏览器进行调试。所以我们现在的问题就转变为了，如何在浏览器中打开我们的网页。通过新建一个 html 文件并链接上自己本地的 .js 文件，就可以在浏览器中打开自己的网页；而在开发更常见的一种情况是，我们运行项目的集成框架，会打开一个服务使得我们的网页可以利用本地地址来访问。
接下来我将根据这两种情况并结合配置的参数，来展示一下如何配置和执行。

### 打开单个文件

在文件夹目录中新建你的 html 文件，然后在 script 元素中填写你的 JS 文件名。

``` json
{
  "name": "Launch index.html",
  "request": "launch",
  "type": "chrome",
  "file": "${workspaceFolder}/index.html"
},
```

按照如上配置，在选择该配置进行调试的时候，VS Code 会在你的 Chrome 浏览器中打开这个文件，而你的 JS 文件就能依靠浏览器这个环境来进行调试。但是有一个问题是，你对文件的修改，并不能触发浏览器的自动刷新，每次修改代码之后，你需要对网页进行刷新后才能看到修改后的效果。

在这个配置中，有一些属性需要进行一些讲解。属性 name 是你给调试配置起的名字，request 决定了调试程序是如何运行的，两个属性分别为 launch 和 attach，“启动”和“附加”。这两个属性的选择，来自你的使用习惯。表现上来说，选择“启动”的话，VS Code 会以调试模式替你打开你的 Chrome 浏览器，并在地址栏输入你的 html 文件地址；而“附加”需要你自己去打开这个文件。启动模式下，VS Code 会以调试模式打开你的程序进程后，并自动将调试器连接到你的程序进程的调试端口上；而附加模式需要你根据运行代码的程序，手动配置好调试端口。

### 附加模式配置

# 备注

## 运行时

在看文档的时候，经常会注意到文档中会提到一个叫做 runtime 的词，比如：
"VS Code has built-in debugging support for the Node.js runtime and can debug JavaScript, TypeScript, or any other language that gets transpiled to JavaScript."
"For debugging other languages and runtimes (including PHP, Ruby, Go, C#, Python, C++, PowerShell and many others), look for Debuggers extensions in the VS Code Marketplace or select Install Additional Debuggers in the top-level Run menu."
code.visualstudio.com/docs/editor/debugging
runtime 这个词一般被翻译成运行时。这有点像 run-time，但是实际上有一些区别。在之前谈 event loop 的文章里，我简单提到了一下这个概念，但是我感觉还是有必要再深入说说我的理解。
我个人的理解是，runtime 是一组内容的概括，属于通过抽象得到的一个概念。它作为操作系统和代码之间的一个抽象层，代码执行时需要的东西，执行造成的影响都可以归到这个概念中去。为了防止混淆，我这里特地使用了“执行”这个词，而不使用“运行”。以代码执行开始和结束的时间为界限，期间涉及到的所以东西都可以算作运行时这一个概念所包括的内容。举例来说，执行代码的 JavaScript 引擎，无论是浏览器还是 nodejs 中对事件循环的具体实现，浏览器或 nodojs 所提供的 api；这些执行代码、去调用系统资源代码、控制事件的这些软件的集合，都是运行时包含的东西。

在执行 JavaScript 代码的时候，JavaScript 运行时实际上维护了一组用于执行 JavaScript 代码的代理。每个代理由一组执行上下文的集合、执行上下文栈、主线程、一组可能创建用于执行 worker 的额外的线程集合、一个任务队列以及一个微任务队列构成。除了主线程（某些浏览器在多个代理之间共享的主线程）之外，其他组成部分对该代理都是唯一的。
https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth
Node.js 运行时是负责安装 Web 服务代码及其依赖项并运行服务的软件栈。
cloud.google.com/appengine/docs/standard/nodejs/runtime?hl=zh-cn


