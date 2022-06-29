# 配置 vue 框架的插件开发模版

开发浏览器插件过程中，如果使用原生的 js 去写，无疑效率会非常的低。我们可以借助于前端框架来提升开发效率。只要打包产物符合插件的要求即可。同时需要关注热更新的实现。

这里是用熟悉的 webpack 来处理打包和热更新，后面可能考虑 vit

1. 搭建一个 vue 标准模版项目
1. 考虑插件的项目结构
1. 根据目标项目结构配置打包，产出目标目录结构
1. 

[toc]



## 考虑浏览器插件需要的项目内容

#### 必须的内容

- manifest.json - 清单文件
- manifest.json 中必须的配置  manifest_version、name、version；推荐的配置 description、icons

清单文件有了后就可以开始根据插件需要的文件有进行配置。

## 模版项目结构

我们知道 vue 是挂载在 DOM 上的，一般是单页应用。而浏览器插件有不同的页面，比如 popup、background、homepage...所以我们需要开发的是一个多页应用，这就要求**打包的时候有多个入口和出口**

多页我们使用相同的`index.ejs`文件作为模版，将 **vue 实例分别挂载在不同的模版上**。下面看看我们需要哪些东西！

#### content

`content-scripts` 可以用来直接注入页面，操作页面中的 dom。这样就可以在页面上直接完成一些展示操作，给用户更好的交互体验。

所以 content 是必须的，不过`content-scripts`是纯业务逻辑，么有页面。所以**最后输出的时候只需要配合好他在 manifest.json 中对应的路径即可**。

#### backgroud

`background`作为生命周期最长、权限最高的扩展部分，肯定也是需要的。backgroud 可以设置页面也可以配为 js，看我们具体的需求。

#### popup

`popup`用于和用户的基础交互，一般也是需要的，同时 `popup` 肯定是弹出的 `popup` 面板，所以 `popup `要有页面即需要创建 .vue 文件来画页面。

#### inject

`inject`(web_accessible_resources) 用于页面和插件交互，一般来说不是很需要，看需求。模版文件里我们仍然放上。同时由于他也是和网页内容`content`的交互相关，所以把他放在 content 一起。

#### devtools

devtools 具有一组特有的`Devtools API`，比如拦截网络请求。所以如果要写 mock 插件之类的这个是必备的。devtools 只能是页面，所以需要创建一个 .vue 文件。

同时它还可以创建 devtool panel，可以使用 devtools.vue 页面来作为面板，devtools.js 作为逻辑承载。

#### options

右键插件选项的单独页面，是个很重要的页面，插件的配置全都保存在这里。所以也是必备的，同时也只能是页面，是给用户用的。所以也需要创建一个 .vue 文件。

#### homepage_url

主页配置，如果需要的话，直接在 manifest.json 里面声明就行了，一把可以用自己的网站，打广告之类的。多和插件本身功能无关。就不放在模版里了。

#### 最终项目结构如下

```text
simple-mock-yk
├─ .eslintrc.js
├─ .gitignore
├─ README.md
├─ babel.config.js
├─ config
│  └─ manifest.json // 清单模版
├─ package-lock.json
├─ package.json
├─ public
│  ├─ favicon.ico
│  └─ index.ejs // 多页面的模版
├─ src
│  ├─ assets
│  ├─ components
│  ├─ utils
│  └─ view  // 主要看这里面
│     ├─ background
│     │  ├─ background.js
│     │  └─ background.vue
│     ├─ content
│     │  ├─ content-script.js
│     │  └─ inject.js
│     ├─ devtools
│     │  ├─ devtools.js
│     │  ├─ devtools.vue
│     │  └─ panel.js
│     ├─ options
│     │  ├─ options.js
│     │  └─ options.vue
│     └─ popup
│        ├─ popup.js
│        └─ popup.vue
├─ static
│  └─ icons
│     ├─ mock-ext-128.png
│     ├─ mock-ext-16.png
│     └─ mock-ext-48.png
└─ vue.config.js
```

## 配置

项目结构是开发时用，最终我们生产插件还是要符合插件对目录的要求，这就需要我们在**清单文件中写好读取的目录地址**，同时配置打包**出口到正确的路径**。

后续也可考虑读取清单文件中的配置来动态处理 entry 地址，由于插件项目结构的固定性，暂时用写死的方法处理。

首先我们需要完成我们的模版清单配置，再根据清单的配置信息来配置 webpack 打包输出的配置信息。

#### 清单配置

基础的一些信息不必多说，主要还是配合上面模版具有的项目结构，将需要的一些信息补上。

- **基础信息**

基础信息就是插件名、图表等描述

```json
{
  "manifest_version": 3,
  "name": "vue ext tempalte",
  "version": "0.1",
  "description": "vue 扩展模版",
  "icons": {
    "16": "icons/mock-ext-16.png",
    "48": "icons/mock-ext-48.png",
    "128": "icons/mock-ext-128.png"
  },
  
}
```

- **content**

`content-script`和`inject.js`的路径信息，这两个都是纯 js 文件，配置好注入路径即可

```json
{
  "content_scripts": {
    "matches": [
      "http://*/*"
    ],
    "js": [
      "js/content-script.js" // js/xxxx
    ],
    "run_at": "document_end"
  },
  "web_accessible_resources": [
    "js/inject.js", // js/xxxx
    "panel.html" // 这里还把 devtools 的面板注入了，方便页面和 devtool 互相访问
  ]
}
```

注意上面的路径都是`js/xxxx`，这就要我们后面打包的时候把 js 文件都打包到`js/`这个目录

- **background**

后台页面的话就使用`pages`目录下的 background.html

```json
{
  "background": {
    "page": "pages/background.html"
  }
}
```

其他的我就不一一列举了，下面是其他的配置文件

```json
{
  "action": {
    "default_icon": "icons/mock-ext-48.png",
    "default_title": "simple mock",
    "default_popup": "pages/popup.html" // 配置弹出窗
  },
  "devtools_page": "pages/devtools.html", // devtools 页面
  "options_ui": {
    "page": "pages/options.html"
  }
}
```

**总结**

总的来说就是

- js 文件 -> `js/xxx`

- pages 页面 -> `pages/xxx`

这就要求我们的打包输出符合这样的目录结构。

#### webpack 配置
