# loader

loader，webpack 打包前的预处理任务。深入了解 webpack 的 loader 并手把手写一个。

[toc]

## 什么是 loader？

webpack 打包文件前的预处理工具。webpack 原生只能打包 JS 和 JSON 文件。loader 可以对其他类型的文件进行预处理，将其他文件转化为有效的模块，以供 webpack 打包使用。

官方说法

```text
通过 loader 可以使 webpack 支持多种语言和预处理器语法编写的模块。loader 向 webpack 描述了如何处理非原生模块，并将相关依赖引入到你的 bundles中。
```

#### 触发时机 

`compilation.addEntry -> loader runner` 

所以每次读取一个新的依赖模块时，都会执行一遍配置的 loader

#### 开发准则

- keep simple
- 链式调用，保证 loader 的单一职责
- 模块化的输出，`module.exports = xxx`
- 无状态化，每个 loader 应该完全独立
- 记录 loader 的依赖
- 解析模块依赖关系

  - 不同的模块规范，甚至 CSS 的 @import 引入依赖，需要统一解析出依赖关系
  - 可以通过（1）转化成统一的  CJS 规范；（2）使用`this.resolve`函数解析实际路径
- 提取通用代码
- 避免绝对路径
- 使用`peerDependencies`，提前声明当前`loader`依赖的另一个包（如果存在的话）

#### 执行顺序

前置(pre) > 普通(normal) > 行内 > 后置(post)

**前置、后置配置方法** 

通过在`module`的`rules`中配置 `enforce`字段

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'test-loader',
        enforce: 'pre' // 前置，不配置该项就是 normal 普通
      }
    ]
  }
}
```

可以通过配置该字段来保证某一 loader 在最前或最后执行。优先级比原本的从右往左执行要高。

**行内**

`import Styles from '!style-loader!css-loader?modules!./style.css'` 

不同的 loader 之间通过`!`分隔。可以通过`?`传递参数，效果等同于`loader.options` 

**ps: 最终读取 webpack 配置中的 loader 之后还是会拼成行内的形式去执行** 

**前缀** 

行内引入 loader 时可以配置各类前缀，如上行内引入 loader 所示。**行内引入的分隔符就是`!`号，所以引入了行内 loader 时肯定会忽略 webpack 中配置的 normal laoder，这样也是为了防止相互冲突。**

- `!`：跳过 normal loader
- `-!`：跳过 pre 和 normal loader
- `!!`：跳过 pre、normal 和 post loader

前缀在编写 loader 时很有用。

## 四种 loader

 在 webpack 中一共有四种类型的 loader

1. 同步 loader：（1）直接`return`内容；（2）调用`this.callback()`并且无`return`值；
2. 异步 loader：调用`this.async()`来通知 webpack
3. "Raw" loader：webpack 可以处理 utf-8 编码的文件，也可以直接处理 Buffer。像图片文件等就需要直接传递原始的 Buffer，`compiler`将把它们在 loader 之间相互转换。在 loader 中声明`module.exports.raw = true`那么 loader 接收到的就是原始的 Buffer
4. Pitching loader：每个 loader 都有一个`pitch`方法，用以控制 loader 的调用链

#### 基础 loader

最简单的 loader 如下：

```js
// testLoader.js
module.exports = function (content) { // content 是 webpack 传入的内容
  return '// this is TestLoader \n' + content
}
```

会在解析每个 module 的时候都给代码前的内容加上一句`this is TestLoader`的注释，实际效果如下

![image-20220202164243039](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220202164243039.png?x-oss-process=image/resize,w_400,m_lfit) 

下面分别说明一下四种类型的 loader

#### 同步 loader

直接 return 值或者通过调用`this.callback`并且无返回值

`this.callback(err, content[, sourceMap[, meta]])`

- err：`Error || null`,如果存在 Error 对象，表示编译失败，打包过程中会抛出该错误信息
- content：{ string | buffer } 处理后返回的内容
- sourceMap: 一个可以被正常解析的 sourceMap
- meta：可选，可以是任何东西。用来传递信息？

简单示例如下

```js
// 把所有的 console.log 替换成了 console.warn
module.exports = function (content) {
  console.log("执行 test-loader-sync");
  try {
    const myContent = content.replace(/console.log/g, "console.warn");
    this.callback(null, myContent); // 无错误信息的话第一个参数直接使用 null
  } catch (e) {
    this.callback(e, content); // 会在 webpack 打包过程中收到该错误信息
  }
  return;
};
```

#### 异步 loader

在 loader 处理过程中我们也可以使用异步请求，同时打包过程**会阻塞打包流程并等待异步操作的的返回**。

`const callback = this.async()` 使用方式要注意，**`this.async()`返回一个回调方法。通过调用该回调方法来继续解析文件，而不是直接调用`this.async()`** 。回调方法的参数和同步的`this.callback()`完全相同。

简单示例如下

```js
module.exports = function (content) {
  console.log('执行 test-loader')
  const callback = this.async()
  setTimeout(() => {
    console.log('test-loader 异步执行')
    callback(null, content) // 什么也没做，只是异步处理了
  }, 2000) // 后续的 module 解析会等到这里返回了才会继续，否则拿不到 content 内容
};

