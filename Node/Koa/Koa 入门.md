# koa

> Koa -- 基于 Node.js 平台的下一代 web 开发框架

## 安装

`npm i koa --save`

```js
// 基础示例
const Koa = require('koa')
const app = new Koa()

app.use(async (ctx, next) => {
  ctx.response.status = 200
  ctx.response.body = 'hi, koa'
  await next()
})

app.listen(3000)
```

## 中间件

中间件是 Koa 一个重要的概念，它是一个执行的链条，整个链条组成了一个运行的周期

```
const koa = require('koa2')
const app = new koa()

// 访问权限
app.use(async (ctx, next) => {
  console.log('权限验证通过...')
  await next() // 执行下一个中间件
})

// 日志记录
app.use(async (ctx, next) => {
  console.log('日志记录完成...')
  await next() // 执行下一个中间件
})

// 响应处理
app.use(async (ctx, next) => {
  ctx.response.status = 200
  ctx.response.body = 'hi, koa'
  await next()
})

app.listen(3000)
```

每一个中间件负责特定的小模块，互相配合，组合成一条完整的业务通道

### ctx 

ctx 是上下文参数，是中间件之间的全局变量。

ctx 包含了 HTTP 的请求和响应处理。

**洋葱模型**

![image-20211213151830503](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211213151830503.png)

请求遵循如上的处理方式，就像穿过一个洋葱一样。先进入外层，到达最中间的函数处理完，然后再从内向外。

#### koa-compose

Koa 中间件模型使用模块 [koa-compose](https://github.com/koajs/compose) 实现，第一次读都会被其优雅的设计折服，模块没有任何依赖，算上空行、异常兼容、注释只有 [48 行代码](https://github.com/koajs/compose/blob/master/index.js)，移除异常处理后代码相当简单

```js
function compose(middleware) {
  return function (context) {
    // 从第一个中间件开始调用
    return dispatch(0);

    /**
     * 调用指定 index 的中间件，为其传入 next 参数为下一个中间件的 dispatch
     * @param {Number} i 中间件 index
     * @return {Promise} resolve 后意味着上一个中间件 next() 后的代码可以继续执行
     */
    function dispatch(i) {
      // 当前中间件函数
      let fn = middleware[i];

      // 中间件都被调用后
      if (i === middleware.length) {
        return Promise.resolve();
      }

      try {
        // 调用当前中间件，next 参数设置为下一个中间件的 dispatch
        // 程序执行到 await next() 时进入下一个中间件调用
        const ret = fn(context, dispatch.bind(null, i + 1));

        // 将本次调用结果返回给上一个中间件，也就是 await next()
        return Promise.resolve(ret);
      } catch (ex) {
        return Promise.reject(ex);
      }
    }
  }
}
```

这样就实现了一个洋葱模型，秒、太妙了！！！！

## Koa HTTP 模块

### Koa Request

请求处理。

```
// header
ctx.request.headers  
ctx.request.protocol
ctx.request.type
ctx.request.charset

// method
ctx.request.method
ctx.request.query // get
ctx.request.body // post | 依赖 koa-bodyparse 第三方模块，后面章节有描述

// path
ctx.request.url // path/?get=
ctx.request.path // path

// host
ctx.request.host // hostname:port
ctx.request.hostname // hostname
ctx.request.ip
crx.request.subdomains 

// cookie
ctx.cookies.get('name') // 获取 cookie
ctx.cookies.set(name, value, { // 设置 cookie
  'expires': new Date() // 时间
  'path' : '/' // 路径
  'domain': '0.0.0.0' // 域
  'httpOnly': false // 禁止js获取
})

// error
ctx.throw(404, 'Not found')
```

### Koa Response

响应处理。

```
// header
ctx.set({})

// status
ctx.response.status = 200

// type
ctx.response.type = 'text/html; charset=utf-8' // defaule

// redirect
ctx.response.redirect(url)
```

## Koa Router

如果使用原生的方式来控制路由，需要使用下面这种繁琐的方式

```js
const koa = require('koa2')
const app = new koa()

app.use(async (ctx, next) => {
    if (ctx.request.path === '/') { // 首页
      ctx.response.status = 200
      ctx.response.body = 'index'
    } else if (ctx.request.path === '/list') { // 列表页
      ctx.response.status = 200
      ctx.response.body = 'list'
    } else {
    	ctx.throw(404, 'Not found') // 404
    }
  await next()
})

app.listen(3000)
```

这样通过请求的 path 来区分路由。

一般我们的做法是使用封装好的 koa-router

### koa-router

**安装**

`npm i koa-router --save`

## 其他

官方介绍了三个学习 koa 的网站

- [Kick-Off-Koa](https://github.com/koajs/kick-off-koa) - An intro to Koa via a set of self-guided workshops.
- [Workshop](https://github.com/koajs/workshop) - A workshop to learn the basics of Koa, Express' spiritual successor.
- [Introduction Screencast](https://knowthen.com/episode-3-koajs-quickstart-guide/) - An introduction to installing and getting started with Koa

只要了解了 Node.js 的 http 模块和 koa 的洋葱模型中间件，koa 非常好理解，但实现一个完整的 web 网站需要整合大量的中间件（ koa 核心维护者[死马](https://github.com/dead-horse)整理了一些常用的 https://www.npmjs.com/package/koa-middlewares）

```js
var koa = require('koa');
var middlewares = require('koa-middlewares');
var router = middlewares.router();

var app = koa();

router.get('/', function *(){
  this.body = 'hello koa-middlewares';
});

app.use(middlewares.bodyParser());
app.use(middlewares.conditional());
app.use(middlewares.etag());
app.use(middlewares.compress());
middlewares.csrf(app);
app.use(router.routes());
app.use(router.allowedMethods());

app.listen(7001);
```

