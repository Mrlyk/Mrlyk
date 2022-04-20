# AST 实践

> 在了解了 AST 的基础概念后，我们可以使用 AST 来做一些事情。比如打包时去除代码中一些无用的方法，类似 webpack 的 tree shaking。这里说明一些实践的做法。

先来看一下一个标准的函数解析成 AST 之后是什么样子的，有助于理解我们后续应该怎么操作。

```js
function a() {
  console.log('1', '2');
  debugger;
}
```

```json
{
  "type": "Program",
  "start": 0,
  "end": 53,
  "body": [ // 全局环境下的第一层代码块，第一个也就是函数声明本身
    {
      "type": "FunctionDeclaration", // 函数声明
      "start": 0,
      "end": 53,
      "id": {
        "type": "Identifier", // 标识符
        "start": 9,
        "end": 10,
        "name": "a"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [],
      "body": { // 嵌套的代码块，每一个代码块都会对应一个 body，这里就是函数内部的代码块
        "type": "BlockStatement", // 块级声明
        "start": 13,
        "end": 53,
        "body": [ // 表达式也被认为是一个代码块
          {
            "type": "ExpressionStatement", // 表达式声明
            "start": 17,
            "end": 39,
            "expression": {
              "type": "CallExpression", // 调用的表达式
              "start": 17,
              "end": 38,
              "callee": {
                "type": "MemberExpression", // 类型是内置的成员表达式
                "start": 17,
                "end": 28,
                "object": {
                  "type": "Identifier",
                  "start": 17,
                  "end": 24,
                  "name": "console"
                },
                "property": {
                  "type": "Identifier",
                  "start": 25,
                  "end": 28,
                  "name": "log"
                },
                "computed": false,
                "optional": false
              },
              "arguments": [ // 调用的表达式的参数
                {
                  "type": "Literal",
                  "start": 29,
                  "end": 32,
                  "value": "1",
                  "raw": "'1'"
                },
                {
                  "type": "Literal",
                  "start": 34,
                  "end": 37,
                  "value": "2",
                  "raw": "'2'"
                }
              ],
              "optional": false
            }
          },
          {
            "type": "DebuggerStatement", // debugger 声明，debugger 不被看作一个新的表达式，所以被放在了一个 body 中。如果是还有新的表达式则会被放到新的 body 中
            "start": 42,
            "end": 51
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```

上面的 AST 是在 https://astexplorer.net/ 在线生成的，使用的 acorn 解析器。

由上面生成的 AST 我们就可以想到我们可以做些什么，比如我要去掉所有的 console 方法，那我就遍历 AST 找到 identifier 是 console 的树节点，把他们都删除再转回JS。

所以 AST 实践过程中，我们要做的事就是四件：

1. 解析代码生成 AST 树
1. 遍历 AST 树找到我们要的节点
2. 对节点进行增删改
2. 将修改完的 AST 树重新转换成浏览器认识的 JS

*ps：本文后面介绍的方法都是基于成熟的工具，如果想了解原理，即不借助工具如何去操作 AST。那就要先自己写一个 JS 的解析器，暴露方法给自己用了😣。所以这里只说怎么用，后面会再用进阶文章看一看成熟的工具的原码中是如何做的。*

*pss：其实也不能说是不借助工具如何去操作 AST， 不使用工具都没办法生成 AST（自己写的工具也是工具），也就没有操作一说了...* 

那我们要如何使用工具进行这一步步的操作呢，let's start!!!



[TOC]

## 如何解析 JS -> AST 

在基础篇中就已经提过了很多社区方案来帮我们进行这两步操作，这里介绍两种最流行的方案，一种是 webpack 使用的 Acorn，一种是 babel 使用的 @babel/parser。他们都遵守 ECMAScript 的 `estree规范`

#### acorn

**安装** 

```shell
npm install acorn
```

**使用** 

`acorn.parse(code[, options])`

- code：要转换的源代码
- options（只列举常用配置）
  - ecmaVersion：转换的目标 ES 版本，默认 es2020
  - sourceType ：{ String } script/module，根据是否使用 import 猜测
  - onComment：{ Funtion } 传入一个回调函数，当解析到注释时触发，可以获取注释内容

