# webpack 编译流程

> webpack 作为现在前端开发不可或缺的工具之一，其功能之庞大，原理之复杂已经很难学习，但是又不得不掌握。本篇文章通过自己实现一个简易的 webpack 来了解 webpack 的整体编译流程

学习整体编译流程之前，需要对 webpack 的使用比较熟悉了，最好先学习一下 loader 和 webpack plugin 的编写，对 webpack 的实现核心工具 tapable 至少要懂如何使用。不然很多东西看的云里雾里...

在开始之前先理清几个概念：

- module：webpack 处理资源的最小单位（不一定是文件）
- chunk：module 及其所有依赖组成的一个大对象
- **compiler**：负责整体编译流程，一次编译流程中只存在一个。包含了 webpack 的所有环境配置，如 entry、loader、plugin 等。webpack 启动时会将其实例化，也是 plugin 的 apply 方法中的对象，提供了多个 hook
- **compilation**：负责真正构建过程的对象，每次进行构建时都会创建一个。包含了当前构建流程的 module 信息、编译生成的资源(assets)、变化的文件以及跟踪依赖的状态信息，也提供了很多 hook。在 devServer 中每次热更新时都会创建一个。

**webpack 在编译过程中的两个核心对象就是 compiler 和 compilation ，他们都继承自 tapable！** 

[toc]

## 整体流程

首先我们来过一下 webpack 编译的整体流程

1. 初始化参数：从配置文件和 cli 命令中读取合并参数（命令行优先级更高）
2. 开始编译：通过上一步得到的参数初始化生成 Compiler 对象，**加载所有配置的插件，执行对象的`compiler.run`方法开始编译**
3. 模块递归编译(buildModule)、依赖收集：从入口文件开始递归的找到的每一个 module 使用 loader 进行预处理(loader-runner)，并且记录 module 之间的依赖关系，生成 dependency graph
4. 完成编译：封装(seal) chunk。依据 dependency graph ，对 module 进行 seal，生成一个个对 chunk 对象
5. 输出文件：将上一步得到的 chunk 对象转换成一个单独的文件，**实现 webpack 自己的 require 方法**。然后将文件加入到输出列表（emitAssets）。这也是修改 module 内容的最后机会。最终根据 output 配置，最后写入文件系统

大概的流程就是如上，接下来我们深挖细节，一点点的看看 webpack 如何实现。

*实现查看自己的 webpack-yk 项目* 

## 初始化参数

一般我们使用 webpack 进行打包都是通过命令行启动的，比如

```shell
webpack --mode development
```

这其中就传递了一个 mode 参数。

而另一种就是**通过 webpack.config.js 传递参数，**我们都知道 webpack.config.js 实际上是导出了一个对象

```js
// webpack.config.js
module.export = {
  mode: 'development'
}
```

初始化参数时，我们需要对两个地方的参数进行合并。

