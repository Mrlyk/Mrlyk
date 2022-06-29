# webpack 热更新详解

> webpack 的热更新在实际开发中很常用，了解它有助于我们排查开发过程中的问题。本文从实践深入到源码一起慢看它是怎么实现的。

热更新的作用大家都知道，就是不刷新页面而去更新页面中的资源代码，让我们能更高效的开发。在探究原理之前，我们还是先看看热更新怎么用，从使用去探究原理。

[toc]

## 热更新实践

官方文档：https://www.webpackjs.com/guides/hot-module-replacement/

#### 启用热更新

在本地开发过程中，默认 webpack-dev-server 在 development 模式下会自动开启热更新。我们也可以手动开启，一般两种方式（webpack-dev-server > 4 版本）

- 通过在启动的 cli 命令中携带`--hot`参数，开启该选项后 webpack-dev-server 会自动加载`HotModuleReplacementPlugin`插件（如果未配置的话）
- 在 webpack.config.js 的 `devServer`选项中，配置`hot: true`选项，开启后也不需要手动配置`HotModuleReplacementPlugin`插件

启用了热更新之后，我们就可以通过 webpack 提供的 api 来实现模块热替换。

#### 实现热更新

API 官方文档：https://www.webpackjs.com/api/hot-module-replacement/

当我们启用热更新之后，webpack 会将热更新的 API 暴露在`module.hot`这个对象下面。所以我们首先要检查这个对象是否可访问来判断是否开启了热更新

```js
import { sum } from './src/sum-output.js'

function setButton () {
  const btn = document.createElement('button')
  btn.innerText = 'getSum'
  btn.onclick = sum;
  return btn
}
document.body.appendChild(setButton())

if (module.hot) { // 是否开启热更新
  module.hot.accept('./src/sum-output.js', function () {
    console.log('sum-output 热更新')
  })
}
```

上面看到我们使用了一个`module.hot.accept`方法，它是干什么的呢？我们首先对一些重要 API 方法做个说明

- `accpet(dependencies, callback)`: accpet 方法接收指定的 dependencies 的更新，并触发 callback 来响应。其中 dependencies 可以是一个字符串，也可以是一个字符串数组。**参数可以都为空，仅用来表明该模块可以热替换。该模块更新时就不会触发页面刷新** 
- `deline(dependencies)`：用来拒绝指定 dependencies 的更新，当指定的 dependencies 更新时不会热更新，而是强制刷新页面。用来处理一些需要刷新整个页面的场景。比如模块整体是支持热更新的，但是有个依赖涉及到 windows 对象，它热更新时就必须刷新整个页面，否则会产生问题，就可以用到这个 api。**参数也可为空** 
- `dispose(callback) || addDisposeHandle(callback)`：当前模块热更新时触发，可以用来移除一些定时器操作或者传递数据。**传递的数据可以通过 `module.hot.data`获取到（在当前模块）**
- `removeDisposeHandler(callback)`：和上面的方法对应，移除回调
- `status()`:返回当前热模块进程替换的状态，常用的有：check - 检查以更新、prepare - 准备更新、ready - 准备更新且可用、dispose - 正在调用上面说的 dispose 回调、apply - 正在调用`accept`方法处理、abort - 更新中止，还未更新、fail - 更新已经抛出异常
- `addStatusHandle(callback)`：注册一个 callback 来监听 status 的变化，当然也有对应的`removeStatusHandler`
- `check`：相当于手动检查模块是否存在更新，看完后面的原理流程后会更清楚
- `apply`：应用模块更新，仅能在`status`为`ready`状态时使用

了解这些 api 能让我们在后面分析原理时更好的了解是如何实现的。

#### 要注意的问题

本章开头的例子就是一个开启`sum-output`模块热替换的简单例子，**但是存在一个问题：btn 已经生成了，他的点击事件会自动更新吗？答案是不会**。示例代码中 btn 生成后页面不刷新它绑定的点击事件就还是原来的那一个。所以在实现热更新的时候我们要注意这种情况，**手动在`accept`回调中处理这种情况。**  

#### 服务端的热更新

webpack-dev-server 本身是一个 server，依赖于 express 框架实现了一个本地 node 服务。所以在服务端我们也可以使用 webpack 提供的热更新能力。

