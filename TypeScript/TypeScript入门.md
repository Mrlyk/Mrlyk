# TypeScript 入门

[toc]

## 一、TypeScript 介绍

### 1、基本特点

- 以 `JavaScript`为基础构建的语言
- 一个 `JavaScript`的超集
- 可以在任何支持 `JavaScript`平台中执行
- 扩展了 `JavaScript`并添加了类型
- `TypeScript`不能被 `JavaScript`解释器直接执行，需要编译成 `JavaScript`执行

### 2、`TypeScript`增加了什么

- 类型
- 支持 ES 的新特性
- 添加 ES 不具备的新特性
- 丰富的配置选项

### 3、安装环境

`node install typescript -g`

使用时，因为浏览器不支持直接解析 ts， 所以可以使用 tsc 命令将 ts 文件转换成 js。
`tsc test.ts`
*默认情况下即使 ts 中存在一些错误也会编译成功，除非对编译进行配置。*

## 二、基本类型

|  类型   |       例子       |              描述               |
| :-----: | :--------------: | :-----------------------------: |
| number  |   1, -33, 2.5    |              数字               |
| string  |     'hello'      |             字符串              |
| boolean |   true, false    |             布尔值              |
| object  |  {name: 'Mike'}  |              对象               |
|  array  |    [1, 2, 3]     |              数组               |
| 字面量  |      其本身      |  限制变量的值就是该字面量的值   |
|   any   |        *         |            任意类型             |
| unknow  |        *         |         类型安全的 any          |
|  void   | 空值（undefined) |      没有值或者 undefined       |
|  never  |      没有值      |          不能是任何值           |
|  tuple  |      [4, 5]      | 元素，ts 新增类型，固定长度数组 |
|  enum   |    enum{A, B}    |        枚举，ts 新增类型        |

### 1、类型声明

在变量后使用 `:`声明，注意**声明类型都要小写**

```typescript
let a : number // 声明一个变量 a 为数字类型
let b : boolean = true // 声明一个变量 b 并直接赋值
let c : number | string // 声明 c 为 numer 或 string 类型（联合类型），`|`单竖线代表`或`
let d : string & number // 这样表示'且'，如果是这样没什么意义，但是可以用对象
let d1 : { name: string } & { age: number } // 这样就表示 d1 里面需要包含 name 和 age 属性。和 { name: string, age: number } 的作用是一样的

// 函数参数类型声明，在参数后直接声明。同时可以在括号后声明返回值的类型
function sum (a: number, b: number): number {
  return a + b
}
```

**函数的声明**

ts 中的函数，在规定了参数和参数类型后，无论是参数类型不对，还是**参数个数不对**都会提示语法错误。同时声明了返回值的类型后还会断返回值进行检查。

**类型的别名**

```tsx
let a: 1 | 2 | 3 | 4 // 如果一个变量的类型像这样很复杂很长，就可以使用别名来定义类型
type myType = 1 | 2 | 3 | 4  // 使用 type 定义别名
lat a : myType
```

#### 1.1 类型推断

在 ts 中进行类型**声明并直接赋值**时 `let a = 'string'`，ts 中会**自动推断**出 a 为 String 类型。

如果后续再赋值为其他类型也会报错。如果**只声明不赋值**则 ts 会把变量默认为 any 类型（**隐式的 any**）。

#### 1.2 TS 中的类型介绍

##### 字面量

使用字面量类型声明之后，就不能再赋其他值了，有点类似常量声明。一般应用场景为需要限制某个变量的值。

```typescript
let a : 10 // 声明 a 为字面量 10
let sex : 'male' | 'female' // 声明 sex 只能是 male 或者 female
```

##### any

任意类型，可以任意赋值，相当于对这个变量关闭了 ts 的类型检测，不建议使用。

##### unknow

未知类型的值，和 any 类似，但是比 any 更安全。

```typescript
let b : any
b = 3 
let s : string
s = b  // 这样赋值不会被检查出错误，但是 s 的类型却被 any 影响了，但是如果用 unknow 则会检查出类型问题
let e : unknow
s = e // 这时 e 的类型虽然也不明确，但是在直接赋值给 s 的时候会被检查出类型问题，比 any 更安全

// 如果确认要将这个 unknow 类型的变量赋值给 s，则需要自己进行逻辑判断，这时候就不会报错（类型收窄）
if (typeof e === 'string') {
  s = e
}
```

##### void

空值，函数未设置返回值时，默认的返回类型是 `void`。如果有 return 值会则会使用 return 值的类型。

```typescript
function a () {}  // void
function b () { return true } // boolean
function c ():void { return true } // 编译器会报错因为 void 要无返回值
```

##### never

表示永远不会返回结果，与 void 的区别是他真的没有返回结果，也是一种类型。比如函数执行到中间报错了，真的不会有任何返回了，那他的类型就应该是 never。

```typescript
function a () : never {
	throw new Error('错误！')
}
```

##### object

表示一个 js 对象，但是仅仅声明一个对象没有什么作用。这个对象可以是数组，可以是函数，在 ts 中可以做更细致的约束。

