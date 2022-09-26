# python 基础语法回忆

python 写爬虫、学 AI 都必不可少，还是需要回忆一下曾经学过的 python！

python 是和 js 有很多相似之处，一样的动态类型语言，一样的基于继承的内置方法！而且在 ai 方面具有更多完善的工具！

[toc]

## 基本语法

注释：***py 中单行注释使用 `#`*** ，多上注释使用`""" 注释内容 """`包裹（单引号也行！）

缩进：py 中缩进很重要，py 没有像其他语言一样使用 `{} ;`来区分代码语句和代码块，一般代码块前使用`:` 来表明。**使用四格缩进来表明代码块** 

## 变量和简单数据类型

py 中变量直接声明即可，甚至不需要`let` 这种定义，当然 python 中也有保留字！

```python
message = 'hello world'
x, y, z = 1, 2, 3 # 支持多个变量同时赋值
print(message)
```

常规注意事项：

- 变量名只能包含字母、数字和下划线。变量名能以字母或下划线打头,但不能以数字打头
- 变量名不能包含空格

**py 中变量是可以赋给值的标签,也可以说变量指向特定的值。即一个指向内存地址的指针。**

#### 常量

和 js 一样 py 并没有内置常量类型，使用全大写声明语义化区分即可！

#### 数据类型

py 是动态类型语言，所以变量的类型由其值决定。

##### 字符串

字符串声明不限制单/双引号，都可以解析

- 模版字符串使用 `f'{}'`声明，f 是 format 的简称——`name = f'{first_name} {last_name}'`；*注意要求 py >= 3.6 版本*

  如果是 py < 3.6 版本需要使用这种形式：`name = '{} {}'.format(first_name, last_name)`

**制表符**

- \n 换行
- \t 制表符

和其他编程语言中基本一致，也可以组合使用 `\n\t` 

##### 数字

py 中的数字逻辑运算和 js 相同，也存在 js 相同的数字精度问题！

py 中数字要注意的一些问题

- 任意数相除，结果总是浮点数`4/2 -> 2.0`
- 任意运算中，只要有一个是浮点数，结果总是浮点数
- **在 py 中为了方便大数阅读，可以使用`_`分隔，不会影响值**`100_000_000`，打印的时候下划线也会被自动去除

##### 布尔值

py 中的 bool 值和其他语言差不多，除了他们开头是大写的

- True 真
- False 假

##### None 

在 py 中 None 值表示什么都不返回，相当于 js 的 null

## 列表

列表对应着 js 中的数组，表示方法也一样，访问方法也一样（通过索引访问）

这里主要讲一下不同的地方

#### 访问元素

常规访问和 js 一样通过索引访问

- **支持负索引访问**，-1 表示倒数第一个元素，-2 表示倒数第二个元素...

```python
fruit = ['apple', 'banana']

print(fruit[-1].title()) # Banana
```

- 不能直接给超出声明长度的数组赋值也不能访问，均会报错

```python
fruit = ['apple', 'banana']

fruit[2] = 'peach' # 这样会报错，因为原来只有索引最大只到 1，给 2 赋值超出范围
```

- **支持切片访问**

可以使用`:`来分割表明要访问几号到几号元素，**也支持负数** ，包括开头不包括结尾

```python
nums = [1, 2, 3]

print(nums[1:]) # [2, 3]
```

#### 操作列表（数组）

##### 常规访问、属性操作

这里顺便列一下数组操作和 js 中的对应关系，加深印象

| python              | 描述                                                         | 对应 js 操作 |
| ------------------- | ------------------------------------------------------------ | ------------ |
| append              | 末尾添加元素，无返回值                                       | push         |
| insert(index, args) | 插入元素，**只能插入不能像 js splice 那样删除**              | splice       |
| del array[index]    | 删除元素，前提是知道索引                                     | splice       |
| pop(index)          | 默认删除末尾元素并返回，**接收一个索引可**以弹出任意索引位置的元素 | pop          |
| remove(item)        | 根据值删除元素，需要传入对应的值，省的还要去找了...,**如果有相同的值只会删除第一个**，可以使用循环来全部删除 | \            |
| sort                | 排序，默认升序。接收一个参数`reverse`来降序排列`sort(reverse=True)`。无法像 js 那样自定义。**该方法会修改原始数据！** | sort         |
| sorted(array)       | 排序，默认升序。接收两个参数，第一个为目标列表，第二个为排序方式。**不会修改原始数据而是返回新数据！** | \            |
| reverse             | 反转数组，**会改变原始数据**                                 | reverse      |
| len(array)          | 返回**数组长度**                                             | arr.length   |
| min(array)          | 找出数组中最小值，**如果不是纯数字列表会报错**，py 中不存在 js 的隐式转换 | \            |
| max(array)          | 找出数组中的最大值，**如果不是纯数字列表会报错**             | \            |
| sum(array)          | 统计数组之和，**如果不是纯数字列表会报错**                   | \            |

其他一些 tips ：

- 对于**删除**：如果你要从列表中删除一个元素,且不再以任何方式使用它,就使用`del `语句;如果你要在删除元素后还能继续使用它,就使用方法 `pop()` 

##### 遍历操作

第一种操作和其他语言都差不多，使用 `for` 循环，几乎所有高级编程语言都有这种方式！

```python
iterateList = [3, 1, 2, '哈哈']

for item in iterateList:
    print(item)
```

同时这个 item 是存在全局作用域的，就像 js 中用 var 声明的一样。遍历完成后他的值是`哈哈`，并且能够被访问！

**注意：在 js 中 `item` 输出的是索引，而在 py 中直接就是列表项的值，不要弄混了！** 

##### 复制列表