```

异步主要是用在一些需要读取文件内容的地方，比如我这个 loader 可以支持通过`loader.config.js`配置，就可以通过`fs.readFile()`读取文件并配合该异步 loader 处理。

但 loader 还是要 keep simple，复杂的异步操作会带来极差的体验。因为这里的异步实际是将异步转成了同步操作，会阻塞主线程。

#### "Raw" loader

"Raw" loader 主要是用来处理资源文件，如文本、图片、视频等

`module.exports.raw = true` 其核心就是这个属性，webpack 会读取 laoder 的该属性。如果是 true 则会给我们的 loader 传入原始的 Buffer

然后我们可以对资源文件进行压缩，转换等一系列操作。

**首先我们知道 webpack 原生只能处理 js 和 json 文件，但是如果存在一张图片，要让 webpack 能读取它，我们就需要把它转换成** 

```js
modoue.exports = 'png'
```

**这样的形式。**  （小图片一般都转 base64了，视频资源一般还是用 copy-webpack-plugin 直接拷到打包目录。webpack 也提供一个`emitFile`API 来直接输出文件）

简单示例如下

```js
module.exports = function (content) {
  if (Buffer.isBuffer(content)) {
    console.log("raw-loader: buffer content");
  }
  console.log(this.context);
  // this.callback(null, content);
  return `module.exports = ${JSON.stringify( // 加上 module.exports 输出
    `data:image/png;base64,${content.toString("base64")}` // buffer 转 base64 字符串
  )}`;
};

