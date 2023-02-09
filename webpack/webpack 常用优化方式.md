# webpack 常用优化方式

[toc]

## 常用的打包分析方式

#### `webpack --json > stats.json` 输出打包信息

打包信息中字段对应的信息如下

- **「assets」** ：编译最终输出的产物列表
- **「chunks」** ：构建过程生成的 chunks 列表，数组内容包含 chunk 名称、大小、依赖关系图
- **「modules」** ：本次运行触达的所有模块，数组内容包含模块的大小、所属chunk、分析耗时、构建原因等
- **「entrypoints」** ：entry 列表，包括动态引入所生产的 entry 项也会包含在这里面
- **「namedChunkGroups」** ：chunks 的命名版本，内容相比于 chunks 会更精简
- **「errors」** ：构建过程发生的所有错误信息
- **「warnings」** ：构建过程发生的所有警告信息

同时可以将生成的文件上传到 [webpack 官方的分析平台](https://webpack.github.io/analyse/) ，以可视化的方式查看结果

#### 打包分析插件

- **speed-measure-webpack-plugin** 

分析每一步打包所耗费的时间

- **webpack-bundle-analyzer** 

打包体积分析插件

```shell
webpack --report
```

- **[webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard)** 

命令行可视化工具，能够在编译过程中实时展示编译进度、模块分布、产物信息等。

**配置好插件后需要修改 webapck 启动方式** 

```json
"scripts": {
    "dev": "webpack-dashboard -- webpack-dev-server", # OR
    "dev": "webpack-dashboard -- webpack",
}
```

![image-20220417181012485](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220417181012485.png?x-oss-process=image/resize,w_600,m_lfit) 

- **[UnusedWebpackPlugin](https://www.npmjs.com/package/unused-webpack-plugin)**

查找出未使用的依赖的插件，相比于`npx depcheck`这类工具，能根据 webpack 实际的打包信息查找出更精确的结果。而`npx depcheck`只能根据是否`import`来判断

- **[cpuprofile-webpack-plugin](https://www.npmjs.com/package/cpuprofile-webpack-plugin)** 

生成可视化的图表来查看具体的构建信息，会创建一个文件夹，将可视化的 index.html 直接输出出来，使用浏览器打开即可

![image-20220417180913405](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220417180913405.png?x-oss-process=image/resize,w_600,m_lfit) 

#### 使用 devtools 分析

webpack 可以生成供 devtoools performance 分析的 json 文件

```js
module.exports = merge(require('../webpack.config'), {
  plugins: [
    new wbp.debug.ProfilingPlugin({
      outputPath: path.resolve(__dirname, '../profileEvents.json')
    })
  ]
});

```

将 json 文件上传到 chrome 的 devtools performance 中就可以看到详细的打包流程和耗费时间图

## 打包体积优化

- 图片压缩

- 代码压缩

- gzip 压缩（需web服务支持）配合 `compression-webpack-plugin` 插件使用

- 按需引入

- csdn 引入，特别是入口处的一些框架资源和**大文件、图片**，使用 cdn 是最好的（注意高可用）还可以避免构建时的内溢出问题，加快构建速度

- sideEffects 手动标记为 false，表示包没有副作用，可以被直接 shaking 掉。可以在 package.json 中直接配置全局的，也可以指向某个文件`"sideEffects": ["./src/some-side-effectful-file.js"]` 

- webpack 5 `optimization.splitChunks `分包策略（webpack 4 是`CommonsChunkPlugin`）。**使用多入口的方式更有利于分包**

  ```js
  module.exports = {
    entry: {
      //...
      antd: [ // 将 antd 这个 ui 库的常用组件单独设置打包入口
        'antd/lib/button',
        'antd/lib/icon',
        'antd/lib/breadcrumb',
        'antd/lib/form',
      ]
    },
    /// webpack4 的分包插件
    plugins: [
      new webpack.optimize.CommonsChunkPlugin({ 
        names: ['antd'], // 将 antd 常用组件单独打包
        minChunks: Infinity // 拆分前共享模块的最小 chunks 数，在 4 中必须 >= 2，也可以设置为 Infinity 直接生成一个空包保证没其他的模块会被放入
      }),
    ]
  }
  ```

- webpack 5 `optimization.minizer `，手动配置`terser-webpack-plugin`

## 打包速度优化

减小包的体积中的某些操作也会提高打包速度，因为要打包的内容少了。但是有的可能造成负面影响，比如代码压缩这些操作肯定是耗时的。

#### oneOf 

减少文件的 loader 匹配次数。文件在匹配 loader 的时候会用正则处理，但是 webpack **会把每个配置的 rule 都执行一遍匹配测试**。使用这个选项可以匹配到其中一个后不再匹配其他的（**中断匹配**），提高效率。

```js
{
  module: {
    rules: [
      {
        oneOf: [ // 匹配到下面任意一个之后不再进行匹配测试
          {
            test: /\.css$/,
            use: [...commonCssLoader]
          },
          {
            test: /\.less$/,
            use: [...commonCssLoader]
          }
        ]
      }
    ]
  }
}
```

#### DLLPlugin

预打包不怎么变动的框架文件，webpack 的内置插件。

**使用**

**1.单独打包**

基本配置如下，我们需要先使用这个插件单独配置一份输出，打出第三方的包放在项目中（随 git 上传）。后面除非这些第三方的包更新，否则就不再打包。

- `context` (optional): manifest 文件中请求的上下文(context)(默认值为 webpack 的上下文(context))
- `name`: 暴露出的 DLL 的函数名 ([TemplatePaths](https://github.com/webpack/webpack/blob/master/lib/TemplatedPathPlugin.js): `[hash]` & `[name]` )
- `path`: manifest json 文件的**绝对路径** (输出文件)

```js
module.exports = {
  entry: {
    vendor: ['vuex', 'vue/dist/vue.runtime.esm.js', 'vue-router'], // 配置要打包的入口文件
    vit: '@weiyi/vit'
  },
  output: { // 需要配置单独输出的文件夹
    path: path.resolve(__dirname, '../dll'),
    filename: '[name].dll.js',
    library: 'dll_[name]_[contenthash:8]'
  },
  plugins: [
    new wbp.DllPlugin({
      path: path.join(__dirname, '../dll', '[name]-manifest.json'),
      name: 'dll_[name]_[hash]',
      context: __dirname
    })
  ]
};
```

配置完成后，使用该配置执行一次打包。

*因为这边打包生成的是 js，在主流程打包的时候还是会经过主流程那边的 loader 处理，所以这里不需要配置 loader 之类的*

**2.在真正的主 webpack 配置文件中引入 `DllReferencePlugin`**

```js
module.exports = {
  plugins: [
    new wbp.DllReferencePlugin({ // 多个库需要配置多个
      context: __dirname,
      manifest: require('./dll/vendor-manifest.json'),
      publicPath: '/static/js' // 注意引入路径
    }),
    new wbp.DllReferencePlugin({
      context: __dirname,
      manifest: require('./dll/vit-manifest.json'),
      publicPath: '/static/js'
    }),
  ]
}
```

**3.在 html 中引入**

打包完成后`index.html`不会引入我们的文件，需要我们手动引入或者通过插件配置

```js
module.exports = {
  plugins: [
    new AddAssetHtmlPlugin([ // 通过这个插件可以配置 html-webpack-plugin 生成的 index 的内容
      {
        filepath: require.resolve('./dll/vendor.dll.js'),
        outputPath: 'static/js/',
        publicPath: process.env.BUILD_BRANCH
        ? `/${process.env.BUILD_BRANCH}/`
        : '/'
      },
      {
        filepath: require.resolve('./dll/vit.dll.js'),
        outputPath: 'static/js/',
        publicPath: process.env.BUILD_BRANCH
        ? `/${process.env.BUILD_BRANCH}/`
        : '/'
      }
    ])
  ]
}
```

#### 多进程/并行 打包

- **thread-loader**

原理：

- 启动时，以 `pitch` 方式拦截 Loader 执行链
- 分析 Webpack 配置对象，获取 `thread-loader` 后面的 Loader 列表
- 调用 `child_process.spawn` 创建工作子进程，并将Loader 列表、文件路径、上下文等参数传递到子进程
- 子进程中调用 `loader-runner`，转译文件内容
- 转译完毕后，将结果传回主进程

```js
use: [
  {
    loader: "thread-loader",
    options: {
      // 产生的 worker 的数量，默认是 cpu 的核心数
      workers: 2,
      // 额外的 node.js 参数
      workerNodeArgs: ['--max-old-space-size', '1024'],
      // 闲置时定时删除 worker 进程
      // 默认为 500ms
      // 可以设置为无穷大， 这样在监视模式(--watch)下可以保持 worker 持续存在
      poolTimeout: 2000,
      // 池(pool)的名称
      // 可以修改名称来创建其余选项都一样的池(pool)
      name: "my-pool"
    }
  },
  "babel-loader"
]
```

thread-loader 的兼容性相对比较差，因为没注入 `emitFile`、`emitAsset`等接口，导致和文件写入相关的 loader 无法一起使用，比如`file-loader`。（可以将 `thread-loader`放到他们之前调用，当然如果前面没有其他 loader 也就没意义了）

**ps:**

(1) 多进程启动需要花费 600ms 左右，所以如果本身打包在 15s 以内的没必要开启

(2) `HappyPack`是这个工具的上一代，作者已经明确不会维护。但是他具有更好的兼容性，如果存在兼容问题还是可以用`HappyPack`

- **HappyPack**

HappyPack 的配置相比`thread-loader`略微麻烦一点，既需要配置 loader 也需要配置插件。

HappyPack 启动后，会向其运行的 loader 注入 `emitFile` 等接口，而 Thread-loader 则不具备这一特性，所以他的兼容性更好。

- **parallel-webpack**

上述两个 loader 都仅作用于文件的转译阶段（loader 处理），对 AST 依赖解析、依赖收集、打包、splitchunks 等均没有影响。

praller-webpack 则提供了更强大的方案：创建多个 webpack 实例进行打包（原理也是如此）。**由入口文件数量确定产生的 webpack 实例数量，所以对单入口项目没什么作用，但是多页面应用起飞。**（对多页面应用也会有一些问题，比如压缩代码在多个进程上重复执行）

#### webpack5 持久化缓存

在 webpack5 之前，可以使用`cache-loader`将编译结构写入硬盘缓存，还可以使用`babel-loader`，设置`option.cacheDirectory`将`babel-loader`编译的结果写进磁盘。

在 webpack5 之后，可以使用自带的缓存策略：`cache` 会在开发模式被设置成 `type: 'memory'` 而在生产模式中被禁用

```js
module.exports = {
  cache: {
    type: 'filesystem', // 存储在哪里
    cacheDirectory: path.resolve(__dirname, '/node_modules/.cache/webpack'),
    buildDependencies: {
      config: [__filename] // webpack 配置文件改变时缓存失效
    }
  }
}
```

缓存策略：限制最大 500MB，失效时间 2周。当大小逼近阈值时使用 LRU 算法，优先删除最老的

**除了 cache 需要配置之外。还可以对`optimization`中的`moduleIds`进行配置，这也属于持久化缓存的一部分。**配置可以查看《webpack 属性解析》。

*实测配置之后，二次打包速度提升 100%* 

**注意**

存在一定的副作用，已知的比如：

- node_modules 内文件修改后不生效，因为会读 cache 里的

#### cache-loader 缓存

webpack 5之前没有持久化缓存，可以使用 cache-loader 来缓存耗时的任务，比如 babel 的。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.m?jsx?$/,
        use: [
          {
            loader: 'cache-loader',
            options: {
              cacheDirectory: '/xxxxx/node_modules/.cache/cache-loader',
            }
          },
          {
            loader: 'babel-loader'
          }
        ]
      }
    ]
  }
}
```

除了 cache-loader 之外，webpack 5 之前还可以使用如下 loader 自带的缓存功能。当然配置了 cache-loader 之后，另外两个也不需要配置了，在这里简要说明一下

**babel-loader 缓存**

默认缓存位置: `node_modules/.cache/babel-loader`

```js
module.exports = {
  module: {
    rules: [{
      test: /\.m?js$/,
      loader: 'babel-loader',
      options: {
        cacheDirectory: true,
      },
    }]
  }
};
```

**eslint-loader 缓存**

默认缓存位置: `./node_modules/.cache/eslint-loader`

```js
module.exports = {
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /node_modules/,
      loader: 'eslint-loader',
      options: {
        cache: true,
      },
    }]
  },
};
```

#### TS 项目 transpileOnly

ts-loader 在进行编译时，会对整体项目进行语义进行检查，这是非常耗时的。我们可以开启`transpileOnly`选项来关闭该检查，同时使用官方推荐的另一个插件`fork-ts-checker-webpack-plugin`来开启另一个进程进行检查。

```js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin')

