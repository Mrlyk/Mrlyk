# Rollup

> 新一代打包工具，快就一个字，工作流形式（node 流中有部分说明）

tree-shaking 的概念就源于 rollup。

下面是一个基础命令，表示执行 rollup 的打包动作，以 input.js 为入口，打包为 esm 格式，输出到当前目录的 bundle.js 中。

```shell
npx rollup input.js -f esm -o bundle.js
```

也可以使用 `-c`参数指定配置文件

```js
npx rollup -c rollup.config.js
```



## rollup 中消除副作用

我们知道 tree-shaking会受到副作用的影响，使用webpack打包时，我们可以使用 effect 属性显式的声明哪些模块不具有副作用。

在 rollup 中依赖一个特殊的注释来做到这一点——`/*#__PURE__*/`

因为 vue3 就是使用 rollup 进行打包的，所以如果查看 vue3 的源码会发现到都是这个注释，这一切都是为了能减少最后的打包体积。

## 参考文章

rollup - 构建原理及简易实现：https://juejin.cn/post/6971970604010307620#heading-19