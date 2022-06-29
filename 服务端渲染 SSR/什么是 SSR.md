# 什么是 SSR

[toc]

传统的前端 web 开发，其最终产物是交由客户端即浏览器来渲染（**客户端渲染：CSR - Client Side Render**）。用户打开网页后就是我们常说的“输入 url 之后发生了什么” 的步骤。一般流程如下

**CSR** 

```sequence
title:CSR
客户端->服务端:访问 url
服务端-->>客户端:返回 html、js、img 等
Note left of 客户端:渲染 DOM
客户端->服务端:请求接口
服务端-->>客户端:返回数据
Note left of 客户端: 根据数据回流、重绘 DOM
```

传统的标准渲染流程就是这样，但具有两个明显的缺点：

1. 首屏渲染时间长。因为先发送请求，再解析渲染，非常耗时
2. SEO 不够友好，爬虫基本都是**同步**爬取数据的，传统方式爬虫扫到了，接口却没请求完会导致爬不到数据

**由此 SSR 诞生了** 

**SSR** 

**SSR 又称服务端渲染（Server Side Render），也称同构**。传统情况下渲染交由客户端完成，**在 SSR 项目中，服务端会直接将首屏页面渲染成 HTML 字符串**然后返回，之后再转变成传统的 CSR 模式。但是用户每次刷新的首屏永远是服务端渲染完成的。

SSR 流程如下

```sequence
title:SSR
客户端->node服务: 首次访问或刷新页面（首屏请求）
note right of node服务: 读取 url 对应的静态资源
node服务->服务端: 请求首屏数据
note right of 服务端: 查询数据
服务端-->>node服务: 返回数据
note right of node服务: 解析数据生成 DOM
note right of node服务: 渲染成 html 字符串
node服务-->>客户端:首屏的 html
note left of 客户端: 渲染 html
note left of 客户端: 静态标部分激活 CSR，转为 CSR 模式
客户端-->服务端: 后续异步请求
服务端-->>客户端: 返回数据
note left of 客户端: CSR 渲染
```

## 主要差异

#### API

- cookie 操作方式
- 路由跳转
- 任何 BOM 方法

## 缺点

1. 没有 BOM 等浏览器 API，要特别注意
2. 第三方库不一定能在服务端用
3. 注意异常问题，可能直接导致服务崩溃，要做好降级方案
4. 需要启动一个额外的 node 服务，增加了服务器负载
