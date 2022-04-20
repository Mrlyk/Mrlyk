# 配置 vue 框架的插件开发模版

开发浏览器插件过程中，如果使用原生的 js 去写，无疑效率会非常的低。我们可以借助于前端框架来提升开发效率。只要打包产物符合插件的要求即可。同时需要关注热更新的实现。

这里是用熟悉的 webpack 来处理打包和热更新，后面可能考虑 vit

1. 搭建一个 vue 标准模版项目





## 考虑浏览器插件需要的项目内容

#### 必须的内容

- manifest.json - 清单文件
- manifest.json 中必须的配置  manifest_version、name、version；推荐的配置 description、icons

## 模版项目结构

#### content

content-scripts