**webpack 使用第三方的 [yargs](https://www.npmjs.com/package/yargs) 工具对参数进行合并**。我们则按照 webpack 的编译步骤，自己从头开始实现。

首先启动编译肯定是在 node 环境中，是用 webpack 命令启动的，所以我们就需要导出一个 webpack 方法，他接收 shell 的参数并且合并了 webpack.config.js 中的参数，如下

```js
/* webpack-yk/core/webpack.js */
function webpack(options) {
  // 合并参数
  const mergeOptions = _mergeOptions(options);
}

function _mergeOptions(options) {
  // 命令运行在 node 环境中，可以通过 process.argv 获取 shell 参数
  // 第一个参数是 node 路径，第二个是 webpack 的脚本路径，第三个开始才是我们要的参数
  const shellOptions = process.argv.slice(2).reduce((option, argv) => {
    const [key, value] = argv.split("="); // 参数 --xxx=xxx 转换到 option 对象上
    if (key && value) {
      const parseKey = key.slice(2); // 去除参数前的 --
      option[parseKey] = value;
    }
    return option;
  }, {});
  return { ...options, ...shellOptions }; // shell 参数优先
}

module.exports = webpack;
```

如上就完成了第一步，初始化参数合并，我们只需要在入口文件中调用该方法即可

```js
/* webpack-yk/core/index.js */
const webpack = require('./webpack')
const config = require('../example/webpack.config')

// 一：初始化参数，从 shell 和 webpack.config.js 合成（当然这里还没有返回值，只是初始化了参数）
const compiler = webpack(config)
```

## 开始编译

参数初始化完成后，就要进入下一步，开始编译。在这个阶段我们

1. 据初始化的参数，实例化一个 compiler 对象，并且调用`compiler.run()`方法正式开始编译
2. 生成 compiler 对象后，加载所有 webpack plugin 插件
3. 在`run`方法中根据我们的配置文件查找打包的 entry

所以我们需要实例化一个 compiler 对象，它具有`run`方法，并且进行后面的一系列操作

首先我们声明一个 Compiler 类，它存储初始化的参数方便后面调用，同时具有一个` run`方法

```js
/* webpack-yk/core/compiler.js */
class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
  }

  // 开始编译
  run(callback) {
  }
}

module.exports = Compiler;
```

接着在 webpack.js 中实例化它，将初始化的参数传入

```js
/* webpack-yk/core/webpack.js */
const Compiler = require('./compiler')

function webpack(options) {
  // 合并参数
  const mergeOptions = _mergeOptions(options);
  // 实例化 compiler 对象
  const compiler = new Compiler(mergeOptions);
  return compiler;
}
```

到这里开始编译的结构就搭好了，但是还没有`run`方法的具体实现。

#### 初始化钩子

在开头我们提过 webpack 本身是一个类发布订阅模式，**通过实例化多个钩子并在合适的时机调用他们，以影响编译过程**。而钩子又基于 tapable 实现（tapable 就不再展开说了）。所以我们**也根据我们需要影响的操作**初始化几个钩子（为了方便理解这里只使用最简单的同步钩子）

- 开始编译时的钩子，可以影响开始编译时的操作
- 编译完成输出文件到 output 目录钩子
- 全部编译完成之后的操作

```js
/* webpack-yk/core/compiler.js */
const { SyncHook } = require('tapable')
class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // 创建几个必要的钩子，简单点我们就使用同步钩子
    this.hooks = {
      // 开始编译时的钩子
      run: new SyncHook(),
      // 输出文件时的钩子
      emit: new SyncHook(),
      // 全部编译完成时的钩子
      done: new SyncHook()
    }
  }
  // 开始编译
  run(callback) {
  }
}
module.exports = Compiler;
```

以上只是简单的几个钩子，也只能影响部分构建过程。webpack 中的钩子实际多的多，类型也和例子中的简单的钩子不同，具体可以查看源码`webpack/lib/Compiler.js`。

**有了钩子之后，我们就可以在 plugin 中注册(`hooks.run.tap`)事件，以供我们需要的时候调用(`hooks.run.call`)。** 

#### 加载插件

钩子初始化完成后就可以来加载插件了，我们知道 plugin 是一个个的类实例对象，一定含有一个`apply`方法并且接受`compiler`作为参数。所以我们需要拿到所有的插件，**并且将`compiler`对象传入进去。这也是插件的原理**，compiler 对象贯穿编译始终。

**plugin 是在 webpack.config.js 中声明的，所以在合并后的参数里就能拿到所有的 plugins**

```js
/* webpack-yk/core/webpack.js */
const Compiler = require("./compiler");

function webpack(options) {
  // 合并参数
  const mergeOptions = _mergeOptions(options);
  // 实例化 compiler 对象
  const compiler = new Compiler(mergeOptions);
  // 加载插件
  _loaderPlugin(mergeOptions.plugins, compiler);
  return compiler;
}
/// ...
// 加载插件
function _loaderPlugin(plugins, compiler) {
  if (plugins && Array.isArray(plugins)) {
    plugins.forEach((plugin) => {
      plugin.apply(compiler); // 调用插件一定有的 apply 方法
    });
  }
}

module.exports = webpack;
```

插件的编写这里不再赘述，下面以一个能在我们的 webpack 中运行的插件的为例

```js
// plugins/plugin-a.js
class PluginA {
  apply(compiler) {
    // 在我们在 compiler 对象上声明的 hooks 属性的 run 钩子上注册事件 
    // 我们在 compiler 类的构造函数中对 run 钩子进行类初始化
    compiler.hooks.run.tap('PluginA', () => {
      console.log('PluginA is Running!') // 钩子的调用时机往后看
    })
  }
}

module.exports = PluginA
```

到此为止我们更清晰的知道了 plugin 的工作原理，也知道 compiler 对象是如何存储所有的参数了（构造函数中通过 options 属性存储了），接下来就开始正式编译了

**这里加载的是我们的内部插件，webpack 内置了上百个插件，通过`new WebpackOptionApply().process`方法加载，感兴趣的可以直接去看源码**。

#### 从 entry 入口开始编译

编译是从我们配置的 entry 入口开始的。1）在初始化时已经获得了所有的配置，所以我们可以很方便的拿到 entry 配置。但是 entry 配置的路径是相对于根路径的（webpack 的路径相关配置都依赖于这个根路径），根路径在 webpack 中可以通过`context`选项配置，默认值是当前 ndoe 执行的路径。所以我们也要实现相关功能，2）把 entry 中的相对路径转换为绝对路径。

*这里还要注意 linux 和 windows 的路径分隔符的问题，unix 上以`/`为路径分隔符，windows 上以`\`为路径分隔符但是现在也支持`/`分隔符，所以最好统一转换成`/`，以避免混乱*

```js
/* webpack-yk/core/compiler.js */
const { SyncHook } = require("tapable");
const { toUnixPath } = require("./utils"); // path.replace(/\\/g, "/");
const path = require("path");

class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // 创建几个必要的钩子，简单点我们就使用同步钩子
    this.hooks = {
      run: new SyncHook(),
      emit: new SyncHook(),
      done: new SyncHook(),
    };
    // webpack 配置的 path 根路径
    this.rootPath = this.options.context || toUnixPath(process.cwd());
  }

  // 开始编译
  run(callback) {
    // 在编译开始前手动触发 run 钩子
    this.hooks.run.call();
    const entry = this.getEntry();
  }

  // 获取入口文件路径，是一个对象 默认使用 main 属性
  getEntry() {
    let entry = Object.create(null);
    const { entry: optionsEntry } = this.options;
    if (typeof optionsEntry === "string") {
      entry["main"] = optionsEntry;
    } else {
      entry = optionsEntry;
    }
    Object.entries(entry).forEach(([key, entryPath]) => {
      // 转换为绝对路径
      if (!path.isAbsolute(entryPath)) {
        entry[key] = toUnixPath(path.join(this.rootPath, entryPath));
      }
    });
    return entry;
  }
}

