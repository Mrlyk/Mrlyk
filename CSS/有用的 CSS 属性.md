# 一些有用的 CSS 属性

[toc]

## 传统属性

这里记录的都是在 chrom < 100 之前都可用的属性，符合主流，基本可以随意使用。

#### 多列布局（Multi-Column）

```css
.container {
  column-count: 3; /* 浏览器自动分成 3 列 */
}
```

#### 吸顶布局 position: sticky

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211222103554994.png" alt="image-20211222103554994" style="zoom:50%;" align="left"/>

 chrome >= 56

元素会吸附在最近的滚动祖先元素上，也就是说元素的定位是相对于**最近的**`overflow` **不是** `visible` 的祖先元素上。

```css
div {
  overflow: hidden;
}

div .child {
  position: sticky;
  top: 0;
}
```

如果像上面这样，最近的元素是 hidden，那么 child 就会相对于他来定位。这样往往达不到我们要的效果。

因为 div 元素会被滚动上去，子元素也会被滚动上去，**导致没有预期的吸附效果。**

所以要想达到预期，div 元素的 `overflow` 需要设置为 `visible` 。但是有时候我们需要隐藏溢出的部分，这就需要用到其他属性来配合了。比如`clip-path`

#### 设置长宽比 aspect-ratio

在实现这个属性之前，一般使用 padding-top 撑开元素的 hack 来实现。但是存在额外的计算开销。

```css
.container {
  width: 100%;
  aspect-ratio: 16 / 9;
}
```

- 如果同时设置了width 和 height 则该属性会被忽略，否责使用其中一个来计算另一个
- 该属性的优先级非常低，元素会优先计算最小宽度/高度，然后再来满足此属性。可以手动将 min-width/height: 0; 让元素不再基于内容计算宽高以满足此属性。

chrome >= 88

#### 首屏渲染优化 content-visibility 

基于CSS Containement（大多数浏览器支持）。在首屏渲染时，很多元素一开始是不可见的，不需要立刻渲染出来。通过该属性可以告诉浏览器这些信息

css containement 四个属性：

- `size`: 在计算该元素盒子大小的时候会忽略其子元素
- `layout`: 元素的内部布局不受外部影响，同时该元素以及其内容也不会影响到上级
- `style`: 声明同时会影响这个元素和其子孙元素的属性，都在这个元素的包含范围内
- `paint`: 声明这个元素的子孙节点不会在它边缘外显示。如果一个元素在视窗外或因其他原因导致不可见，则同样保证它的子孙节点不会被显示。

```css
.container {
  content-visibility: auto; /* 如果元素具有 auto 属性且没出现在屏幕上则不会立刻渲染它和它的子元素 */
}
```

- visible - 默认值，正常渲染
- hidden - 表现相当于 display: none; 跳过内容的渲染。

chrome >= 85

#### 混合模式 Blend Modes

描述元素重叠时，颜色如何呈现

```css
h1 {
	font-size: 100px;
	color: #fff;
	mix-blend-mode: difference;
}
```

- `difference` - 取重叠部分颜色的差值；如白(255, 255, 255) - 黑(0, 0, 0) = 白，重叠的部分会变成白色

- `darken` - 变暗

- `lighten` - 变暗

- `screen` - 滤色

  ......

#### 滚动捕捉

 `scroll-snap-type` 属性，子元素的 `scroll-snap-align` 属

- `scroll-snap-type:mandatory` 告诉浏览器，在用户停止滚动时，浏览器必须滚动到一个捕捉点。
- `scroll-snap-align` 可以指定元素的哪一部分吸附到容器上，`start` 指的是元素的顶部边缘。如果你水平滚动，它指的是左边缘。`center` 和 `end` 属性值与此同理。

#### 性能优化 will-change

告知浏览器即将渲染的动画，让浏览器做好准备，提升渲染速度

```js
// 当鼠标移动到该元素上时给该元素设置 will-change 属性
el.addEventListener('mouseenter', hintBrowser);
// 当 CSS 动画结束后清除 will-change 属性
el.addEventListener('animationEnd', removeHint);

function hintBrowser() {
  // 填写上那些你知道的，会在 CSS 动画中发生改变的 CSS 属性名们
  this.style.willChange = 'transform, opacity';
}
function removeHint() {
  this.style.willChange = 'auto';
}
```

#### backdrop-filter 

**`backdrop-filter`** [CSS](https://developer.mozilla.org/zh-CN/docs/Web/CSS/backdrop-filter) 属性可以让你为一个元素后面区域添加图形效果（如模糊或颜色偏移）。因为它适用于元素*背后*的所有元素，为了看到效果，必须使元素或其背景至少部分透明。

#### 裁剪显示区域 clip-path

举例说明

```css
.clip-path-example {
  clip-path: inset(10px 4px 10px 4px); /** 将元素的 上 右 下 左 裁剪 10px 4px 10px 4px */
  clip-path: inset(10px round 10px); /** round 可以用来创建圆角，也支持上右下左分别设置*/
}
```

裁剪矩形的规则和设置`margin`、`padding`类似，也可以缩写为两个或者一个。

## 新的工具

这里记录的都是比较新的属性，大部分都是在 chrome > 100 之后才可用，使用时要注意兼容性。

#### @container

官方文档：https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Container_Queries

CSS 为了更好的响应式布局，推出了 @container 媒体查询。

之前我们使用的媒体查询是相对于整个 viewport 视窗的，并不是很好用。

@container 则是相对于容器的媒体查询，使用分为两步：

1. 标记容器，容器查询会查找最近的容器，也可以使用`container-name` 来命名具体容器
2. 使用，使用方式和 @media 一致

下面是一个例子

```html
<style>
.post {
  container-type: inline-size;
}

  /* 默认的卡片标题样式 */
.card h2 {
  font-size: 1em;
}

/* 如果容器宽度大于 700px */
@container (min-width: 700px) {
  .card h2 {
    font-size: 2em;
  }
}

</style>

<div class="post">
  <div class="card">
    <h2>卡片标题</h2>
    <p>卡片内容</p>
  </div>
</div>
```