```js
const acorn = require("acorn");

const myFunction = `
function test () {
  console.log('Hello')
}
`;
acorn.parse(myFunction, { ecmaVersion: 2015 }); // 生成的就是在基础篇中类似的语法树

```

解析的结果：

![image-20220121182715490](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220121182715490.png?x-oss-process=image/resize,w_600,m_lfit) 

*注：解析时会默认把注释删掉*

**acorn 配置插件** 

acron 要增加对一些特性的支持需要使用继承的的方法`acorn.extend`

```js
const {Parser} = require("acorn")

const MyParser = Parser.extend(
  require("acorn-jsx")(), // 支持 jsx
  require("acorn-bigint") // 支持大数
)
console.log(MyParser.parse("// Some bigint + JSX code"))
```



#### @babel/parser

**安装** 

```shell
npm i @babel/parser
```

**使用** 

`parse.parse(code[, options])`

- code：要转换的源代码
- options（只列举常用配置）
  - strictMode：{ Boolean }默认 false 是否开启严格模式，否则遵从代码中的规定
  - sourceType ：{ String } script/module，根据是否使用 import 猜测
  - attachComment：{ Boolean }，默认 true 是否将注释附加到相邻的 AST 节点，设置为 false 时将去除注释
  - allowImportExportEverywhere： { Boolean }，默认 false 是否允许 import、export 出现在非代码顶层位置
  - **plugins**：{ Array }包含要启用的插件的数组。比如开启 JSX 支持`plugins: ['jsx']`

```js
const parser = require('@babel/parser')

const myFunction = `
function test () {
  console.log('Hello')
}
`;
parser.parse(myFunction)
```

解析的结果：

![image-20220121184707958](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220121184707958.png) 

@babel/parser 解析的结果可以看到是和 acorn 类似，但是也有很多不同，这些不同会影响他们的遍历和解析，后面会讲到。

通过上面的两个工具可以将 JS 转换成 AST，也是当前的的主流方案。如果我们要写 webpack 的插件也是使用的 Acorn 这个工具。拿到 AST 之后就可以进行下一步操作了，遍历 AST 以找到我们要的树节点。

## 如何遍历 AST 树

在上面拿到 AST 树之后（返回一个树形对象的数据），其实我们已经可以通过原生的迭代方法遍历树了。但是树不可能只有一层，直接遍历的话会非常的麻烦。幸运的是上面的两个工具都提供了方便的遍历方法。

#### acorn-walk

acorn-walk 是 Acorn 官方提供的抽象语法树遍历工具，通过递归算法遍历每个树节点。

**安装** 

```shell
npm install acorn-walk
```

**使用** 

acorn-walk 遍历的方式有好几种，这里列举两种常用的

`walk.simple(AST, visitors[, base[, state]])` 简单遍历，**回调函数只有一个参数就是找到的树节点**

`walk.ancestor(AST, visitors[, base[, state]])` 简单遍历，**回调函数由两个参数，第一个是当前节点，第二个是当前节点的所有祖先节点数组** 

- AST：由 acorn 解析生成的 AST
- visitors：直译叫访问者，可以看作要访问的树节点的描述对象，比如我要访问 AST 中所有的字面量`Literal`，就可以在里面声明一个`Literal`属性，作为回调函数会将所有的`Literal`节点回传
- base：遍历的算法
- state：开始状态？不太明确

```js
const acorn = require("acorn");
const walk = require("acorn-walk");

const myFunction = `
function test () {
  console.log('Hello')
}
`;

const visitors = {
  Literal(node) { // 字面量类型的访问者。这里是 "hello" 这个字面量的节点
    console.log(`Found a literal: ${JSON.stringify(node)}`);
  },
  CallExpression(node) { // 调用表达式类型的访问者。这里是 console.log 这个表达式节点
    if (
      node.callee &&
      node.callee.object &&
      node.callee.object.name === "console"
    ) {
      node.arguments.push(
        JSON.stringify({
          type: "Literal",
        })
      );
    }
    console.log(`Found a Callee: ${JSON.stringify(node)}`);
  },
};

walk.simple(acorn.parse(myFunction, { ecmaVersion: 2015 }), visitors);

```

