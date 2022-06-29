# 第三方常用插件

> webpack 除内置插件外，我们经常会用到一些第三方的插件。注意 webpack 5 是比较大的升级，下面的说明大部分是以 webpack 5 做的，如果有些选项在老版本中不能用需要去查一下文档老版本是怎么用的。

[toc]

## dotenv-webpack 读取 .env 环境变量

官方文档：https://www.npmjs.com/package/dotenv-webpack

```javascript
// webpack.config.js
const Dotenv = require('dotenv-webpack');

module.exports = {
  ...
  plugins: [
    new Dotenv()
  ]
  ...
};
```

在项目中建立如 `.env.development`文件即可。插件只会将代码中明确使用的参数输出到最终的打包产物中。

*ps：vue-cli 中就是使用的该工具来读取不同环境后缀的变量文件*

```js
const logger = debug('vue:env')
// mode = process.env.VUE_CLI_MODE
const basePath = path.resolve(this.context, `.env${mode ? `.${mode}` : ``}`)
const localPath = `${basePath}.local`

const load = envPath => {
  try {
    const env = dotenv.config({ path: envPath, debug: process.env.DEBUG })
    dotenvExpand(env)
    logger(envPath, env)
  } catch (err) {
    // only ignore error if file is not found
    if (err.toString().indexOf('ENOENT') < 0) {
      error(err)
    }
  }
}

load(localPath) //先读取了本地配置以 .local 结尾的
load(basePath)  // 再读取了环境变量配置，所以理论上后面的配置会覆盖前面的
```

## cross-env 跨平台环境变量设置

官方文档：https://www.npmjs.com/package/cross-env

```js
//package.json
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config build/webpack.config.js"
  }
}
```

#### cross-env & webpack.DefinePlugin

cross-env 与 **webpack 内置**的 DefinePlugin 都用于处理环境变量，区别是

- cross-env：定义编译时的环境变量，处理跨平台环境变量设置方法不统一导致环境变量无法配置的问题
- DefinePlugin：用于读取不同环境的环境变量。和上面的 dotenv-webpack 很类似。只不过这是 webpack 内置的插件。使用这个插件一般就不需要上面的 dotenv-webpack 了

#### --mode & NODE_ENV

- --mode：webpack4 及以上版本可以使用`mode: development`选项，也可以在 cli 命令中使用。只有`development`、`production`、`none`三个选项，默认`production`。使用后在打包产物中可以通过`proess.env.NODE_ENV`获取到环境变量，**但是在 webpack.config.js 中无法获取，即编译前无法读取到。**
- NODE_ENV：在一般的 unix、linux 系统中，cli 命令使用 `NODE_ENV=development webpack`这种方式打包可以在编译前临时设置上环境变量。**但是 windows 等某些平台上无法这样配置，会被阻塞。为了处理这个问题就需要使用到上面说的 cross-env 工具。**

## postcss

css 语法的降级工具，同时提供了一些 css 转换功能。相当于 babel 之于 js，功能很强大。

官方文档：https://github.com/postcss/postcss/blob/main/docs/README-cn.md

```shell
npm i postcss postcss-loader postcss-preset-env
```

可以在 webpack 的 loader 中直接配置，也可以在使用配置文件`postcss.config.js`

```js
module.exports = {
  plugins: [
    [
      'postcss-preset-env',
      {
        browsers: 'last 2 versions' // 所有浏览器的最后两个版本。基于 browseslist 工具
      }
    ]
  ]
}
```

## copy-webpack-plugin 拷贝静态资源文件

项目中直接写死的静态文件，比如一些下载下来的第三方 js 库或者图片。不想参与 webpack 打包，或者在 index.html 中直接引入的（webpack 不会打包）的文件。可以使用本插件，将静态资源直接拷贝到输出目录中。

提供基于`globby`的文件匹配模式

