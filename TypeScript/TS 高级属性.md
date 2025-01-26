# TS 高级属性

[toc]

## 面向对象

具体的实物反应到程序中就是一个对象。比如洗衣机是一个对象，洗衣机有洗衣服的功能，有脱水的功能，具有一系列功能的实物就可以看作是一个对象。很多方法都可以基于这个对象来完成，我们不需要关心对象是如何实现的，我们只需要调用对象的这个方法即可。

```text
JavaScript 的设计是一个简单的基于对象的范式。一个对象就是一系列属性的集合，一个属性包含一个名和一个值。
```

#### TS 中的 class 和 JS 中的 class

TS 中的 class 和 JS 中的很相似，所以下面只讲一下 TS 中的 class 和 JS 中的区别

```typescript
class Person {
  /** 
  * 1、ts class 属性支持 readonly 标志，构造函数中初始化后，无法通过实例再赋值
  * 2、声明了属性之后一定要在构造函数中初始化，否则 ts 检查会报错
  * 3、ts 中可以使用抽象类
  * 4、ts 中可以使用 private/public 修饰符
  */
  readonly name: string // 和 js 一样，属性不一定要在类里面先声明，在构造函数的参数中定义也可以
  constructor (options: {name: string}) {
    this.name = name
  }
}
```

#### 抽象类

**抽象类**：指只能用来继承，不能实例化的类，抽象类中可以添加抽象方法

**抽象方法**：定义了方法结构，但没有具体实现的方法，认为是抽象方法。**只能定义在抽象类中，且子类必须对抽象方法进行重写**

- 抽象类中也可以对属性进行抽象声明，子类也必须实现这个属性且保持类型一致。当然使用了抽象声明后就不能使用 static 来声明是静态属性了。
- 抽象类中仍然可以声明普通的实例方法

```typescript
/**
* ts 支持使用 abstract 关键字声明抽象类
*/
abstract class Animal {
  name: string;
  age: number;
  constructor(options: { name: string; age: number }) {
    this.name = options.name
    this.age = options.age
  }
  // 抽象方法，没有具体实现，但是定义了无返回值
  abstract bark(): void
  //对属性抽象声明
  abstract type: string 
}
```

#### interface 接口

接口就是定义了的规范，用来约束子类，和抽象类很类似。

一般描述一个对象的类型，我们可以使用 type 来描述（type 用来定义别名）

```typescript
type myType = {
  name: string,
  age: number
}
const obj1 : myType = {
  name: 'lyk',
  age: 12
}
```

**接口则用来描述一个类的结构**，定义一个类中应该包含哪些属性和方法。可以和 type 一样当成类型声明去使用。

interface 像抽象类：

- 可以在 interface 中声明方法的结构，但是不能定义其具体实现
- 对于属性也是一样的，只能定义其属性，不能声明其值
- 与抽象类的区别是抽象类中仍然能定义普通的方法，在接口中所有方法都只能是抽象方法
- 接口可以继承自接口或类
- 抽象类的继承使用 extends 关键字，但是接口不是继承而是实现，使用 implements 关键字

```typescript
interface myInterface {
  name: string;
  age: number;
  sayHello() : void;
}

const obj2: myInterface = { // 当成类型声明使用
  name: "lyk_1",
  age: 25,
};
```

实际使用中用哪个来定义对象都可以，但是更推荐使用接口以方便扩展。

**如果要声明一个构造函数，需要具有 new 方法**

```ts
interface Person {
  new (options: any): Person;
  speak: () => void;
}
```

有时候认为是要声明一个 `constructor` 方法，实际不是这样，这点要注意！

**implements 使用类来实现一个接口**

在 ts 中可以使用 implements 来实现一个父类或者接口，用来实现的类需要实现父类或父接口中所有声明的方法，可以重写和拓展

实现：指满足父类或父接口的要求，即如果父接口中声明了`name : string`，那么在子类的实现中**必须包含该属性**

```typescript
interface myInter {
  name: string;
  sayHello(): void;
}

class Myclass implements myInter {
  name: string = 'myclass'
  sayHello(): void {
    console.log('mycalss')
  }
}
```

**implements 与 extends 的区别：**

- extends 继承一个类，implements 实现一个类或接口

**type 和 interface 的区别：**

- type 声明的别名具有唯一性，比如上面如果再声明一个`type myType = {}`会报错。但是接口不会，接口会将同名的进行属性合并，当然后面定义的接口和之前定义的接口中已定义的同名属性类型需要相同
- interface 可以被子类进行实现而不只是单单用来描述一个类的结构

##### extends 用作条件判断

