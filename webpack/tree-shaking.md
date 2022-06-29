# tree-shaking

webpack 的 tree-shaking 和我以为的不太一样，我以为是直接剔除没有用到的模块，然而没有用到的模块本就不会被打包。实际上 tree-shaking 是剔除模块中没有用到的变量、方法等，更高级一点。

那 webpack 是如何做到的呢？本文一起来看一下！



## 参考文章

Tree shaking问题排查指南: https://mp.weixin.qq.com/s/SfZbBg0lWhvgzS023wTjYA 