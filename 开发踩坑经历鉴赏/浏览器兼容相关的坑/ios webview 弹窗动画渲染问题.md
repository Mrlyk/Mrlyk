# ios webview 弹窗动画渲染问题

## 现象

现象如下图，可以看到弹窗内容先完全展示了一下（闪烁了一下），然后又变成正常的从下方动画往上出来。

![2023-10-30 20.31.31](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/2023-10-30%2020.31.31.gif) 

## 原因

推测：在某些版本的 ios webview 浏览器中调用 GPU 渲染 CSS 样式中的 3D 动画（`tranlate3D`）是交给不同线程处理的，导致存在这样一个延迟。

先展示了整个弹窗，之后才解析处理交给 GPU 渲染动画，最后合成了这样一个结果。

## 解决方法

由于存在上面这样一个渲染延迟，所以我们需要我们的弹窗一开始是不可见状态。

```css
.pad-init {
  transform: translateY(100%); /* 这里不能再使用 translate3d(0, 100%, 0) 来处理 */
}
```

在延迟最短时间之后再开始执行动画，如果不延迟还是会立刻闪烁一下。

同时要让动画停留在最后一帧，否则弹窗出来之后又会消失了。

```css
.pad-animation {
  animation-delay: 0.01s;
  animation-fill-mode: forwards; /* 停留在最后一帧 */
}
```

## 参考文章

- [兼容性问题 —— ios webview内h5动画抖动问题处理 ](https://juejin.cn/post/7195208760904646717) 

