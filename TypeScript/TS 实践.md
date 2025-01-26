# TS 实践

[toc]

## 一、基于 webpack 打包

#### 打包环境

**依赖**

```shell
npm i typescript -D
npm i ts-loader -D
```

**配置文件**

`tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "es2015",
    "target": "ES5",
    "strict": true
  }
}
```

有了上述基础环境之后就可以使用 webpack 进行打包了

## 二、实战

**在使用 ts 的时候，要具有和使用 js 时不同的思维。简单点说就是我们在编写 ts 的时候将方法与对对应起来，方法最好是在对象上而不是直接去定义，面向对象编程。**

一般来说三步走：

1. 定义目标类
1. ....

#### 别名

在 webpack、vite 等打包工具中配置别名后需要在`tsconfig.json`中配置`path`来解析路径，否则会提示找不到模块

```json
{
  "paths": {
      "@/*": [
        "./src/*"
      ],
      "@api/*": [
        "./src/api/*"
      ]
    }
}
```



## 三、JS 与 TS 的交叉 - 类型定义文件（*.d.ts）

使用 ts 时，由于很多库都不是用 ts 写的（主流的基本都还是 js 写的），导致这些第三方的库无法进行类型推导。比如直接在 index.html 中通过 script 标签引入 jquery。在 ts 文件中使用`${'#head'}`时，ts 会报错表示不知道什么是`$`。这个时候就要使用**类声明语句**了。**在项目中一般把这些声明语句归类放在一个文件中，约定的就是`*.d.ts`这种类型定义文件。**

需要配置正确的  `typeRoots` 或 `types` 选项，让 ts 能解析到这些文件。

#### 类声明方法

delare 用于声明一个全局变量、全局函数、全局模块等，它的作用是**告诉编译器这个变量、函数、或模块等已经在其他地方定义过了，不需要编译器再次检查他们的定义。**这种声明方法被称为“外部声明（External Declarations）”。

- `declare var` 声明全局变量
- `declare function`声明全局方法
- `declare class`声明全局类
- `declare enum`声明全局枚举类型
- `declare namespace`声明（含有子属性的）全局对象
- `interface` 和 `type`声明全局类型
- `export`导出变量
- `export namespace`导出（含有子属性的）对象
- `export default`ES6 默认导出
- `export =`commonjs 导出模块
- `export as namespace`UMD 库声明全局变量
- `declare global`扩展全局变量
- `declare module`扩展模块
- `/// <reference path="..." />`三斜线指令

**一般的声明方法:**

```typescript
// global.d.ts
declare const myObj {
  name: string
}

// export = 
interface CookieStatic<T extends object> { ... }
declare const Cookies: Cookies.CookiesStatic;
export = Cookies;

// 使用 declare global 可以在 npm 包或者 UMD 库的声明文件中扩展全局变量的类型
// 注意项目中不能使用会报错，只能在第三方库中用
declare global {
    interface String {
        prependHello(): string;
    }
}
export {};

// export as name space
declare function foo(): string;
declare namespace foo {
    const bar: number;
}

export as namespace foo;
export default foo;
```

这里说一下 namespace 的声明，了解一下即可。很少用，但是很多历史库可能用了）

```text
由于历史遗留原因，在早期还没有 ES6 的时候，ts 提供了一种模块化方案，使用 module 关键字表示内部模块。但由于后来 ES6 也使用了 module 关键字，ts 为了兼容 ES6，使用 namespace 替代了自己的 module。
随着 ES6 的广泛应用，现在已经不建议再使用 ts 中的 namespace，而推荐使用 ES6 的模块化方案了。

namespace 被淘汰了，但是在声明文件中，declare namespace 还是比较常用的，它用来表示全局变量是一个对象，包含很多子属性。
```

```typescript
// 注意使用 namespace 声明内置的全局变量会报错：重复声明。比如 declare namespace Window{ }
// 现在这个一般只用来对第三方包进行一些声明，下面以 jQuery 举例
// jQuery.d.ts
declare namespace jQuery {
  const version: number  // 声明含有一个属性 version
  function ajax(url: string, settings?: any): void // 声明 jQuery.ajax 方法
  
  // 嵌套声明
  namespace fn {
    function extend(object: any): void
  }
}
```

