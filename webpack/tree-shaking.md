# tree-shaking

webpack 的 tree-shaking 和我以为的不太一样，我以为是直接剔除没有用到的模块，然而没有用到的模块本就不会被打包。实际上 tree-shaking 是剔除模块中没有用到的变量、方法等，更高级一点。

~~区分 DCE 和 tree shaking。~~

tree-shaking 因 rollup.js 这个打包工具而普及。

想要实现 tree-shaking 必须满足一个条件——**模块必须是 ESM**。因为tree-shaking依赖于ESM的静态结构。





## 参考文章

- 《vuejs 设计与实现》
- Tree shaking问题排查指南: https://mp.weixin.qq.com/s/SfZbBg0lWhvgzS023wTjYA 