module.exports = Compiler;
```

通过以上转换操作就拿到了我们要的入口对象。

在 webpack 中配置 entry 还可以配置很多其他属性，所以实际上的`getEntry`方法会复杂的多，我们这里就以最简单的两种配置方式做说明，主要是理解这个编译过程。

**这里还要强调一点，钩子的触发。可以看到我们在`run`方法开始时调用了`this.hooks.run.call` 方法，就是这样 webpack 在特定的时机触发特定的钩子。这样不论是我们定义的外部插件还是 webpack 自己注册的内部插件，在一开始加载完成并且注册过的所有 plugin 都会触发。这也是 webpack 的核心架构模式，自身并不做繁杂的操作，而是将整个过程交给各类钩子上注册的事件来处理。**

到此我们的开始编译阶段就完成了，再强调这个阶段一下做了三件事

1. 根据初始化的参数，实例化 compiler 对象，在对象上存储了所有的初始化参数并且初始化了所有需要的钩子
2. 根据参数加载了所有配置的插件，将插件中的事件在各个钩子上注册上
3. 找到入口文件，生成入口文件对象。并且调用符合该周期（run）的钩子，发布事件

接下来就是真正的模块递归编译阶段

## 模块递归编译

在模块编译阶段就是对各个 module 真正的处理，调用 loader-runner 对各个文件进行预处理并且会根据依赖关系生成“依赖关系图”。最终将一个个存在依赖关系的 module 组装成 chunk，这也算是核心步骤了。主要做的事情如下：

1. 根据上一个阶段生成的入口文件对象拿到入口文件路径，使用匹配到的的 loader 对文件进行处理
2. 将 loader 处理完成的文件使用 webpack 进行编译（基于 acorn 生成 AST）
3. 根据 AST 树找到当前文件依赖的文件，重复上面两个步骤（使用 loader 处理，之后编译）
4. 如果依赖的文件也存在依赖，则递归调用依赖模块进行编译
5. 所有依赖递归编译完成后，组装成一个个包含多个模块的 chunk

接下来我们就按照这些步骤，继续完善我们的 compiler 对象。我们知道 webpack 在整体的编译流程中，是一个一个模块处理的，然后最后统一输出 chunk 文件。所以在编译流程中，**我们需要对处理完的模块，chunk，待输出的文件进行存放，最后统一打包成 assets 输出**。

所以在正式从入口文件下手前，我们需要做一些准备工作：**声明几个 Set 对象来存储这些编译流程中规定内容** 

**注意：到这里在 webpack 中其实已经交给 compiler 创建的 compilation 对象来处理了，我们的项目中简化了这一步骤，将逻辑全部写在了 compiler 对象中。所以下面的几个属性其实是在 compilation 对象上的。**

```js
/* webpack-yk/core/compiler.js */
const { SyncHook } = require("tapable");
const { toUnixPath } = require("./utils");
const path = require("path");

