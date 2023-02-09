# 一些需要了解的 npm 包

> 很多工具可能会用到一些我们自己不常用，但是很重要的包，需要了解一下它的作用，这里记录一下

[toc]

#### core-js-compat（兼容）

npm：https://www.npmjs.com/package/core-js-compat

用于查询和获取兼容目标浏览器所需要的 polyfill 列表

```js
const {
  list,                  // array of required modules
  targets,               // object with targets for each module
} = require('core-js-compat')({
  targets: '> 2.5%',     // 最低兼容的浏览器版本
  filter: /^(es|web)\./, // optional filter - string-prefix, regexp or list of modules
  version: '3.20',       // 使用的 core-js 版本，
});

// targets 最终输出 { 'es.promise.finally', 'es.promise.all-settled' } 等
```

#### depcheck

通过引用关系排查未使用的包。可以直接使用如下命令，不用安装。

```shell
npx depcheck
```

