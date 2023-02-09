# CSS 工程化方案

在项目中 CSS 由于没有作用域可言，容易导致很多问题。现代的前端开发中解决这个问题大致可以分为以下三类方案：

1. **CSS 命名方法论**：通过人工的方式来约定命名规则。
2. **CSS Modules**：一个 CSS 文件就是一个独立的模块。
3. **CSS-in-JS**：在 JS 中写 CSS。

其又有具体的一些策略如下，本文将对这些方案进行简单说明

![976317407-6069e77a17772_fix732](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/976317407-6069e77a17772_fix732.webp?x-oss-process=image/resize,w_800,m_lfit)  

[toc]

## CSS 命名方法论

顾名思义，命名方法论是在名字的基础上使用统一规范的方法。

#### BEM

Block Element Modified。

#### 

#### OOCSS





#### SMACSS



## CSS Modules

命名方法论固然不错，但是替考了开发者的心智负担，需要项目所有开发者共同遵守。而现代工程化方案更推荐使用工具来统一处理。

css-loader 则是最常用的工具之一

工程化方案中，我们使用 css-loader 来统一提取文件中的 css。当然他的功能远不止于此。

css-loader 官方文档：https://github.com/webpack-contrib/css-loader

可以看到他支持`modules`选项，用于开启 CSS Modules

一般配置如下：

```js
{
  test: /\.(c|sa|sc)ss$/i,
  exclude: /node_modules/,
  use: [
    'style-loader',
    {
      loader: 'css-loader',
      options: {
        // The option importLoaders allows you to configure how many loaders before css-loader
        importLoaders: 2,
        // 开启 CSS Modules
        modules: true,
        // 借助 CSS Modules，可以很方便地自动生成 BEM 风格的命名
        localIdentName: '[path][name]__[local]--[hash:base64:5]',
      },
    },
    'postcss-loader',
    'sass-loader',
  ],
}
```

当然除了原生的 CSS 还可以配置 Less、Sass 一起使用！

**CSS Modules 特性：**

- **作用域**：模块中的名称默认都属于本地作用域，定义在 `:local` 中的名称也属于本地作用域，定义在 `:global` 中的名称属于全局作用域，全局名称不会被编译成哈希字符串

- **命名**：对于本地类名称，CSS Modules 建议使用 camelCase 方式来命名，这样会使 JS 文件更干净，即 `styles.className`

- **组合**：使用 `composes` 属性来组合另一个选择器的样式，这与 Sass 的 `@extend` 规则类似。

  ```css
  .className {
    color: green;
    background: red;
  }
  
  .otherClassName {
    composes: className; /* 关键字 composes 实现组合 */
    color: yellow;
  }
  ```

- **变量**：使用 `@value` 来定义变量，不过需要安装 PostCSS 和 [postcss-modules-values](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcss-modules%2Fpostcss-modules-values) 插件。

**编译结果对比：**

```js
/* style.css */
:global(.card) {
  padding: 20px;
}
.article {
  background-color: #fff;
}
.title {
  font-size: 18px;
}
/*========使用=========*/
import style from './films/films.module.css'

<NavLink activeClassName={style.linkActive} to='/films/nowplaying'>正在上映</NavLink>

/*=======编译结果=============*/
<style>
  .card {
    padding: 20px;
  }
.style__article--ht21N {
  background-color: #fff;
}
.style__title--3JCJR {
  font-size: 18px;
}
</style>
```

**注意点**

1. 类名最好使用驼峰命名，使用中划线的形式会无法识别
2. 使用具体的自定义类名，如果使用标签选择器也不会被编译到本地，而是放到全局。（如果要用需要放在一个具体的类名下面再用）



## CSS-in-JS



#### styled-components 💅

官方文档：https://styled-components.com/

styled-components 是目前最流行的 CSS-in-JS 库，在 React 中被广泛使用。



## 参考文章

CSS 模块化方案探讨：https://segmentfault.com/a/1190000039772466