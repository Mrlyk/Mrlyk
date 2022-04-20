# ndb 调试 node 应用

> ndb is an improved debugging experience for Node.js, enabled by Chrome DevTools 。ndb 是一次对 node 调试体验的升级，通过原生的 CDP 协议提供支持（和 puppeteer 相似，并且它依赖于 puppeteer）

官方仓库：https://github.com/GoogleChromeLabs/ndb

## 安装

环境要求：node > 6

```shell
npm i ndb -g
```

## 使用

#### 调试 js 程序

```shell
ndb app.js
```

#### 调试可执行的命令

比如有些 npm 脚本，如`webpack`命令，他们最终也是执行的 js 方法，也可以通过`ndb`来调试

```shell
ndb npm run dev
```



## 参考文章

使用 ndb 调试 node 应用：https://zhuanlan.zhihu.com/p/45851471