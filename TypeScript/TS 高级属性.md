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

#### 接口

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

#### 泛型推断 infer

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



## 参考文章

轻松拿下 TS 泛型：https://juejin.cn/post/7064351631072526350