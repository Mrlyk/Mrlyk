# rollup 配置说明

[toc]

## input 入口

类似于 webpack 的 entry

```js
// rollup.config.js
const config = {
  input: 'input.js',
}
```

## output 输出

#### output.format

有时候我们希望用户能通过 script 标签引入我们的包后就能使用，这就需要用到 IIFE。最终打包出的产物需要是如下形式。

```js
var Vue = (function () {
	// dosth
})()
```

通过这个配置就可以实现

```js
// rollup.config.js
const config = {
  output: {
    format: 'iife' // 也可以是 esm、cjs 等
  }
}
```

最终打包的内容会被包裹在一个 IIFE 函数中，一般如下：

```js
(function () {
  'use strict';
	// dosth
})();

```

