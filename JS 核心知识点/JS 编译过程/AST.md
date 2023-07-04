# AST（abstract syntax code）

> 抽象语法树

```text
在计算机科学中，抽象语法树或者语法树（syntax tree），是源代码的抽象语法结构或者树状表现形式，这里特指编程语言的源代码。
```

在说明 AST 之前，先了解一下我们所写的代码（源代码）是如何变成计算机可执行的机器语言的

```mermaid
graph LR
A(源代码)--字符流-->B(词法分析)
B--词法单元-->C(语法分析)
C--AST-->D(语义分析)
D--AST-->E(中间代码生成)
E--字节码-->F(机器无关代码优化)
F--字节码-->G(目标代码生成)
G--机器码-->H(机器相关代码优化)
H-->I(机器码)
```

流程中到中间代码生成都可以算作是前端编译的范畴（前端是指高级语言层面的），生成中间代码之后编译产生机器代码的过程就是后端编译的过程（后端指底层语言比如C、C++）。

*JS 和 WebAssembly的编译引擎是 `SpiderMonkey`，它决定了 JS 是如何编译产生机器语言的。官方文档：https://spidermonkey.dev/*

基本上所有的高级语言都遵循上面这一套流程，作为前端开发。我们首先应该对前端编译范畴的过程有一定的了解，有利于排查各种异常问题和**编写各类插件**。因为我们知道 webpack/babel 等 js 预处理工具也是**先将我们的代码转换成 AST 之后再处理的**。

JS 与其他语言相比的一大特性是他的编译不是发生在构建前的，而是执行前的几微秒，所以 JS 使用了 JIT 等很多方法来优化编译性能。

这里顺便介绍一下 JIT

```text
JIT(Just in time)即时编译，通常我们写的高级语言要运行每次都要经过上面一系列的编译过程之后才能运行。JIT 则是将我们频繁操作的方法或代码块即时编译成机器码并缓存起来，以提高编译效率。JIT 是高效的。（实际更复杂，这里只是简单说明）
```




[toc]

## 什么是 AST ？

AST 抽象语法树：一种用来描述我们的源代码的树形结构表示方法，树上的每个节点都表示源代码中的一种结构。抽象是指把代码进行了结构化的处理，转化为一种数据结构。

示例如下，左边是我的代码，右边是生成的 AST

![image-20220121113658392](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220121113658392.png?x-oss-process=image/resize,w_1200,m_lfit) 

*在线 AST 生成网站：https://astexplorer.net/* 

#### AST 的作用

AST 是一种清晰的树形结构。正是由于这种清晰的描述让我们在这里我们可以做很多代码层面很难做到的事，比如

- IDE代码格式化、错误提示、补全等
- 代码的混淆压缩
- webpack、babel、rollup 打包等
- TS、JSX 转换为 JS 
- vue、react 模版编译
- ...

#### 如何生成 AST

如我们开头所画的流程图，生成 AST 的第一步是将传入的字符流作词法分析（Lexical analysis），然后是语法分析（Syntacitc analysis）。我们就按照这个步骤一步步看一下 JS 的解析器是怎么做的。

ps：后面的分析以 `let a = 1;`为例进行分析

#### 词法分析

首先我们要清楚一些概念

**token-词法单元** 

官方定义

```text
A lexical token or simply token is a string with an assigned and thus identified meaning. 
```

意思是说：词法分析的里的 token 是指一个被**分配的可识别的字符串**。有点保留字的味道，但不是，它可以是任意被识别的字符串。

*这里的 token 和 web 程序中通信的 token 要区分开来，根本不是一个东西*

**被分配的可识别的字符串**：这个怎么理解呢？就是说解析器在解析字节流的时候是根据自己的一套正则表达式的规则去解析的，所以解析器会内置好如何识别这些字符串的正则规则。比如区分空格，区分保留字如 js 中的 const 等等规则。

```text
The first stage is the token generation, or lexical analysis, by which the input character stream is split into meaningful symbols defined by a grammar of regular expressions.
```

token 的结构是 `(token_name: String, token_value?: String )`这样的，具体可以看下面的例子

| token_name           | token_value          |
| -------------------- | -------------------- |
| identifier（标识符） | `a`,` b`, `x`, `y`, `color` |
| keyword（关键字）    | `if`,`return`       |
| spearator（分隔符）<br />包含一系列 punctuator（标点符号）等 | `(`,`}`,`;` |
| operator（操作符） | `+`,`>`,`=` |
| literal（字面量）<br />字面量在计算机中表示一种固定值的符号，如 Numeric<br />注意和 JS 中 这种 { } 声明式的方法区分开 | `1`,`true`,`"music" ` |
| comment（注释） | `//`,`/* */ ` |

了解了 token 之后再看一下最开始的例子

```js
let a = 1;
// 下面是 C 语言的 AST，比较标准，作说明用
// token: [(keyword, let), (identifier, a), (operator, =), (literal, 1), (sperator, ;)]
```

**词法分析就是对我们的源代码进行扫描，生成 token 的过程。token 包含了标识符、关键字...等等一系列关键信息** 

JS 中如果使用 Esprima 库进行词法分析生成 token，它生成的词法分析结果是下面这样的。

