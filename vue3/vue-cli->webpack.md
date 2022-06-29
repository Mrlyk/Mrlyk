# vue-cli -> webpack

记录项目从 vue-cli 迁移到 webpack 的踩坑过程

基本步骤如下：

1. 安装 webpack webpack-cli
2. 卸载原来的 @vue/cli 所有配置和插件
3. 添加 webpack.config.js 配置文件
4. 配置 vue-cli 原自带的插件
5. 迁移 vue.config.js 配置文件
6. 修改 package.json 中的 script 脚本

第一步没什么好说的，直接从第二步开始

## webpack 配置

可以先在原来的项目上使用`vue inspect > output.js`把原配置输出出来

#### 必须的配置

这些配置都可以参照 output.js 输出的原配置来配

- entry 入口文件
- output 输出
- alias 别名
- extensions 扩展名
- module.rules 模块解析的 loader 配置
- devServer 开发服务

#### 其他配置

- `babel-plugi-dynamic-import-node`: webpack 打包出来是使用 require 加载模块，不支持异步加载，比如异步路由等。使用该插件可以将 require 转换成`Promise.then(xxx => require('xxx'))`的形式，以支持异步。可以**提高 vue 热更新速度**，否则每次更新，整个 chunk 都会被更新。**但要注意如果使用了魔法注释 /* webpackChunkName: "xxxx" */，两者之间会有冲突** 

## 渲染 vue 必须的插件

注意 vue2 和 vue3 由于边跟比较大，插件也要适当调整版本

- vue-loader: 处理 SFC 的 vue 文件

  配置参考：https://vue-loader.vuejs.org/zh/options.html#compiler 

  **一般我们需要通过配置 vue-loader 的 options 来配置 vue-template-loader**，比如去除 template html 标签之间的空格`preserveWhitespace: false`（默认 true）

- vue-template-loader: 编译 vue 模版，配合 vue-loader 使用

- `const VueLoaderPlugin = require('vue-loader/lib/plugin');`这个 webpack 插件是必须的，将 vue SFC 中 js、css 添加参数，分配给正确的 loader 处理

## 踩坑

#### 框架样式丢失

**原因**

vue-loader-plugin 没配置，导致 SFC 中的样式未分离出来。`const VueLoaderPlugin = require('vue-loader/lib/plugin')`

**处理方式** 

1. 配置 vue-loader-plugin
1. 引入加上具体的 `index.css` path
2. vue-style-loader  可以直接用 style-loader 替换，vue-style-loader 实际上就是 style-loader

#### babel 需要重新配置

卸载原来的`@vue/cli-plugin-babel`，手动安装`@babel/preset-env`

#### 图片目录结构不对 

**原因** 

`url-loader` 的 fallback 不支持对象，只能设置字符串。

**处理方式** 

fallback 的 loader 如果要配置参数，直接在`url-loader`中配置即可，`url-loader`会自动读取自己的参数传入 fallback

```js
// 错误的方式，不支持
{
  loader: 'url-loader',
  options: {
    limit: 4096,
    fallback: {
      loader: 'file-loader',
      options: {
        name: 'static/fonts/[name].[fullhash:8].[ext]'
      }
    }
  }
}

// 正确的方式
{
  loader: 'url-loader',
  options: {
    limit: 8192,
    name: 'static/fonts/[name].[contenthash:8].[ext]', // fallback 参数直接写在这里
    fallback: 'file-loader'
  }
}
```

#### 图片无法加载，路径 [object]

**原因**

file-loader 默认处理资源文件最后输出使用的是 ES Module 规范，但是 vue-loader 会把 template 中的路径转换成 CommonJS 的格式，导致无法在 template 中无法加载，返回一个空的 module 对象

**处理方式**

修改 file-loader 配置，将 ES 关闭

```js
{
  {
    test: /\.(png|jpe?g|gif|webp)$/,
      use: [
        {
          loader: 'url-loader',
          options: {
            limit: 8192,
            name: 'static/img/[name].[contenthash:8].[ext]',
            esModule: false, // 手动关闭该项
            fallback: 'file-loader'
          }
        }
      ]
  }
}
```

#### 其他问题

1. babel es6.promise...... `@babel/preset-env`版本问题
2. vue-style-loader 其实就是 style-loader（fork 过来的）。作为 vue-loader 的依赖，一般我们不需要单独安装，切换到 webpack 之后可以手动安装 style-loader 来处理 css 文件
3. **core-js 和 babel 中的版本不同导致的解析问题**。一般手动配置了`resolve.modules`才有这个问题。可以通过配置`alias`来映射解决