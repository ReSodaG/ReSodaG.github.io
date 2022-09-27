---
title: 在 Visual Studio Code 上搭建一个 TypeScript 开发环境
tags:
  - TypeScript
  - VS Code
---

先放结论：

首先需要安装 Node.js，然后使用命令`npm install -g typescript`全局安装 TypeScript，环境准备好后处理 VS Code 的配置。在项目的目录`.vscode`中新建文件`launch.json`，并新建启动配置如下：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch TypeScript Program After Build Task",
      "program": "${fileDirname}/${fileBasenameNoExtension}.ts",
      "request": "launch",
      "skipFiles": ["<node_internals>/**"],
      "type": "node",
      "preLaunchTask": "tsc: 使用 args 构建单个文件 - tsconfig.json"
    }
  ]
}
```

在同目录下新建文件`tasks.json`进行任务配置，具体参数如下：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "shell",
      "problemMatcher": ["$tsc"],
      "command": "tsc",
      "args": ["${file}", "--sourceMap", "true", "--outDir", "${workspaceFolder}\\debug\\${relativeFileDirname}\\"],
      "group": "build",
      "label": "tsc: 使用 args 构建单个文件 - tsconfig.json"
    }
  ]
}
```

---

## 从 JavaScript 谈起

由于 TypeScript 和 JavaScript 之间特殊的关系，谈 TypeScript 绕不开的是 JavaScript。一般来说，TypeScript 代码需要使用 tsc 编译为 JavaScript 代码才能运行。所以我们这里先搭建一个 JavaScript 的运行和调试环境，利用 JavaScript 来熟悉一下 VS Code 的调试和配置方式，再对 TypeScript 来进行设置和配置。

在 VS Code 的调试功能^[1]^介绍中，提到了其有一个内置的调试器，可以对 JavaScript 进行调试。作为调试控制器，它的作用是用来调试代码的，并不能启动代码，你的 JavaScrip 代码必须要在一个运行时上执行，这个调试器才能连接到运行时上，完成在 VS Code 上调试的效果。而普遍的 JavaScript 运行环境，就是浏览器和 nodejs。对于 VS Code 的调试和启动的配置，是通过文件夹`.vscode`中的`launch.json`文件来设置的，虽然打开后有一些默认的配置项，但是如果默认配置项不能实现自己的目的的时候，还是建议自己新建文件去进行配置。

### 浏览器调试

虽然从功能和方便程度来说，使用 Node.js 作为你的 JS 运行时是最为方便的，不过我们先不安装 Node.js，使用浏览器这个自带的运行时来配置一个开发环境。在浏览器上运行 JS 代码非常的容易，只要打开 DevTools 的 Console 就可以编写和运行 JS 代码。而对于一个网页的 JS 资源，可以在文件上面打断点来进行调试。

对于 VS Code 的调试工具，只要告诉它你的浏览器和 url 地址，它就可以完成连接并使用浏览器进行调试。所以我们现在的问题就转变为，如何在浏览器中打开我们的网页。通过新建一个`html`文件并链接上自己本地的`.js`文件，就可以在浏览器中打开自己的网页；而在开发更常见的一种情况是，我们运行项目的集成框架，会打开一个服务使得我们的网页可以利用本地地址来访问。

接下来我将根据这两种情况并结合配置的参数，来展示一下如何配置和执行。

### 打开单个文件

在文件夹目录中新建你的`html`文件，然后在`script`元素中填写你的 JS 文件名。具体内容我们这里就省略了，直接跳到如何进行启动配置。

```json
{
  "name": "Launch index.html",
  "request": "launch",
  "type": "chrome",
  "file": "${workspaceFolder}/index.html"
},
```

在`.vscode`中的`launch.json`文件中按照如上配置，在选择该配置进行调试的时候，VS Code 会在你的 Chrome 浏览器中打开这个文件，而你的 JS 文件就能依靠浏览器这个环境来进行调试。但是有一个问题是，你对文件的修改，并不能触发浏览器的自动刷新，每次修改代码之后，你需要对网页进行刷新后才能看到修改后的效果。