class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // ....
    // webpack 配置的 path 根路径
    this.rootPath = this.options.context || toUnixPath(process.cwd());
    // 通过一下几个对象存储内容
    // 存放所有入口模块对象
    this.entries = new Set();
    // 存放所有依赖模块对象
    this.modules = new Set();
    // 存放所有代码块对象
    this.chunks = new Set();
    // 存放本次产出的文件对象
    this.assets = new Set();
    // 存放编译产生的所有文件名
    this.files = new Set();
  }
  // ...
}
module.exports = Compiler;

```

可以看到上面声明了 5 个 Set 对象来存储这些内容。可能有人疑问，为什么使用 Set 对象而不是普通的数组呢？这是因为 Set 具有天然的去重特性，可以在编译流程中占用尽量小的内存。同时 Set 提供了更方便的一些操作方法，比如`has`、`clear`。在对象大小特别大时，Set 也具有更好的性能。（待确认？？？）

**上面的 5 个存储对象`entries/moduels/chunks/assets/files`贯穿整个编译流程始终，存储编译流程中不同阶段的产物。** 

准备工作完成后，就可以开始对入口文件对象进行编译了

#### 编译入口文件

在准备工作中我们声明了`entries`对象用来存储我们对入口文件对象，以便我们后续使用（待确认在哪里能用到？？？）。所以我们需要在编译入口文件时将入口文件对象存储到`this.entries`中，当然文件本身还是交给统一的`buildModules`方法来处理。

```js
const path = require("path");

class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // 存放所有入口模块对象
    this.entries = new Set();
   //
  }

  // 开始编译
  run(callback) {
    // 在编译开始前手动触发 run 钩子
    this.hooks.run.call();
    const entry = this.getEntry();
    // 编译入口文件
    this.buildEntryModule(entry);
  }
  // 获取入口文件路径，是一个对象 默认使用 main 属性
  getEntry() {
    let entry = Object.create(null);
    // ...
    Object.entries(entry).forEach(([key, path]) => {
      // 转换为绝对路径
      if (!path.isAbsolute(value)) {
        entry[key] = toUnixPath(path.join(this.rootPath, value));
      }
    });
    return entry;
  }

  // 编译入口文件，主要是存储 entries 对象
  buildEntryModule(entry) {
    Object.entries(entry).forEach(([entryName, path]) => {
      const entryObj = this.buildModule(entryName, path);
      this.entries.add(entryObj); // 存储入口文件对象
    });
  }

  // 模块编译方法
  buildModule(moduleName, modulePath) {
    // ...
    return {};
  }
}

module.exports = Compiler;
```

在如上的代码中，如果我们的入口文件配置是`entry:{ main: './src/main.js'}`，那么在`buildEntryModule`中，就会存储如下这样一个对象

```js
// this.entries 
{
  'main': '/Users/xx/xxx/src/main.js'
}
```

在真正的模块编译方法`buildModule`中，就可以使用`main`作为 moduleName，`'/Users/xx/xxx/src/main.js'` 作为 modulePath 来查找和编译模块了。

所以接下来就是 webpack 的核心 compilation 过程 —— buildModule

#### buildModule 模块编译方法

首先我们需要知道编译一个模块会做哪些事情

- `buildModule`方法接收两个参数，模块名称和对应的路径。**有了路径我们就可以通过 node 的文件系统`fs`来读取文件内容（我们的源代码），webpack 使用自己封装过一层的 inputFileSystem 来读取文件，但还是继承自 fs** 
- 读取到文件内容后，就可以使用匹配的 loader 来处理了，通过 loader 处理并返回结果
- 得到 loader 的处理结果后，使用 Acorn 将代码转换成 AST 来分析（Acorn 处理 AST 会麻烦一些，他没有提供`@babel/types`这样的工具让我们方便的生成和查找节点，也没提供将 AST 重新转回 js 代码的 API，需要借助其他工具，所以我们的项目中使用`@babel/parser`来处理源码）**这一步主要是分析 module 的依赖，生成含有依赖关系的 module 对象：module -> AST -> dependences -> module** 
- AST 分析完成后，如果该模块不再有任何依赖的模块了（没有 require 语句了），那么就返回编译后的 module。当然如果还存在依赖的模块，那么就递归的调用`buildModule`方法

所以我们一步步来

##### 读取文件内容（直接用 node 的 fs）

```js
/* webpack-yk/core/compiler.js */
const fs = require("fs");