```typescript
let obj : object // a = [] ;  a =  {}; 都是可以的

// 更细致的约束 
let obj2 : {} // 和 object 一样，但是可以指定属性的类型
let b : { name: string, age: number } // 约束 b 对象 name 属性是 string 类型，age 是数字，可以多个
b.name = 1 // 这时编译器就会提示错误
b = { name: 'Mike' } // 同时约定了几个属性就要有几个属性，这里没有 age 属性也会报错

// 属性可选, 在属性名后加 '?' 符号表示属性可选
let c = { name: string, age?: number}

// 任意类型的属性
let d = { [key:string]: any } // 方括号中的 key 可以是任意字符（比如[prop: string]），js 对象的属性名也基本都是 string。方括号外面的表示属性的类型，这里是 any
```

*定义函数结构的类型声明*

````typescript
let fn : (a: number, b: number) => number; // 语法：（形参: 类型, 形参：类型 ...） => 返回值类型
````

##### array

```typescript
let arr : string [] // 字符串数组，数组的每一项都要是字符串
let arr : Array<string> // 和上面的作用一样，第二种声明方式
```

##### tuple

元组 - 固定长度的数组；存储效率比数组更高

```typescript
let arr : [numebr, string] // 声明一个只有两项的元祖
```

##### enum

枚举

```typescript
enum MyNumber {
  one,
  two,
  three
} // 声明一个枚举类型，里面含有 one、two、three 三项,
let a : MyNumber
a = MyNumber.one // a 就是 0，访问枚举类型的属性，未声明的情况下会从前到后按 0、1、2....排下去,同时访问第 0 项就是 one，第 1 项就是 two

// 如果手动声明了则会使用手动声明的值（反向映射操作失效）
enum MyNumber1 {
  one = 2, // 手动声明
  two = 'a' // 如果是字符则不会实现反向赋值操作，即只会有 MyNumber1.two = a ，不会有 MyNumber.a = two
}
```

#### 1.3 类型断言

##### as

有时候我们需要手动指定一个值的类型。

```ts
/*
* 类型断言
* 语法：
* 	as 语法
*   <> 尖括号语法
*/
// 或者使用 类型断言，告诉 ts 编译器这个 e 就是 string 类型
s = e as string
s = <string> e // 类型断言的第二种写法
```

一般具有以下用途：

1. 联合类型参数，如果不指定具体类型我们就只能使用公共的方法，但是可以通过`as`类型断言来确定类型并使用私有方法

   ```ts
   interface Cat {
       name: string;
       run(): void;
   }
   interface Fish {
       name: string;
       swim(): void;
   }
   
   function swim(animal: Cat | Fish) {
       (animal as Fish).swim();
   }
   ```

2. 将父类断言为更加具体的子类

3. 将 any 断言为一个具体的类型

**注意不要滥用类型断言，编辑器不会对不满足类型断言的对象发出警告。**

##### is（类型谓词）

使用 as 类型断言只能帮我们临时处理错误，is 则可以帮我们实现类型保护。

```ts
// 这个方法相当于告诉编译器，如果返回结果为 true，那么 val 是（is） string 类型
function isString(val: any): val is string {
  return typeof val === 'string'
}

function logIfString(val: any) {
  if (isString(val)) { // 这里即使再传入一个 any 类型的数据，在 isString 方法中会被编译器认为是 string
    console.log(value)
  }
}
```

我们可以使用它进行更深层次的控制

```ts
const getType = (val: unknown) => Object.prototype.toString.call(val).slice(8, -1).toLowerCase()

const isArray = <T>(val: unknown): val is T[] => getType(val) === 'array'
// 这里相当于告诉编译器，val 会是一个 string 类型的数组，所以每一项都具有 toLowerCase 方法
const foo = (val: unknown) => isArray<string>(val) ? val.map(item => item.toLowerCase()) : val
```

## 三、编译选项

`ts` 需要编译成 `js`后才能执行，所以可以对编译选项进行配置 

### 1、tsc 编译命令

```shell
tsc // 编译项目目录下的所有文件，需要项目根目录下有 tsconfig 配置文件

tsc myfile.js -w // 编译单个文件
	-w // 观察文件变化自动编译
```

- 不带任何输入文件的情况下调用`tsc`，编译器会从当前目录开始去查找`tsconfig.json`文件，逐级向上搜索父目录。
- 不带任何输入文件的情况下调用`tsc`，且使用命令行参数`--project`（或`-p`）指定一个包含`tsconfig.json`文件的目录。

### 2、tsconfig 常用配置

> tsconfig 和 jsconfig 都是用来配置 tsc 的编译选项，jsconfig 是 tsconfig 的子集

**单个 * 号表示任意文件，双 \** 号表示任意目录**

