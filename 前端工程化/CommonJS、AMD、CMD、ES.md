# CommonJS、AMD、CMD、ESM

当前前端工程中，模块化是和我们吃饭一样重要的一环。前端流行的模块化方案就是CommonJS、AMD/CMD、ESM 这四种，今天我们来简单的复习一下这四种模块化方案（规范）。

| 规范          | 简述                                                         | 使用环境    | 实现                                                         |
| ------------- | ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| **CommoJS**   | Mozilla 的开发者 09 年开发的一个项目，当前最流行的模块化规范之一 | 服务端      | [NodeJs ](http://nodejs.cn/api/modules.html#modules-commonjs-modules) |
| AMD           | Async Module Definition 顾名思义，异步模块定义。为了处理 CommonJS 在前端（web）不能直接使用的问题 | 前端        | [RequireJS](https://github.com/requirejs/requirejs)          |
| CMD           | Common Module Definition，国内大神在 SeaJS（一个 web 端的模块加载器）中提出的一种异步模块的实现方式，在国内有一些使用者 | 前端        | [SeaJS](https://github.com/seajs/seajs/issues/242)           |
| **ES Module** | ES6 提出的 ES Module，当前另一个流行的模块化方案，浏览器原生支持 | 前端/服务端 | [Modules](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules) |

开始之前我们先想想 Why modules？为什么我们需要模块化

[toc]


**在没有模块化之前是怎么样的？ **

最开始 JS 作为脚本语言，只处理一些简单的逻辑。到后来越来越复杂，前端工程师们开始抽离逻辑到单独的 JS 文件中，与 HTML 部分解藕。但是会带来一些问题

1. 全局变量污染
2. 脚本文件之间的依赖关系

最开始简单处理一下这些问题，比如为了处理全局变量污染，使用命名空间，如

```js
var app = {}
// a.js
app.a.name = 'test'

// b.js
app.b.name = 'test'
```

这样可以一定程度上解决全局变量污染的问题，但是在 b.js 中也可以直接修改 a.js 中的变量，会带来很多问题。虽然可以**使用 IIFE 和 闭包特性来模拟私有变量**处理。但是又会带来闭包导致的复杂性（为了让其它文件能访问我，我需要给要暴露的每个变量写个 getter）和内存泄漏问题。而且还是没有处理模块间的依赖关系。**前端需要模块**，不仅是上面说的两个问题。前端的工程越来越大，也需要模块化来拆分、解耦、复用模块。

## CommonJS（cjs）

> 不是前端的 bing，革了前端的命

CommonJS 最开始并不是给前端用的，而是在服务端。

```text
CommonJS is a project with the goal to establish conventions on the module ecosystem for JavaScript outside of the web browser.
```

根据 wiki 上的记录，CommonJS 是一个旨在浏览器之外建立模块生态系统的规范。

在 node 把 JS 带到服务端之后，由于后端存在的复杂业务场景，急需一种模块化的解决方案，于是 nodejs 中实现了 CommonJS 规范，CommonJS 也因此被广大群众所知。同时它影响了后面其他的模块化方案。所以一开始 CommonJS 被创建是为了处理服务端的复杂常见问题的而不是前端。

#### 使用

基于 nodejs 中对 CommonJS 的实现

**模块导出** 

```js
// a.js
module.exrpots = function sum (a, b ) { return a + b; }
// 或者使用 exports。注意不能直接 exports = xxxxxx，会改变 exports 的指向
// exports 本来也只是 moodule.exports 的引用而已，require 加载模块最终也是在 module.exports 中寻找导出的对象
exports.sum = function (a, b ) { return a + b; } // 相当于 module.exports.sum = xxxxxx
```

**最终导出的对象是对声明对象的浅拷贝**，也就是说基本类型相当于直接复制，对象则导出的是引用关系。

**模块引入** 

根据导出的方式不同，引入的方式有所差别，具体看代码

```js
// b.js
// 如果是 module.exports = xxxx 直接导出（可类比于 es6 的 export default）
const sum = require('./a.js');
sum(1, 2) // 3

// 如果是 exports.sum = xxxxxx / module.exports.sum = xxxx
const sum = require('./a.js'); // 这时 sum 不再直接是函数了，而是一个包含 sum 函数的对象
sum.sum(1, 2) // 3
```

#### 原理

在说明原理之前先说明 CommonJS 中的几个概念（基于 node 的实现）

- 模块：在 CommonJS 中，一个单独的脚本文件被视为一个模块
- 作用域：每一个模块都有单独的作用域，在作用域声明的变量、函数、类都是私有的
- module：module 是每个模块的引用对象，它不是全局的，而是每个模块本地的

在使用`require`第一次**加载脚本**时（后面加载时会有缓存策略`require.cache`），会**执行被加载的整个脚本**，并在内存生成一个对象，结构如下

```js
{
  id: 'xxx', // 模块名（不是文件名，一般是文件的路径）
  filename: '/xxx/xxx/a.js', // 文件名
  exports: { xxx }, // 导出的对象
  parent: [ ... ], // 父模块
  children: [ ... ], // 子模块
  paths: [ ... ], // 模块可能存在的路径
  loaded: true // 是否加载完毕
  ...
}
```

require 也是挂在 module 原型上的一个方法（非全局的），当我们用 require 去引入模块的时候。调用的是`module.load`方法，这里不说方法的具体实现，只说流程。load 方法则根据`paths`去找到模块，最后通过`module.__compile`（将模块读取成字符串再编译）方法解析找到的模块，完成加载。

`require`加载的整个过程是同步进行的，但存在一个循环引用的问题，就是 a.js `require`了 b.js，b.js 又`require`了 a.js。 CommonJS 是怎么处理这个问题的的呢？

#### 循环引用问题

上面说到，通过`require`去加载模块的时候，会执行整个模块。

```js
// a.js
console.log("a 开始执行");
exports.a1 = "1";
const sum = require("./b.js"); // 执行 b.js
exports.a2 = "2";
console.log("sum", sum.sum(2, 3));
console.log("a 执行结束");

// b.js
console.log("b 开始执行");
const a = require("./a.js"); // 执行 a.js，a.js 在内存中已存在，返回已经执行完成的部分
console.log('a1', a.a1)
console.log('a2', a.a2)
exports.sum = function (a, b) {
  return a + b;
};
console.log("b 执行结束");
```

示例代码如上，当我执行 a.js ，在 a .js 中去加载 b.js 时，b.js 则开始执行了。执行到`const a = require("./a.js");`则出现了循环引用的问题。**CommonJS的做法是出现这种情况时，返回已经执行完的部分**。什么意思呢？就是在 b.js 中加载 a.js 时，a.js 的`exports`中只有`a1`这个变量。`a.a2`就会是`undefined`。然后 b.js 继续往下执行。

输出结果如下

```js
/*
a 开始执行
b 开始执行
a1 1
a2 undefined
b 执行结束
sum2 5
a 执行结束
*/
```

同样也是为了处理这种循环引用问题，CJS 只支持一个文件一个 module。但是在浏览器（前端）环境下，使用 CJS 就会存在一些问题，比如 html 通过 script 标签引入 js 是通过 http 请求的，这就会带来异步问题。而 CJS 的引用又是同步的，进而引发更多问题。也就是在这种背景下 AMD 诞生了。

**注意实际开发中，虽然 CJS 自己处理了循环引用的问题，但是我们仍然应该避免这种问题。** 

#### CommonJS & CommonJS2

在配置 webpack 的`externals`时有时候会定义对这个包的引用方式，其中有时会出现`commonjs2`这种方式。不知道的同学可能有些疑惑是个啥。

```js
{
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash'
    }
  }
}
```

`commonjs2`和`commonjs`有什么关系呢？

[CommoJS](https://github.com/commonjs/commonjs/blob/master/docs/specs/modules/1.0.html.markdown) 中定义了`exports`这种模块导出方法。而 node.js 对 CommonJS 的实现做了一些扩展：

- 将`module.exports`作为 commonjs 规范的模块导出实现
- 将`exports`作为`module.exports`的[引用(或者说别名？)](https://nodejs.org/docs/latest/api/modules.html#modules_exports_alias)。

**我们把这种扩展后的 CommonJS 规范称为 CommonJS2。** 目前也只在配置 webpack 时用到了这个概念

顺便说一下 webpack 打包也是一种 CommonJS 的实现方式。

## AMD

> CommonJS 你有什么毛 bing

AMD 是为了处理 CJS 在**前端应用**的问题而诞生的。上面说的 CJS 方案，在`require`的时候就去加载并执行文件，这在服务器端是可以接受的，但是在 web 端不能每次要加载模块的时候才去发个请求把模块拉下来，不确定因素太多了。如果阻塞执行会非常影响用户体验，如果不阻塞依赖关系无法保证。

AMD 是流行的前端模块处理规范，受 CJS 影响，它内部也实现了一些 CJS 的特性。

```text
AMD provides some CommonJS interoperability. It allows for using a similar exports and require() interface in the code, although its own define() interface is more basal and preferred.
```

我们所使用的 AMD 规范由 [requirejs](https://github.com/requirejs/requirejs) 实现。它主要解决的就是**在浏览器环境下多个模块间的依赖关系**。CJS 一个文件只能有一个 module，那么在浏览器环境下由于异步问题就会让多个模块间的依赖关系难以为继。

#### 使用

基于 requirejs 实现

首先需要引入`requirejs`，可以通过 cdn 的方式`<script src="require.js"></script>`或者`npm`安装

**模块导出** 

`define([[id,] dependencies,] factory)`：通过`define`方法定义要导出的模块

- id：{ string} 可选参数，没有提供就是文件名，**最好声明，不然容易出现 Mismatched anonymous... 错误** 
- dependencies：{ array }可选参数，当前模块的依赖，**如果不设置，导出值默认为`["require", "exports", "module"]`** 
- factory：模块初始化工厂方法（在依赖加载完后执行）。如果是函数，**参数是依赖模块的导出值**，返回值就是 exports 的值。如果是对象，那么这个对象就是 exports 的值

```js
// a.js
// 注意：在使用 cdn 的引入方式时，最好声明要定义的模块 id
// 否则 requirejs 为了防止匿名函数导致的冲突问题，会直接报错 Mismatched anonymous define....
define('moduleA', function () {
  let name = 'Study'
	return {
    name
  }
});
```

**与 CJS 相同，AMD 导出的模块中的对象也是使用的浅拷贝。** 

**模块引入** 

`requirejs(modules, factory)`：模块引入使用 requirejs 库在全局暴露的 `requirejs`方法。**网上很多文章还是用 define 方法来引入依赖，给我看懵了。大家要注意下 define 仅仅是定义模块，要引入还得用 `requirejs`** 

```js
// b.js
requirejs(["moduleA"], function(a) {
    console.log(a.name); // study
  	return {
			name: a.name + 'hard'
    }
});
```

AMD 采用异步的方式来加载模块，也就是说模块加载不会阻塞后面代码的执行。可以把对模块有依赖的代码定义在`factory`对象中，所有依赖模块加载完后才执行`factory`函数或返回`factory`对象。这样就保证了模块的依赖关系不会因为异步的问题而混乱。

#### **循环引用** 

在 requirejs 中也存在循环引用的问题，但是 requirejs 自身并没有处理这个问题。需要我们在`factory`中自己处理。常用的方案有：

- 使用 AMD `factory`回调中的 `require`方法，在 return 时才去动态的加载依赖的包
- 由于 AMD 几乎实现了 CJS 的所有特性，所以可以使用 `factory`中的默认回调参数，使用 CJS 的方案去加载模块

#### 其他

**`requirejs.config`**方法

用于定义基础引入路径、别名等

```js
requirejs.config({
  paths: {
    vue: 'vue.min.js' // 之后我们通过 define 方法引入 vue 的时候直接写 vue 就可以了
  }
})
```

## CMD

> AMD 行我也 xing

官方文档：https://seajs.github.io/seajs/docs/

CMD 是国内大神在实现异步模块规范时提出的一种“规范”，也可以说是一种实现方式。

它的使用方法可以说和 AMD 基本相同，大家参照 AMD 的使用方法和官方文档，这里重点说一下他和 AMD 的区别。

#### AMD 与 CMD 的区别

AMD 和 CMD 最大的区别是模块加载时机的不同： AMD 在执行`factory`前需要完成模块的加载，而 CMD 可以类似动态加载，在需要时引入。

- AMD：推崇**提前执行**，即要用什么依赖，你提前给我写好
- CMD：推崇 lazy laoder 懒加载，即什么时候要，我什么时候加载

```js
// AMD 
define(["a", "b"], function (a, b) {
  // 使用提前定义好的 a、b 模块
});

// CMD
define(function (require, exports, module) {
  // 按需引入（可能造成堵塞）
  const a = require("./a.js");
  const b = require("./b.js");
});
```

requirejs@^2.0 提供了[语法糖](https://requirejs.org/docs/whyamd.html#sugar)，就像 CMD 一样，以官方例子

```js
define(function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');

    return function () {};
});

// AMD 加载程序将使用 Function.prototype.toString() 解析出 require('') 调用，然后在内部将上述定义调用转换为
define(['require', 'dependency1', 'dependency2'], function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');

    return function () {};
});
```

虽然有这个语法糖，但他还是先做了转换去把模块加载完成再执行`factory`的，所以和 CMD 还是不同。

AMD 和 CMD 两种实现方式不能说孰优孰略，只能说适合你的就是好的。

## ES Module（mjs）

> 真行还得看我正黄旗的 ES Module

ES Module，浏览器原生支持的老正黄旗的模块化方案。这个相信是现代的前端最熟悉的方案了。

```js
import xxx form xxx;

export { xxxx }
```

这两个语句应该都不知道写了多少遍了，唯一的缺点就是浏览器的兼容问题。

使用就不再赘述了，应该没有人不清楚吧😁（不清楚的去把红宝书翻10遍）。这里主要阐述一下他和上面所述的几个规范的区别

#### 动态引用

ES Modules 使用真正的动态引用。上面所述的 CJS、AMD 导出值时，都是对值的浅拷贝。而`import`文件时实际上只是生成了一个引用，就像我们在 JS 中声明一个对象一样（纵使我们 export 只是一个基础类型的变量）。等我们真的去访问 import 进来的值时候才去模块中取值。

**How?** 

这里最大的问题其实是 ES Module 是怎么做到的？

对于 ES Module 的加载一般是三个步骤：

1. 构造：查找和解析所有的文件作为模块记录
2. 实例化：在内存中找到所有模块记录的变量存放地址，但不填充他们，并将导出和导入都指向同一个内存地址
3. 运行：运行代码时以变量实际值填充内存地址

实际里面包含的东西比较多，简单来说就是这三步，让 ES Module 在加载时实现动态引用。详细可以查看这篇文章[es-modules-a-cartoon-deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)。

#### 循环引用

由于 ES Module 动态引用的特性，所以 ES Module 中不会产生像 CJS 一样的循环引用问题。我们只需要保证在取值的时候能取到就行了。

#### 注意事项

在项目中我们经常能看到这种代码

```js
import 'xxxx/lib/index.css';
```

这会让我们误以为 ES Module 支持 css 等文件的导入。**实际上 ES Module 只支持 js 文件的导入导出**，在项目中这样写之所以可行，是由于`webpack`等打包工具，在打包过程中会将 import 的所有文件视为 module，交给各自的 loader 处理后输出。而不是真的通过 import 引入的。

## END

最后，随着 webpack 等一系列工具等出现，我们已经很少关注 AMD/CMD 这两种方式了。但我们在阅读一下老的代码库的源码的时候，或者在自己写一个工具库的时候总会碰到他们。有个概念总是好的！

## 参考文章

Why AMD？：https://requirejs.org/docs/whyamd.html

CommonJS 介绍：https://zhuanlan.zhihu.com/p/113009496

CommonJS和ES6模块循环加载处理的区别：https://juejin.cn/post/6844903747290660878

require 源码解读：https://www.ruanyifeng.com/blog/2015/05/require.html

ES Module：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Module

es-modules-a-cartoon-deep-dive: https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/