class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // ...
  }
  // ...

  buildModule(moduleName, modulePath) {
    // 读取源码，同时在当前实例上存储一份，以便插件也能访问
    const originSourceCode = (this.originSourceCode = fs.readFileSync(
      modulePath,
      "utf-8"
    ));
    // ...
  }
}

module.exports = Compiler;

```

第一步很简单，直接使用 node 的 fs 读取文件内容。**这里要在 complier 实例上存储一份源码，让我们在后续的操作中能使用**（比如 loader 解析的有问题了，直接不转了使用源码）

读取到文件内容后就可以调用 loader 对模块进行处理，loader 的作用方式和写法就不再赘述了。

##### 调用 loader 处理模块

在 compiler 实例上有初始化参数，我们依然可以在初始化参数中获取到配置的 loader。

同时根据我们传入的 path 可以获取到文件类型，以和 webpack.config.js 中的 test 进行匹配是否使用该 loader

```js
/* webpack-yk/core/compiler.js */
const fs = require("fs");

class Compiler {
  constructor(options) {
    this.options = options; // 存储参数
    // ...
  }
  // ...

  buildModule(moduleName, modulePath) {
    // 读取源码，同时在当前实例上存储一份，以便插件也能访问
    const originSourceCode = (this.originSourceCode = fs.readFileSync(
      modulePath,
      "utf-8"
    ));
    // 同时需要复制一份给 loader 处理使用
    this.moduleCode = originSourceCode;
    // 调用 loader
    this.handleLoader(modulePath);
    // ...
  }

  // 处理 loader
  handleLoader(modulePath) {
    const matchLoaders = [];
    const rules = this.options.module.rules;
    rules.forEach((loader) => {
      const testRule = loader.test;
      if (testRule.test(modulePath))  // 使用正则匹配
        // 仅处理 { test: /\.js$/, loader: 'babel-loader' } / { test: /\.js$/, use: ['babel-loader'] } 两种情况
        if (loader.loader) {
          matchLoaders.push(loader.loader);
        } else {
          matchLoaders.push(...loader.use);
        }
      }
    });
    // 倒序调用 loader
    for (let i = matchLoaders.length - 1; i >= 0; i--) {
      const loaderFn = require(matchLoaders[i]);
      // loader 处理完成
      this.moduleCode = loaderFn(this.moduleCode);
    }
  }
}

module.exports = Compiler;
```

通过这种方式我们所有声明的 loader 也就都处理完成了，**并且将处理的后的代码存在了`this.moduleCode`中**。

在这一步就处理好了入口文件，但入口文件可能存在依赖还要去处理。webpack 则需要记录这种依赖关系才能在后续打包 chunk 的时候将相关的模块打包到一个 chunk 中。那 webpack 是如何处理这种依赖关系的呢？

**将源码转成 AST，通过分析 AST 查找当前 module 的依赖，调用 `module.addDependency`方法将依赖存储到 module 的 dependencies 中，最终返回一个类似如下结构的 module**

```js
// module object
{
  id: 'path', // 当前模块相对于 rootPath 的路径
  dependencies: { }, // 一个存储了当前模块所有依赖的 Set 类型的对象
  name: 'main', // 标识当前模块属于哪个入口文件，因为不同入口文件会生成不同的 chunk
  _source: 'code' // loader 处理过后的代码
}
```

所以我们接下来就要将上一步 loader 处理完成的代码编译成如上结构的符合 webpack 中的 module 对象（**实际上 wbepack 具有 50 多个 module 子类**）

##### 生成 module 对象

为了更好的分析源码，webpack 使用 Acorn 来解析 code 生成 AST。但是我们的项目为了方便，使用 `@babel/parser`来操作。

```js
/* webpack-yk/core/compiler.js */
const path = require("path");
const fs = require("fs");
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const generator = require("@babel/generator").default;
const t = require("@babel/types");