参考：[TS关键字extends用法总结](https://juejin.cn/post/6998736350841143326)

extends 关键字用作条件判断时，如：`type P = 'x' extends 'x' ? string : number`

其意义是**满足前面类型需求的也一定满足后面的类型需求则为真（前面是后面的子）**。上面的`x`类型和后面的相同，肯定是满足的，所以最终返回`string`。

如果是这样

```ts
type P = 'x' | 'y' extends 'x' ? string : number
```

满足前面类型需求的`y`并不能满足后面的类型需求，所以上面返回的是`number`。

##### extends 与联合类型的分配律 

```text
对于使用extends关键字的条件类型（即上面的三元表达式类型），如果extends前面的参数是一个泛型，当传入该参数的是联合类型，则使用分配律计算最终的结果。分配律是指，将联合类型的联合项拆成单项，分别代入条件类型，然后将每个单项代入得到的结果再联合起来，得到最终的判断结果。
```

```ts
type P<T> = T extends 'x' ? string : number
type myT = P<'x' | 'y'> // type myT = string | number

// 等价于
type myT = 'x' extends 'x' ? string : number | 'y' extends 'x' ? string : number
```

上面的 T 指代的实际是正在遍历的成员元素，不能理解为完整的联合类型！！！

**注意 never**

1. never 是所有类型的子类型
2. **never 被认为是空的联合类型** 

基于第一点，`type P = never extends 'x' ? string : number` 会返回 `string` 类型

基于第二点，`type P<T> = T extends 'x' ? string : number` ，因为 never 是空的联合类型，所以进行上面说的分配计算后发现没有可以分配的值，也就不会有最终结果，**返回的一直是`never`** 

**注意 any**

- any 也被认为是所有类型的子类型

但是和 never 不同的是 any 会返回”符合“和”不符合“两个条件的 数据

```ts
type P = any extends number ? true : false
// 等价于
type P = (number extends number ? true : false) | (no-number extends number ? true : false) = true | false = boolean
```

**消除分配律**

有时候我们并不想要这种分配律特性，我们就希望他联合判断，这时候怎么办呢？——使用`[]`

```ts
// 具有分配律的写法
type ToArray<Type> = Type extends any ? Type[] : never; //
type StrArrOrNumArr = ToArray<string | number>; // 结果是：`string[] | number[]` 即要么是数字数组要么是字符串数组

// 消除分配律的写法
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;
type StrArrOrNumArr2 = ToArray<string | number>; // 结果是：`(string | number)[]` 即数字或字符串数组
```

##### extends 与对象类型的判断

我们很容易混淆 extends 与联合类型的判断和 extends 与对象类型的判断。

- 对于联合类型：使用分配律判断前面的是否都是后面的子集
- 对于对象类型：**判断的原则仍然不变，前面的是后面的子。但是要注意“子“的概念是父拥有的子都要有，同时子可以拥有一些自己的扩展方法**

```ts
type UselessType<T extends {a: 1, b: 2}> = T;

type Test1 = UselessType<{a: 1,b: 2, c: 3}> // 符合
type Test1 = UselessType<{a: 1}> // 不符合，子没有继承父所有的
```

**注意不要混淆联合类型和对象的判断！！！** 

##### 父子类型关系图

从上往下，父 -> 子

![image-20230704100049846](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20230704100049846.png?x-oss-process=image/resize,w_600,m_lfit) 

#### 属性的封装

类声明后，一般情况下我们可以任意修改属性的值，只要类型符合即可。但是有些属性是不能被随意更改的，除了在构造函数中进行逻辑判断限制，还可以像其他强类型语言一样，使用`private、public`声明私有和公有属性。

- private：声明私有属性，只允许类内部访问。可以在类内部添加方法来编辑该属性，同时做相应的逻辑验证。这样逻辑就不会都写在构造函数中，而是按功能拆分了。
- public：允许所有其他实例化对象访问
- protected：声明受保护的属性，仅允许类自身以及继承他的子类访问

```typescript
class Person {
  name: string
  private _age: number // 声明私有
  constructor(name: string, age: number) {
    this.name = name
    this._age = age
  }
  // getAge、setAge 被称为属性的存取器
  getAge() : number{ // 内部才可以访问
    return this._age
  }
  setAge(age: number):void {
    if (age <= 18) {
      this._age = age
      return
    }
    console.log('这个人还没超过18岁呢')
  }
}

const per1 = new Person('lyk', 18)
per1.setAge(10)
per1.setAge(19)
console.log(per1)
// console.log(per1._age); // 编译错误，私有属性无法访问
console.log(per1.getAge())
```

**属性的存取器 setter/getter**

存取器是 ts 和 js 都具有的，使用如下

```typescript
class Person {
  name: string;
  private _age: number;
  protected type: boolean
  constructor(name: string, age: number, type: boolean) {
    this.name = name;
    this._age = age;
    this.type = type
  }
  get age() : number {
    return this._age
  }
  set age(val) { // 注意 set 不能有返回值声明
    if (age <= 18) {
      this._age = age
      return
    }
    console.log('这个人还没超过18岁呢')
    this._age = val
  }
}

const per1 = new Person("lyk", 18, true);
console.log(per1.age) // 这是当我们访问私有属性的 _age 时相当于时调用了 get age 方法
```

好处是在使用时符合我们的使用习惯，不需要在访问一个属性时去调用一个方法，不够便捷。

存储器可以用于任何类型的属性，但用在 public 上没什么意义......

##### public 等修饰器在 ts 中的简写作用

在声明一个类时，我们通常需要外部传入一些属性并保存在实例上，如下

```ts
class Person {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}
```

这样写有一些繁琐，name 和 this.name 实际上是重复声明了 name 属性。ts 通过 public 等修饰器提供了一种简写方法，如下

```ts
class Person {
  constructor(public name: string) {
  }
}

// 编译结果 js
class Person {
  name;
  constructor(name) {
    this.name = name;
  }
}
```

除了`public`修饰符，构造方法的参数名只要有`private`、`protected`、`readonly`修饰符，都会自动声明对应修饰符的实例属性。



#### 泛型

当遇到类型不明确的变量时，可以使用泛型。如我们不确认后端返回的参数的类型时，就可以使用。

```typescript
// 如下使用 尖括号 包裹，其中的字母任意，不一定是 T。只要前后对应即可
function fn2<T>(a: T): T{
  console.log(a)
  return a
}

fn2(10) // 可以直接调用方法，ts 会进行类型推断
fn2<number>(12) // 也可以手动声明泛型来指定
```

上面这个函数的目的是原封不动的返回传入的参数，如果我们直接使用 any 那就关闭了类型检查。而使用泛型就可以保证参数与返回值的类型一致性，不会破坏函数的初衷。

还可以同时声明多个泛型

```typescript
function fn3<X, Y>(a: X, b: Y) : X { // 使用逗号隔开
  return a
}

fn3<number, string>(123, 'Hello')
```

**泛型也可以继承自 interface 或者 class，用来限制泛型的范围**

```typescript
// 继承自 interface 
interface inter {
  length: number
}

function fn4<T extends inter>(a: T) : number { // 注意这里统一就是使用 extends 关键字，而不是 implents。因为他是使用 interface 描述对象结构的本质，而不是去实现这个接口
  return a.length
}
fn4<inter>({length: 123})
fn4('string') // 这里传个 字符串 也行，因为字符串的原型链上具有 length 属性

// 继承自 class
class Fan {
  length: number = 12;
}

function fn5<T extends Fan>(a: T): number {
  return a.length;
}

fn5({ length: 24 });
```

如果是继承自 class ，私有属性不能访问的这些特性依然会保留

当然泛型不仅可以用在变量声明上，像类的声明在不确定属性的类型时也可以使用泛型

```typescript
class Fan2<T>{
  name: T
  constructor(name: T) {
    this.name = name
  }
}
const fanclass = new Fan2<string>('haha')
```

**TS 会在真正调用时对泛型对类型进行推断**

总之泛型就是用来给不明确的变量一个声明。

#### 映射类型

在 ts 入门中我们声明对象时，没必要重复声明所有的项，只要符合某个映射规则即可，这就是映射类型。

```ts
type OnlyBoolsAndHorses = {
  [key: string]: boolean; //映射类型 [key: string]
};
 
const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

#### 索引类型访问 Indexed access

我们可以像访问普通对象一样，直接获取某个类型

```ts
type Person = {age: number, name: string}
type Age = Person['age'] // type Age = number

type Age = Person['age' | 'name'] // type Age = number | string
type Age = Person[keyof Person] // type Age = number | string
```

**我们还可以通过`number`关键字来访问数组中的元素，如下：**

```ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number] // typeof {name: 'Alice', age: 15} -> {name: string, age: number}
```

如果使用范型数组 `T[number]` 这样访问 `number` 属性，可以认为返回的是一个联合类型数组。

## 类型收窄

类型收窄介绍的官方文档：https://www.typescriptlang.org/docs/handbook/2/narrowing.html 

当我们像这样声明一个变量时

```ts
const OBJ = {
  name: 'yk',
}
```

我们会认为 ts 将 OBJ.name 推断为 'yk' 类型，实际上它会被推断为 string 类型。有时候我们需要让 ts 认为这就是一个固定的值就是 'yk'，希望收窄这个类型。那应该怎么做呢？下面说明几种在 ts 中收窄类型的方法。

#### as const 断言

使用 as const 断言可以很好的解决上面的问题。

```ts
const OBJ = {
  name: 'yk' as const,
}
```

这样 name 就会被推断为具体的值而不是 string 类型了。

官方文档中介绍了很多收窄类型的方法。比如用 typeof 做判断，用 === 值判断，使用 is 操作符等。详细的可以直接看官方文档。

## 协变与逆变

**参数为父，返回值为子 —— 参数是逆变，返回值是协变**。如何理解呢，看下面的例子：

假设我有如下三种类型：

> ```
> Greyhound ≼ Dog ≼ Animal
> ```

`Greyhound` （灰狗）是 `Dog` （狗）的子类型，而 `Dog` 则是 `Animal` （动物）的子类型。由于子类型通常是可传递的，因此我们也称 `Greyhound` 是 `Animal` 的子类型。

**问题**：以下哪种类型是 `Dog → Dog` 的子类型呢？

1. `Greyhound → Greyhound`
2. `Greyhound → Animal`
3. `Animal → Animal`
4. `Animal → Greyhound`

第一种：假设传入的是一只其他品种的狗，符合`Dog -> Dog` 但是不符合`Greyhound`， 所以不是；

第二种：同第一种，不是；

第三种：传入任意品种的狗都是`Animal`，所以参数没问题。但是返回值可能不是狗，而是其他动物，不符合`Dog`。所以也不是；

第四种：同第三种参数没问题，返回值`Greyhound` 是狗的一种，也符合。所以他是`Dog -> Dog` 的子类型；

## 函数重载

在 ts 中，有时会有这样一种场景：我的一个函数参数不确定，但是后面要依据这个参数类型做不同的事情。所以泛型在这里也不



## 常用操作符

#### infer - 泛型推断

- infer： 表示在 extends 条件语句中**推断**的类型变量

```typescript
type Pa<T> = T extends (...args: infer P) => any ? P : T;
interface User {
  name: string;
  age: number;
}