```js
[
  {
    "type": "Keyword",
    "value": "let"
  },
  {
    "type": "Identifier",
    "value": "a"
  },
  {
    "type": "Punctuator",
    "value": "="
  },
  {
    "type": "Numeric",
    "value": "1"
  },
  {
    "type": "Punctuator",
    "value": ";"
  }
]
```

*使用 Acorn 的话会更详细一些，太长了这里就不列了。*

生成了 token 之后相当于给源代码中的每个单词打上了标签，但是生成的 token 之间互相是没有关联的，所以接下来就要进行第二步，语法分析。

#### 语法分析

首先语法分析是做什么呢？还是先看一下官方定义

```text
which is checking that the tokens form an allowable expression. 
```

意思是说语法分析是检查上面生成的 token 能否组成一个允许的表达式。具体怎么做的一般有两种方法：

- [context-free grammar](https://en.wikipedia.org/wiki/Context-free_grammar) ：上下文无关法。简单来说就是递归的查找可以构成表达式的词法单元以及他们必然出现的顺序，如果符合就生成一个合格的表达式
- [attribute grammars](https://en.wikipedia.org/wiki/Attribute_grammar) ：属性法。这是一种比较语义化的处理方式，简单来说就是描述了某个 token 属性在语句结束符之前可以包含哪些属性值

*这里暂时不介绍语法分析是怎么实现的（太蓝了😭），如果想深入了解的话可以去看一下 Acorn 等工具的源码*

语法分析最终的结果就是将词法分析生成的 token 转换成 AST。我们还是以开头的例子为例，看下最终生成的结果，同时做一个通俗的说明（Acorn 解析生成）

```js
{
  "type": "Program", // type: 程序
  "start": 0, // 字符开始的位置
  "end": 10, // 字符结束的位置
  "body": [ // 解析的内容，一个代码块一个 body，可以嵌套
    {
      "type": "VariableDeclaration", // type：变量声明
      "start": 0,
      "end": 10,
      "declarations": [ // 声明内容
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 9,
          "id": {
            "type": "Identifier", // type: 标识符（变量名）
            "start": 4,
            "end": 5,
            "name": "a" // 变量名的值
          },
          "init": { // 初始字面量
            "type": "Literal", // type：字面量（值）
            "start": 8,
            "end": 9,
            "value": 1, // 字面量的值
            "raw": "1"
          }
        }
      ],
      "kind": "let" // 类型（keyword）：let
    }
  ],
  "sourceType": "script" // 源代码类型，是普通的 script 还是模块代码 module
}
```

**语法分析的局限性** 

在大部分语言的解析器中，语法分析的范围是非常宽松的。这是因为计算机语言的记忆力是有限的，换句话说，在平常我们编写代码的过程中，我们必须先声明变量然后再使用它。然而在语法分析的过程中，计算机语言是无法记住这个变量是否被声明过的，如果要遵循代码语言的规范就需要更强大的约束。而这种约束计算机语言又不能有效的解析。所以在编译的语法分析阶段，解析的策略是非常宽松的。

如果要做到代码语言的规范就要往下一步，语义分析。这里暂时只介绍道 AST 的生成，语义分析等后面进阶再表。

## 社区方案

ECMAScript 存在一个自己的 AST 规范

**estree：**https://github.com/estree/estree 

大部分成熟的框架都是使用的已有的 JS 解析器来生成 AST，像 webpack、 babel 使用的都是 [Acorn](https://github.com/acornjs/acorn) 。
下面是几个流行的使用 JavaScript 编写的 JavaScript 解析器：

- [Esprima ](https://github.com/jquery/esprima) ：第一个用JavaScript编写的符合`estree规范`的JavaScript的解析器，不原生支持 JSX
- [Acorn](https://github.com/acornjs/acorn) ：当前最流行的方案，生成的 AST 也是符合 `estree规范的`，性能也是当前最好的，不原生支持 JSX
- [UglifyJS 2](https://github.com/mishoo/UglifyJS) ：可以生成 AST，但主要用于压缩代码
- [@babe/parser](https://github.com/babel/babel/tree/main/packages/babel-parser) ：最开始 babel 是使用的 Acorn，后来经过自己的改善和升级变成了自己的 @babel/parser

除此之外还有一些其他强大的工具库

[recast](https://github.com/benjamn/recast) ：一个强大的解析工具，支持多种解析器，并且可以自定义传入需要的解析器，也可以用来生成 JS 的 AST

[swc](https://github.com/swc-project/swc) ：速度极快的 web 编译工具，用它重写的 babel 比原生的快 7 倍以上。它自己也说明是用来对标 babel 的。

## 补充知识点

#### CST 具体语法树

与抽象语法树相对的是具体语法树（Context Syntax Tree）。**抽象语法树是具体语法树简化为仅需表达代码含义的结果**。具体语法树中则包含更多的信息，比如运算符的优先级，多余的空格、注释等。但一般我们不需要包含这些信息，因为更底层的编译器会帮我们处理这个（如 YACC -  linux 上生成编译器的编译器）。所以一般在顶层编译成 AST （又快又小）即可。

*python 的编译器会先生成 CST 再转成 AST* 

## 参考文章

wiki 词法分析说明：https://en.wikipedia.org/wiki/Lexical_analysis

wiki 语法分析（解析说明）：https://en.wikipedia.org/wiki/Parsing

高级前端基础-AST：https://juejin.cn/post/6844903798347939853#heading-12