class Compiler {
    // 存放所有依赖模块对象
    this.modules = new Set();
    // ...
  }

  // ...

  buildEntryModule(entry) {
    Object.entries(entry).forEach(([entryName, path]) => {
      const entryObj = this.buildModule(entryName, path);
      this.entries.add(entryObj);
    });
  }

  buildModule(moduleName, modulePath) {
    // 1.读取源码，同时在当前实例上存储一份，以便插件也能访问
    const originSourceCode = (this.originSourceCode = fs.readFileSync(
      modulePath,
      "utf-8"
    ));
    // 2.同时需要复制一份给 loader 处理使用
    this.moduleCode = originSourceCode;
    this.handleLoader(modulePath);
    // 3.生成 module 对象
    const module = this.handleWebpackCompiler(moduleName, modulePath);
    // 返回 module
    return module;
  }
  // ...

  handleWebpackCompiler(moduleName, modulePath) {
    // 将当前模块路径相对于根路径的地址作为 模块 ID
    const moduleId = "./" + path.posix.relative(this.rootPath, modulePath);
    const module = {
      id: moduleId,
      dependencies: new Set(),
      name: [moduleName], // entry 入口的名称
    };
    // 将 loader 处理过的代码转 ast
    const ast = parser.parse(this.moduleCode, { sourceType: module });
    // 遍历 ast ，找到所有 require 标识符
    traverse(ast, {
      CallExpression: (nodePath) => {
        const node = nodePath.node;
        // 如果存在引入的依赖，则需要处理他们的路径，获得绝对路径
        if (node.callee.name === "require") {
          // 获得原来的引入路径
          const requirePath = node.arguments[0].value;
          // 获取模块所在文件夹的路径
          const moduleDirName = path.posix.dirname(modulePath);
          // 获取绝对路径，获取前面的几个路径都是为了在 tryExtensions 中获取到真正文件的绝对路径
          // tryExtensions 是处理默认的文件后缀，就是 webpack resolve 里配置的 extensions 选项
          const absolutePath = tryExtensions(
            path.posix.join(moduleDirName, requirePath),
            this.options.resolve.extensions,
            requirePath,
            moduleDirName
          );
          // 依赖的模块也需要生成一个子 module
          const moduleId =
            "./" + path.posix.relative(this.rootPath, absolutePath);
          // webpack 自己实现的 require 方法，在浏览器上 cjs 规范是行不通的
          node.callee = t.identifier("__webpack_require__");
          node.arguments = [t.stringLiteral(moduleId)];
          // 为原来的 module 添加依赖，可以看到 webpack 只记录了依赖的引用路径，没有立刻去解析依赖
          module.dependencies.add(moduleId);
        }
      },
    });
    // 将 AST 重新转回 code
    const { code } = generator(ast);
    // 将新代码挂载回模块对象
    module._source = code
    return module
  }
}

module.exports = Compiler;
```

可以看到在我们的项目中，将源码转换成 AST 主要是为了替换 require 方法和 require 的实际路径，为什么要这么做呢？

- 浏览器本身并不支持 cjs 的 require 方法，所以需要由我们自己来实现`__webpack_require`（参考 webpack 打包方式）
- require 的路径可能是相对当前模块的路径，这里需要统一转换成相对根路径的地址，后面才能读取到真正的文件

而且我们注意到**module 对依赖的 module 仅保持引用关系，而不去真正解析。**到了这一步，整个 entry 算是编译完成了，接下来就是根据依赖关系，递归的处理依赖模块了。

##### 递归处理

经过上面的一系列操作，我们得到了如下的一个`this.entries`对象（_source 中的示例代码是自己随便写的，详见`webpak-yk/example/src`）

![image-20220213135843017](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220213135843017.png?x-oss-process=image/resize,w_600,m_lfit) 

接下来我们要去遍历 entries 中每个对象的`dependencies`，直到所有的依赖都处理完成。所以我们可以直接在`handleWebpackCompiler`中加入一段递归处理的逻辑（深度优先遍历）

```js
/* webpack-yk/core/compiler.js */
  // ...
  handleWebpackCompiler(moduleName, modulePath) {
    // ...
    // 将 AST 重新转回 code
    const { code } = generator(ast);
    // 将新代码挂载回模块对象
    module._source = code
    // 遍历 dependencies set 对象，递归处理
    module.dependencies.forEach(dependency => {
      // 交给 buildModule 让 loader 来处理
      const depModule = this.buildModule(moduleName, dependency)
      // 将编译后的依赖模块存入 this.modules 对象
      this.modules.add(depModule)
    })
    return module
  }
}