通过上面的`walk`方法即完成了我们对 AST 的遍历。在`visitor`对象声明对属性中，我们可以看到类似`token_name`的声明，他们是 acorn 可以识别的词法类型。我们要获取这种类型的回调也需要在`visitor`中作出相应的声明。除了例子中的`Literable`字面量和`CallExpression`调用表达式外还有很多，在附录中有简要说明。

在实践中如果不记得当前语法要找的节点是哪个类型，可以在在线网站上先转一下看看。

#### @babel/traverse

@babel/traverse 是 babel 官方提供的用来遍历 @babel/parser 生成的 AST 的工具。

**安装**

```shell
npm install @babel/traverse
```

**使用** 

`traverse.default(AST, visitors)`

```js
const traverse = require('@babel/traverse')
const parser = require('@babel/parser') 

const myFunction = `
function test () {
  console.log('Hello')
}
`;

const ast = parser.parse(myFunction)
const visitors = {
  CallExpression: function (path) { // Acorn 的 visitors 回调中参数是节点类型。在 babel 中是 path。表示两个 node 之间的连接对象。如果要访问节点本身则需要使用 path.node 
    console.log(path)
  }
}
traverse.default(ast, visitors) 
```

总的来说和 Acorn 基本差不多（毕竟一开始是从 Acorn 那里 fork 出来的）。

#### 注意

1. 工具之间是互不相通的。虽然他们都遵守`estree规范`，但是实现上会有所不同。比如起始的根节点不同，节点的描述属性上有差异等。所以不能用 acorn-warl 去遍历 @babel/parser 生成的 AST，反之亦然
2. 因为工具都遵守`estree规范`，所以他们描述节点类型大部分是相同的（babel 会有更多的类型），比如都使用`Identifier`类型来声明标识符
3. visitors 中的节点回调属性会在每次遍历到该类型的节点时触发（多次回调）。**Acorn 和 @babel/traverse 的遍历方法有所不同：**
   - Acorn：顺序遍历，**在遍历到存在子节点的父节点时，由子节点的最小叶子节点向子节点的根节点收拢**
   - @babel/parser：深度优先遍历法

