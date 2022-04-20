# plugin

webpack 自身可以说是建立在插件上的，它内置了数百个插件。很多核心逻辑并不由内部直接实现，而是通过插件的方式引入，让其灵活性达到了一个新的高度。这也是为什么 webpack 这么强大的原因。

这里不说明内置插件，主要对如何编写插件做一个说明。最好是对 tapable 有一个深入了解，会更加清楚 plugin 的作用机制。

[toc]

## 什么是 plugin？

从结构上看，plugin 是一个带有`apply`函数的类

```js
class MyPlugin {
  apply (compiler) { // webpack 构建流程中详细说明了 compiler 实例
  }
}
```

**通过`apply`函数的`compiler`参数，可以调用`hook`对象以注册 webpack 各种钩子的回调函数**，如下

```js
class MyPlugin {
  apply (compiler) {
    // compiler/compilation.hooks.<hookName>.tap/tapAsync/tapPromise(pluginName,(xxx)=>{/**dosth*/}) 具体使用可以参考 tapable 的文章
    // 在 compiler 对象触发 thisCompilation 阶段注册了一个同步的插件
    compiler.hooks.thisCompilation.tap('MyPlugin', (compilation) => {
      // 在 compilation 对象触发 optimizeChunkAssets 时注册了一个异步的插件
      compilation.hooks.optimizeChunkAssets.tapAsync('Myplugin', (callback) => { callback(); })
    })
  }
}
```