```js
const CopyWebpackPlugin = require("copy-webpack-plugin");

{
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.join(__dirname, "./public"),
          to: path.join(__dirname, "./dist/public"), // 默认直接使用 output.path
          toType: "dir", // 文件类别，如果没有扩展名默认识别为文件夹
          globOptions: { // 匹配选项
            gitignore: true, // 同步 gitignore 中的配置，默认 false
            dot: true, // 匹配以 . 开头的文件，比如 .npmrc，默认 false
            ignore: [
              ".DS_Store",
              '**/index.html',
            ],
          },
        },
      ],
    }),
  ]
}

```

## webpack-chain

官方文档：https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans

参考文章：https://juejin.cn/post/6947851867422621733

webpack 链式配置插件，vue 中的 chainWebpack 就是使用的这个插件，使用总的来说主要是两点

- 配置新增某项属性，就是用某项属性的 key
- 配置修改某项属性，就用`tap`方法

**配置插件**

`config.plugin('name').use(plugin)`：新增插件

- name 是使用 webpack-chain 配置该属性后，该属性的标识，可以用来删改目标配置。同时会在配置文件里生成一行注释`/* config.plugin('name') */`

`config.plugin('name').tap(args => [ ... ])`：修改插件选项

- 修改标识为 name 的插件配置项，tap 方法的回调中`args[0]`是原来的配置，可以自己手动合并

```js
{
  chainWebpack: (config) => {
    config.plugin('html').use(HtmlWebpackPlugin, [ // 新增 HtmlWebpackPlugin 插件
      {
        templage: path.resolve(__dirname, './public/index.html')
      }
    ])
    // 修改 HtmlWebpackPlugin 插件配置项
    config.plugin('html').tap(args => [...args[0], minify: {minifyJS: true}])
  }
}
```

**配置loader**

配置 loader 和插件大同小异，**如果要配置 loader 的执行顺序，可以使用 post 这种 loader 配置。需要注意这个配置是在 rules 层级的，所以需要创建一个不同名但是相同匹配规则的 loader 并设置`post()`来处理** 

```js
{
  config
    .module
    .rule('babel-name').test(/\.js$/)
    .exclude
    	.add(path.resolve(__dirname, 'node_modules'))
    .use('babel-loader') // 会直接加在已有的 loader 后面，优先级最高
    	.loader('babel-loader')
    	.options({
    		presets: ['@babel/preset-env']
  		})
}
// 修改也是使用 tap
config.module
.rule('babel-name')
.use('babel-loader')
  .tap(options => {
    // 修改它的选项...
    options.include = path.resolve(__dirname,  'test')
    return options
  })
```

**使用 when 根据环境配置**

```js
{
  config.when(process.env.NODE_ENV === 'test', config => {
    ...
  })
}
```

## html-webpack-plugin

html 模版渲染插件

## compression-webpack-plugin

官方文档：https://github.com/webpack-contrib/compression-webpack-plugin 

assets 输出前的的压缩插件，支持自定义压缩算法

```js
module.exports = {
  // ...
  plugins: [
    new CompressionWebpackPlugin({
      // 开启gzip压缩
      algorithm: 'gzip', // 压缩算法
      test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'), 
      threshold: 10240, // 仅处理大于此大小的资源（以字节为单位）
      minRatio: 0.8 // 压缩比大于此值才处理
    })
  ]
}
```

## add-asset-html-webpack-plugin

自动添加一个生成好的文件到 html-webpack-pluin 输出的 html 上。有时候我们需要引入一些第三方的 js 或者 DLL 预打包的 js 就可以使用该插件来引入。

与我们手动写死相比，该插件支持动态配置引入路径

- outputpath 引入的文件放置的地址
- publicPath 引入标签 script 上 src 的前缀地址

```js
module.exports = {
  // ...
  plugins: [
    new AddAssetHtmlPlugin([
      {
        filepath: require.resolve('./dll/vendor.dll.js'),
        outputPath: 'static/js/',
        publicPath: process.env.BUILD_BRANCH
        ? `/${process.env.BUILD_BRANCH}/static/js/`
        : '/static/js/'
      }
    ]),
  ]
}
```

