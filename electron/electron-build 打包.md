# electron-build 打包

electron 开发只要我们熟悉 node 还是比较简单的。重点是开发完成后的打包配置，需要学习一下。

[toc]

首先先看一个常规的打包配置，然后再对其中一些常用的和坑做说明。

一个常规的打包配置如下：

```js
const buildConfig =  {
  "appId": "com.test", // appid
  "productName": "test",// 程序名称
  "files": [ // 需要打包的文件目录，如果根目录有项目的文件，需要把这些文件都写进去，不要的不用写
    "build/icons/*",
    "src/**/*",
    "main.js", 
    "preload.js",
    "renderer.js",
    "index.html",
    "node_modules/**/*"
  ],
  "directories": {
    "output": "build/app", // 打包输出的目录
    "app": "./", // package所在路径
    "buildResources": "assets"
  },
  "nsis": { //nsis安装器配置(windows 安装程序用)
    "oneClick": false, // 是否需要点击安装，自动更新需要关掉
    "allowToChangeInstallationDirectory":true, //是否能够选择安装路径
    "installerIcon":"./dist/icons/installer.ico",// 安装程序图标（最好用256 × 256以上的图标）
    "installerHeader":"./build/icons/installerHeader.bmp",// 安装时界面头部右边图片，只能用bmp格式的
    "uninstallerIcon":"./dist/icons/uninstall.ico",//卸载程序图标（最好用256 × 256以上的图标）
    "perMachine": true, // 是否需要辅助安装页面
    "createDesktopShortcut": true, // 创建桌面图标
    "createStartMenuShortcut": true,// 创建开始菜单图标
    "license":"./src/license/license.html" //安装界面的软件许可证，如果不配置，不会出现软件许可证界面
  },
  "win": {
    "icon":"./dist/icons/aims.ico",//图标文件，分辨率至少在256*256以上，不然会报错
    "target": [
      {
        "target": "nsis", // 输出目录的方式
        "arch": [ // 这个意思是打出来32 bit + 64 bit的包，但是要注意：这样打包出来的安装包体积比较大，所以建议直接打32的安装包。
          "x64",
          "ia32"
        ]
      }
    ]
  },
  "linux": { //Linux打包配置，现在只是在windows打包，未测试过
    "icon": "./dist/icons/main.png" //图片当前格式只能是512x512分辨率的
  },
  "mac": { //Mac打包配置
    "target": "dmg",
    "icon": "./dist/icons/main.png" //图标的分辨率至少在512x512以上
  },
  "dmg": { //mac打包dmg格式配置
    "title": "Mac程序",
    "icon": "./dist/icons/main.png",
    "contents": [
      {
        "x": 110,
        "y": 150
      },
      {
        "x": 240,
        "y": 150,
        "type": "link",
        "path": "/Applications"
      }
    ],
    "window": {
      "x": 400,
      "y": 400
    }
  },
  "publish": [//版本更新配置推送
    {
      "provider": "generic",
      "url": "http://127.0.0.1:8080/updata/"
    }
  ]
}　　

```

*要注意一点：electron-builder.js 即打包配置文件和使用了 vue-cli-plugin-electron-builder 插件后在 vue.config.js 中的配置是不相同的。虽然 vue.config.js 中的配置会被传给 electron-builder，但是会在其中做一些转换，不能把它视作和在 electron-builder.js  中一样编写。*  

## 图标相关

可以直接在配置最外层设置`icon`来配置图标，网上很多说的图标只能是`ico`格式的，经实验发现不是，**普通的 png 图片也可以**。

```js
module.exports = {
  // ...
  icon: './public/xxx-icon.png'
  // ...
}
```

也可以和常规配置中一样，单独为 mac、windows 设置不同的图标，其中

- windows 推荐使用`ico`图标
- mac 推荐使用`icns`图标

当然使用图标也行，就是在高分屏下会模糊