module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'eslint-loader',
        options: {
          transpileOnly: true  // 仅编译
        }
      }
    ]
  },
  plugins: [
    new ForkTsCheckerWebpackPlugin()
  ]
}
```

同时还可以一定程度上处理编译大量文件造成的内存溢出问题

#### 其他配置最佳实践

除了上述的使用各种插件、缓存来优化打包速度，我们还可以在 webpack 配置的细节上进行处理。**递归操作比我们想象的要更加耗时。** 

1. **使用最新版本的 webpack**：webapck 的性能、功能都在随着其不断迭代而提升（webpack 的团队太强啦）
2. 缩小搜索范围（减少递归），常见方式：
   - `resolve.modules` 指定确认的第三方包目录，比如就是当前根目录下的`node_modules`
   - `resolve.extensions` 明确扩展名，当然最好的方式是在引入时写明（`resolve.enforceExtension = true`可以强制要求声明，但不是很推荐）
   - `resolve.mainFiles` 明确主文件，比如我们`import`一个目录的时候，会默认去引入目录下的`index.xx`文件，可以通过该项明确主文件到底是哪一个
3. `module.noParse `跳过部分文件编译：对于一些完全独立的代码，不需要在做重复的依赖分析、转译等，可以直接配置该选项。比如 Vue 的 `node_modules/vue/dist/vue.runtime.esm.js` 。*如果使用了 DllPlugin 预编译了就没必要了。*
3. loader 的 `include/exclude`明确 loader 作用范围
3. `optimization` minimizer 手动配置，参考《webpack 属性解析》，这一项即影响打包体积，又影响打包速度

## 美化

上面说的都是针对实用性的优化，但有时候我们也需要一些输出上的美化。比如打包完成输出一大坨消息，看着很混乱，我们希望他更清晰。