module.exports = Compiler;
```

最终生成的的依赖`this.modules`就如下 

![image-20220213143114608](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220213143114608.png?x-oss-process=image/resize,w_400,m_lfit) 

可以看到有两个重复的`modules.js`，那是因为两个入口文件都引用了这个`module`。在真正的项目中这种情况很常见，**webpack 在 `seal`方法（将 module 分装成 chunk ）时会触发`optimizeModules`钩子调用`splitChunksPlugin`进行优化。** 

我们的项目中就使用简单的判断来处理一下

```js
/* webpack-yk/core/compiler.js */
// ...
class Compiler {
    // 存放所有依赖模块对象
    this.modules = new Set();
    // ...
  }

  // ...

  handleWebpackCompiler(moduleName, modulePath) {
    // ...
    // 将 loader 处理过的代码转 ast
    const ast = parser.parse(this.moduleCode, { sourceType: module });
    // 遍历 ast ，找到所有 require 标识符
    traverse(ast, {
      CallExpression: (nodePath) => {
        const node = nodePath.node;
        // 如果存在引入的依赖，则需要处理他们的路径，获得绝对路径
        if (node.callee.name === "require") {
          // ...
          // 依赖的模块也需要生成一个子 module
          const moduleId =
            "./" + path.posix.relative(this.rootPath, absolutePath);
          // webpack 自己实现的 require 方法，在浏览器上 cjs 规范是行不通的
          node.callee = t.identifier("__webpack_require__");
          node.arguments = [t.stringLiteral(moduleId)];
          // 为原来的 module 添加依赖，可以看到 webpack 只记录了依赖的引用路径，没有立刻去解析依赖
          const alreadyModules = Array.from(this.modules).map(
            (module) => module.id
          );
          if (!alreadyModules.includes(moduleId)) {
            module.dependencies.add(moduleId);
          } else {
            // 否则只新增入口，这也是为什么上面入口时一个数组格式
            this.modules.forEach((module) => {
              if (module.id === moduleId) {
                module.name.push(moduleName);
              }
            });
          }
        }
      },
    });
    // 将 AST 重新转回 code
    const { code } = generator(ast);
    // 将新代码挂载回模块对象
    module._source = code
    return module
  }
}

module.exports = Compiler;
```

到这里整个模块的递归编译阶段就结束了，回忆一下我们一共干了哪几件事

- 从入口文件开始，读取文件内容，调用 loader 进行处理
- 通过分析 ast 查找到入口文件的依赖，生成 module 对象，记录依赖关系
- 递归的遍历依赖模块，使用 loader 处理，在`this.modules`中记录依赖模块
- 将入口文件生成的 module 对象记录到 `this.entries`中

**在这个过程中，真正的 webpack 还有一系列的优化操作，如 tree-shaking、代码压缩等等。也会触发许多的钩子让内置的或者我们的 plugin 来影响编译过程** 

这一步完成后，webpack 核心的编译过程算是结束了，接下来就是完成编译的阶段，将 module `seal`成 chunk，准备输出到我们的文件系统中。

## 完成编译

分装成 chunk 很简单，首先想一想 chunk 对象需要记录些什么

- name：chunk 对象肯定要有个名字，作为最后输出的名字，默认就是我们的入口名
- entryModule：入口的文件对象，一般来说不同的入口文件生成的肯定不是一个 chunk 对象。所以这里也知道我们需要在遍历入口文件对象的时候去生成 chunk 对象
- modules：入口文件对象的所有依赖，`this.modules`

所以我们需要生成一个如下对象

```js
/* webpack-yk/core/compiler.js */
// ...
buildEntryModule(entry) {
  Object.entries(entry).forEach(([entryName, path]) => {
    const entryObj = this.buildModule(entryName, path);
    this.entries.add(entryObj);
    this.buildUpChunk(entryName, entryObj); // 构建 chunk 对象，webpack 中是 seal 分装成 chunk 对象
  });
  console.log("entries:", this.entries);
  console.log("modules:", this.modules);
}
// ...
buildUpChunk(entryName, entryObj) {
  const chunk = {
    name: entryName,
    entryModule: entryObj,
    // 根据 name 来查找当前这个依赖模块是不是属于这个 chunk
    modules: Array.from(this.modules).filter((module) =>
                                             module.name.includes(entryName)
                                            ),
  };
  this.chunks.add(chunk);
}
```

至此 chunk 对象也生成了，只剩下最后一步，依据这个 chunk 对象生成我们最终的 assets 文件

## 输出文件

这一步应该要参考《webpack 打包方式》来处理。总的来说就是要实现自己的`__webpack_require__`

当然我们首先要通过配置获取到文件要被输出到哪个位置，然后才是核心实现

```js
/* webpack-yk/core/compiler.js */
run(callback) {
  // 在编译开始前手动触发 run 钩子
  this.hooks.run.call();
  const entry = this.getEntry();
  // 编译入口文件
  this.buildEntryModule(entry);
  // 输出文件到 assets 中
  this.creatChunkAssets(callback);
}