type Func = (user: User) => void;
/**
* 如上面的声明，Func 这个别名声明符合 (...args: infer P) => any
* ...args -> user / P -> User 
* 所以最终 T 推断出类型是 P，也就是 User
* 同理 Param2 是个字符串类型，不是符合结构的函数，所以直接返回了这个字符串类型
*/
type Param = Pa<Func>; // Param === User 
type Param2 = Pa<string>; // Param2 === string
```

上面的例子中 infer 是用在参数位置。除此之他也可以用在其他的位置上

```typescript
// 用在返回值位置上
type ReturnType<T> = T extends (...args: Array<any>) => infer P ? P : T
type Func = () => User;
type Test = ReturnType<Func>; // Test === User

// 用在构造函数上
// 通过嵌套两层的方法获取到构造函数的参数
type ConstructorParameters<T extends new (...args: any []) => any => T extends new (...args: infer p) => any ? P : never>
// 获取实例类型
type InstanceType<T extends new (...args: any []) => infer P ? P : any>
```

**为了方便使用，TS 内置了几个常用的类型获取方法**

- `ReturnType<T>`：获取函数 T 的返回类型，如`ReturnType<typeof setTimeout>`
- `ConstructorParameters<T>`：获取构造函数的参数类型
- `InstanceType<T>`：获取实例类型

#### 类型提取 Pick / Omit

##### Pick

Pick 的作用是从类型定义的属性中，选取需要的属性返回一个新的类型定义（和 lodash 的 pick 方法类型）

Pick 接收两个参数，第一个是要提取的对象，第二个是要提取的属性。下面是一个示例：

```ts
interface Person {
  id: string;
  name: string;
  age: number;
}

