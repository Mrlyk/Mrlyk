# electron-vue

使用 electron 结合 vue 开发桌面端应用

## 创建项目

一般我们不用手动配置项目模版，可以使用 `vue-cli-plugin-electron-builder` + `vue cli`帮助我们创建

1. `vue create xxx`创建 vue 项目
2. **安装 vue-cli-plugin-electron-builder 插件**：`vue add electron-builder`
3. 正常编写交互页面
4. 涉及到 node 操作的交给 preload 处理，通过 icpRender 和 main 进程通信

## 踩坑

#### preload 不加载

一定需要在 vue.config.js 中声明

```js
module.exports = {
  // ...
  pluginOptions: {
    electronBuilder: {
      preload: './src/preload.js'
    }
  }
  // ...
}
```