```js
// 官方示例
const webpackDevServer = require('webpack-dev-server');
const webpack = require('webpack');

const config = require('./webpack.config.js');
const options = {
  contentBase: './dist',
  hot: true,
  host: 'localhost'
};

webpackDevServer.addDevServerEntrypoints(config, options); // 主要是开启热更新和确认替换的目录
const compiler = webpack(config);
const server = new webpackDevServer(compiler, options); // 使用 webpack server
server.listen(5000, 'localhost', () => {
  console.log('dev server listening on port 5000');
});
```

*webpack 还提供 webpack-dev-middleware 中间件来在服务端开启 HMR* 

总的来说热更新的实践还是比较简单易懂的，实际使用中像 vue loader，style-loader 等各类 loader 会在后台使用`module.hot.accept`等方法来帮我们处理热更新。

接下来就来看看在 webpack-dev-server 中，热更新到第是如何实现的

## 热更新原理

**先简单说明一下原理**：webpack-dev-server 通过启动一个 node 服务来观察静态文件的变化，同时在编译完成的钩子上注册一个事件，文件变化或每次编译完成时通知客户端（客户端与 devServer 服务在 webpack 打包时注入）。客户端收到通知后触发更新事件`webpackHotUpdate`，通过前面注入的 devServer 检查模块更新，之后通过注入的客户端发起 http 请求拉取更新的清单文件和实际文件。最终将更新的模块重新引入，完成更新。

接下里我们从 webpack-dev-server 的启动开始，一步步探究这个热更新是如何实现的。下面是整体流程图  ![webpack-hmr-e](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/webpack-hmr-e.png) 

我们就根据这张流程图一步步做个说明

#### wepack serve 命令

源码：@webpack-cli/serve/lib/index.js

这个是我们通过 shell 启动 webpack-dev-server 的命令，webpack 本身并没有实现这个命令，所以需要通过 webpack-cli 来处理构建命令并执行。

1. 执行`cli.makeCommand`方法构建 serve 命令
2. 在`cli.makeCommand`的回调中，创建 webpack-dev-server 实例，同时启动该实例

![image-20220214154310845](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220214154310845.png?x-oss-process=image/resize,w_500,m_lfit) 

![image-20220214161730405](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220214161730405.png?x-oss-process=image/resize,w_500,m_lfit) 

在上一步启动该实例后，则会进入到 webpack-dev-server 包中进行下一个阶段的处理

#### webpack-dev-server

源码：webpack-dev-server/lib/Server.js

在这一步主要是服务端的处理，我们上面说过服务端要通知客户端文件更新，客户端要向服务端请求更新清单和文件。这一步就要构建这样一个服务。

1. 使用原生的 node net 模块创建进程间的 IPC 通信，这一步主要是建立客户端和服务端的 socket 通信

2. **initialize 一系列的初始化操作**，说其中三个重点的

   - 通过`addAdditionalEntries` 向我们的打包产物中注入了 webpack-dev-server/clinet/index.js（前提是开启了 hot 配置）。其他操作可以看上面的流程图

     ![image-20220215173813312](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220215173813312.png?x-oss-process=image/resize,w_800,m_lfit)   

   - **`setUphooks` ：像插件一样，在`compiler.hooks.done`上注册了一个事件，每次编译完成都会通知客户端**

   - `new Express()`创建 node 服务，其中放入了`webpack-dev-middleware`中间件，用来生成`hot-update.json`清单和`hot-update.js`。也就是每次热更新时客户端发送请求，请求的两个文件，这里面的逻辑就不赘述了

服务端创建完成后就可以通过 ws 来通信了，下面是一个通信列表

![image-20220215174133244](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220215174133244.png?x-oss-process=image/resize,w_800,m_lfit) 

**其中要注意这里发送了一个 hash 值，客户端就是通过比对 hash 来判断文件是否有更新的** 

客户端接收本次通信的信息来触发后续操作

#### webpack-dev-server/client

在客户端中处理了服务端发来的各类信息，对于异常信息我们就不说了，只说明正常的流程。

