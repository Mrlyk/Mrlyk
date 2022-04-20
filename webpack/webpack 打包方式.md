# webpack 打包方式

webpack 是现在最流行的打包工具，这里探讨一下它的打包方式，并对不同文件的打包方式区别做一个说明。webpack 的打包方式只是 webpack 整体编译过程中的最后一步，将 module 合并成 chunks 这一步中。

首先引用一下官网的话

```text
本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
```

## 打包方式

webpack 打包的 mode 分为`development`和`production`两种，两种 mode 由于默认的配置不同，打包出来的最终产物有所不同，不过打包的方式是相同的。我们通过打包的产物，来逆推一下 webpack 的打包方式。

#### production

这是我的示例代码，非常简单

```js
// sum-output.js
module.exports = function sum(a, b) {
  return a + b;
};


// index.js
const sum = require("./src/sum-output");
console.log("这是 index");
console.log("sum(1, 2)：", sum(1, 2));

```

下面是去掉代码压缩、混淆后`production`模式下的产物。（注释与笔记为后来添加）

![webpack package](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/webpack%20package.jpg)

可以清晰的看到 webpack 打包最终就是将关联的模块放到了一个文件中，首先使用`__webpack_modules__`对象来记录所有关联的导出模块。再通过自定义的`__webpack_require__`函数去对象中找引入目标模块（顺便提一下这里用的是设计模式中的策略模式），期间做一些缓存操作，这样一个可以在浏览器执行 js 文件就打包好了。

在`development`环境下打包的产物有所不同。

#### development

在 development 环境打包时，不需要关注太多的性能问题和安全问题，只追求一个字“快”。

以我上面的示例代码为例，就这么几句代码，在`production`模式下的打包速度比在`development`模式下慢三倍！！！

![image-20220125155441816](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220125155441816.png?x-oss-process=image/resize,w_400,m_lfit) 

![image-20220125155511457](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220125155511457.png?x-oss-process=image/resize,w_400,m_lfit) 

为什么会有这种差异呢？就是因为 webpack 在`development`模式下默认的打包配置和`production`不同。`development`模式下**webpack 默认直接使用`eval`来直接执行目标目录的代码**。

也就是说在默认情况下，它打包出来的代码是下面这样的。

```js
(() => {
  var __webpack_modules__ = {
    "./index.js": () => {
      eval("\n\n//# sourceURL=webpack://webpack-demo/./index.js?"); // 直接使用 eval 执行
    },
  };
  var __webpack_exports__ = {};
  __webpack_modules__["./index.js"](); // 通过路径和文件名标记
})();

```

当然大家都知道 webpack 可以配置`devtool`属性，根据配置的不同，打包产物也会有所不同。

但是无论哪种模式，webpack 的打包方式都是相同的。**那从我们的代码 -> 产物发生了什么变化呢？** 简单来看就是两点变化。

1. 模块被打包到了一个文件中，并且放到了一个`IIFE`函数中
2. 原来的`module.export`、`require`没有了，被 webpack 重写了，或者说被 webpack 重新实现了

具体是怎样的流程查看我的《webpack 编译流程》， 本篇主要说明编译流程中的最后一步，打包方式。

## 不同文件的打包方式

#### JS 打包



#### 文件打包



#### vue 打包



## 注意点

- webpack 打出来的包是基于 ES5 的，不支持 ES6 Module（Rollup 支持）
- 使用 webpack 打包时，ESM 和 CJS 可以混用，webpack 会把这种会用的方式统一转换成自己的 CJS 实现

## 附录

#### `eval`中的`//# sourceURL`

在 Chrome 的调试工具中有一个 soruce 面板，大家肯定都用过。source 面板左侧的 page 标签栏中有当前页面所有的资源文件，根据不同的来源地址分类。而`eval("\n\n//# sourceURL=webpack://webpack-demo/./index.js?")`作用就是把目标文件加载到 source panel 中。

效果如下

![image-20220125165038602](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220125165038602.png?x-oss-process=image/resize,w_800,m_lfit) 

加载进去后大家就可以断点和调试这个文件了，注意不忘要忘了`//# `。ps：安全性的高的网站都不允许这样的注入方式哈。

## 参考文章

做了一夜动画，让大家十分钟搞懂Webpack：https://juejin.cn/post/6961961165656326152

