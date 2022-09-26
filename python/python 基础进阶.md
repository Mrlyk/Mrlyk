# python 基础进阶

在回忆了基础语法后，开始学习 python 中的逻辑控制、字典、用户输入、函数等进阶内容！

[toc]

## 逻辑控制

逻辑控制也就是其他语言中常用的

- if 条件控制
- while 循环
- 用户输入

其实高级语言中这些都很类似。这里对 py 中的基础做一个示范，同时列出和 js 相比的一些不同点！

#### if 条件控制

和其他语言基本一致，只不过没有`() {}`这些用以语法分析的标志符号！

下面是个基础🌰：

```python
cars = ['audi', 'bmw', 'volvo']

for car in cars:
    if car == 'bmw':  # 注意 1）: 表明代码块 2）py 没有 === 的语法，因为 py 没有变量类型， == 就是全等判断
        print(car.upper())
    elif car != 'audi': # 也可以有多个
        print(car)
    else:
        print(car.title())

```

- `==` 相等判断，**没有**`===`，py 没有变量类型
- `!=` 不等判断，也没有`!==`
- **else if 在 py 中简写为 `elif`**，这点和其他编程语言不通，要注意！

##### 检查多个条件

- `and` 替代 js 中的 `&&`
- `or` 替代 js 中的 `||`

下面是一个🌰：

```python
num = 1
for car in cars:
    if car == 'bmw' or car == 'audi':
        print(car.title())
    if car == 'bmw' and num == 1:  # if 通过后执行后面所有缩进的语句，相当于用 {} 包裹了一样
        num += 1
        print(car.upper())
```

##### 包含检测

在 js 中如果有多个条件判断，我们通常会使用`includes` 判断。**在 py 中则是实用`in`、`not in` 操作符！**

```python
cars = ['audi', 'bmw', 'volvo']

if 'bmw' in cars:   # 包含
    print('have bmw car')
if 'toyota' not in cars:  # 不包含
    print('not have toyota')
    
# 当然对字符串也是可以用 in 操作服判断对
if '1' in '123':
  	print('have 1')
```

##### 列表的非空检测

在 js 中如果直接用 `if` 判数组空是么效果的，原因大家都知道。

**在 py 中可以这么做，`if` 条件是一个数组时，判断的是数组是否有内容**

```python
pizza = []
if pizza:
    print('have pizza')
else:
    print('you need a pizza')  # 会输出这里
```

*在`if `语句中将列表名用作条件表达式时,Python将在列表至少包含一个元素时返回 `True`  ,并在列表为空时返回`False`* 

#### while 循环

py 的 while 循环和其他高级语言也基本一致。

基本🌰：

```python
current_number = 1
while current_number <= 5: # 打印 1 ～ 5
    print(current_number)
    current_number += 1 # 注：py 不认 ++ 这种语法
```

也可以使用 `break` 关键字立刻退出循环，`continue` 关键字退出当前这一次循环！和其他语言没什么区别！

在前面我们使用了`for`循环来遍历列表，我们当然也可以使用`while`循环来进行相同的操作！同时**`for`作为一个经典的遍历手段，不应该在`for`遍历时修改列表。这时候就可以使用`while`循环了！**

下面是一个使用循环和列表`remove`方法删除所有同一类元素的方法🌰：

```python
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']

while 'cat' in pets:
    pets.remove('cat')

print(pets)
```

## 字典

字典对应着 js 中的对象，也是用`{}`来声明，访问和修改方式也相同。当然 py 的字典有着自己的特性，下面会着重讲一讲 py 自己的特性。

下面是个基础的🌰：

```python
dog = {'color': 'black', 'weight': '20kg'}

dog['age'] = 3
dog['weight'] = '30kg'

print(dog['color'])
```

说一下和 js 中的基本不同点