在这个配置中，有一些属性需要进行一些讲解。属性`name`是你给调试配置起的名字，`request`决定了调试程序是如何运行的，这个属性可以选择的两个值分别为`launch`和`attach`，即“启动”和“附加”。这两个属性的选择，来自你的使用习惯。从调试时程序的表现上来说，选择“启动”的话，VS Code 会以调试模式替你打开你的 Chrome 浏览器，并在地址栏输入你的`html`文件地址；而“附加”需要你自己去浏览器打开这个文件。启动模式下，VS Code 会以调试模式打开你的程序进程后，并自动将调试器连接到你的程序进程的调试端口上；而附加模式需要你根据运行代码的程序，手动配置好调试端口。

附加模式配置：

```json
{
  "name": "Attach index.html",
  "request": "attach",
  "type": "chrome",
  "port": 9222
},
```

附加调试需要将你的执行环境（浏览器）以调试模式的方式启动。以 Chrome 浏览器为例，这需要你在浏览器的安装目录中（一般为`C:\Program Files\Google\Chrome\Application`）以调试模式打开新的浏览器实例，并配置一个端口用来和外部调试器连接，例如`chrome.exe --remote-debugging-port=9222 --user-data-dir=remote-debug-profile`^[2]^，这里建议你使用管理员模式打开命令行，以防止在浏览器进程文件夹中新建`remote-debug-profile`的配置目录失败。当新的浏览器实例启动后，就可以在 VS Code 中选择`Attach index.html`来附加到浏览器。此时你会发现，图下功能区域的最后一个图标从“停止”变成了“断开连接”，表示当前已经实现了调试器对浏览器的连接，即便你还没有打开目的文件或地址。之后在这个新的浏览器实例中打开你的静态文件，就可以像上面启动模式一样进行调试了。

{% asset_img debug-bar.jpg 调试栏 %}

`type`属性决定了你可以打开什么浏览器，这里除了填`chrome`以外，还可以填`msedge`来用 Edge 浏览器进行调试。如果你安装了其他浏览器，也可以通过修改参数来打开，以下是两种打开 Chrome 开发者版的方法，一个是配置运行时的执行参数来控制浏览器的版本，另一种是直接设置参数为具体的程序地址，只要浏览器的调试接口设计能兼容你在`type`属性中填的值，就可以实现调试功能。

```json
{
  "name": "Launch index.html (Chrome Dev)",
  "request": "launch",
  "type": "chrome",
  "runtimeExecutable": "dev",
  "file": "${workspaceFolder}/index.html"
},
{
  "name": "Launch index.html (exe)",
  "request": "launch",
  "type": "chrome",
  "runtimeExecutable": "C:\\Program Files\\Google\\Chrome Dev\\Application\\chrome.exe",
  "file": "${workspaceFolder}/index.html"
},
```

### 使用服务器运行网页

上面是通过打开静态页面的方式进行调试的方法，而我们更经常遇到的一种情况是启动一个 WEB 服务来访问。在 VS Code 上可以通过安装 Live Server 插件来实现。此时只要把先前配置的`file`属性改为`url`，并填上服务运行后本地可以访问的链接。

```JSON
{
  "name": "Launch Chrome Dev (Server)",
  "request": "launch",
  "type": "chrome",
  "runtimeExecutable": "dev",
  "url": "http://localhost:5500",
},
```

## 用 Node.js 调试

### 启动调试

首先需要先安装 Node.js^[3]^ 在你的电脑上，安装成功后，可以在命令行中输入`node`来检测安装是否成功。此时，即便你的目录中没有`launch.json`进行配置，也可以通过直接点击“运行和调试”来进行调试（记得重启你的 VS Code）。或者也可以选择下图中的 Node.js 选项，然后在弹出条中选择运行当前文件来实现调试。

{% asset_img debug-option.png 调试选项 %}

如果要写一个配置的话，在配置文件中进行如下配置：

```json
{
  "name": "Launch Program (Node.js)",
  "program": "${fileDirname}/index.js",
  "request": "launch",
  "skipFiles": [
      "<node_internals>/**"
  ],
  "type": "node"
},
```

