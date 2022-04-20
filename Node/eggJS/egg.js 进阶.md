# egg.js 进阶

[toc]

## 插件机制

作为服务端的框架，虽然基于`koa`也有中间件，但是会存在一些问题。

插件提供了一套更强大的机制来管理独立的业务逻辑

#### 使用

## cookie 与 session

官方文档：https://www.eggjs.org/zh-CN/core/cookie-and-session

#### cookie

通过 `ctx.cookies`，我们可以在 controller 中便捷、安全的设置和读取 Cookie。

```js
class HomeController extends Controller {
  async add() {
    const ctx = this.ctx;
    let count = ctx.cookies.get('count'); // 获取 cookie
    count = count ? Number(count) : 0;
    ctx.cookies.set('count', ++count); // 设置 cookie，返回的时候会被放入响应头中，支持第三个参数
    ctx.body = count;
  }
}
```

cookie 的参数可以参考官方文档，常见的如下

- maxAge: { Number } 过期时间，单位毫秒
- path: { String } 所在的路径
- domain: { String } 

如果想要 Cookie 在浏览器端不能被修改，不能看到明文：

```js
ctx.cookies.set(key, value, {
  httpOnly: true, // 默认就是 true
  encrypt: true, // 加密传输
});
```

*建议使用 base64 编码并 encode 后在写入* 

另外 egg 还提供加密方法，具体可以查看官方文档

#### session

使用 session 和 cookie 一样简单（Session 的实现是基于 Cookie 的）

```js
ctx.session = null; // 删除 session
```

需要 **特别注意** 的是：设置 session 属性时需要避免以下几种情况（会造成字段丢失，详见 [koa-session](https://github.com/koajs/session/blob/master/lib/session.js#L37-L47) 源码）

- 不要以 `_` 开头
- 不能为 `isNew`

由于在 egg 中 session 基于 cookie 实现，所以可能存在大小的问题

框架提供了将 Session 存储到除了 Cookie 之外的其他存储的扩展方案，我们只需要设置 `app.sessionStore` 即可将 Session 存储到指定的存储中。

```js
// app.js
module.exports = (app) => {
  app.sessionStore = {
    // support promise / async
    async get(key) {
      // return value;
    },
    async set(key, value, maxAge) {
      // set key to store
    },
    async destroy(key) {
      // destroy key
    },
  };
};
```

*官方不推荐使用外部存储，认为 session 也需要是精简的信息，不应该放在外部，容易丢失* 

**实现用户登录时刷新 session 时间**

框架提供了一个 `renew` 配置项用于实现此功能，它会在发现当用户 Session 的有效期仅剩下最大有效期一半的时候，重置 Session 的有效期。

```js
// config/config.default.js
module.exports = {
  session: {
    renew: true,
  },
};
```

## 连接数据库

以 mongodb 为例

egg-mongoose: https://github.com/eggjs/egg-mongoose

要连接数据库分为三步

1. 安装对应的插件，这里为`egg-mongoose`
2. 配置插件和连接信息

## 常见问题

#### web 安全 csrf

egg 内置 `egg-security` 插件可以帮我们处理常见的安全问题

通常情况下我们需要携带 token 来经过这道防火墙，在测试阶段可以暂时关闭或者放行部分请求地址

```js
config.security = {
    csrf: {
      ignore: '/api/', // 忽略 /api/ 这个地址的请求，支持正则
      // enable: false // 直接关闭（不推荐）
    },
  };
```

#### cors 跨域