- py 中字典的键值只能是字符串，**写成其他的会被当作变量去查找**，不会像 js 那样自动转成字符串
- 访问字典中的属性只能使用`[]` ，不支持`.`操作符访问。同时这样访问如果 key 不存在会报错
- **如果声明了相同的属性，后面的会覆盖前面的的**（js 其实也会）
- 不支持简写 key value，即不能像 js 一样不写值而把变量的值默认作为值
- **如果只有键没有值，那么就不是字典而是 set 了** ，注意不要混淆

#### 字典操作

##### 删除属性

删除和 js 中类似，js 使用 `delete`，py 中使用 `del`

```python
dog = {'age': 5, 'name': 'Wang'}

del dog['name']
```

##### 使用 get 访问

在上面我们通过`[]`访问字典的 key 时，如果 key 不存在会报错。所以 py 提供了另一种字典的访问方式`get`，在**指定的键不存在时返回一个默认值,从而避免这样的错误**

```python
person = {
    'name': 'Mike'
}

# print(person['age']) # 访问不存在的报错
print(person.get('age', 'no age'))  # 返回 no age，如果没有第二个参数则返回 None
```

##### 遍历字典

终于说到遍历了，在 js 中我们遍历一个对象一般是`Object.keys` 或者`Object.values`，在通过数组的方式遍历。

在 py 还是使用 `for in`来遍历，到目前为止我们可以看到 py 中的遍历都是统一使用 `for in`来操作的！

但是 **py 中的字典不是直接可以枚举的，需要遍历的是字典的`items()`方法返回的值**，要记住这一点！

```python
user_0 = {
    'name': 'user0',
    'age': 20
}

print(user_0.items())  # dict_items([('name', 'user0'), ('age', 20)])
for key, value in user_0.items(): # 遍历 items() 方法返回的值
    print(f'{key}:{value}')

```

在上面可以看到使用了`key`和`value`来承载对象的键和值，**这其实是一种类似 js 解构的写法**，如果我们只有单个参数
即像这样：`for item in user_0.items()`，那么这个`item`就会是一个个点 tuple `('name', 'user0')`...

**还可以仅仅只遍历键或只遍历值**

| 方法                         | 描述                                                    | 对比 js         |
| ---------------------------- | ------------------------------------------------------- | --------------- |
| `for key in user_0.keys()`   | 仅遍历键                                                | `Object.keys`   |
| `for key in user_0.values()` | 仅遍历值，重估的会被保留。如果要去重可以使用`set()`函数 | `Object.values` |

**使用`set`函数去重**

```python
language = {
    'a': 1,
    'b': 2,
    'c': 1,
}

for v in set(language.values()): # 使用 set 函数包裹
    print(v)  # 1, 2
```

**从Python 3.7起,遍历字典时将按插入的顺序返回其中的元素**。但是我们也可以使用`sorted`函数自定义顺序！

```python
for k, v in sorted(programmer.items()): # 默认按 key 的升序
    print(f'{k}:\t{v}')
```

字典的其他特性和 js 的对象基本一致，比如不限制字典值的种类（**当然 key 只能是字符串**）！

#### 集合

上面说了字典如果只有 key 就是一个集合，当然我们不能这么认为，字典和集合并没有什么关系。

集合 set 在 js 中个有，效果也一样，就是一个数据不重复的数据存储合集！

```python
my_set = {1, 2, 3}
```

## 用户输入

程序的交互操作依赖于用户的输入，在 C 语言中我们使用`scanf`来读取用户输入。在原生 js 中没办法直接获取用户的输入，在 web 端需要结合 html 表单。在服务端可以使用 nodejs 提供的 `readline`模块。

在 py 中和 C 一样提供了原生的接收用户输入的函数`input()`

基础🌰：

```python
num = input() # 可以给 input 传一个参数作为提示语 input('please enter a num')

print(f'{num} is your input')
```

运行上面的小程序后，**程序会挂起等待用户输入完成。之后按下回车程序才会继续执行。同时将用户输入的值赋值给声明的变量！**

