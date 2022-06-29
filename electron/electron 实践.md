# electron 实践

本文记录一些 electron 的实践经验



## API 相关

#### render 进程 electron 引入报错

在 render 进程中直接引入 electron 会报错

![image-20220614145231847](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220614145231847.png?x-oss-process=image/resize,w_600,m_lfit) 

这是因为 electron 本来就是 node 环境下的包，在渲染进程（可以看作浏览器环境）是没有 fs 这些 node 模块的。

所以我们需要做两件事：

1. 对产物而言：在打包的配置中开启`nodeIntegration: true`集成 node 运行环境
2. 对开发而言：如果想要在渲染进程中使用 electron 的部分模块，需要将模块暴露到 window 对象上：
   - 通过 preload 模块，将需要使用的模块暴露`contextBridge.exposeInMainWorld`。然后再在 window 上使用即可`window.electron.ipcRenderer.send("test")`



## 界面相关

界面展示的一些配置

#### 自定义顶部 title

首先需要如下配置来去掉默认的标题栏，去掉之后整体展示的就是自己的 render。

```js
mainWindow = new BrowserWindow({
  useContentSize: true,
  frame: false  // 去掉默认的标题栏
})
```

之后自己画整体页面即可，记住实现常规的

- X - 退出程序
- — - 最小化程序
- [] - 最大化程序