# css 自定义字体引入

有时候我们需要引入自定义的字体，或者引入图片式的字体，让我们更好的对样式进行更改。一般的引入方法如下

```css
@font-face {
  font-family: 'logo-font'; /* 定义引入的字体名 */
  src: url('logo.eot?8et5zy');
  src: url('logo.eot?8et5zy#iefix') format('embedded-opentype'), /* format 见下方说明*/
    url('logo.ttf?8et5zy') format('truetype'),
    url('logo.woff?8et5zy') format('woff'),
    url('logo.svg?8et5zy#icomoon') format('svg');
  font-weight: normal;
  font-style: normal;
}

[class^="we-logo-"], [class*=" we-logo-"] { /* 可以对 logo 进行更详细样式设置，这是图片不具有的优势 */
  /* use !important to prevent issues with browser extensions that change fonts */
  font-family: 'logo-font' !important;
  speak: none;
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  line-height: 1;

  /* Better Font Rendering =========== */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.we-logo-icon{ 
  margin-right: 5px
}
.we-logo-icon::before {
  content: "\e901";
}
.we-logo-text{
  font-size: 24px;
}
.we-logo-text::before {
  content: "\e900";
}

```

#### format

format ：字体的格式，主要用于浏览器识别，一般有以下几种

- **truetype格式(.ttf)** — Windows和Mac上常见的字体格式，是一种原始格式，因此它并没有为网页进行优化处理。
  浏览器支持：IE9+,FireFox3.5+,Chrome4.0+,Safari3+,Opera10+,IOS Mobile Safari4.2+

- **opentype格式(.otf)** — 以TrueType为基础，也是一种原始格式，但提供更多的功能。

  浏览器支持：FireFox3.5+,Chrome4.0+,Safari3.1+,Opera10.0+,IOS Mobile Safari4.2+

- **embedded-opentype(.eot)** — IE专用字体格式，**可以从 truetype 格式创建此格式字体** 
  浏览器支持：IE4+

- **Web Open Font格式(.woff)** — 针对网页进行特殊优化，因此是Web字体中最佳格式，它是一个开放的TrueType/OpenType的压缩版，同时支持元数据包的分离。

  浏览器支持：IE9+, FireFox3.5+, Chrome6+, Safari3.6+,Opera11.1+

- **SVG格式(.svg)** — 基于SVG字体渲染的格式。
  浏览器支持：Chrome4+, Safari3.1+, Opera10.0+, IOS Mobile Safari3.2+

对于@font-face而言，兼容性问题就是各浏览器所能识别的字体格式不尽相同。

#### #iefix 有何作用？

IE9 之前的版本没有按照标准解析字体声明，**当 src 属性包含多个 url 时，它无法正确的解析而返回 404 错误**，而其他浏览器会自动采用自己适用的 url。因此把仅 IE9 之前支持的 EOT 格式放在第一位，**然后在 url 后加上 ?，这样 IE9 之前的版本会把问号之后的内容当作 url 的参数**。至于 **#iefix 的作用，一是起到了注释的作用，二是可以将 url 参数变为锚点**，减少发送给服务器的字符。

#### 为何有两个src？

绝大多数情况下，第一个 src 是可以去掉的，除非需要支持 IE9 下的兼容模式。**在 IE9 中可以使用 IE7 和 IE8 的模式渲染页面，微软修改了在兼容模式下的 CSS 解析器，导致使用 ? 的方案失效**。由于 CSS 解释器是从下往上解析的，所以在上面添加一个不带问号的 src 属性便可以解决此问题