钩子的核心逻辑定义在[tapable](https://github.com/webpack/tapable)库中，webpack 的插件机制（也是最核心的机制）也是由 tapable 实现的。tapable 的相关知识在这里不再赘述，参考另外两篇文章。

## 什么时候会触发 plugin？

webpack 编译过程中的两个重要对象 Compiler 和 Compilation 都会触发不同的钩子（具体触发哪些看 webpack 的编译流程）。通过在这些钩子的触发事件中，即可加入我们自己的逻辑，也就是加入 plugin。有了 plugin 之后如何影响 webpack 的编译状态呢？

#### plugin 如何影响编译

在文章开始 plugin 的示例中，我们可以看到 webpack 将上下文信息（compiler）以参数的形式传入了 plugin。**虽然 hooks 的触发时机由 webpack 自身决定，但是透过传入的参数我们就可以对上下文进行修改以影响编译**

## plugin 中的常用对象

我们知道 plugin 是在 webpack 触发 hook 时调用的，这里说明一些常用的 plugin 中常用的对象

- `comiler.hooks`
- `compilation.hooks`

上面两个是核心中的核心，贯穿 webpack 整个流程，他们的 hook 属性上则挂载了各种 hook

- `compiler.hooks.contextModuleFactory`
- `JavascriptParser Hook`
- `NormalModuleFactory Hook`

下面这三个则是用的比较多的 hook

下面就对这些对象进行一一说明（虽然在编译流程中有对 compiler 和 compilation 的说明，强调重要性这里再做个简单说明）

#### compiler

compiler 对象中保存着完整的 webpack 环境配置，它通过 cli 或者 node api 传递的所有选项创建出一个 compilation 实例。

compiler 在 webpack 首次启动时创建，可以认为是单例的，通过它的如下属性可以进行一些操作

- `compiler.options`：options 属性中存储了上面说的 webpack 配置，比如`compiler.options.entry`就是 webpack.config.js 中配置的入口信息，还有 loaders、plugin 等等
- `compiler.inputFileSystem/compiler.outputFileSystem`：webpack 中文件操作相关模块，可以看作是 node 中 fs 模块的拓展。在 webpack 中我们尽量使用它来操作文件，以保持和 webpack 的一致性。比如使用 devServer 启动服务时，webpack 实际是将生成的文件写入内存中，如果我们还使用 fs 就无法和 webpack 保持一致。**该方法使用异步回调的形式，虽然由 fs 拓展而来，尝试使用 node 的 promisefy 方法 prosmise 化，报错了。内部有一些变更，比如用了缓存之类的**
- `compiler.hooks`：compiler 对象的各种钩子都在这个对象上，在这些钩子上注册事件即可在不同的声明周期中对编译产物进行操作

#### compilation

一个 compilation 对象代表一次资源的构建，比如通过 devServer 开发时，存在热更新。每次都会创建一个新的 compilation 对象。

compilation 对象会对*构建依赖图*中的所有模块进行编译，主要进行以下操作：

- loade 加载模块
- seal 封存模块
- optimize 优化
- chunk 分块
- hash 生成 hash
- restore 重新创建

具体查看编译流程。由于以上特性，在 compilation 对象中我们就可以**获取/操作本次编译当前模块资源、编译生成资源、变化的文件以及被跟踪的状态信息？？？同样 compilation 也基于 tapable 拓展了不同时机的 Hook 回调** 。这句话不明白可以直接看后面的例子

compilation 的主要属性如下

- modules：webpack 处理资源的最小单位（不是所有类型的 module 都是文件，有的就是一串代码），它是 set 类型的
- chunks：module 及其依赖的 modules 组成的一个大的对象
- assets：打包产生的文件
- hooks：compilation 也通过该属性提供各类事件的注册

*注：有个小坑，node < 14.17.0 时会无法打印出 compilation，这是由于 node 自身的 bug 引起的*

**在 webpack5 中直接提供了各类 API 给用户操作，可以参考官方 API 文档**

#### compile.hooks.contextModuleFactory

了解这个之前先要了解下 `require.context`[官方文档](https://webpack.js.org/api/module-methods/#requirecontext) 

**`require.context(directory, useSubdirectories, regExp, mode)`** 用于引入整个文件夹下的所有模块

- directory：要加载的目标文件夹，path
- useSubdirectories：是否检索子文件夹，默认 false
- regExp：匹配文件的正则表达式，比如 /\\.js$/
- mode：加载模式，默认同步 sync，其他参考官方文档

而这个 hook 则用于在 webpack 处理 module 中通过`require.context`引入整个上下文时的操作

```js
class DonePluginContext {
  apply(compiler) {
    compiler.hooks.contextModuleFactory.tap(
      'Plugin',
      (contextModuleFactory) => {
        // 在 require.context 解析请求的目录之前调用该 Hook，其他调用时机参考文档
        // 参数为需要解析的 Context 目录对象
        contextModuleFactory.hooks.beforeResolve.tapAsync(
          'Plugin',
          (data, callback) => {
            // 在这里就可以对解析进行操作，比如新增一个通用的模块
            console.log('data: ------------', data);
            callback();
          }
        );
      }
    );
  }
}

module.exports = DonePluginContext;
```

`contextModuleFactory.hooks`的调用时机可以参考[文档](https://webpack.js.org/api/contextmodulefactory-hooks/) 

总的来说这个钩子用的不多。有引入整个上下文的对象，自然也有普通对象的，也就是我们用的最多的直接`require`的，和他相关的钩子就是`compiler.hooks.normalModuleFactory`

#### compiler.hooks.normalModuleFactory

这个钩子则会在**每次** require 文件的时候触发，可以修改我们 require 的对象，具体看下面的示例代码

```js
class DonePluginNormal {
  apply(compiler) {
    compiler.hooks.normalModuleFactory.tap(
      'MyPlugin',
      (NormalModuleFactory) => {
        NormalModuleFactory.hooks.beforeResolve.tap(
          'MyPlugin',
          (resolveData) => {
            console.log(resolveData, 'resolveData');
            // 返回 false 的时候，依赖会被忽略掉。所以这里只会解析 ./src/index.js
            return resolveData.request === './src/index.js';
          }
        );
      }
    );
  }
}

module.exports = DonePluginNormal;
```

`normalModuleFactory.hooks`的调用时机可以参考[文档](https://webpack.js.org/api/normalmodulefactory-hooks/) 

还有一个`JavascriptParser Hook`，看名称也能知道，它是在解析 module 内容生成 AST 的时候调用的 hook

#### JavascriptParser Hook

一般这个 hook 会和 `normalModuleFactory`配合使用，因为在`normalModuleFactory`中可以获取到每个依赖的内容（但是要注意，其他插件也会被获取到）。使用可以参考以下示例

```js
const t = require('@babel/types');
const g = require('@babel/generator').default; // 后面可以交由 babel-loader 继续处理，所以这里可以使用 babel 来处理 webpack 中的 ast 树
const ConstDependency = require('webpack/lib/dependencies/ConstDependency');

class DonePlugin {
  apply(compiler) {
    // 解析模块时进入
    compiler.hooks.normalModuleFactory.tap('pluginA', (NormalModuleFactory) => {
      // tapable HookMap 生成一个 parser 钩子
      const hook = NormalModuleFactory.hooks.parser.for('javascript/auto');

      // 在上面生成的 paser 钩子上注册事件
      hook.tap('pluginA', (parser) => {
        parser.hooks.statementIf.tap('pluginA', (statementNode) => {
          const { code } = g(t.booleanLiteral(false)); // 把所有 if 中的块级声明替换为 false 
          const dep = new ConstDependency(code, statementNode.test.range);
          dep.loc = statementNode.loc;
          parser.state.current.addDependency(dep);
          return statementNode;
        });
      });
    });
  }
}

module.exports = DonePlugin;
```

## 编写 plugin

了解了 plugin 之后，就可以开始编写自定义的 plugin 了。编写 plugin 最大的一个问题是：

- 在那个对象的哪个钩子上注册我们的事件？compiler 和 compilation 都有很多 hooks，如何确认我们需要在哪个钩子上注册事件呢？

**这就需要我们对 webpack 编译流程中，各个阶段做了什么有一定的了解。**本文也会列举一些常用的 API 简单说明。具体查看《webpack 编译流程》

也可以参考官方文档说明：https://webpack.js.org/api/compiler-hooks/#environment 

#### 思路

编写 plugin 大致可以按照下面的思路走

1. **根据需求，找到在哪个对象上注册事件** 
2. 编写相关逻辑代码
3. 导出 plugin

## 常用 API

- `compilation.addModule`：添加模块，可以在原有的 `module` 构建规则之外，添加自定义模块
- `compilation.emitAsset`：直译是“提交资产”，功能可以理解将内容写入到特定路径
- `compilation.addEntry`：添加入口，功能上与直接定义 `entry` 配置相同
- `module.addError`：添加编译错误信息



## 其他

#### 有了 loader 为什么还要 plugin？

plugin 比 loader 提供了更完备的功能，使用阶段式的构建回调。同时 webpack 给我提供了非常多的 hooks 让我们在构建阶段有多个机会去引入自己的行为。

比如我们想在打包完成时提示用户某些事情，那就需要在打包完成事件的发布中注册钩子。这也是 loader 做不到的，loader 只能对编译内容进行处理。或者说它只存在于`compilation.addEntry`这一个阶段中。plugin 却可以在多个阶段中触发。



## 参考文章

官方 API 文档：https://webpack.js.org/api/compiler-hooks/#environment 

全方位探究Webpack5中核心Plugin机制：https://juejin.cn/post/7046360070677856292