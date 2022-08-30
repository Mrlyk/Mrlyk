# less

CSS 预处理器 less，提供了许多方便的功能，比如混入、函数、变量。让我们能编程式的编写 CSS，less 最终被编译为浏览器认识的 CSS 。这里主要对其使用做一个记录！

[toc]

## 安装

安装不需要多赘述，直接安装

- `less` 
- ``less-loader `

配置好即可

## 使用

使用会记录一些常用的和有用的 less 用法，类似于嵌套语法这种日常使用的就不再赘述了！

#### 变量 @

```less
/* 作为属性值*/
@width: 100px;

.header {
  width: @witdh;
}

/* 作为属性名 */
@selector: header;

.@{selector} {
  width: @width;
}
```

#### 混合 mixing ()

作用：声明公共样式，需要的直接混入

```less
.font {
  font-size: 18px;
  font-bold: 500;
}

.artical {
  .font(); /* 混合上面声明的 */
}

/* 支持嵌套的形式 */
.font {
  .button {
    width: 50px;
  }
}

.test-button {
  .font.button(); /*  嵌套式的引入 */
}

/* 支持传递参数 */
.font-weight(@weight) {
  font-weight: @weight;
}
```

#### 运算 + - * /

作用：变量值运算

```less
@width: 10px + 10px; /* 20px */

/* 不等同的单位换算会失效，优先级以 px -> cm -> rad -> %; 5% * 2 = 10px; */
@height: 2 + 5px - 3cm; /* 4px */
```

#### 转义 ~

作用：原样输出声明的值

```less
@min768: ~"(min-width: 768px)";

.header {
  @media @min768 { /*  @media (min-width: 768px)  */
    font-size: 1.2rem;
  }
}

/* 从 less > 3.5 开始可以直接使用 () 而不需要 ~ 符号 */
@min768: (min-width: 768px);
```

#### 映射 []

作用：复用一个单独的属性

```less
#colors() {
  primary: blue;
  secondary: green;
}

.button {
  color: #colors[primary];
  border: 1px solid #colors[secondary];
}
```

## less 函数

函数比较重要，单独说明。在 less 中分为内**置函数**和**自定义函数**。先看一下如何自定义函数，再说明一些常用的内置函数。

#### 自定义函数

**less 对函数的支持比 sass 差一点，需要我们在 webpack less-loader 选项中手动开启`javascriptEnabled: true`** ，这样才能使用如下的带有业务逻辑的自定义函数。

```less
.remMixin() {
  @functions: ~`(function() {
    const clientWidth = '375px';
    function convert(size) {
    return typeof size === 'string' ? 
    +size.replace('px', '') : size;
    }
    this.rem = function(size) { /* 通过 this 挂载 */
      return convert(size) / convert(clientWidth) * 10 + 'rem';
      }
})()`;
}

.remMixin();

.el-function {
  width: ~`rem("300px")`;
  height: ~`rem(150)`;
  background: blue;
}
```

#### 循环

**语法** 

- `loop`定义循环次数，`when`条件判断，符合进入函数，不符合不进入函数。之后次数`+1`，形成循环。
- `loop`函数调用，直接传值`1`。

```less
.loop(@i) when (@i < length(@bgcardList)+1){
  .backgroundcard(extract(@bgcardList, @i),extract(@bgcardList, @i));
  .loop(@i+1);
}
.loop(1);

```





## 参考

less 中文文档：https://less.bootcss.com/#%E6%A6%82%E8%A7%88

less 函数：https://juejin.cn/post/6902698973287907336