在 js 中复制数组一般直接使用 slice 浅拷贝。在 py 中如何复制列表呢？

和 js 中的 slice 一样，**使用切片复制，也是浅拷贝**

```python
foods = [{'a': 1}, 'milk']

my_foods = foods[:] # 切片复制，不输入开始和结束就可以浅拷贝一个列表
```

#### 数值列表

在 py 中还存在单独的数值列表，py 会对数值列表有优化，纵使是几百万的数据性能也很好！

**py 特别擅长处理数据，这也是为什么大数据和深度学习都用它，千万级别的数字求和都能在几秒内搞定！**

##### 通过 range 函数创建数值列表

```python
rangeArr = range(1, 100) # 创建数值列表 1-100

for i in rangeArr:
    print(i)
```

注意 `range` 包含开头不包含结尾，即 `range(1, 100)` 相当于` [1, 2, 3, ... 98, 99]` （只是内容相同但不是一个东西要注意）**这是编程语言中常见的差一行为的结果。**

如果只传入一个参数，那么将默认从 0 开始到这个参数为止，即`range(100) == range(0, 100)`

**设置步长**	

`range`函数接收第三个参数设置步长

```python
rangeArr = range(0, 6, 2) # 表示每一步加 2 

for i in rangeArr:
    print(i) # 0, 2, 4 ，从 0 开始每一步加 2
```

**列表可用的方法对 `range` 创建的数值列表也都可用**  

##### range 列表转普通数字列表

`range`函数创建的列表是 range 类型的数字列表，可以使用`list`函数将其转为普通数字列表

```python
rangeArr = range(0, 3)

print(list(rangeArr)) # [0, 1, 2]
```

#### 列表解析语法

通常我们要获取一个列表的值加 1 的新列表，需要遍历这个列表并且每个值加 1 然后放入新列表。这样至少要 3 行代码！

python 提供了一种更加方便的列表解析的语法

```python
sum = [i+1 for i in range(3) if i < 2] # 列表解析（ps：这里声明的 i 外部无法访问，用完销毁）

print(sum) # [1, 2]
```

语法很简单

1. 中括号包起来
2. 左边是列表值计算的表达式
3. 中间是要遍历的列表
4. 右边是加入的逻辑判断（这是可选的）

在一些 py 程序中经常会看到。

## 元祖

在 py 中列表的数据是可变的，而元祖这种变量类型类似于列表，但是**其内容是不可变的**！

**定义元祖使用`()`**

```python
tup = (10, 100)

print(tup) # (10, 100)
print(tup[0]) # 10
print(tup[1]) # 100
```

注意 严格地说,元组是由逗号标识的,圆括号只是让元组看起来更整洁、更清晰。如果**要定义只包含一个元素的元组,必须在这个元素后面加上逗号**: `tup(10,)`

**列表可使用的方法元祖也基本都可用**。比如

- `len(tup)` 返回元祖长度
- `list(tup)` 元祖转列表
- `tuple(list)` 列表转元祖（列表都包含`range`定义的数值列表）
- `tup[:1]` 元祖切片
- `for i in tup` 遍历元祖

当然元祖数据不可变，不能对声明对元祖中添加、删除、修改数据！

**如果需要存储的一组值在程序的整个生命周期内都不变,就可以使用元组。**

#### 为什么有了列表还要元祖？

如果说列表的发明是为了解决一个变量存储多个数据的问题，而元组的发明是为了解决列表数据可被编辑的不安全性！

在 py vm 的底层实现中大量使用了 C-API 操作 py Objects，tuple 的不可变性实现了对函数返回值的保护！

总之还是利用了 tuple 不可变性的特点，而这个特点在有些情况下很有用！

## python 之禅

> Beautiful is better than ugly.
> Explicit is better than implicit. # 显示的编写好于隐式
> Simple is better than complex. 
> Complex is better than complicated.  # 越简单越不容易出错
> Flat is better than nested.  # 扁平化好于嵌套
> Sparse is better than dense. 
> Readability counts.  # 强调可读性
> Special cases aren't special enough to break the rules. # 特殊情况也要遵守规则
> Although practicality beats purity.  # 注重实用性
> Errors should never pass silently. # 异常要主动抛出不应该被隐藏
> Unless explicitly silenced.  # 除非显示的表明应该隐藏
> In the face of ambiguity, refuse the temptation to guess. # 面对困惑，不要猜测
> There should be one-- and preferably only one --obvious way to do it. # 要有一种也最好只有一种方法显示的解决它（意思是遵循设计模式）
> Although that way may not be obvious at first unless you're Dutch.  # 纵使一开始可能不明显（除非你是 py 的作者哈哈）
> Now is better than never.  # 学无止境，不要试图编写完美的代码，先编写有效的代码再来优化会更好
> Although never is often better than *right* now.
> If the implementation is hard to explain, it's a bad idea. # 如果实现很难理解，那他就是垃圾
> If the implementation is easy to explain, it may be a good idea. # 如果实现一眼就明，那他就是神
> Namespaces are one honking great idea -- let's do more of those! # 多使用命名空间吧，很棒！

## 其他

#### 代码风格推荐

python 遵循 pep8 代码格式指南。一般我们使用 ide 工具提供的 autopep 8 工具自动格式化！

pep 8 指南部分核心内容如下：

- 每次缩进使用 4 个空格
- 行长建议不超过 80 字符，注释建议不超过 72 字符
- 类名推荐使用双驼峰命名法
- 方法名和变量名推荐使用下划线分隔
- 需要同时导入标准库中的模块和你编写的模块时,先编写导入标准库模块的import 语句,再添加一个空行,然后编写导入你自己编写的模块的import 语句