module.exports.raw = true; // 声明接收一个原始数据类型 buffer
```

其他示例可以参考[img-loader](https://github.com/vanwagonet/img-loader/blob/main/index.js) 

#### pitching loader

在说明 pitching loader 之前先要说清楚 loader 的调用顺序，如下图

![IMG_B45AE788A987-1](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/IMG_B45AE788A987-1.jpeg?x-oss-process=image/resize,w_800,m_lfit) 

webpack compilation 调用 loader runner 之后，先正序遍历配置的 loader，同时执行 loader 的 pitch 方法（如果存在的话）。之后再从后往前的倒序执行 loader。

**所以导出了 pitch 方法的 loader 就是 pitching laoder**。同时注意，如果 pitching loader 有返回值则会中断链式调用。效果如下

![IMG_A0C00B8070BE-1](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/IMG_A0C00B8070BE-1.jpeg?x-oss-process=image/resize,w_800,m_lfit) 

```js
/**
* 注意：前两个参数返回的都是路径
* @param { string } remainingRequest：剩余的 loader，webpack 将 loader 合并成行内的形式后，后面的 loader。如上面合并成行内* * 就是 !loader1!loader2!loader3!./file.js。如果当前 pitch 在 loader2 上，那么 remainingRequest 就是 * /Users/xxx/xxx/loader3.js!/Users/xxx/xxx/src/file.js
* @param { string} precedingRequest：前面的 loader，参考上面的说明
* @param { object } data：pitch 传递给 loader 执行阶段的数据对象。在执行阶段可以通过 this.data 获取
*/
module.exports.pitch = function (remainingRequest, precedingRequest, data) {
  return 'xxx';
}
```

**使用 pitch loader 需要注意两点：**

1. pitch function return 的值中不能有空格符，否则 webpack 无法转换和传递，会直接报错
2. pitch function retrun 的值会作为处理后的代码传递给下一个要处理的 loader。比如我在 loader2 `return 'xxxx'`，那么剩下会执行的是 loader1，它接收到的 content 也就是 `xxxx`。

四种类型的 laoder 就介绍到这里。可以看到在 loader 的示例代码中，我们使用了很多`this.xxx`，那这个`this`指代什么呢？

#### loader 中的 this

loader 中的 this 指代的就是 loade runner。因为 loader 最终就是被 loade runner 这个对象作为方法调用，所以我们可以通过这个 this 去调用 loader runner 上的一些其他方法。下面是一些常用的方法（API）

**常用 API** 

官方文档：https://www.webpackjs.com/api/loaders/

- `this.addDependency(file: string)`：用于 loader 实时编译处理，对 add 的文件进行监听，每次文件改变时都会重新编译
- `this.clearDependencies`：清除 loader 的所有依赖
- `this.cacheable(flag: Boolean)`：用于告知 webpack 是否缓存 loader 的处理结果，默认为 true
- **`this.getOptions()`：**最常用的 API，用于获取用户配置的参数
- `this.context()`：获取文件所在目录
- **`this.data`：**pitch loader 和执行阶段共享的数据对象
- **`this.loaders`：**所有 loader 对象（包含 loader 的一系列属性，path、query 等）组成的数组，**可以在 pitch 阶段写入** 
- **`this.resolve(context, request, callback)`：**解析一个 request，相当于手动 require 一个文件
- `this.resource`：获取当前 request 路径，包含 requset 等参数 query。也可以用`this.resourcePath`仅获取路径
- `this.sourceMap`：{ bool }，是否生成一个 sourceMap，由 webpack 配置决定
- **`this.emitFile(name, content, sourceMap)`：**直接导出一个文件到打包目录，file-loader 就是使用的该 api。**配合`__webpack_public_path__`获取到在 webpack 配置的`output.publicPath`**，即可完成非 js 资源文件的导出处理。**最后 exports 路径即可，路径可以通过 **`loaderUtils.interpolateName`生成（一般取`[contenthash].[ext]`）

*request 是 loader 处理模块的请求对象* 

## loader 工具

#### loader-utils

官方仓库：https://github.com/webpack/loader-utils

官方提供的 loader 开发工具，可以用来获取配置和处理一些文件

三个常用方法：

- `loaderUtils.getOptions()`：获取用户在`rules`中配置的`options`，**在 webapck 5 中被 `this.getOptions`替代**
- `loaderUtils.interpolateName(this, name, options?: {context?, content? })`：处理生成文件的名字，需要用这个 api 是因为他可以生成和 webapck 相同配置规则的 hash 名和自动获取扩展名。**注意 options 中的 content 是生成 contenthash 的依据，所以如果要用 contenthash 这个选项必填**。有时配合`this.emitFile`处理资源文件的名称
- `loaderUtils.urlToRequest(request, root?)`：把资源路径处理成 webpack 模块请求，默认 root 是 `/`

## loader 配置

在 webpack 属性解析中已经有配置说明了。这里说一些注意点

- 可以通过配置 webpack `resolveLoader`的路径来引入本地 loader
- 可以通过配置绝对路径来引入 loader：`loader: path.resolve(__dirname, 'loaders/test-loader.js')`
- 可以设置相同的`module.rules`匹配，同时多个 loader 会按声明顺序拼接（相同的 loader 会重复执行），之后按 loader 的后序顺序执行

## 其他

#### 1.可以配合 @babel/parse、acorn 等 AST 解析工具来处理代码

#### 2.在使用各类 loader 时，我们要时刻记得两点

1. loader 是链式调用的 ，上一次处理后的内容会流转到下一个 loader；
2. webpack 执行 loader 是随着`compilation.addEntry`依赖收集一个个文件每次都执行的，一个文件（module）就要走一次 loader 链；

#### 3.loader 为什么从右往左执行？

1. 更语义化，module 最终被拼为一个带有 loader 链的路径，靠近文件的先执行，由内往外将源文件格式转换成目标格式更符合语义；
2. ？？？

#### 4.一个示例，简单的 style-loader 实现

```js
// my-style-loader
const loaderUtils = require("loader-utils"); // 注意默认使用 CJS 的方式引入
module.exports = function (content) {};

module.exports.pitch = function (r, p, data) {
  /**
  * 使用 loaderUtils.urlToRequest 获得剩余的（css-loader）的实际执行地址，并再次 require
  * 使用 '!!' 跳过其他 loader 执行，仅执行 css-loader 来帮我们获取 style
  * 在 pitch 中 return 也会跳过其他 loader 的执行，一般 style-loadr 是第一个，所以解析 css module 时只会 	 * 执行到这就返回了
  * 最终通过 document.head.appendChild 插入元素
  */
  return `
    let style = document.createElement('style');
    style.innerHTML = require('${"!!" + loaderUtils.urlToRequest(r, '')}');
    document.head.appendChild(style);
    `;
};

```

## 参考文章

手把手教你写一个 loader / plugin：https://juejin.cn/post/6976052326947618853

揭秘 webpack loader：https://zhuanlan.zhihu.com/p/104205895