*`/// <reference path="..." />`三斜线指令*

用到的也不多，主要是用来编译。注释内容会被视为编译指令。有点像 webpack 打包时的
`/* webpackChunkName: "name1" */` 指令。

- `/// <reference path="..." />`：用于声明文件间的依赖。一般是浏览器加载文件的顺序不固定，但是某些文件的定义依赖于后面才加载完的文件，导致报错。这时候就可以用这个手动声明依赖。`///<reference path="testa.ts" />`
- `/// <reference types="..." />`：用于定义类声明文件的位置，编译期间 ts 会自动添加这类语句，我们也可以手动配置

声明完成后，ts 的语法检查不会再报错（前提是 tsc 的编译配置里要把声明文件包含进去）。

**如果通过 webpack 来打包依然检查到编译错误，一般就是 tsconfig.json 中没有手动`includes`声明文件**。因为使用 webpack 打包一把不需要声明 includes 文件，是由 webpack 进行的编译和打包。但是做类型检查时会出现问题。

#### 社区方案

一般成熟的第三方库自己提供了这类声明文件的依赖，需要我们单独安装。

比如 loadsh，js-cookie 等，可以直接安装相关的 ts 声明文件依赖：

```shell
npm i @types/lodash -D
npm i @types/js-cookie -D
```

大部分成熟的 js 库都可以直接试试安装`@type/xxx`这种定义文件。

如果库没有提供这种文件，则需要我们使用上面的声明方法自己定义了。

**`@types`和`typeRoots` 的编译时自动引入说明** 

默认所有*可见的*"`@types`"包会在编译过程中被包含进来。 `node_modules/@types`文件夹下以及它们子文件夹下的所有包都是*可见的*； 也就是说， `./node_modules/@types/`，`../node_modules/@types/`和`../../node_modules/@types/`等等。

如果指定了`typeRoots`，***只有***`typeRoots`下面的包才会被包含进来。 比如：

```json
{
   "compilerOptions": {
       "typeRoots" : ["./typings"]
   }
}
```

这个配置文件会包含*所有*`./typings`下面的包，而不包含`./node_modules/@types`里面的包。

如果指定了`types`，只有被列出来的包才会被包含进来。 比如：

```json
{
   "compilerOptions": {
        "types" : ["node", "lodash", "express"]
   }
}
```

这个`tsconfig.json`文件将*仅会*包含 `./node_modules/@types/node`，`./node_modules/@types/lodash`和`./node_modules/@types/express`。/@types/。 `node_modules/@types/*`里面的其它包不会被引入进来。

指定`"types": []`可以禁用自动引入`@types`包。

**注意，自动引入只在你使用了全局的声明（相反于模块）时是重要的**。 如果你使用 `import "foo"`语句，TypeScript仍然会查找`node_modules`和`node_modules/@types`文件夹来获取`foo`包。

## 其他工具

#### ts-node 直接运行 ts 文件

官方文档：https://www.npmjs.com/package/ts-node

通常我们需要先将 ts 转换为 js 才能在 node 环境中运行。使用 ts-node 可以直接执行 ts 文件（需要配合安装 typescript ）

```shell
ts test.js
```

其原理是在 require hook 中对 ts 类型的文件进行处理

```js
const originalJSLoader = Module._extensions['.js']
Module._extensions['.ts'] = function(module, filename) {
    // 修改代码
    module._compile(修改后的代码, filename); // 使用 ts 编译
}
```

实践中发现效率不是很高，最简单的 js 也需要 1 秒左右的转译时间

#### esno 直接运行 ts 文件

~~官方文档：https://github.com/antfu/esno~~

esno 已经被集成到 tsx：https://github.com/esbuild-kit/tsx，可以参考这个文档

**使用** 

```shell
esno test.ts
```

依赖于`esbuild-register`这个包，所以直接安装这个包，使用

```shell
$ node -r esbuild-register file.ts # 也可以执行
	-r, --require # 表示预加载模块
```

其原理和 ts-node 类似，都是对 require hook 的劫持然后编译 ts。

**优点是基于 esbuild 来解析文件，速度会比 node 原生加载快的多。** 

*ps: esno 支持 cjs 模块，如果要支持 es module，需要安装 esmo 来运行。* 
