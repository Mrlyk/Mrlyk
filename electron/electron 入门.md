# electron 入门

> Electron是一个使用 JavaScript、HTML 和 CSS 构建桌面应用程序的框架。 嵌入 [Chromium](https://www.chromium.org/) 和 [Node.js](https://nodejs.org/) 到 二进制的 Electron 允许您保持一个 JavaScript 代码代码库并创建 在Windows上运行的跨平台应用 macOS和Linux——不需要本地开发 经验。

electron 主要分为两类进程，主进程和渲染进程，主进程通过 `BrowserWIndow`模块来创建不同的窗口，不同的窗口有不同的渲染进程。和浏览器扩展有一定的相似之处，主进程的权限更高，但是无法和 UI 交互，渲染进程则是渲染 UI 的。我们还可以使用 vue、react 这样的框架来编写 UI。

他们之间**通过 IPC 通信**来处理各自职责无法做到的事情！通信底层由 electron 封装好了，我们只需要像 h5 开发一样监听各类事件即可！

[toc]

## 创建项目

和一般的 node 项目相同，安装 `electron`即可。

最基础的项目代码如下：

```js
// index.js
const { app, BrowserWindow } = require('electron')

const createWindow = () => {
  const win = new BrowserWindow({ // 创建窗口
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}

// 生命周期
app.whenReady().then(createWindow)
```

- [`app`](https://www.electronjs.org/zh/docs/latest/api/app) 模块，控制应用程序的事件生命周期
- [`BrowserWindow`](https://www.electronjs.org/zh/docs/latest/api/browser-window) 模块，创建和管理应用程序 窗口

之后使用`electron .`命令运行即可，默认查找 package.json 中 `main`指定的文件并执行

#### app 管理生命周期

**打开程序**

```js
app.whenReady().then(() => {
  createWindow()
 
  // 点击 x 只是放到后台，不是关闭。从后台打开会触发该事件
  app.on('activate', () => { // app 的 actvive 事件
    if (BrowserWindow.getAllWindows().length === 0) createWindow() // 如果没有窗口打开则创建一个（从台打开的情况下）
  })
})
```



**退出程序**

```js
// 监听所有的窗口关闭事件
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit() // 手动退出程序（mac 一般自己右键才退出）
})
```

## 从渲染器访问 DOM

**我们不能直接在主进程中编辑DOM，因为它无法访问渲染器 `document` 上下文。 它们存在于完全不同的进程！**

参见 electrom 进程模型：https://www.electronjs.org/zh/docs/latest/tutorial/process-model

如果我们要访问`window`和`document`，需要 **预加载脚本**！

```js
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js') // 写入预加载脚本的地址
    }
  })

  win.loadFile('index.html')
}

/*============preload.js=========*/
// 在预加载脚本中访问 window 对象，操作 DOM
window.addEventListener('DOMContentLoaded', () => {
  console.log('preload.js')
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    console.log('preload.js', element)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

## 打包

#### @electron-forge/cli

使用官方提供的`@electron-forge/cli` 工具进行打包。cli 工具还有其他功能，打包主要依赖于 [electron-packager](https://github.com/electron/electron-packager)

```shell
npm install --save-dev @electron-forge/cli
npx electron-forge import # 自动安装打包需要的依赖和写入打包命令
```

之后执行命令打包即可

```json
"scripts": {
  "package": "electron-forge package", // 打包，用于调试，未经过压缩，提及较大
  "make": "electron-forge make" // 打包，用于分发，代码经过一定压缩，自动生成 zip 包
},
```

**打包平台**

两个命令都可以使用`--arch(-a)` 和`--platform(-p)`参数来指定系统结构和平台

```shell
npx electron-forge package --arch x64 --platform darwin
```

但是需要注意的是**只能打包你当前机器的平台包**，比如你用OSX是无法打出windows平台安装包的。

#### eslint-builder

在 package.json 中使用 build 配置

```json
"build": {
  "appId": "com.example.app"
}
```

或者在根目录创建``electron-builder.[yml|js|json|toml]`文件

也可以在打包时使用`--config`手动指定

**构建命令**

- build for macOS, Windows and Linux

```
electron-builder -mwl
```

- build NSIS 32-bit installer for Windows

```
electron-builder --windows nsis:ia32
```

**打包平台** 

```shell
electron-builder --win --x64 # win系统64
electron-builder --win --ia32 # win32
electron-builder --win --armv7l # win arm

electron-builder  --darwin --x64 # mac系统
```

#### eslint-builder vs @electron-forge/cli

electron-builder 与 electron-packager 相比各有优劣

- electron-builder 配置项较多，更加灵活，**打包体积相对较小**，同时上手难度大
- electron-packger 配置简单易上手，自动处理了很多事比如打包平台，打包命令，但是**打出来的应用程序包体积相对较大**

## 上下文隔离 contextIsolation *

上下文隔离功能开启后可以隔离我们的渲染进程和 preload 的上下文对象，保证安全性。因为 preload 具有很高的权限，可以直接访问主进程。

如果在预加载脚本中设置 `window.hello = 'wave'` 并且启用了上下文隔离，当网站尝试访问`window.hello`对象时将返回 undefined。

自 Electron 12 以来，默认情况下已启用上下文隔离，并且它是 *所有应用程序*推荐的安全设置。开启之后才可以在 preload 中使用`contextBridge`对象

**配置**

```js
new BrowserWindow({
  name: 'app',
  width: 800,
  height: 600,
  webPreferences: {
    preload: path.join(__dirname, './preload.js'),
    contextIsolation: true
  }
})
```



## 通信*

由于渲染进程和主进程权限不同，两边经常需要通信来请求对方帮自己完成任务。通信依赖于 electron 提供的 IPC 模块

官方文档：https://www.electronjs.org/zh/docs/latest/api/ipc-renderer

- 在渲染进程中，使用`ipcRenderer`模块
- 在主进程中，使用`ipcMain`模块

#### 渲染进程 -> 主进程 （单向）

```js
/*==============preload.js || renderer.js==========*/ 
const { ipcRenderer } = require('electron')
// 发送异步消息
btns[0].addEventListener('click', () => {
  // 下面是异步消息
  // 也可以使用 sendSync 发送同步消息 const result = ipcRenderer.sendSync() ，主进程使用 ev.returnValue 返回
  ipcRenderer.send('render-evnet', '这是一条来自于异步的消息')
})
// 监听消息
ipcRenderer.on('render-evnet-rv', (ev, data) => {
  console.info(data)
})
/*===============主进程==========*/ 
const { ipcMain } = require('electron')
ipcMain.on('render-evnet', (ev, data) => {
  console.info(data)
  // 发送消息给渲染进程
  ev.sender.send('render-evnet-rv', '这是一条来自主进程的反馈消息')
})
```

#### 渲染进程 -> 主进程 （双向）

```js
// Main
const { ipcMain } = require('electron')
ipcMain.handle('read-file', async (event, path) => {
  if (!pathIsOK(path)) throw new Error('forbidden')
  const buf = await fs.promises.readFile(path)
  return buf
})

// Renderer
const { ipcRenderer } = require('electron')
const data = await ipcRenderer.invoke('read-file', '/path/to/file')
// ... do something with data ...
```



#### 主进程 -> 渲染进程

主进程没有直接给渲染进程发送消息的方法，但是可以借助`BrowserWindow`对象，使用`webContents.send`方法。

渲染进程依然使用`icpRenderer`接收。

```js
/*===============主进程==========*/ 
BrowserWindow.getFocusedWindow().webContents.send(
    'mtp',
    '主进程发送消息给渲染进程'
)
/*==============渲染进程==========*/ 
ipcRenderer.on('mtp', (ev, data) => {
    console.info(data)
})
```

#### 渲染进程间的通信

和 vue 兄弟组件传值的思想相同，子传父，父传子。利用上面两个方法即可

```js
/*==============发起消息的渲染进程==========*/ 
ipcRenderer.send('mti', '这是条来自于 modal 的消息')

/*===============主进程==========*/ 
ipcMain.on('mti', (ev, data) => {
    // 通过 id 获取到对应的渲染进程,然后消息传递
    BrowserWindow.fromId(mainId).webContents.send('mti2', data)
})

/*==============接收消息的渲染进程==========*/ 
ipcRenderer.on('mti2', (ev, data) => {
	console.info(data)
})

```

除此之外还有

- 使用 localStorage 来实现
- sendTo 通信

#### remote 模块(electron@^14 弃用，推荐 invoke)

remotr 模块提供了更为语义化的模块 remote。使用它就可以在渲染进程中**不显示的发送信息而调用 main 进程的方法**。

1. 在创建渲染进程的时候显示的声明集成 node，并关闭上下文隔离（高版本好像不需要了？）

```js
const win = new BrowserWindow({
  width: 800,
  height: 600,
  webPreferences: {
    nodeIntegration: process.env.ELECTRON_NODE_INTEGRATION, // 集成 node true
    contextIsolation: !process.env.ELECTRON_NODE_INTEGRATION // 上下文隔离 false
  }
})
```

2. 在渲染进程获取主进程

```js
/*======渲染进程===========*/
const remote = require('electron').remote; 
const BrowserWindow = remote.BrowserWindow; // 获取主进程的 BrowserWindow

var win = new BrowserWindow({ width: 800, height: 600 }); // 直接调用主进程的方法
win.loadURL('https://github.com');
```

**remote 的几个方法及属性**

- remote.getCurrentWindow()

返回 BrowserWindow 即此网页所属的窗口

- remote.getCurrentWebContents(）

返回 webContents 即此网页的 web 内容

- remote.getGlobal(name)

返回主进程中 name (例如 global[name]) 的全局变量。

- remote.process

主进程中的 process 对象。这与 remote.getGlobal('process') 相同, 但已被缓存。

***注意:** 反向操作（从主进程访问渲染进程），还可以可以使用 `webContents.executeJavascript`* 

## 通知

两种方式

1. 通过渲染进程通知
2. 通过主进程通知

**通过渲染进程通知**

就是我们写的  view 层的通知。如果用 vue 写直接写在 vue 组件中即可。

```js
// preload.js 
const NOTIFICATION_TITLE = 'Title'
const NOTIFICATION_BODY = 'Notification from the Renderer process. Click to log to console.'
const CLICK_MESSAGE = 'Notification clicked!'

new Notification(NOTIFICATION_TITLE, { body: NOTIFICATION_BODY })
  .onclick = () => document.getElementById("output").innerText = CLICK_MESSAGE
```

**主进程通知**

主进程的话只需要等进程启动完的任意时间后，直接调用通知接口即可。

```js
function showNotification () {
  new Notification({ title: NOTIFICATION_TITLE, body: NOTIFICATION_BODY }).show()
}
app.whenReady().then(createWindow).then(showNotification)
```

## 右键菜单

右键菜单只能由主进程创建

菜单的具体属性可见：https://www.electronjs.org/zh/docs/latest/api/menu-item

菜单的功能由`role`定义，比如复制（copy）electron 都已经根据 role 帮我们实现好了，不许要我们再手动处理。

```js
import { Menu, MenuItem } from 'electron'

const menu = new Menu()
menu.append(new MenuItem({
  label: '复制',
  role: 'copy'
}))
win.webContents.on('context-menu', (e, params) => { // 监听内容的右键菜单事件
  menu.popup({ window: win, x: params.x, y: params.y }) // 调用 popup 方法，弹出右键菜单
})
```

*放在 app.whenReady 之后调用*

## 应用菜单（顶部菜单）

顶部菜单由主进程创建

```js
const { app, Menu } = require('electron')

const isMac = process.platform === 'darwin' // mac 和 windows 有点区别

const template = [
  // { role: 'appMenu' }
  ...(isMac ? [{
    label: app.name,
    submenu: [
      { role: 'about' },
      { role: 'services' },
      { type: 'separator' },
      { role: 'quit' }
    ]
  }] : []),
  // { role: '文件' }
  {
    label: '文件',
    submenu: [
      isMac ? { role: 'close' } : { role: 'quit' }
    ]
  },
  // { role: '编辑' }
  {
    label: '编辑',
    submenu: [
      { role: '撤销' },
      { role: '重做' },
      { type: 'separator' },
      { role: '剪切' },
      { role: '复制' },
      { role: '粘贴' },
      ...(isMac ? [
        { role: 'pasteAndMatchStyle' },
        { role: 'delete' },
        { role: 'selectAll' },
        { type: 'separator' },
        {
          label: 'Speech',
          submenu: [
            { role: 'startSpeaking' },
            { role: 'stopSpeaking' }
          ]
        }
      ] : [
        { role: '删除' },
        { type: 'separator' },
        { role: 'selectAll' }
      ])
    ]
  },
  // { role: '查看' }
  {
    label: '查看',
    submenu: [
      { role: 'reload' },
    ]
  },
  // { role: '窗口' }
  {
    label: '窗口',
    submenu: [
      { role: 'minimize' },
      { role: 'zoom' },
      ...(isMac ? [
        { type: 'separator' },
        { role: 'front' },
        { type: 'separator' },
        { role: 'window' }
      ] : [
        { role: 'close' }
      ])
    ]
  },
  {
    role: '帮助',
    submenu: [
      {
        label: 'Learn More',
        click: async () => {
          const { shell } = require('electron')
          await shell.openExternal('https://electronjs.org')
        }
      }
    ]
  }
]

const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)
```

*放在 app.whenReady 之后调用*

## 系统托盘（右下角图标）

系统托盘只能由主进程创建

```js
import { Tray } from 'electron'

let tray = null
const path = require('path')
const iconPath = path.join(__dirname, 'assets/icon.png') // 在 mac 上图标使用原始图片大小，要注意使用合适的图标
tray = new Tray(iconPath)
const contextMenu = Menu.buildFromTemplate([ // 右键点击提示
  {
    label: '最小化',
    role: 'minimize',
    click: () => win.minimize()
  },
  {
    label: '退出',
    role: 'quit',
    click: () => app.quit()
  }
])
tray.setToolTip('鼠标触摸提示，默认是工具名')
tray.setContextMenu(contextMenu)

tray.on('click', () => { // 左键点击展示窗口
  win.show()
})
```

*放在 app.whenReady 之后调用*

## 文件浏览器

通过 electron 的方式加载文件浏览器（其实也可以通过 h5 的方式加载，但是 h5 ）

```js
import { dialog } from 'electron'

dialog.showOpenDialog(win, {
  properties: ['openFile'], // openDirectory - 文件，multiSelections - 多选
  filters: [ // 过滤器
    { name: 'Images', extensions: ['jpg', 'png', 'gif'] },
    { name: 'Movies', extensions: ['mkv', 'avi', 'mp4'] },
    { name: 'Custom File Type', extensions: ['as'] },
    { name: 'All Files', extensions: ['*'] },
  ],
}).then(res => {
  if (!res.canceled) {
    const file = res.filePaths[0]
    win.loadURL(`file://${file}`)
  }
})
```

上面的是 promise 版本，还支持 `dialiog.showOpenDialogSync`这样的同步语法。

## 通知类对话框

和文件浏览器类似，使用不同的弹窗

1. 消息弹窗：`dialog.showMessageBox`
2. 错误警告弹窗：`dialog.showErrorBox`

## 屏幕自适应

electron 提供 `screen`对象来暴露显示器信息

```js
const { app, BrowserWindow } = require('electron')

let mainWindow = null

app.whenReady().then(() => {
  // 必须要等项目 ready 之后才能获取 screen
  const { screen } = require('electron')

  const primaryDisplay = screen.getPrimaryDisplay()
  const { width, height } = primaryDisplay.workAreaSize

  mainWindow = new BrowserWindow({ width, height }) // 这样就打开后就会全屏，也可以根据比例缩放
  mainWindow.loadURL('https://electronjs.org')
})
```

## 参考文章

官方文档：https://www.electronjs.org/zh/docs/latest/tutorial/quick-start