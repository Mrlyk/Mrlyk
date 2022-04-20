# babel

>JS 编译器，将ECMAScript 2015+ 语法编写的代码向后兼容

[toc]

## 安装

```shell
npm i @babel/core @babel/preset-env -D
```

- @babel/core： babel  的核心
- @babel/preset：预设的配置

其他一些可安装的依赖

```shell
npm i babel-loader core-js -D
```

- babel-loader：babel 文件的预处理工具
- core-js：一个标准 js 库，实现了所有高级语法的 polyfill，在 babel-polyfill 中继承了 ^2.0.0 版本。在 @babel/core@^7.0.0 以上版本，如果要使用更新的，支持更全面的 core-js@3.0 则需要手动安装 core-js 并在 @babel-preset-env 的选项中配置`corejs: 3`。

*babel@^7.0.0 以上已经不再内置 @babel/polyfill，而是在 @babel/preset-env 中内置了 core-js@^2.0.0*

## 配置项

对 babel 对配置项做说明

#### presets 预设选项

在 babel 中我们可以手动一项项配置各个编译选项，也可以使用别人已经预设的配置。两者之间会有一个 merge 操作（待确认）

**配置方法** 

```js
module.exports = {
  presets: [ // 接收一个数组
    '@vue/app', // 配置一个预设包
    ['@babel/preset-env',  // 配置一个预设包并设置 option
     {
       corejs: '3',
       useBuiltIns: 'usage'
     }
    ]
  ]
}
```

**presets 的加载规则** 

- `@vue/app`：这种会在 app 前加上 `babel-preset-`的前缀，最终引入的预设包路径是 `babel-preset-app` 
- `@vue/cli-plugin-babel/preset`：有两个`/`时则直接使用配置的路径去加载
- `app`：`babel-preset-`前缀是加在真正的包名前，不是在私有包名（`@vue`）前加，这里就是`babel-preset-app` 

源码待查看！！！

#### plugins

插件

**plugins 的加载规则** 

- @babel/proposal-optional-chaining：会在包名前加`plugin`前缀，最终引入的是`@babel/plugin-proposal-optional-chaining`。如果自已经写了则不会再加

源码待查看！！！

## 其他

#### 为什么使用 babel 指定兼容 IE11 在 IE11 中依然不能运行？

webpack 打包后为了代码作用域不受影响，将**所有打包后的放在了一个箭头函数中。**而 IE11 是不支持箭头函数的所以依然会报错。所以这里要注意几个问题：

- **babel 只会对我们写的代码进行降级，对 webpack 工具自身的方法不会进行处理。**正如上面所说的打包后的代码被放入一个自执行的箭头函数中是 webpack 自身的行为，而不是我们手写的代码。（要处理这种问题要对 webpack 自身打包进行配置，配置 output 文件的 environment 让他不要使用 arrowFunction 即可）
- 如果方法在浏览器中使用 polyfill 都无法实现，比如一些 api 老的浏览器可能根本就不提供这种功能，使用 babel 降级也是没用的

#### babel 与 npm link

 npm link 产生的是软连接，webpack 在编译时如果配置了 exclude: /node_modules/ 。由于 webpack 打包默认的规则是基于真实文件的地址，所以 npm link 的文件不会被排除，而是被额外的预编译。

webpack 提供了 resolve.symlinks = [Boolean] 进行控制。默认是 true 基于真实路径，改成 false 可以解决上面的问题。

#### babel 与 tsconfig 中的 target

TS 代码编译有三种方式

- TS 自带的 `tsc`编译方法，为了 targert 兼容老版本，内部引入了core-js@^2.0.0 版本来实现 polyfill。缺点是不会按需加载 polyfill 方法，导致编译较慢且文件体积较大。
- 配合 webpack 的 ts-loader，内部引入了` babel-polypill`。也相当于配置了 core-js@^2.0.0 版本
- 配合 webpack 的 babel 以及 @babel/plugin-transform-typescript 也可以预处理 ts 代码。同时配置 @babel/preset-env 的 options 还可以指定 core-js 的版本为 3.0 及以上，以兼容更新的 ES2021 以上语法

所以无论使用哪种方法都能对代码中的新预发实现降级，不过 babel 可指定 core-js 版本使得他能兼容的语法更新。