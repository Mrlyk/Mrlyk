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

