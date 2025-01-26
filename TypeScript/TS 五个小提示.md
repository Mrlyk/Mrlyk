# TS 五个小技巧

本文记录 TS 实践中常用的的五个小提示，

[toc]

## | 和 &



我们知道`|`是联合类型，`& `是交叉类型

粗看他们的符号我们可能会产生这样的**误解**：`|` 代表或，`&` 代表且。

实际上并不是这样，下面是一个例子:

```ts
type User = {
   getId(): number;   
   getName(): string;
}

type Admin = {
   getId(): number;
   getUserName(): string;
}

declare function Person(): User | Admin;

const person = Person()

person.getId() // succeeds
person.getName() //fails
person.getUsername() // fails
```

我们可以看到下面两个方法在类型检查时会报错。

这是因为**`|`联合类型实际上是取交集**，在上面的例子中就是取`User`和`Admin`类型都有的属性。

同样的**`&`取的是并集**，也就是说如果把上面例子中的`｜`换成`&`就没问题，而不是将`&`理解为`&&`既要 A 有也要 B 有。

这两个区别一定要区分清楚！

## 利用联合类型而不是可选操作符

经常有需要处理接口字段的场景，我们需要区分请求成功和失败的情况，而请求成功和失败的情况返回的字段肯定不一样，这时候我们一般使用可选操作符来处理`?`来处理这种情况。下面是一个例子

```typescript
interface APIRes {
  success: boolean;
  data?: number;
  error?: string;
}

function handleAPIRes(res: APIRes) {
  return res.success ? res.data : res.error;
}
```

这样写虽然不会报错，但是在类型推断方面会有一个问题——在成功请求的情况下`res.data`肯定是`number`，但是因为使用`?`，所以这里`res.data`还会被推断为`undefined`类型！

在后续方法中如果要接收这个返回值作为数字处理又会出现类型推断错误。

如何处理这种情况呢？

**创建两个类型并使用联合类型**。

```typescript
interface SuccessRes {
 success: true;
 data: number;
}

interface ErrorRes {
 success: false;
 error: string;
}

interface APIResponse = SuccessRes | ErrorRes;

function handleAPIRes(res: APIResponse) {
 (response.success) ? res.data: res.error
}
```

这样在类型推断的时候，`res.data` 就只会是`number`类型而不会被推断为`undefined`！

## 使用类型谓词 is 收窄类型

有时候我们不清楚一个对象的类型是什么，需要根据条件判断，虽然我们知道了对象的类型，但是编译器还不知道。这里我们就可以使用类型谓词来收窄类型，获得更好的类型推断。

**类型谓词判断返回的是一个布尔值！** 下面是一个例子：

```typescript
type Fish = {
 swim:() => void
}

type Bird = {
 fly:() => void
}

function isFish( pet: Fish | Bird ): pet is Fish {
 return (pet as Fish).swim !== undefined 
}
```

在上面的例子中，我们将`pet is Fish` 作为函数返回值，相当于告诉编译器，**如果`(pet as Fish).swim !== undefined ` 这个条件判断成立，那么 `pet` 的类型可以收窄为`Fish`类型而不是原来的联合类型。**

## 使用枚举改进代码结构

在原生的 ts 中并没有枚举类型，所以在遇到一些数字常量表示状态的时候我们总是使用一个 Map 映射来表示。如：

```js
const SEX = {
  feamale: 0,
  male: 1,
  other: 9,
}
```

这样写的缺点是如果数据本身就是数字，我们还需要做一个转换来查找数字对应的值。或者声明两个映射，一个用值做键，一个用数字做键。

而枚举类型直接就支持到了这样的双向映射：

```typescript
enum Weekdays {
 Sunday = 1,
 Monday,
 Tuesday,
 Wednesday,
 Thursday,
 Friday,
 Saturday
}

console.log(Weekdays.Sunday) // 1
console.log(Weekdays[1]) // Sunday
```

## 使用范型

这一点不必多说，当我们在运行时才能确定函数类型的时候可以使用范型。

更多的可以看我们的类型体操。

