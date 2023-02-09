# 初识 Node.js

[toc]

## 一、什么是 node.js 

> Node.js® is a JavaScript runtime built on [Chrome's V8 JavaScript engine](https://v8.dev/).

### 定义

1. 不是一门语言，而是一个运行环境。和浏览器一样，只不过浏览器运行在客户端而它运行在服务端
2. 使用的语言是 `JavaScript` （不包含 BOM、DOM API，增加了 Stream、网络等 API）
3. Node.js 依靠 Chrome V8 引擎运行 JavaScript

对应到 Java 可以把 Node.js 看作 JDK，装上之后就能在服务端运行 JS 代码了。

Chrome 和 Node.js 都是 JS 都运行环境，都使用了 V8 引擎，区别在于 V8 只实现了 ECMAScript 的数据类型、对象和方法，Chrome 运行时提供了 Window、DOM、BOM，而 Node.js 运行时提供了 global、Buffer、net 等模块

### 核心特征

#### 1、事件驱动

node 环境中的 Event Loop 与浏览器中的大致相同。不同的是 node 中有一套自己的模型。node 中 Event Loop 依靠的是 Libuv 引擎，以 Chrome V8 引擎作为解释器，V8 将 JS 代码分析后去调用对应的 node api，而这些 api 最后由 libuv 引擎驱动，执行对应的任务。

```tex
libuv: 高性能的事件驱动的异步 I/O 库，本身由 C 语言编写。封装了不同平台底层对于异步I/O模型的实现。最初专为 Node.js 而设计。
```

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210616170129714.png" alt="image-20210616170129714" style="zoom:40%;" />

node 的 Event Loop 如上图。从上面这个模型中，我们可以大致分析出node中的事件循环的顺序：

外部输入数据-->轮询阶段(poll)-->检查阶段(check)-->关闭事件回调阶段(close callback)-->定时器检测阶段(timer)-->I/O事件回调阶段(I/O callbacks)-->闲置阶段(idle, prepare)-->轮询阶段

- timers: 这个阶段执行定时器队列中的回调如 `setTimeout()` 和 `setInterval()`。
- I/O callbacks: 这个阶段执行几乎所有的回调。但是不包括close事件，定时器和`setImmediate()`的回调。
- idle, prepare: 这个阶段仅在内部使用，可以不必理会。
- poll: 等待新的I/O事件，node在一些特殊情况下会阻塞在这里。
- **check**: `setImmediate()`的回调会在这个阶段执行。
- close callbacks: 例如`socket.on('close', ...)`这种close事件的回调。

#### 2、非阻塞 I/O

基本 Event Loop 将异步事件放入队列，执行完成之后再放回主线程中，不阻塞主线程的执行。

### 

### 为什么 node.js 是高性能的？

1. CPU 运算远快于 I/O 操作
2. Web 是典型的 I/O密集场景
3. JavaScript 是单线程的，但是 JavaScript 的 runtime Node.js 并不是。负责 Event Loop 的 libuv 由 C 和 C++ 编写。因此一样具有多线程的性能。

其他依赖多线程解决高并发问题的方式多是一个线程处理一条用户请求，处理完成后释放线程。在阻塞 I/O 模型下，I/O 期间会产生多个用户线程占用 CPU 资源。打个比方：类似于饭店一个厨师配备一个服务员，服务员把菜单给厨师后就什么都不做在等着，造成资源浪费。这是完全没必要的。在高并发、I/O 密集场景下，没必要一个厨师配备一个服务员，整个饭店一个服务员就差不多了。其他的资源可以用来做其他事情，效率更高。

**所以 node.js 快就快在：**

1. 非阻塞 I/O
2. Web 场景 I/O 密集
3. 没多线程 Context 切换开销，多出来的开销是维护 EventLoop

在其他场景下，node.js 性能其实不高。Node.js 在 I/O 密集的 Web 场景相对于使用多进程模型语言有性能优势，这个优势不是来源于语言，而是操作系统实现，Java 按照这种模型实现性能一样很高

```tex
得益于 V8 的优化和 C/C++ 拓展，Node.js 执行 CPU 密集任务性能并不差，但如果长时间进行 CPU 运算会阻塞后续 I/O 任务发起，用 Java 实现非阻塞模型也会遇到一样问题。
```

所以**快是相对的**，是在特定场景下的。

## 二、模块系统

### 1. ES6 的模块语法

export：导出

import：引入

### 2. ES6 之前

类似于 commonJS

module.exports：导出

require：引入

`require.resolve()`: 返回引入的文件名，但不真正加载模块

### 3. 着重说一下 require

一个项目中，可能包含多个 node_modules 文件夹。第三方模块的查找遵循就近原则，逐层向上追溯，直到根据 NODE_PATH 环境变量到达根目录。

此外 node.js 还会搜索以下的全局目录

- $HOME/.node_modules
-  $HOME/.node_libraries
-  $PREFIX/lib/node

其中 `$HOME` 是用户的主目录， `$PREFIX` 是 Node.js 里配置的 `node_prefix`。强烈建议将所有的依赖放在本地的 node_modules 目录，这样将会更快地加载，且更可靠。

#### 缓存

Node中引入模块，需要经历3个步骤：（1）路径分析，（2）文件定位，（3）编译执行

为了减少二次引用时的开销，Node对引入过的模块都会进行缓存，缓存的是编译和执行后的`对象`。

> 不论是核心模块还是文件模块，`require()`方法对相同模块的二次加载都一律采用缓存优先的方式，这是**第一优先级**的。

那么“require是根据什么来判断是否从文件加载还是从缓存加载？”

> 每一个编译成功的模块都会将其文件路径作为索引缓存在`Module._cache`对象上，以提高二次引入的性能。

所以如果是引用同一路径的模块，比如`const Vue = require('vue')`，那么后续引入的都是缓存中的同一对象！！！

### 4. 单次加载 & 循环依赖

模块在加载一次后会被缓存到 `Module.cache`，多次加载不会导致模块重复执行。node.js 根据实际的文件名缓存模块。

#### 循环依赖

当多个模块循环依赖时，会返回依赖的**未完成副本**给引入的模块，如下所示：

当 main.js 加载 a.js 时，a.js 又加载 b.js,此时，b.js 会尝试去加载 a.js。为了防止无限的循环会返回一个 a.js 的 exports 对象的 **未完成的副本** 给 b.js 模块，然后 b.js 完成加载，并将 exports 对象提供给 a.js 模块

> 未完成副本：即执行到这一句之前的所有挂载了的 exports 对象会被作为副本返回。

```js
// a.js
console.log('a 开始');
exports.done = false;
const b = require('./b.js');
console.log('在 a 中，b.done = %j', b.done);
exports.done = true;
console.log('a 结束');

// b.js
console.log('b 开始');
exports.done = false;
const a = require('./a.js');
console.log('在 b 中，a.done = %j', a.done);
exports.done = true;
console.log('b 结束');

// main.js
console.log('main 开始');
const a = require('./a.js');
const b = require('./b.js');
console.log('在 main 中，a.done=%j，b.done=%j', a.done, b.done);

// output
/*
main 开始
a 开始
b 开始
在 b 中，a.done = false
b 结束
在 a 中，b.done = true
a 结束
在 main 中，a.done=true，b.done=true
*/
```



### 5. ES6 模块和 CJS 模块的区别

> ES6模块是编译的时候绑定的作用域，运行时加载改变作用域。编译时会把变量挂载到这个模块的作用域上，已经 excute 过的编译时不再执行。如果引入一个模块中的含有块级作用域的变量，如果这个模块已经 excute 过，那么引入的地方就会报错。

#### JS 引擎解析一个模块时

1. 解析:读取模块的源代码,并检查语法错误。
2. 加载:加载所有的导入模块(递归进行),这是还未标准化的部分。
3. 链接:对于毎个新加载的模块,在实现上都会创建一个作用域,并把模块中声明的所有变量都绑定在这个作用域上,包括从其他模块导入的变量。如果你想试试 `import { cake } from "paleo"`,但是 paleo 模块没真正导出名为cake的变量,你会得到一个错误。
4. 运行时间:最后,开始执行加载进来的新的模块中的代码。这时,整个 import过程已经完成
   了,所以前面说代码执行到 import这一行声明时,什么都没有发生。

### 5. 工作原理

node.js 中每个文件都是一个模块，模块内的变量都是局部变量。在执行模块代码之前，node.js 会使用一个如下的函数封装器封装模块

```js
const exports = module.exports; // 所以在导出的时候可以 exports.test 直接赋值导出。但是不能 exports = { } 赋值，会更改 exports 的指向
(function(exports, require, module, __filename, __dirname) {
	// 模块的代码实际上在这里
});
```

- __filename：当前模块文件的绝对路径
-  __dirname：当前模块文件据所在目录的绝对路径
- module：当前的模块实例
- require：加载其它模块的方法，module.require 的快捷方式
- exports：导出模块接口的对象，module.exports 的快捷方式

### 6. 原生支持

v12 之前配置 babel，开启 esmodules

在 v12 后可以使用原生方式支持 ES Module

1. 开启 `--experimental-modules` 
2. 模块名修改为 `.mjs` （强烈不推荐使用）或者 package.json 中设置 `"type": module`

## 三、调试

```shell
node --inspect-brk [path]
```

> --inspect-brk 会让用户代码第一行执行前停住，防止没来及 debug 代码就执行结束了，Web 服务脚本会一直在后台运行。否则使用 --inspect 即可

## 四、npm scripts

即在 npm 的 package.json 文件中声明的脚本命令。可以通过`npm run env`查看 npm 配置的环境变量。

### 工作原理

在执行`npm run command`时，会在当前路径下的 package.json 中搜索 command

搜索到后新建一个 shell，在 shell 内执行 command 的内容。windows 下默认使用 cmd.exe

可以使用 --key=value 的形式传递参数

**npm scripts 执行前会把当前目录的 `node_modules/.bin` 目录加入到环境变量 PATH 中，执行完成后恢复，这样该目录下的所有脚本都可以不写路径直接使用**

### pre、post 钩子

当使用命令 `npm run xxx` 时，npm 会尝试执行 package.json `scripts` 中配置的 xxx 脚本命令，但 npm 同样会尝试在 package.json `scripts` 中查找是否配置了 prexxx，postxxx 脚本命令。如果都配置了，npm 会按照以下顺序执行脚本

- npm run prexxx
- npm run xxx
- npm run postxxxvvv
