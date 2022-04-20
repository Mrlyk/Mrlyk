# 解读 source map 与源码对应关系

经过打包工具打包后的代码一般都经过压缩和混淆，防止泄密。但是这种方式也会给调试带来不便，让我们无法精准定位异常发生的具体位置。source map，顾名思义就是对源码的映射文件。浏览器会分析代码的 source map 以呈现给我们未混淆前的代码。

[toc]

开始之前先要对 source map 有个基本了解

## source map 组成

如下是一个最简单的 source map 文件，他打包后的代码只有两句

*源码*

```js
const word = 'hello';
console.log(word);
```

*打包后代码* 

```js
console.warn("hello");
//# sourceMappingURL=index.d0b7cf56.js.map 这一句很重要，调试工具用他来查找 sourcemap 文件
```
在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/SourceMap) 上指出**在文件的响应头里添加 `SourceMap: <url>` 字段，也能指定文件的 map 位置**  

*打包后代码对应的 source map* 

```json
{
  "version": 3,
  "file": "index.d0b7cf56.js",
  "mappings": "AAEAA,QAAQC,KADK",
  "sources": [
    "webpack://webpack-demo/./index1.js"
  ],
  "sourcesContent": [
    "//@keep this is TestLoader \nconst word = 'hello'\nconsole.warn(word)\n"
  ],
  "names": [
    "console",
    "warn"
  ],
  "sourceRoot": ""
}
```

#### version

source map 版本，source map 到现在一共有 3 个版本的规范。

- 2009 年 Google 介绍他的一个编译器Cloure Compiler时，也顺便推出了一个调试插件Closure Inspector，可以方便调试编译后的代码，这个就是 sourcemap 的雏形
- 2010年制定了一些  source map 规范，并决定使用 base64 编码
- 2011年推出第 3 个版本，使用至今，主要是相比 2 压缩了 map 体积

#### file

source map 对应的源文件名（打包后的）

#### mappings

**源码和 map 对应关系的 base64 VLQ 字符串，也是最重要的内容** 

#### sources

转换前的源文件数组，比如`a.js`依赖于`b.js`，那这个数组里就会有`a.js`和`b.js`。当然示例中没有依赖关系，所以只有一个

#### sourcesContent

源代码，包含 sources 中所有源文件的内容（经过一定的压缩，但是没有混淆）。也正是因为 map 文件包含所有的源码，所有 map 文件总会比源文件要大

#### names

maps 中引用的标识符数组，以 AST 的角度来看就是源码中声明的 Identifier（但是不是所有的，有点奇怪？）

#### sourceRoot	

文件中**所有相对路径**的根路径地址吗，比如 file 中的源文件。sourceRoot 为空，可以知道 file 和 map 在同一个目录下（编译后的文件）。（后续修改该路径值并不会影响代码映射，不清楚其是否只是一个表示作用无实际意义？？？）

## source map 生成方法

生成 source map 的方法有很多，最常见的就是使用 webpack 开启 devtools 选项，即可自动生成 source map 文件。

当然 souce map 在有了规范之后随之产生了标准的 source map 生成工具

**根据上面说的 source map 标准生成 source map 的工具**：https://github.com/mozilla/source-map

除此之外，以下几种方式也可以生成 source map，具体可以参考：https://code.tutsplus.com/tutorials/source-maps-101--net-29173

