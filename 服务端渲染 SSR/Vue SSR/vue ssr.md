# Vue SSR

本文将简单记录 Vue SSR 的构建基本原理和开发指南

[toc]

## 基本原理

vue ssr 的基本原理并不不复杂。实际上是在 webpack 构建服务时，两个不同的入口（client 和 server 各一个）构建出两份不同的包 `client-bundle.js`和`server-bundle.js`。

其中`server-bundle.js`用来生成首次访问的 html 字符串并返回给客户端，到了客户端之后实际挂载执行的是`client-bundle.js`。这个流程在客户端叫"Hydrate"（融合）。

![image-20220423112709014](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220423112709014.png?x-oss-process=image/resize,w_1000,m_lfit)  

**融合？** 

SSR 如何融合呢？**实际上 SSR 渲染最后生成的 html 字符串上，通过 `script`引用的静态产物和降级使用 CSR 渲染的是相同的（client-bundle.js）**。所以最后才会又回到传统的 CSR 模式（即 SPA 应用）。只不过首屏页面的整个 DOM 结构已经渲染好了，和传统的 CSR 相比多的也就是首屏的 DOM。

只不过每次刷新第一次执行的总是`server-bundle.js`。

## vue2 SSR

vue2 和 vue3 SSR 有些许差别，这里先讲解 vue2 SSR 的简单实现，稍后在讲 vue3 中的。

vue2 SSR 依赖于两个东西

- `vue-server-renderer`(注意是 `renderer`)
- node 服务

node 服务是我们自己实现的，先不去提，我们先看看 vue-server-renderer

#### vue-server-renderer

**作用：**vue-server-render 的作用是**将传入的 vue 实例渲染成 html 字符串** 

```js
#! /usr/bin/env node
'use strict'
const Express = require('express')
const Vue = require('vue')
// 引入渲染器
const renderer = require('vue-server-renderer').createRenderer()

const page = new Vue({
  template: `<div>Hello World</div>`,
})

const app = Express()

app.get('/', async (req, res) => {
  let html = ''
  try {
    // 注意是异步方法
    html = await renderer.renderToString(page) // 将 vue 实例渲染成 html 字符串
  } catch (error) {
    res.status(500).send('服务器内部错误')
  }
  res.send(html)
})

app.listen(3010, () => {
  console.log('server is running at http://localhost:3010')
})
```

这样一个最简单的 vue ssr 页面就完成了。接下来我们从这个简单的使用来稍微深入一点探究一下`vue-server-render`的原理。

**`vue-server-render`原理** 

要实现 SSR （同构），就需要有两份打包产物，一份用于 SSR，一份用于 CSR。在`vue-server-renderer`中实现了两个个 webpack 插件，会分别使用不同的 webpack 配置打包出两个产物

- vue-ssr-server-bundle.json
- vue-ssr-client-manifest.json

我们使用在上面实例中将 vue 实例转换为 html 模版使用了`createRenderer` ，还有另一个方法`createBundleRenderer`

- createRenderer: 接收一个 Vue实例，输出 html 字符串
- createBundleRenderer: 接收 webpack 打包的产物，输出 HTML（就是原来的静态打包）

#### 路由设计

SSR 渲染需要考虑路由设计，及把 node 服务的路由映射到前端路由。

简单来说就是下面这样

```js
const getRoutePage = ({path}) => {
  // todo: 获取到前端路由对应的组件
}

app.get('/page/news', async (req, res) => {
  let html = ''
  try {
    const page = getRoutePage({path: '/page/news'})
    // 注意是异步方法
    html = await renderer.renderToString(page)
  } catch (error) {
    res.status(500).send('服务器内部错误')
  }
  res.send(html)
})
```

#### vue-server-renderer/client-plugin

这个 plugin 的作用是生成一个名为 **vue-ssr-client-manifest.json **的文件。这个文件将在我们做服务端渲染的时候用到。

**vue-ssr-client-manifest.json**

其一般文件结构如下

![image-20220802170719973](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220802170719973.png?x-oss-process=image/resize,w_600,m_lfit) 

- publicPath：访问静态资源的根相对路径，与webpack配置中的publicPath一致
- all：打包后的所有静态资源文件路径
- initial：页面初始化时需要加载的文件，会在页面加载时配置到 preload 中
- async：页面跳转时需要加载的文件，会在页面加载时配置到 prefetch 中
- modules：项目的各个模块包含的文件的序号，对应all中文件的顺序

所以，我们能够通过vue-ssr-client-manifest.json做什么呢？**其最重要的作用就是我们能根据initial拿到客户端渲染的js代码。**

#### vue-server-renderer/server-plugin

这个插件会生成一个 json 文件：**vue-ssr-server-bundle.json**

其文件内容结构如下，主要存放的是资源的映射信息

![image-20220802170810512](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220802170810512.png?x-oss-process=image/resize,w_1000,m_lfit)

- entry 是服务端入口的文件
- files 是服务端依赖的文件列表
- maps 是sourcemaps文件列表。

下面的`.js` 就是根据请求生成 html 的 js 代码！

## 最佳实践

#### 异步请求应该放在哪里

我们知道，在常规的 Vue 前端渲染中，组件请求 Ajax 一般是在 mounted (created) 中调用 `this.fetchData`，然后在回调里面把返回数据写到实例的 data 中，这就 ok 了。
 在 SSR 中，这是不行的，因为服务器并不会执行 mounted 周期。

那么我们是否可以把 `this.fetchData`提前到 created 或者 beforeCreate 这两个生命周期中执行？

同样不行。原因是：`this.fetchData`    **是异步请求，请求发出去之后，没等数据返回呢，后端就已经渲染完了，无法把 Ajax 返回的数据也一并渲染出来。**

所以我们一般使用 ssr 应用特有的 asyncData 周期来请求数据

- 官方的推荐的做法之一是将数据都存到 vuex 然后再通过 vuex 去读取
- **nuxt 之类的或者自己编写的框架可以在 asyncData 的时候 return 回数据**，然后和组件合并。等到真正渲染的时候直接读取即可！

*注意：asyncData 是根据路由跳转的组件来获取执行的，所以**路由的组件的子组件不会执行 asyncData*** 

## 参考文章

vue-ssr在项目中的实践：https://zhuanlan.zhihu.com/p/95294219