这个例子有一个最大的问题，就是要根据你的文件名去修改`program`属性中的文件名。这是就要利用配置的预定义变量来优化配置，例如使用`${workspaceFolder}/${fileBasename}`就可以获取当前文件名和后缀来进行调试，具体的变量列表可以参考文档中的 Variables Reference^[4]^ 页面来实现你的需求。

### 附加调试

#### 使用自带的 JavaScript 调试终端

在新建配置的时候，有一个选项是“JavaScript 调试终端”，点击后，你可以在下方功能栏看到打开了一个`JavaScript Debug Terminal`在这个终端里，你只要运行你的 JS 代码，就可以实现调试的效果。例如，输入`node index.js`就可以调试终端目录下的`index.js`文件。这时我们会发现，调试条最后显示的是“断开连接”，这也就是表示了这个功能是使用附加模式来进行调试。

{% asset_img js-debug-terminal.png JS 调试终端 %}

使用 Node.js 附加的配置时，主要要配置的是 Node.js 程序的启动参数，`node --inspect-brk index.js`^[5]^命令会以调试模式启动程序，并等待调试器连接，此时只要在 VS Code 上选择“Attach (Node.js)”来附加到 Node.js 上即可。

```json
{
  "name": "Attach (Node.js)",
  "port": 9229,
  "request": "attach",
  "skipFiles": [
      "<node_internals>/**"
  ],
  "type": "node"
},
```

## 调试 TypeScript

> 如果你只是要运行 JS 代码，只要安装 Node.js 后，在 VS Code 的商店中使用 Code Runner 扩展即可方便的运行 JS 代码。

TypeScript 不像 JavaScript 可以直接在浏览器的终端中调试，甚至不能直接在 Node.js 环境中运行，需要利用 tsc 编译成 JS 文件，或者使用 ts-node 才行。利用 tsc 的`sourceMap`属性，可以编译出一个文件后缀为`.map`的中间文件，但是如果不配置的话，不会自动配置，需要你手动编译后才能进行调试。

为了在 VS Code 上调试和运行 TypeScript 代码，你需要先安装对应的编译器。`npm install -g typescript`然后使用`tsc -v`来验证你的安装。对于你的 TypeScript 代码，通过命令`tsc filename.ts`即可将其编译为一个 JS 文件，然后就可以按照 JavaScript 的方式进行调试。

这样编译文件会产生一个问题，你编译产生的 JS 文件会和`.ts`文件在同一个目录中，假设有同名文件的话会产生覆盖的问题，这里就要使用`tsc -init`来生成`tsconfig.json`文件对你的编译行为进行设置。我们只需要设置`"outDir": "debug"`并不携带任何参数的运行`tsc`命令，就可以把当前目录下所有的`.ts`文件进行编译，并将产生的`.js`文件按照相对目录输出到根目录下的`debug`文件夹中。

但是这样依旧是在 JS 文件中调试，并没有实现我们在 TypeScript 中调试的目标。我们需要使用`sourceMap`^[7]^属性，来实现在 TypeScript 源代码和在 Node.js 中运行的 JavaScript 代码间进行映射。只需要在`tsconfig.json`文件中设置`"sourceMap": true`，就可以在运行`tsc`命令编译时,生成一个后缀为`.map`的文件来帮助调试器进行调试。此时完全兼容先前的 Node.js 启动配置，使用选项“Launch Program (Node.js)”就可以在`.ts`文件中进行调试。

### 使用 VS Code 的任务功能

按照上面的操作，虽然可以实现调试的目标，但是使用起来相当的不便。不但需要运行编译的命令，一次还会生成多个文件，对于目前不需要重新编译的文件，需要进行多次重复操作。针对这些问题，我们可以使用`lauch.json`来自己设置一个配置，一步步的解决上面的问题。首先，我们要解决每次都要执行编译命令的问题。