这一步接收到`type: ok`信息之后，如果存在之前的异常信息弹出层（overlay）会先将其关掉，**然后通过`hotEmitter.emit("webpackHotUpdate", status.currentHash);`方法来发送事件**。这里的事件完全使用 node 的 events 类。

![image-20220215183510671](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220215183510671.png?x-oss-process=image/resize,w_600,m_lfit) 

clinet 主要的作用就是用来和 server 建立 ws 通信，最终还是通过 webpack 编译时注入`dev-server.js`的和 HotModuleReplacementPlugin 来实现热更新。

所以在这里发送事件后，就被`webpack/hot/dev-server.js`接收到了

#### webpack/hot

热更新的实际执行逻辑还是包含在了 webpack 中，webpack-dev-server 则用来观测和通知变化。这里面的逻辑就稍微复杂一些。

当`dev-server.js `接收到`webpackHotUpdate`事件后，热更新的真正逻辑开始了

![image-20220215184536160](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220215184536160.png?x-oss-process=image/resize,w_600,m_lfit) 

1. 调用`module.hot.check`方法检查更新，该方法在执行时由`HotModuleReplacement.runtime.js `的`createModuleHotObject`方法创建
2. 在`check`方法中首先通过 http 请求向服务端要更新的文件清单`hot-update.json`。拿到清单后又去请求`hot-update.js`-更新后的文件。其中要注意请求方法是在 webpack 编译时创建的，编译过程中全局替换`$xxx$`这样的标志而成。
3. 获取到更新的 js 文件后，**通过 jsonp 的方式注入我们的项目** 
4. 最后通过`JavascriptHotModuleReplacement.runtime.js`的`apply`方法执行最后的更新操作

第三步的操作是经由`LoadScriptRuntimeModule.js`模块完成的，他主要用来在 webpack 运行过程中通过字符串实时构建函数。

第四步中还会对过时的 module、chunk、依赖进行处理，同时执行`module.hot.accpet`方法。这里详细说明下 apply 方法（代码是简化后的）

```js
apply: function (reportError) {
  // 插入新的代码，appliedUpdate 中就是我们 http 请求下来的更新的模块 {'./src/test.js': xxxxx}
  // $moduleFactories$ 编译后对应 __webpack_require__.m，其中存储了当前项目所有的模块，只要完成对他的更新，我们的新代码也就替换完成了
  for (var updateModuleId in appliedUpdate) {
    if ($hasOwnProperty$(appliedUpdate, updateModuleId)) {
      $moduleFactories$[updateModuleId] = appliedUpdate[updateModuleId];
    }
  }

  // currentUpdateRuntime 由 webpack 编译时生成，也是 http 请求下来的 hot-update.js 中的方法（见下方图）
  // 这里主要存储本次更新的文件的 hash 值，用于下次请求
  for (var i = 0; i < currentUpdateRuntime.length; i++) {
    currentUpdateRuntime[i](__webpack_require__);
  }

  // 这里面主要是处理依赖中的 module.hot.accept 方法
  for (var outdatedModuleId in outdatedDependencies) {
    // ...
  }

  // 加载自执行的模块
  for (var o = 0; o < outdatedSelfAcceptedModules.length; o++) {
    var item = outdatedSelfAcceptedModules[o];
    var moduleId = item.module;
    item.require(moduleId);
  }

  return outdatedModules;
}
```

**currentUpdateRuntime：** 

![image-20220215194513081](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220215194513081.png?x-oss-process=image/resize,w_800,m_lfit)  

以上就是热更新的整个过程，其中还有很多细节，可以通过源码去了解，这边主要阐释其主要过程。

## 其他

#### 如何将文件写入内存

基于 [memory-fs](https://www.npmjs.com/package/memory-fs) 工具将 js 格式的数据写入内存（保持数据不会被垃圾回收即一直存在于内存了...）

#### 如何观察静态文件变化

基于 [Chokidar](https://www.npmjs.com/package/chokidar)工具

## 参考文章

官方文档-模块热替换：https://www.webpackjs.com/guides/hot-module-replacement/

webpack 热更新原理探究：https://juejin.cn/post/7006260787031310372

轻松理解webpack热更新原理：https://juejin.cn/post/6844904008432222215