- [Closure Compiler](https://developers.google.com/closure/compiler/) ：google 的 js 编译器，使用 js 写的 js 编译器，并不生成编译后的机器玛文件吗，只是对代码进行分析和优化。安装后可以使用命令`google-closure-compiler --js file.js --js_output_file file.out.js`生成编译后的 js 文件，也可以生成 source map
- [UglifyJS2](https://github.com/mishoo/UglifyJS2) ：就是我们经常在 webpack 中用来压缩代码的工具，也可以生成 source map。具体可以查看官方文档
- **TypeScript** ：ts 的编译器也可以在转换 ts 时直接生成 source map 文件。配置允许编译 js 文件的话也可以直接生成 js 文件的 source map

## source map 源码对应关系

source map 与源码的对应关系主要关注 map 文件中的两个属性`names`和`mappings` 

#### mappings 

```map
{
  "mappings": "AAEAA,QAAQC,KADK",
}
```

转换后的源码

````js
console.warn("hello");
````

在 map 文件中我们可以看到 mappings 属性是一个字符串，组成基本就是英文字母和逗号，其实还有分号。在 demo 代码中没体现出来。

- 分号`;` ：表示**行**对应，每个分号对应**打包后**源码的一行，如`"AAEAA;QAAQC"`，`AAEAA`就是第一行，`QAAQC`就是第二行。demo 中因为转换后的源码就只有一行，所以没有分号
- 逗号`,` ： 表示分段，一般如果使用了代码压缩工具，打包后的代码可能就一行了没有分号了。但是分段是肯定有的，分段是以标识符进行拆分。比如`console.log(a)`，这一句中就有三个 Identifier 表示符`console、log、a`，就会分成三段
- 字母：字母表示的是源码与打包后的代码之间的行列对应关系，内容是 Base64 VLQ 编码的（如果直接用数字会导致文件更大，用这种方式就是为了压缩体积）

其中最需要了解的是字母表示的对应关系。

#### 字母表示的对应关系

在上面的 mappings 中可以看到子母最长 5 个，短的也有4个。以 5 个字母为例，每个字母的对应关系如下：

1. 打包后代码的列数（**打包后的代码一般情况就一行了，所以行数一般省略了**，也可以通过分号来判断行数）
2. 对应的源代码文件，从 map 文件的 sources 数组中取，比如 0 就是`sources[0]`对应的文件
3. 源代码对应的行
4. 源代码对应的列
5. `names`里标识符的索引（**这里存在 names 中没有的情况，比如打包后把常量直接转换掉了。所以有时候会有 4 个字母的情况**）

**注意：行和列都是从索引 0 开始，并且对应的源码是`sourcesContent`中的。和我们自己写的源码比起来，可能 loader 或者一些其他工具会给源码中塞入一些其他东西。所以不要用自己的源码去找对应关系，而是真正的输入源码** 

*有时候会有更多个字母，比如`mBAIAA`，这是因为超出了 VLQ 编码的单字符表示范围{-15, 15}，所以需要多个字符来表示，具体可以看后面的转玛规则*  

**以 demo 为例** 

在我们的 demo 中，源码共有三行，第一行是我的 loader 加入的一个注释

```map
"sourcesContent": [
	"//@keep this is TestLoader \nconst word = 'hello'\nconsole.warn(word)\n"
],
```

打包后的代码 names 中有两项`console、warn`，word 直接被转成了常量，所以在 names 中没展示，但是在 mappings 中还是分成了三段。我们一个一个来看一看

- `console`：打包后的代码是**从索引 0 开始的（第 0 列）**，对应的 sources 中的**第 0 项**。在源码中对应**第 2 行**，**索引 0 （第 0 列）**。同时对应的是 names 中的**第 0 项**。所以通过打包后的代码和这个对应关系。我们就可以复现出源代码——源代码的第 2 行的开头是 console。**得到对应关系 0 0 2 0 0**
- `warn`：打包后的代码是**从索引 8 开始（第 8 列）**，对应 sourcs 中的**第 0 项**。在源码中对应**第 0 行**，**第 8 列**。同时在 names 中对应**第 1 项**。所以源码中第 2 行，第 8 列是 warn。**得到对应关系 8 0 0 8 1**

**这里可能有人有疑惑，为什么 `warn`在源码中是第 0 行，明明也是第 2 行啊。这里就说到另一个问题，Base64 VLQ 表示的数字范围是 {-15, 15}（超出就需要多个字符表示），所以为了防止数字过大。<font color="red">所有下一行的数据都是相对于上一行的相对数字</font>** ，上一行是第二行，这一次也是第 2 行，所以相对数字就是 0 。列同理。

- `hello`：打包后的代码**从索引 5 开始（相对于索引 8）**，对应 sources 中的**第 0 项**。在源码中对应第 -1 行（相对于上一次的第 0 行，也就是`console`中的第 2 行，**注意相对关系**），第 5 列开始（`word`）。由于被转换为常量了，所以在 names 中没有。所以在源码中第 1 行（2 - 1）第 5 列对应的是`hello`，也就是`word`常量。**得到对应关系5 0 -1 5** 

这样我们就获得了三组对应关系，将他们转换成 Base64 VLQ 编码就是 mapping 中的英文字母。
至于具体是怎样对应的在其他中简单说明。详细的可以查看参考文章或百度。主要是了解其中的对应关系即可。转换规则没必要记得太死。

- 00200 - AAEAA
- 80081 - QAAQC
- 50-15 - KADK

**Base64 VLQ 在线转码网站**：https://www.murzwin.com/base64vlq.html

#### 一图说明对应关系

![sourcemap映射-1](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/sourcemap%E6%98%A0%E5%B0%84-1.jpg?x-oss-process=image/resize,w_1000,m_lfit)   

## 总结

- source map 通过 Base64 VLQ 编码的 mappings 属性来映射打包后代码与源码之间的关系
- 通过`//# sourceMappingURL`或者响应头中的``SourceMap: <url>` `字段启用
- css 文件也可有 map 文件，它只表示一种对应关系，对语言没有要求

## 其他

#### Base64 VLQ 转码规则

- VLQ 编码以 5bit 为一组进行分段，不足 5 bit 的补0
- VLQ 编码是倒序的，其最后一位是符号位置（0正，1负）
- VLQ 编码以 1 表示后面还有数字（比 15 大的要有连续位），0 表示没有

以数字 10 为例

1. 转换为 2 进制：1010
2. 补充符号位：00000、10100，同时已经足 5 位无需补位
3. 倒序：整段倒序，10100、00000（因为第二段没数，所以全部补 0 了）
4. 补充连续位：10 比 15小（后面都是 0 ），没有连续位，所以补 0 -> 010100 000000
5. 转 Base64 编码：分段转码，010100 ->16 + 4 = 20，000000 -> 0，所以最终结果是 20，20对应的 Base64 字符是 U

得到结果 10 --VLQ--> U，

如果要编码转回数字，只需要倒序过来，并去除补充的符号位（最后一位）即可

#### 浏览器 devtools 开启 source map 支持

默认是开启的

![image-20220311153822010](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220311153822010.png?x-oss-process=image/resize,w_500,m_lfit) 

#### //# sourceMappingURL 注释的作用

在 webpack 打包方式中记录过**`//# sourceURL`**注释的作用，可以把文件加载到 source panel 中。`//# sourceMappingURL `的作用其实是相同的，只不过他加载的是源码对应的  map 文件，而`//# sourceURL`则直接加载指定的文件。

举个例子：

- `//# sourceMappingURL=index.d1422a24.js.map`：加载 map 文件，浏览器识别 map 文件，并加载 map 文件中 sources 对应的源码文件到 source panel 中以完成映射
- `//# sourceURL=index.d1422a24.js.map`：将 index.d1422a24.js.map 直接注入 source panel 中，浏览器不会解析他

#### Base64 编码字符集

![image-20220316170912650](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220316170912650.png?x-oss-process=image/resize,w_600,m_lfit)  

## 参考文章

来聊聊SourceMap：https://mp.weixin.qq.com/s/ELIzEaHoo8gpcICLYEj-4w 

source-maps：https://code.tutsplus.com/tutorials/source-maps-101--net-29173