虽然可以利用 VS Code 的构建功能`Ctrl+Shift+B`^[6]^来自动监视 TypeScript 文件和生成 JS 文件，但是如果能把这个行为集成到启动配置中会更加方便。为了实现这个目标，我们需要先了解一下 VS Code 的任务功能，为了方便对多个工具进行自动化。

我们可以为项目配置任务，并在启动脚本中进行使用，任务的配置在`.vscode`中的`tasks.json`文件中。你可以在使用构建功能的时候生成这个文件，使用`Ctrl+Shift+B`功能键，然后在弹出的选项条中点击小齿轮，选择对应的配置任务就可以自动生成和编辑这个配置文件。以 tsc 构建任务为例，我们来看一看任务的配置文件是怎么写的。

```json
"tasks": [
  {
    "type": "typescript",
    "tsconfig": "tsconfig.json",
    "problemMatcher": [
      "$tsc"
    ],
    "group": "build",
    "label": "tsc: 构建 - tsconfig.json"
  }
]
```

task 数组是你的任务配置列表，目前配置的内容是，一个名（标签）为`tsc: 构建 - tsconfig.json`的任务，通过配置和自动检测，它能够将你的 TypeScript 代码根据文件目录中的`tsconfig.json`配置自动编译为 JS 文件。现在我们可以通过组合按键`Ctrl+Shift+B`触发任务，但是按照需要，这个任务如果可以自动执行或者在我们调试前先执行就更好了。在`launch.json`中有个属性是`preLaunchTask`^[8]^，可以配置在调试开始前要执行的任务，需要的参数是你这个任务的`label`属性，就可以在调试开始前启动这个任务。

```json
{
  "name": "Launch TypeScript Program After Build Task",
  "program": "${fileDirname}/${fileBasenameNoExtension}.ts",
  "request": "launch",
  "skipFiles": [
    "<node_internals>/**"
  ],
  "type": "node",
  "preLaunchTask": "tsc: 构建 - tsconfig.json",
},
```

接下来就是实现从编译本目录下全部的 TypeScript 文件到只编译单个文件的操作。而这里就需要转到 task 中进行配置和修改。

tsc 编译单个文件的时，可以手动将所有 tsconfig 中的参数写到命令行，例如`tsc index.ts --sourceMap true --outDir debug`。所以利用这个方式，我们可以使用任务`args`属性来操作。要注意的是，默认的 TypeScript 配置并不能添加属性，所以需要切换为使用 shell 运行命令的方法。现在，你就可以调试单个 TypeScript 文件了。

```json
{
  "type": "shell",
  "problemMatcher": [
    "$tsc"
  ],
  "command": "tsc",
  "args": [
    "${file}",
    "--sourceMap",
    "true",
    "--outDir",
    "${workspaceFolder}\\debug\\${relativeFileDirname}\\"
  ],
  "group": "build",
  "label": "tsc: 使用 args 构建单个文件 - tsconfig.json"
},
```

## 备注

### 运行时

在看文档的时候，经常会注意到文档中会提到一个叫做 runtime 的词，比如：

> "VS Code has built-in debugging support for the Node.js runtime and can debug JavaScript, TypeScript, or any other language that gets transpiled to JavaScript."
> "For debugging other languages and runtimes (including PHP, Ruby, Go, C#, Python, C++, PowerShell and many others), look for Debuggers extensions in the VS Code Marketplace or select Install Additional Debuggers in the top-level Run menu."^[9]^

> Node.js 运行时是负责安装 Web 服务代码及其依赖项并运行服务的软件栈^[10]^。

runtime 这个词一般被翻译成运行时。这有点像 run-time，但是实际上有一些区别。在之前谈 [Event loops](storh.github.io/2022/08/21/js-async-01/) 的文章里，我简单提到了一下这个概念，但是我感觉还是有必要再深入说说我的理解。