*注意这样读取的数据都是字符类型的！*

#### 让用户决定何时退出循环

我们还可以把 `input` 和 `while`集合，根据用户输入的值来退出循环

```python
message = '' # 不声明也不会报错（也有变量提升？？？），但是最好声明
while message != 'quit':
    message = input('enter a word:')
    print(message)
```

## 函数

函数终于来了，在任何语言中函数都是很重要的。一起来学习巩固一下 python 中的函数！

#### 函数定义

py 中函数使用`def`关键字定义，下面是一个基础🌰：

```python
def gree_user(): # 函数定义
    name = input('Please enter your name:')
    print(f'Hello {name}')

gree_user() # 函数调用，都一样
```

**在函数中定义的变量外部也一样无法访问！** 

#### 函数传参

传参也和 js 相同，直接传入实参即可

```python
def gree_user(name): # 形参也支持和 js 一样给默认值，比如 name = 'world'
    # name = input('Please enter your name:')
    print(f'Hello {name}')

gree_user('py') # 传入实参
```

下面有两点要特别注意：

- **如果使用了默认参数，要注意在多个参数时默认参数后面跟的其他形参也必须有默认参数！**即不能只给第一个参数设置默认参数，而第二个不设置，这样会报错：`SyntaxError: non-default argument follows default argument`!
- 实参的个数和形参的个数不匹配时会报错。比如实参个数超过了形参个数会报错，形参未赋默认值时未传形参会报错！总之**只要实参和形参不匹配就会报错！** 

##### 关键字实参

在 js 中，形参和实参的位置关系时一一对应的。但有时会弄混或者参数过多时参数很难对应。

**在 py 中提供了关键字实参，让我们能够直接指定实参对应的形参**

```python
def describe_pet(pet_name, pet_type):
    print(f'The pet name is {pet_name}, it is a {pet_type}')

describe_pet(pet_type='dog', pet_name='Wang') # 顺序不对应没关系，直接关键字对应
```

**要注意关键字实参和普通实参不能混用**，要么都是用关键字实参，要么都使用普通实参（**位置实参**）！

**在 py 的异常追踪器中： ** 

- arguments 表示实参
- parameter 表示形参

##### 任意数量的参数

在 js 中我们可以使用`arguments`对象获取用户传入的所有参数，纵使我们没有声明任何形参！

在 py 中我们可以在一个关键字前加`*`来接收任意数量的实参！**这个关键字会被转成一个元祖，其中包含所有传入的实参。** 

```python
def make_pizz(*args):
    print(args) # 输出一个元祖 ('water', 'hot', 'hi')

make_pizz('water', 'hot', 'hi')
```

**任意数量的实参也可以结合普通实参一起使用**。所以这个任意数量的实参的`*`作用其实和 js es6 中形参的`...`作用是相同的——收集剩余参数！

**当我们传入普通实参后，剩余的实参会被收集起来。如果没有传入任何实参，那么所有的实参都会被收集！**

##### 任意数量的关键字实参

在 py 中如果我们想接收任意数量的关键字需要使用`**`两个星号。`**`标识的形参创建指向一个空字典，其中存储了所有未显示声明的剩余的关键字实参（也只包含关键字实参不包含普通实参）。

```python
def build_profile(first, last, **kwargs):
    print('args', kwargs) # args {'age': '18', 'love': 'lly'}
    args['fist'] = first
    args['last'] = last
    return kwargs # {'age': '18', 'love': 'lly', 'fist': 'yk', 'last': 'l'}


print(build_profile(first='yk', last='l', age='18', love='lly'))
```

#### 返回值

py 函数的返回值和 js 一样，使用 return 关键字来返回，可以返回任何类型的值。而且支持一次返回多个值。

```python
def getNumbers():
  	return 3, 4
  
x, y = getNumbers()  # x:3 y:4  
```

在 js 中这样是不行的，除此之外就和 js 没什么区别这里就不再赘述了！