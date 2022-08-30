# electron 实践

本文记录一些 electron 的实践经验

[toc]

## 界面相关

界面展示的一些配置

#### 自定义顶部 title

首先需要如下配置来去掉默认的标题栏，去掉之后整体展示的就是自己的 render。

```js
mainWindow = new BrowserWindow({
  useContentSize: true,
  frame: false  // 创建一个无边框窗口，去掉默认的标题栏
})
```

之后自己画整体页面即可，记住实现常规的

- X - 退出程序
- — - 最小化程序
- [] - 最大化程序

如果要保留常规的按钮，可以只隐藏默认的顶部但是保留按钮

```js
mainWindow = new BrowserWindow({
  useContentSize: true,
  titleBarStyle: 'hidden' // 隐藏顶部，保留按钮
})
```

#### 图标设置

图标分为两类

- 一类是展示在标题栏的图标，也就是界面上的图标
- 一类是`.exe`可执行文件的图标

展示在标题栏的图标还是通过`BrowserWindow`来配置，而可执行文件的图标则需要 通过打包配置，具体见打包说明。

下面是标题栏图标的配置

```js
mainWindow = new BrowserWindow({
  icon: '../../dist/icons/icon.jpg'
})
```

*经实验发现未生效，原因待探究*

## 踩坑

#### render 进程 electron 引入报错

在 render 进程中直接引入 electron 会报错

![image-20220614145231847](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220614145231847.png?x-oss-process=image/resize,w_600,m_lfit) 

这是因为 electron 本来就是 node 环境下的包，在渲染进程（可以看作浏览器环境）是没有 fs 这些 node 模块的。

所以我们需要做两件事：

1. 对产物而言：在打包的配置中开启`nodeIntegration: true`集成 node 运行环境
2. 对开发而言：如果想要在渲染进程中使用 electron 的部分模块，需要将模块暴露到 window 对象上：
   - 通过 preload 模块，将需要使用的模块暴露`contextBridge.exposeInMainWorld`。然后再在 window 上使用即可`window.electron.ipcRenderer.send("test")`

#### 打包后页面打开空白

打包后页面打开空白的原因有很多种，这里列举我碰到的。**一般可以使用 `cmd + alt + i` 打开 devtools 查看**。有时候比较卡需要多按几次快捷键。

##### 1、preload 没加载

在`new BrowserWindow`的时候声明了`webPreferences.preload`

路径要用绝对路径，如果用相对路径可能不会报错，但是也不会加载成功

如果使用了 vue-cli-plugin-electron-builder 插件还需要在 vue.config.js 中声明 preload 的路径，详见 《electron-vue》。