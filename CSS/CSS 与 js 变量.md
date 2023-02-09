# CSS 与 js 变量

如果要在 css 中使用 js 变量，一般我们选择行内样式的方式。但是这样会产生大量重复的局部样式！

css in js 的一大优点就是可以方便的使用 js 中的变量。

css modules 则可以用下方法来共享 css 变量：

- 通过 `:import、 :export` 伪类做 css 变量的导入导出，
- 用 webpack-loader 实现 js 中引用 css 变量
- 用 css variable 实现 css 引用 js 变量。