# ESLint

[toc]

## 介绍

**官方文档**：http://eslint.cn/docs/user-guide/configuring

ESlint 被设计为完全可配置的，这意味着你可以关闭每一个规则而只运行基本语法验证，或混合和匹配 ESLint 默认绑定的规则和你的自定义规则，以让 ESLint 更适合你的项目。有两种主要的方式来配置 ESLint：

1. **Configuration Comments** - 使用 JavaScript 注释把配置信息直接嵌入到一个代码源文件中。
2. **Configuration Files** - 使用 JavaScript、JSON 或者 YAML 文件为整个目录（处理你的主目录）和它的子目录指定配置信息。可以配置一个独立的 [`.eslintrc.*`](https://eslint.bootcss.com/docs/user-guide/configuring#configuration-file-formats) 文件，或者直接在 [`package.json`](https://docs.npmjs.com/files/package.json) 文件里的 `eslintConfig` 字段指定配置，ESLint 会查找和自动读取它们，再者，你可以在[命令行](https://eslint.bootcss.com/docs/user-guide/command-line-interface)运行时指定一个任意的配置文件。

有很多信息可以配置：

- **Environments** - 指定脚本的运行环境。每种环境都有一组特定的预定义全局变量。
- **Globals** - 脚本在执行期间访问的额外的全局变量。
- **Rules** - 启用的规则及其各自的错误级别。

### 规则

- `"off"` or `0` - 关闭规则
- `"warn"` or `1` - 将规则视为一个警告（不会影响退出码）
- `"error"` or `2` - 将规则视为一个错误 (退出码为1)

## 配置项

对 eslintrc.js 的常用配置做个说明

#### root

默认情况下，ESLint 会在所有父级目录里寻找配置文件，一直到根目录，以根目录的配置文件为准。

有时我们不想这样做，开启此选项后，一**旦找到该配置文件就会停止继续往父目录中查找**

**使用**

```js
// .eslintrc.js
module.exports = {
  root: true
} 
```

#### env

定义**全局环境变量**，让我们在使用全局环境变量的时候不会报错。

比如在代码里使用`setTimeout`，实际上是在`windows`对象上的。如果不在 env 中声明`browser`环境，eslint 检查就会报错。

**使用**

```js
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    node: true,
    es6: true,
    'vue/setup-compiler-macros': true
  }
} 
```

#### extends

extends 相当于配置好的 eslintrc 文件，是一个风格的实践方案。里面可能还有其他的 plugin 甚至 extends

**使用**

```js
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:vue/vue3-recommended' // 后面的会覆盖前面的
  ]
}
```

#### plugins

plugin 插件主要是为 eslint 新增一些**具体的检查规则（rules）**，举个例子：`eslint-plugin-vue`就会对`vue`项目做一些定制的`eslint`规则

**使用** 

```js
// .eslintrc.js
module.exports = {
  plugins: [
    'eslint-plugin-vue'
  ]
}
```

#### parserOptions

支持 ES6 语法并不意味着同时支持新的 ES6 全局变量或类型（比如 `Set` 等新类型）。对于 ES6 语法，使用 `{ "parserOptions": { "ecmaVersion": 6 } }`；对于新的 ES6 全局变量，使用 `{ "env":{ "es6": true } }`。
`{ "env": { "es6": true } }` 自动启用es6语法，但 `{ "parserOptions": { "ecmaVersion": 6 } }` 不自动启用es6全局变量。

解析器选项可以在 `.eslintrc.*` 文件使用 `parserOptions` 属性设置。可用的选项有：

- `ecmaVersion` - 默认设置为 3，5（默认）， 你可以使用 6、7、8、9 或 10 来指定你想要使用的 ECMAScript 版本。你也可以用使用年份命名的版本号指定为 2015（同 6），2016（同 7），或 2017（同 8）或 2018（同 9）或 2019 (same as 10)
- `sourceType` - 设置为 `"script"` (默认) 或 `"module"`（如果你的代码是 ECMAScript 模块)。
- ecmaFeatures

  - 这是个对象，表示你想使用的额外的语言特性:
  - `globalReturn` - 允许在全局作用域下使用 `return` 语句
  - `impliedStrict` - 启用全局 [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) (如果 `ecmaVersion` 是 5 或更高)
  - `jsx` - 启用 [JSX](http://facebook.github.io/jsx/)
  - `experimentalObjectRestSpread` - 启用实验性的 [object rest/spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread) 支持。(**重要：**这是一个实验性的功能,在未来可能会有明显改变。 建议你写的规则 **不要** 依赖该功能，除非当它发生改变时你愿意承担维护成本。)

**使用**

```js
// .eslintrc.js
module.exports = {
  parserOptions: {
    parser: '@typescript-eslint/parser', // 使用 ts 编译器作为参数，供外层的解析器调用
    sourceType: 'module'
  }
}
```

**parserOptions.parser 和 parser 的区别**

有时候我们在 vue 项目中看到将`babel-eslint`设置在 `parserOptions.parser`上，他们的区别是什么呢？

这里只要记住`parserOptions`就是解析器的选项，**parserOptions 中的参数除了官方指定的一些必须要有的，也可以传入自定义的被外层的`parser`使用** 。比如 `vue-eslint-parser`的源码中就会去读取`parserOptions.parser`参数来作为 vue 文件中 script 标签 js 的解析器。所以二者可以认为是一个从属关系！！

在 vue 项目中，都会配置`plugin:vue/essential`的扩展，其中会引入 vue 的 eslint 解析器`vue-eslint-parser`，**而其他的如 babel 配置都需要配置在 `parserOptions.parser`中，防止对 vue 解析器的覆盖**

#### rules

rules 中规定具体的规则配置，他的优先级最高，会覆盖 extends 和 plugins 中的

**使用**

```js
// .eslintrc.js
module.exports = {
  rules: {
    'generator-star-spacing': 'off',
    'indent': ['error', 2] // 混合选项，需要查看文档的具体混合选项规则。这里的意思是缩进 2 格，否则 error
  }
}
```

#### globals

开头说了`env`可以引入默认的环境变量，但是对于自定义全局变量就需要在这里定义了，否则 eslint 检查会报错。

**使用**

```js
// .eslintrc.js
module.exports = {
  globals: {
    'Vue': true,
    'BMap': true,
    'GLOBAL_CONFIG': true,
    'U': true, // 全局@/utils/auth方法
  }
}
```

#### processor

预处理器。项目中的有的文件不是 js 格式的但是其中有 js 代码，要对这类文件中的 js 进行处理就需要配置预处理器

```js
module.exports = {
    "plugins": ["a-plugin"],
    "processor": "a-plugin/a-processor" // 使用 a 插件中提供的 a 预处理器，要使用 / 连接
}
```

#### parser

源码转 AST 的解析器，默认是 espree，也可以手动指定如

- `@babel/eslint-praser` 将代码生成 ast 的时候，会被转换成 ESLint 可以理解的 ESTree 兼容结构
- `@typescript-eslint/parser` 将 TypeScript 转换为 ESTree 兼容形式的解析器

## 行内注释禁用规则

- **禁止两句注释之间的所有规则** 

```js
/* eslint-disable */

alert('foo');

/* eslint-enable */
```

- 禁止下面一句代码的检查
```js
// eslint-disable-next-line
```

- 禁止指定规则

```js
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */
```

## QA

#### 当 eslint 无效时如何查看错误信息？

执行 `npx eslint file.js`可以查看到具体的报错信息