**[visitor](https://en.wikipedia.org/wiki/Visitor_pattern) 访问者模式** 

acorn-walk 和 @babel/traverser 使用的节点访问方法都是采用的访问者模式。但是它们的访问方式有细微的不同，除了我上面说的遍历顺序。还有就是 **visitor 这种访问模式实际上是进入节点到最小叶子节点之后又返回到父节点上。所以我们在遍历时有两次机会来访问到同一个节点。**@babel/traverse 将其暴露了出来如下，acorn 没有暴露出来。

```js
// babel visitor
{
  CallExpression: {
    enter(path) {
      console.log('CallExpression enter');
    },
    exit(path) {
      console.log('CallExpression exit');
    }
  }
}
// babel 还支持对两种表达式使用同一种处理逻辑，使用'|'来串联，注意中间不能有空格
'DebuggerStatement|CallExpression': function (path) {
  console.log('DebuggerStatement｜CallExpression')
 },
```

**acorn、@babel/traverse 直接声明函数表达式时都是在 enter 时访问** 

通过遍历 AST 节点，加上一些逻辑判断，我们就可以找到我们要操作的节点了。那我们如何去对节点进行处理呢？

## 如何操作节点

我们还是按工具来区分操作方法，不同的工具有些许差异

#### Acorn

Acorn 本身并没有提供什么方便的 API 给我们。所以我们直接写原生的业务逻辑去操作即可，这里举两个例子

**删除节点** 

以统一删除代码中的 debugger 为例子

```js
const acorn = require("acorn");
const walk = require("acorn-walk");

const myFunction = `
function test() {
  // debugger;
  console.log("Hello");
}
`;

const ast = acorn.parse(myFunction, { ecmaVersion: 2015 });
walk.simple(ast, {
  BlockStatement: function (node) {
    let { body } = node;
    if (!body) return;
    node.body = body.filter((e) => { // 直接过滤掉 DebuggerStatement 这个类型的节点即可
      return e.type !== "DebuggerStatement";
    });
  },
});
```

**修改节点** 

以向`console.log`打印的参数中添加`World!`为例

```js
const acorn = require("acorn");
const walk = require("acorn-walk");

const myFunction = `
function test() {
  console.log("Hello");
}
`;

const ast = acorn.parse(myFunction, { ecmaVersion: 2015 });
walk.simple(ast, {
  CallExpression: function (node) {
    try {
      const { callee: { object: { name } } } = node;
      if (!name || name !== "console") return; // 判断当前是否是 console 的调用表达式
    } catch (e) {
      console.log(e)
      return false;
    }
    let { arguments } = node;
    console.log("arguments: ", arguments);
    if (!arguments) return;
    // 只需要放入节点的关键信息即可，像位置属性 start、end 可以不写（目前没观察到不写的影响，不写结果依然符合预期）
    arguments.push({ // 直接向参数列表中推入新的参数（也可以修改原来参数的值做到一样的效果）
      type: "Literal",
      value: "World!",
    });
  },
});
```

Acorn 就和我们平常操作对象的属性一样直接去处理即可。

#### @babel

@babel 比 Acorn 要先进一点，它**提供了 @babel/types 让我们可以方便的访问、操作节点**。这些 API 在我们编写 babel 插件的时候也很有用，在附录中会简要说明一些常用的方法，详细介绍查看我的《babel 插件编写》。

安装 @babel/types

```shell
npm install @babel/types
```

我们使用 babel 提供的 API 完成上面相同的操作

**删除节点**

```js
const traverse = require("@babel/traverse");
const parser = require("@babel/parser");

const myFunction = `
function test () {
  debugger
  console.log('Hello')
}
`;

const ast = parser.parse(myFunction);
const visitors = {
  DebuggerStatement: function (path) {
    path.remove(); // 直接调用 path 的 remove 方法即可删除节点
  },
};
traverse.default(ast, visitors);
```

**修改节点** 

```js
const traverse = require("@babel/traverse");
const parser = require("@babel/parser");
const types = require('@babel/types')

const myFunction = `
function test () {
  debugger
  console.log('Hello')
}
`;

const ast = parser.parse(myFunction);
const visitors = {
  // 整体思路和 acorn 差不多，区别是在添加节点时使用 babel 提供的 types 规范，使添加进去的节点符合 babel 的规范。如果和 acorn 一样直接往里面放入对象的话在这里也能达到预期效果（未确认是否会有其他影响）
  CallExpression: function (path) {
    try {
      let {
        callee: { object },
      } = path.node;
      if (!types.isIdentifier(object, { name: "console" })) return; // 使用 types 判断是否目标节点
      const { arguments } = path.node
      arguments.push(types.stringLiteral('World!')) // 使用types 格式化我们要放入的节点
      // 也可以使用直接替换第一个参数的方式，当然这就属于业务逻辑了
      // arguments[0] = types.stringLiteral('Hello World!');
    } catch (e) {
      console.log(e);
      return false;
    }
  },
};
```

可以看到总体思路是想同的，只不过 babel 提供了 @babel/types 这个包来帮我们更好的生成符合它 AST 规范的树节点。

babel 也提供了 @babel/template 工具来帮我们通过字符串直接生成 AST 树节点，比我们直接写要方便很多。实际操作中我们可以对照[在线生成](https://astexplorer.net/)的我们要转换的目标 AST 一步步处理，注意各种边界情况。

*Acorn 没有找到 @babel/types 类似的工具，考虑参照 @babel/types 写一个？* 

到此我们就完成了最重要的三部，解析生成 AST、遍历查找节点、修改节点。只剩下最后一步，将编辑后的 AST 重新转换生成 JS 代码。

## 如何将 AST -> JS

这里依然两个工具分开说明。将 AST -> JS 在前人帮我们造好轮子的情况下是非常简单的，我们需要知道他是怎么做的。说简单点就是和解析的时候反着来，反正 AST 就是对代码的抽象描述。这里实践只说怎么用！先了解怎么用，再去深入其原理。

#### acorn - escodegen

acorn 自身不提供将 AST 转回 JS 的 API，但 acorn 自身是遵守 ECMAScript 的`extree规范的`，所以我们可以使用同样遵守该规范的 AST -> JS 生成工具。

escodegen 是当前最流行 ECMAScript 规范的的 AST -> JS 生成工具

官方文档：https://github.com/estools/escodegen/wiki/API

**安装** 

```shell
npm install escodegen
```

**使用** 

`escodegen.generate(AST[, options])`

- AST：ECMAScript 标准的 AST
- options 仅列举常用的
  - comment: { Boolean } 默认 false，是否将注释输出到代码，前提是在解析的时候没把注释扔掉...
  - format: { Object } 格式化方法
    - newline: { String } 默认 '\n' 换行符
    - quotes：{ String } single | double | auto 默认 auto，输出代码的引号格式，默认单引号
    - compact: { Boolean } 默认 false 是否包含空白和换行符，开启后不会再包含空白和换行符

```js
const acorn = require("acorn");
const escodegen = require("escodegen");

const myFunction = `
function test () {
  // test
  debugger
  let a = 1;
  console.log('Hello')
}
`;
const ast = acorn.parse(myFunction, { ecmaVersion: 2015 });

console.log(escodegen.generate(ast, { format: { quotes: 'double', compact: true }}));
// function test(){debugger;let a=1;console.log("Hello","World!");}
```

除了 escodegen 之外还有一些其他工具也可以完成这项操作，比如[astring](https://www.npmjs.com/package/astring)。但综合下来 escodege 是体积最小速度最快的解决方案。（我已知的）

#### @babel/generator

babel 则贴心的提供了官方工具来让我们将 @babel/parser 解析生成的 AST 重新转换回 JS。虽然 babel 也遵守 `estree规范`，但正如前文所说的他们的实现有些不同，所以无法使用上面的 escodgen.

**安装** 

```shell
npm install @babel/generator
```

**使用** 

`generator.default(AST[, options[, code]])` 

- AST
- options 仅列举常用的
  - comments：{ Boolean } 默认 true，是否包含注释。和 acorn 一样，首先解析的时候要将注释保留下来，@babel/parser 默认保留了
  - compact：{ Boolean } 默认 false，空格换行是否保留
  - concise：{ Boolean } 默认 false，减少空白但没有 compact 减少的那么多，仍然会留下一些空白保证一定的可读性
  - minified：{ Boolean } 默认 false，是否开启代码压缩，开启后无论 compact 是否开启都会将空白部分去除
  - auxiliaryCommentAfter： { String } 在编辑的 node 末尾添加注释字符串，comments 要开启且这里的注释是加到我们编辑过的 node 的后面，相当于告诉别人我们为什么要在这里直接用 AST 做修改。如果我们没有编辑过的 node 这里也不会加上去
  - auxiliaryCommentBefore： { String } 在编辑的 node 开头添加注释字符串
  - decoratorsBeforeExport：{ Boolean } 默认 false，修饰输出文件。比如会去除掉 debugger、给注释单独换行

```js
const traverse = require("@babel/traverse");
const parser = require("@babel/parser");
const generator = require("@babel/generator");
const types = require("@babel/types");

const myFunction = `
function test () {
  debugger
  console.log('Hello')

}
`;

const ast = parser.parse(myFunction);
const visitors = {
  DebuggerStatement: function (path) {
    path.remove();
  },
  CallExpression: function (path) {
    try {
      let {
        callee: { object },
      } = path.node;
      if (!types.isIdentifier(object, { name: "console" })) return;
      const { arguments } = path.node;
      arguments.push(types.stringLiteral("World!"));
    } catch (e) {
      console.log(e);
      return false;
    }
  },
};
traverse.default(ast, visitors);
// 使用 generator
console.log(generator.default(ast, { concise: true, auxiliaryCommentBefore: 'edited'}, myFunction).code);
// function test() { console.log('Hello', /*edited*/ "World!"); }
```

到此我们的实践部分就完成了，掌握了基础和实践知识，我们就可以去编写一些插件了。熟练一点之后就可以看看 Acorn 或者 babel 的源码他们是怎么实现的，模仿写个小的解析器，深入学习。

## 作用域 scope

我们知道 JS 有多种作用域，如全局作用域，函数作用域（也被称为词法环境）。JS 中每当函数创建时都会产生新的词法环境，并且词法环境会包含当前函数声明所在的作用域（作用域链与闭包的原理）。在操作 AST 的时候我们尤其要关注的是其中的作用域 Scope，不要破坏他们之间的引用关系。

在 AST 中作用域可以用以下数据结构表示（使用 @babel/parser）

```js
{
  path: path,  // 当前节点的路径
  block: path.node, // 当前节点
  parentBlock: path.parent, // 当前节点的父节点路径
  parent: parentScope, // 父节点作用域，子作用域可以访问父级的
  bindings: [...] // 变量和作用域间的绑定关系
}
```

有了这些信息就可以知道当前这个变量是否被其他作用域引用，提升变量声明

- `path.scope.hasBinding('test')`：检查本地变量是否被绑定
- `path.scope.hasOwnBinding('test')`：检查作用域是否有自己的绑定
- `path.scope.generateUidIdentifier("uid")`：创建一个不会重复的 uid，参数固定就是 uid
- `path.scope.rename('test', 'myTest')`：重命名绑定及其引用
- `path.scope.parent.push(id, init: path.node)`：变量提升，`id = path.scope.generateUidIdentifierBasedOnNode(path.node.id)`，生成新的标识符 id 并把它推入父级，以完成变量提升

## 附录

#### Acorn 节点类型（常用的）

以下面这段代码做说明

```js
function test() {
  const obj = {};
  console.log('Hello');
  debugger;
  return;
}
```

- Program：根节点 - AST 树本身
- Identifier：标识符节点 - `test、console、log、obj` 
- Literal：字面量节点 - `Hello` 
- FunctionDeclaration：函数声明 - `function () {}` ，**在 Program 中 ** 
- FunctionExpression：函数表达式 - 例子中没有。`const a = function () {}`这种形式
- VariableDeclaration：变量声明 - `const obj = {}`，变量声明语句。其中 kind 属性表示是什么声明类型，这里就是`const`
- ObjectExpression：对象表达式 - `{}`
- ArrayExpression：数组表达式 - `[]`
- BlockStatement：块级声明 - `{ ... }`就是 js 中的块级作用域的范围，**在 FunctionDeclaration 的中** 
- ExpressionStatement：表达式声明 - `console.log('hello')` 整个表达式本身，**在 BlockStatement 中 **
- **CallExpression**：函数调用表达式 - `()`，例子中的`console.log()`调用，**在 ExpressionStatement 中 ** 
- MemberExpression：成员表达式 - `console.log`这句就是 CallExpression 的成员。computed 属性为 false 表示它的属性是一个 Identifier，为 true 表示它的属性是一个表达式
- EmptyStatement：空的声明 - 例子中没有。`;`就是一个单独的分号这样没有任何语句的块
- DebuggerStatement：debugger 声明 - `debugger`只有这一个，**在 BlockStatement 中 ** 
- ReturnStatement：return 声明 - `return` 只有这个一个，**在 BlockStatement 中 ** 
- IfStatement：if 声明，例子中没有，就是我们常写的`if`语句，其他还有`SwitchStatement、TryStatement、TrowStatement、ForStatemen...`等等类似的
- AssignmentExpression：赋值表达式 - `=`，注意和变量声明表达式区分开，在变量声明之后再赋值才是赋值表达式
- BinaryExpression：二元运算表达式 - `> !== ===` 运算符
- UnaryOperator：一元运算表达式 - `!、+` 运算符
- ConditionalExpression：条件运算表达式 - `a ? a : 0`三元运算符

**如果父节点有多个子节点那一般会将子节点放在父节点的 body 属性中，否则就是单独的一个子属性** 

常见的就是以上这些，有个大概的印象即可

#### @babe/types babel 节点类型

大部分和 Acorn 是相同的，但是 babel 在 Acorn 的基础上还扩展了一些。比如根节点是`File`文件节点，字面量不都是使用`Literal`而是区分变量类型如：`StringLiteral、NumericLiteral`。具体可以看下面的官方文档

官方文档：https://babeljs.io/docs/en/babel-types#api

**types 的一些操作方法** 

- `types.isIdentifiler(node, { name: "console" })`：判断是否标识符类型，类似的还有很多 `types.isVariableDeclaration、types.isisMemberExpression、types.isBinaryExpression`就不一一列举了。**注意一点：第一个参数接收的是一个节点不是 path，在 types 操作的时候特别容易混淆**
- `types.assertIdentifier(node, { name: "console" })`：和上面的 `isIdentifier`等价，像测试工具的语法
- `types.stringLitera(value)`：转换为对应方法类型的 node ，上面有判断这里就有转换。和 babel 提供的节点类型是一一对应的关系。当然转换成不同的节点类型需要的参数不同
- `types.identifier(value)`：转换为对应方法类型的 node
- `types.functionExpression(Identifier = null, params, body, generator = false, async = false)`：转换为函数声明表达式

#### @babel Path 操作

和 Acorn 中 visitors 的回调直接就是一个树节点。在 babel 中为了提供更多方便的 API 则进行了一层包装，这里说明一些常用的属性和方法

**常用属性** 

- `path.node`：返回当前 path 的树节点
- `path.node.id`：返回当前 path 树节点的 id。可以用于变量提升，通过 id 插值
- `path.node.type`：返回当前节点的类型，如`Identifier`
- **`path.node.name`**：返回当前标识符节点的 name，前提是他是一个标识符，否则会返回 undefined
- `path.node.value`：返回当前字面量的 value 同上
- **`path.node.params`**：返回函数声明的参数数组
- `path.node.operator`：返回二进制表达式的操作符，比如`a > 1`的`>`
- `path.node.left`：返回二进制表达式的左边节点，比如`a > 1`的`a`，注意返回的是个节点不是 path
- `path.node.right`：返回二进制表达式的右边节点，比如`a > 1`的`1`，注意返回的是个节点不是 path

**常用方法** 

*查找节点* 

- **`path.get('left')`：**和 path.node.left 类似，但返回是 path ，可以使用 path 路径上的方法如`replaceWith`。**注意如果 get 的参数在 path 上无意义或不存在，则返回的内容相当于`path.get('body')`返回的内容**
- **`path.traverse(visitors)`**：遍历路径上的子节点
- `path.getFunctionParent()`：获取最接近的父函数
- **`path.getStatementParent()`**：向上查找获取父节点的路径
- **`path.find((path) => path.isObjectExpression())`**：找到特定的路径，从当前节点往上找（包含当前路径），**返回 return 的值**（有点废话，但这里是用的回调函数，怕大家可能产生误解）
- `path.inList()`：判断是否有同级路径
- `path.getSibling(index)`：通过索引获取同级路径
- **`path.container()`**：获取路径的容器，包含所有的同级节点
- `path.key()`：获取路径所在容器的索引

*操作节点* 

- **`path.remove()`**：删除节点
- **`path.insertBefore\path.inserAfter`**：插入一个兄弟节点
- **`path.replace(types.binaryExpression("**", path.node.left, t.numberLiteral(2))`**：替换一个节点。**注意替换节点后，如果是同级节点或子节点（vistor 未访问到的后面的节点），vistor 会访问到替换的该节点。所以要注意不要造成死循坏了（不停的替换不停的访问）** 
- ` path.replaceWithMultiple([])`：替换多个节点
- `path.replaceWithSourceString(function a () {})`：用源代码替换节点，接收一个表达式（**不是字符串**），效率很低，不建议使用
- `path.parentPath.remove()` ：删除父节点
- `path.parentPath.replaceWith()`：替换父节点
- `path.ski()`：停止遍历

## 参考文章

acorn-walker：https://github.com/acornjs/acorn/tree/master/acorn-walk

使用 Acorn 来解析 JavaScript：https://juejin.cn/post/6844903450287800327

使用 AST 优雅定制你的代码：https://www.wynneit.cn/2021/04/12/ast/#%E4%BD%BF%E7%94%A8AST%E4%BC%98%E9%9B%85%E7%9A%84%E5%AE%9A%E5%88%B6%E4%BD%A0%E7%9A%84%E4%BB%A3%E7%A0%81

babel-plugin handbook：https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-introduction
