# react 工程配置

记录一下使用 react 需要的 eslint 和 babel 配置，webpack 后续再补充

## react-script@2.0.0

如果我们使用 create-react-app 创建一个基础的 react 项目，实际使用的脚本是 react-script 这个脚本。如果我们想将官方的配置展示出来并且自己修改则需要调用下面这个 scripts。

- `"eject": "react-scripts eject"`

如果我们不想完全的自定义配置，只是做一些简单的修改，react-script 也会默认读取项目中的一些配置文件来修改配置。

#### proxy

正常我们想更改 proxy 需要在 webpack 的 proxy 中处理。如果要基于 react-script 去修改，则需要

1. 手动安装`http-proxy-middleware`

2. **在项目的`src/setupProxy.js` 中添加配置**，一个实例如下

   ```js
   const { createProxyMiddleware } = require('http-proxy-middleware')
   
   module.exports = (app) => {
     app.use(
       '/maoyan',
       createProxyMiddleware({
         target: 'https://i.maoyan.com/ajax',
         changeOrigin: true,
         pathRewrite: {
           '^/maoyan': ''
         }
       })
     )
   }
   ```

这样就可以更改代理配置了。

#### CSS Modules

react-script 会自动集成 CSS Modules 配置。

如果我们要自己配置 react 工程则需要：[react-css-modules](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgajus%2Freact-css-modules)、[babel-plugin-react-css-modules](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgajus%2Fbabel-plugin-react-css-modules) 这两个插件（第一个已废弃，直接配置第二个即可）

- 使用 react-script 自动集成的配置。需要在命名 css 文件的时候加上`module`路径，如`films.module.css`

- 使用自己手动点配置没有上面的限制，直接写即可

  ```js
  {
    "plugins": [
      ["react-css-modules", {
        "option": "value"
      }]
    ]
  }
  ```

**举个🌰**

```react
import style from './films/films.module.css'

<NavLink activeClassName={style.linkActive} to='/films/nowplaying'>正在上映</NavLink>
```





## 自定义配置

#### eslint

eslint 主要是要配置好 react 的 extends 配置

```js
module.exports = {
  root: true,
  parser: '@babel/eslint-parser',
  parserOptions: {
    ecmaVersion: 2020,
    ecmaFeatures: { // 这个配置在 react 的 extends 里自带了，不配也可以
      jsx: true
    }
  },
  settings: {
    react: {
      version: 'detect'
    }
  },
  env: {
    browser: true,
    node: true,
    es2021: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended', // 这个是 react 必须的
    'standard',
    'plugin:react/jsx-runtime' // 这个是 react jsx 语法支持必须的
  ]
}
```

#### babel

babel 的配置直接使用预设就可以了，比较常规

```js
module.exports = {
  presets: [
    '@babel/preset-react'
  ]
}
```

## 浏览器插件

#### redux

官方文档：https://github.com/zalmoxisus/redux-devtools-extension

该插件下好后是不能直接用的，需要在项目中配置一些参数，在生产环节中去除，防止其他用户看见。

配置有两种，基础配置只能看到同步操作。一般我们直接受用进阶配置：

```js
// redux/store.js

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose
const store = createStore(reducer, composeEnhancers(
  applyMiddleware(reduxThunk)
))
```