type Female = Pick<Person, 'id' | 'name'> 
// 上面的相当于
interface Female {
  id: string;
  name: string;
}
```

##### Omit 

Omit 和 Pick 作用类似，只不过是取除了传入的参数之外的

```ts
interface Person {
  id: string;
  name: string;
  age: number;
}

type Female = Omit<Person, 'age'> 
// 上面的相当于
interface Female {
  id: string;
  name: string;
}
```

与 Omit 类似的还有一个 `exclude`，不过 `exclude`用于从联合类型中排除类型，Omit 则用与对象和接口。

#### typeof - 引用变量类型 

```ts
let s = 'hello'
let n : typeof s // let n : string
```

当然也可以引用复杂的类型

**对于函数其返回变量类型为返回值的类型** 

```ts
function f() {
  return {x: 10, y: 3}
}

type P = ReturnType<typeof f> // type P = {x: number, t:number}
```

#### keyof - 从对象生成联合类型

```ts
interface Person {
  id: string;
  name: string;
  age: number;
}

type key = keyof Person // type key = 'id' | 'name' | 'age'
```

#### in - 取联合类型的值

```ts
type name = 'a' | 'b'
type KName = {
  [key in name]: sting
} // type KName = {a: sting, b: string}
```



## 参考文章

轻松拿下 TS 泛型：https://juejin.cn/post/7064351631072526350

深入理解 TypeScript: https://jkchao.github.io/typescript-book-chinese/tips/covarianceAndContravariance.html