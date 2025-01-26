# viewport 与虚拟键盘

移动端开发中我们经常需要与 viewport 打交道，那么他和虚拟键盘有什么关系呢？



重要 API：https://developer.mozilla.org/zh-CN/docs/Web/API/VisualViewport

可以获取被虚拟键盘挤压后的视口高度。

以此我们可以做一些定位操作，如

- 将元素固定在虚拟键盘上

**fixed 定位的位置也是基于 viewport 的。** 