```json
{
  "include": ["src/**/*", "test/**"], // 定义要被编译的文件目录
  "exclude": ["./src/public/**/*"], // 定义被排除编译的文件目录
  "extends": "./config/base", // 定义要继承的配置文件
  "files": ["a.ts", "b.ts"], // 定义要被编译的文件
  "paths": { // 定义如何解析目录，类似于 webpack 的 alias
  	"jqyery": ["node_modules/jquery/dist/jquery],
  	"@/*": [
  		"/Users/mrlyk/xxx/xxx/xxx/project/src/*"
  	]
  }
  "references": [{ "path": "./tsconfig.node.json" }], // 引用其他配置文件
  "compilerOptions": {} // 编译选项
}
```

**重要的编译选项配置`compilerOptions`**

```json
{
  "compilerOptions": {
    "target": "ES3", // 指定项目中编译生成的 js 版本规范 "ES5"， "ES6/ES2015"， "ES2016"， "ES2017"或 "ESNext" 默认 ES3。会将语法降级到该版本，但未实现的功能如后面才有的 async/await 无法降级的会被原样保留，这也是和 babel 的区别。babel 会通过 polyfill 将不支持的语法实现降级
    "module": "es2015", // 指定编译的模块化规范  "None"， "CommonJS"， "AMD"， "System"， "UMD"， "ES6"或 "ES2015" 等。 默认值：target === es3 or es5 ? 'commonjs' : 'es6'
    "lib": ["dom"], // 用来指定项目中的库，一般情况下不用设置，在非浏览器环境下运行的时候可以指定要使用的库（可以随便写一个来看看提示可以用哪些库，比如 dom（dom 操作）、webworker...）
    "outDir": "./dist", // 编译后文件的输出目录，结构和项目的结构相同
    "outFile": "./dist/app.js", // 所有全局作用域中的代码会合并成一个文件，但是如果用了模块化之后就合并不了了。一般不使用这个选项而是交给打包工具处理
    "allowJs": false, // 配置是否编译 js 文件。默认是 false
    "checkJs": false, // 配置是否检查 js 文件的语法，和检查 ts 一样，也会有类型判定之类的。默认是 false，开启该选项的前提是 allowJs 配置要开
    "removeComments": false, // 是否移除注释，默认 false
    "noEmit": false, // 不生成编译后的文件，默认 false。只执行编译过程，可以用来检查语法
    "noEmitOnError": false, // 当出现错误的时候不生成编译文件
    "types": [ // 指定哪些类型声明文件在编译时被包含进来，默认是所有可见的都会
      "lodash",
      "/Users/mrlyk/Desktop/xxxx/app/node_modules/@types/webpack-env"
    ],
    
    "alwaysStrict": false, // 严格模式，默认不，如果想要在生成的 js 文件中使用严格模式则需要开启该配置。模块文件会自动启用
    "noImplicitAny": false, // 不允许隐式的 any 类型，默认 false。开启后如果在非主动声明变量的情况下使用参数，比如函数的参数，且不注明类型，则会报错。不开的话则不会
    "noImplicitThis": false, // 不允许隐式的 this，js 中 this 是调用时确定的。开启该配置后如果 this 的还没确定就使用则会报错。可以在要使用前声明，比如 this: Window, this: any
    "strictNullChecks": false, // 严格的 null 检测。比如使用 find 方法查找一个对象后再调用这个对象的方法。但是可能 find 没找到，就会导致报错。开启该配置后 ts 将会对这种情况进行检测，需要我们自己对 null 的情况进行判断
    "strict": false, // 所有的严格检查都打开，包括上面的所有常用检查配置以及一些没提到的
    "allowSyntheticDefaultImports": true, // 允许 import xxx from 'xxx'直接导入默认输出的包
  }
}
```

**compileOptions 中一些可能会用到的选项**

```json
{
  "sourceMap": false, // 生成代码对应的 map，会产生一个单独文件
  "inlineSourcemap": false, // 将生成的文件内嵌到 js 文件中
}
```

**module 和 target 的区别**

- module: 指定 ts 编译成 js 的模块化方案（ts 内置 corejs@2-参考我的 babel 文章说明）
- target: 指定 ts 编译成 js 的 ESMAScript 语法使用的版本，会通过 polyfill 实现降级（基于 corejs@2）

不管这两个使用的是哪个，都不影响项目中使用`imoport`还是`require`，因为最终编译是通过`tsc`，这两种`tsc`都可以解析，但是最终生成 js 用的哪种就由上面的`module`配置决定。

#### 可选 与 非空操作符号

- ? ：在 ts 中除了和 js 一样`?`用于三元运算符和可选链之外，还可以用于声明类或者参数是否可选。如在类中声明：`name?: string`，表示 name 这个属性是可选的。
- ! ：在 ts 中除了和 js 一样`!`操作符号还表示非空，即这个字段或参数一定要存在。`name!: string`表示 name 这个属性是一定存在的，即不能为 null 或者 undefined

如果我们开启了 tsconfig 的严格模式，那么在获取某些数据如 dom 元素，获取的结果可能是 null 或 undefined 的时候，ts 编译器会提示我们。这个时候就可以通过上面两个符号告诉编译器这个值是可选的还是一定存在的了。

```js
this.element = document.getElementById("food")!;
```

