# npm 包管理工具详解

> npm@^6.0.0，7.0 以上版本有较大改动，先记录 6.0 版本

[toc]

- npm 安装时如果存在可复用的依赖时如何处理？
- npm 安装依赖的先后顺序会对项目产生影响吗？
- npm 存在哪些问题需要特别注意

## npm 依赖复用之扁平化

在项目中可能存在这样一种情况，比如我自己全局安装了`core-js@3`。我的 babel 的依赖的依赖 `@babel/preset-env`它又依赖于`core-js@2`，就会出现这样一种情况：（图片仅演示，babel 不会直接全量安装 core-js@2，而是通过 core-js-compat 查询和单独安装目标浏览器需要的 babel-plugin）

![image-20220127183220866](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220127183220866.png?x-oss-process=image/resize,w_600,m_lfit) 

**对于这种情况 npm 是怎么处理的呢？** 

```text
Searches the local package tree and attempts to simplify the overall structure by moving dependencies further up the tree, where they can be more effectively shared by multiple dependent packages.
```

官方是说为了简化整体结构和多个依赖共享包，会查找本地的依赖关系树并将依赖关系移到树上**更高**（能多高多高，没限制 @ 作用域的就直接最顶层，使树变平）的位置。具体可以看下面扁平化策略的说明。

#### 为什么需要扁平化？

没有扁平化的策略依赖如果存在子依赖，会一层层的嵌套下去。如果层级过深，目录超过 256 个字符，windows 甚至无法识别。同时重复的依赖也会被重复安装，包体积会持续增大同时影响安装速度。

#### 扁平化如何处理

扁平化会提升子依赖的层级，比如我们只安装了一个 core-js，但是我们去查看 node_modules 目录，会发现里面有很多包。这就是扁平化的结果，不再是子依赖一层层嵌套下去。

除了普通的扁平化处理，还有两种特殊情况：

**相同依赖的不同版本如何处理** 

如果存在 A -> C@1.0.0， B -> C@1.0.1 这样的情况，则取决于 A 和 B 在 package.json 中声明的顺序，先声明的会被扁平化，后声明的依然嵌套。如果 A 先声明，则 C@1.0.0 会被提升到 A 同级，C@1.0.1 依然会被作为子依赖安装放在 B 下面，不会变平化它。如果已经生成了 package-lock.json 则以它为准，不会再进行扁平化操作。

总的来说就是：**不同版本的依赖不会被安装到同一层级，先声明的依赖会先被扁平化，后声明的会嵌套安装。**

另外如果安装 A -> C@1.0.0（A 和 C由于扁平化会在一层目录中），再在项目中直接安装 C@1.0.1。那么 C@1.0.0 会被移动到 A 的子目录下，C@1.0.1 会和 A 在一层。

详情看下面参考文章。

#### 还存在的问题

扁平化通过提升依赖层级的方式解决了上述问题，但是又带来了**新的问题：**  

- **仍然会存在被重复安装的包，存在不确定性**（原因看上面的扁平化如何处理的）

- 模块可以访问没有声明依赖的包，因为扁平化会把依赖的包提升到顶级。这会导致有时我们写代码没问题，但是别人拉我们的项目（没有 package-lock.json）的时候会出现错误。

- 扁平化操作算法负责，也很耗时

**即使相同版本还是存在重复安装的问题**：假如存在这样一个两层嵌套的依赖结构：第二层有一个依赖 A，声明了子依赖`core-js@3.x.x`，由于扁平化它会和第一层中的`core-js`做比较，是否重复安装，重复的话不会在安装，否则安装在 A 的 node_modules 下。同时它还有两个子依赖 B 和 C，由于扁平化的关系他们会在同一层（也就是 A、B、C 都在第二层）。B 和 C 也都有子依赖`core-js@^2.x.x`。按扁平化的操作来说他们会被安装在最第一层，但是第一层已经有了`core-js` 3.x 版本的，版本不在模糊版本控制的范围内，无法实现扁平化操作了（因为按顺序来说 `core-js@3.x.x`和 B、C 是在都属于 A 的子依赖，所以`core-js@3.x.x`的优先级比 B、C 中的 2.x.x 更高）。这时 npm 就会比较“笨”的将他们安装在各自目录下的 node_modules 下了，纵使它们是完全相同的（因为没地方放了）。

## 其他参考

- `sideEffects: false` ：告诉 tree-shaking 工具，这个包支持 ESM 规范（ESM 规范的包 webpack 打包时会直接将需要的代码插入行内），原来的包文件可以直接抖掉。



## 参考文章

关于现代包管理器的深度思考——为什么现在我更推荐 pnpm 而不是 npm/yarn?：https://juejin.cn/post/6932046455733485575

一文弄懂 npm & yarn 包管理机制：https://jishuin.proginn.com/p/763bfbd655cc

npm 包管理机制：http://www.conardli.top/blog/article/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%8C%96/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%8C%96-%E5%89%96%E6%9E%90npm%E7%9A%84%E5%8C%85%E7%AE%A1%E7%90%86%E6%9C%BA%E5%88%B6%EF%BC%88%E5%AE%8C%E6%95%B4%E7%89%88%EF%BC%89.html#_3-3-lock%E6%96%87%E4%BB%B6