我个人的理解是，runtime 是一组内容的概括，属于通过抽象得到的一个概念。它作为操作系统和代码之间的一个抽象层，代码执行时需要的东西，执行造成的影响都可以归到这个概念中去。为了防止混淆，我这里特地使用了“执行”这个词，而不使用“运行”。以代码执行开始和结束的时间为界限，期间涉及到的所以东西都可以算作运行时这一个概念所包括的内容。举例来说，执行代码的 JavaScript 引擎，无论是浏览器还是 Node.js 中对事件循环的具体实现，浏览器或 Nodo.js 所提供的 api；这些执行代码、去调用系统资源代码、控制事件的这些软件的集合，都是运行时包含的东西。

在执行 JavaScript 代码的时候，JavaScript 运行时实际上维护了一组用于执行 JavaScript 代码的代理。每个代理由一组执行上下文的集合、执行上下文栈、主线程、一组可能创建用于执行 worker 的额外的线程集合、一个任务队列以及一个微任务队列构成。除了主线程（某些浏览器在多个代理之间共享的主线程）之外，其他组成部分对该代理都是唯一的^[11]^。

### TypeScript 和 JavaScript 的特殊关系

> TypeScript is JavaScript’s runtime with a compile-time type checker.
> TypeScript 是带有编译时类型检查器的 JavaScript 运行时^[12]^。

TypeScript 能够运行在任何 JavaScript 可以运行的地方，是由于它们之间具有的特殊的关系。正如上面引用内容所说的，TypeScript 是一个 JavaScript 运行时，编译的过程可以理解为在类型检查通过后，剔除和类型相关的内容后，生成 JS 代码。基于这一特点，使用 TypeScript 需要安装 tsc 实现对代码的编译，而之后生成的 JS 代码就可以放到无论是浏览器还是 Node.js 中执行了。

这句话也正好可以用来理解上面的关于运行时的概念。一般从直觉上来看，虽然 TypeScript 是 JavaScript 的一个超集，但是我们还是会把它们看作两个程序语言。但实际情况是，TypeScript 文件中的代码先由类型检查器的处理编译生成了 JS 代码，编译好的代码最终是在 Node.js 或者浏览器这样的 JavaScript 运行时中执行，TypeScript 这一单词的含义，就变成了将`.ts`文件中的代码转换为 JS 代码后，由 JavaScript 运行时来执行代码这一个组合概念。

### 关于 tsc 的更多

tsc 是 TypeScript 的编译器，基本的使用方法是通过命令`tsc file.ts`来把你的 TypeScript 代码 编译为 JS 代码^[13]^。当然，除了编译成 JavaScript，TypeScript 也是可以执行的（取决于对执行这一概念的理解）。Deno 就通过将 TypeScript 代码通过自己设计的转换工具处理后进行缓存，而不是把源代码转换为 JavaScript，利用这个缓存实现了运行 TypeScript 这一行为^[14]^。

## 参考链接

1. [Debugging](https://code.visualstudio.com/docs/editor/debugging)

2. [Browser debugging in VS Code](https://code.visualstudio.com/docs/nodejs/browser-debugging)

3. [Node.js](https://nodejs.org/en)

4. [Variables Reference](https://code.visualstudio.com/docs/editor/variables-reference)

5. [Node.js debugging in VS Code](code.visualstudio.com/docs/nodejs/nodejs-debugging)

6. [Integrate with External Tools via Tasks](https://code.visualstudio.com/Docs/editor/tasks)

7. [TypeScript tutorial in Visual Studio Code - Debugging](https://code.visualstudio.com/docs/typescript/typescript-tutorial#_debugging)

8. [Debugging TypeScript](https://code.visualstudio.com/docs/typescript/typescript-debugging)

9. [Debugging - Debugger extensions](code.visualstudio.com/docs/editor/debugging#_debugger-extensions)

10. [Node.js 运行时环境](https://cloud.google.com/appengine/docs/standard/nodejs/runtime?hl=zh-cn)

11. [深入：微任务与 Javascript 运行时环境](developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth)

12. [TypeScript for the New Programmer - Learning JavaScript and TypeScript](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html#learning-javascript-and-typescript)

13. [TypeScript Tooling in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-tooling-in-5-minutes.html)

14. [Overview of TypeScript in Deno](deno.com/manual@v1.28.0/advanced/typescript/overview)
