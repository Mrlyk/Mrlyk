# 常用 loader

> 这里记录一些常用的 loader 说明和配置方法。过于常见的不记了，应该都知道。

## cache-loader

缓存一些性能开销较大的 loader 的编译结果，当然保存和读取也会产生性能开销，所以只对性能开销大的 loader 使用是比较好的选择。

**安装：**`npm i cache-loader -D`

**使用** 

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: 'cache-loader',
            options: { // 配置项非必需
              cacheKey, // 自定义缓存的标识
              read,  // 自定义缓存读取方啊
              write, // 自定义缓存写入方法
            }
          },
          'babel-loader'
        ],
        include: path.resolve('src')
      }
    ]
  }
}
```