creatChunkAssets(callback) {
  // 首先获取我们的输出路径
  const output = this.options.output;
  this.chunks.forEach((chunk) => {
    // 支持我们平常的这种 [name] 写法
    const parseFileName = output.filename.replace("[name]", chunk.name);
    this.assets[parseFileName] = renderRequire(chunk); // 实现 __webpack_require__
  });
  // 别忘了调用输出文件时的钩子
  this.hooks.emit.call();
  // 输出目录不存在时首先创建
  if (!fs.existsSync(output.path)) {
    fs.mkdirSync(output.path);
  }
  // 存储所有输出文件名
  this.files = Object.keys(this.assets);
  Object.keys(this.assets).forEach((filename) => {
    const filePath = path.join(output.path, filename);
    // 输出文件
    fs.writeFileSync(filePath, this.assets[filename]);
  });
  // 也别忘了输出文件后触发的钩子
  this.hooks.done.call();
  // 回传参数给回调函数
  callback(null, {
    toJson: () => {
      return {
        entries: this.entries,
        modules: this.modules,
        files: this.files,
        chunks: this.chunks,
        assets: this.assets,
      };
    },
  });
}
```

在上面输出 chunks 的过程中，大部分逻辑都很简单，主要就是通过`fs`将文件写入了磁盘。那核心的`__webpack_require__` webpack 是怎么实现的呢？

首先我们知道 cjs 的 require 在浏览器是不被支持的，所以 webpack 需要实现自己的 cjs 。（处理方式可以参考《webpack 打包方式》，这里使用一种简单实现）

我们直接采用比较偷懒的方式（核心理念和 webpack 是相同的）

```js

function renderRequire(chunk) {
  const { name, entryModule, modules } = chunk;
  return `(() => {
    var __webpack_modules__ = {
      ${modules.map((module) => {
    return `'${module.id}': module => { ${module._source} }`;
  }).join(',')}
    };

    var __webpack_module_cache__ = {};

    function __webpack_require__(moduleId) {
      var cacheModule = __webpack_module_cache__[moduleId]
      if(void 0 !== cacheModule) return cacheModule.exports
      var module = (__webpack_module_cache__[moduleId] = {
        exports: {}
      })
      __webpack_modules__[moduleId](module, module.exports, __webpack_require__)
      // 返回 module 中的 module.exports 对象
      return module.exports
    }

    (() => {
      ${entryModule._source}
    })()
  })()`;
}
```

直接通过模版字符串来渲染成浏览器能认识的代码：

- 将 modules 映射成 id 对应的模块存储在`__webpack_modules__`中
- 实现`__webpack_require__`方法，去我们声明的`__webpack_modules__`中取源码
- 执行入口文件的源码

这样一个自己的 cjs 规范就实现了

webpack 源码中是通过生成 AST 再转 code 统一处理的，在它的`JavascriptModulesPlugin`中处理，参见`webpack/lib/javascript/JavascriptModulesPlugin.js`。**这里也再次强调 webpack 是通过触发各种钩子通过在这些钩子上来执行真正的逻辑操作的设计模式，这种模式非常益于扩展。可以在我们的项目中借鉴。**

## 总结

通过这一系列操作，一个建议的 webpack 就实现了，也让我们更加清晰的了解了 webpack 的整个编译过程。下面是手画的流程图

![Webpack编译流程-1 2](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/Webpack%E7%BC%96%E8%AF%91%E6%B5%81%E7%A8%8B-1%202.jpg?x-oss-process=image/resize,w_800,m_lfit) 

## 参考文章

一文吃透 Webpack 核心原理：https://zhuanlan.zhihu.com/p/363928061

webpack 编译流程：https://juejin.cn/post/6844903935828819981

Webapck5核心打包原理全流程解析：https://juejin.cn/post/703154640003494710