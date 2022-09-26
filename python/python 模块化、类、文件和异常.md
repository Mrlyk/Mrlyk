# python 模块化、类和文件

本文将走进 python 的高级特性——模块化、类、文件管理以及异常，学习完基础、基础进阶以及本文后。就基本掌握了 python 的开发方法，可以动手开发项目了！

[toc]

## 模块化

模块化是现代软件开发工程的必备特性，python 也自带这种特性。

先说说模块的导入

#### 模块导入

在 py 中模块导入和 js 一样使用 `impor`关键字，但语法不太一样。下面用🌰说明

- 导入整个模块：`import module` 导入的模块会是一个字典，要用里面的方法需要使用`.`操作符访问
- 导入所有函数：`from module import *` 使用 `*` 关键字可以导入所有的函数，函数也可以直接使用
- **导入特定的函数**：`from module import module_func` 可以看到 py 导入模块时是先指定模块，再`import`具体要导入的方法。这时候再使用函数就不需要`.`操作符了
- **给函数指定别名**：`from module import module_func as my_func` 和 js 一样使用 `as` 关键字可以为函数指定别名
- 给模块指定别名：`import module as my_module` 也可以使用`as`给整个模块提供别名

可以看到如果直接`import `那么导入的就是整个模块，如果先`from`指定模块再`import`就是导入其中的函数。记住这个语法！

在 py 中模块导入相当于在运行时将导入的模块直接拿了过来！其实和 js 中的 ems 规范也差不多！

#### 模块导出

在 py 中模块并不需要使用关键字导出。一般下面两种规则：

##### 在同一目录中的模块

如果要导入的模块和当前文件在同一目录下，那么直接使用`import 文件名`即可导入模块（**模块名必须是英文**）

##### 在自定义目录中的模块

在现代 python 工程中，我们一般单独创建一个比如`lib/packages`目录来存放公共模块。

**要想让 python 能导入自定义目录下的模块，我们需要在目录下创建一个`__init__.py`文件**，文件内容可以为空。这样 python 才能识别出该模块。

🌰如下：

目录结构：

```text
python-study
├─ README.md
└─ chapter-03
   ├─ 01-模块化.py
   └─ utils
      ├─ __int__.py
      └─ index.py
```

```python
# chapter-03/01-模块化.py
import utils.index

utils.index.make_pizza(16, 'water', 'hot')
```

上面在目录 utils 下创建了一个模块，同时创建了一个空的 `__init__.py`文件让 python 识别。

之后在模块中引入，然后就可以使用其中的方法了。

关于`__init__.py`的作用后面再介绍！

##### 增加 python 模块查找路径

**除了上面的方法之外我们还可以通过增加默认包的查找路径来导入模块** 。

```python
import sys # 引入内置的 sys 模块
sys.path.append("~/lib/packages")  # 设置自定义包的搜索路径，自动到当前磁盘目录下搜索模块
import helloworld
helloworld.show() 
```

## class 类

类，他来了。面向对象编程，每个现代高级语言都具有的特性，py 也不例外。因为他真的很高效、便捷！这一章来学习一下 py 中的类。

#### 创建类

在 js 中创建类使用`class`关键字，但是他也只是一个语法糖而已。 js 中并没有原生意义上的类。

在 py 中也是使用 `class`关键字来创建类。区别是构造方法和属性的定义不同。

```python
class Dog:
  #  -> None 表明返回值是 none，类似于 ts 类型解析的作用，可以删了，也可以保留
  def __init__(self, name, age) -> None:
    self.name = name
    self.age = age

  def sit(self): # 注意这个 self 是必须声明的，否则会报错
    print(f'{self.name} sit')
```

在 py 的类中：

- 属性都是函数，都要使用`def` 关键字来定义
- `self` 相当于 js 中的 this，**并且会传递给每一个属性，让实例能够访问类中的属性和方法**
- **类方法中第一个参数`self`是必须声明的，否则会报错** 

##### 构造函数

py 的类中使用`__int__`方法作为构造函数，接收用户实例化时传入的参数。**开头和末尾各有两个下划线,这是一种约定，旨在避免Python默认方法与普通方法发生名称冲突**。

##### 创建实例

py 中创建类实例不需要 js 中的 new 关键字，直接调用类即可创建。

```python
my_dog = Dog('mike', 19)

print(my_dog.name) # mike
my_dog.sit()
```

#### 使用类

py 中类的使用和 js 并无区别，依然可以用来创建具有一系列功能的对象。

类实例的调用方法也和 js 相同，使用`.`操作符直接在类实例上调用即可！

这里列举 py 中类的一些特性用法并和 js 做对比

| 特性       | 描述                                                         | 对比 js                    |
| ---------- | ------------------------------------------------------------ | -------------------------- |
| self.__xxx | 私有属性，外部无法访问                                       | js 原生不支持，ts：private |
| __xxx      | 私有属性，直接挂载在类上面的。在**类内部**用`self`或者**类内部**直接用类名都可以访问 | static                     |

#### 继承

py 中类的继承没有`extends`关键字，而是直接将父类传入。同时在子类的构造函数中，需要使用`super`关键字调用父类的`__init__`方法。

下面是一个🌰：

```python
class Animals:
    def __init__(self, type, color) -> None:
        self.type = type or 'animals'
        self.color = color

    def bark(self, voice):
        print(f'{self.type}: {voice} {voice}')


class Cat(Animals): # 继承 Animals
    def __init__(self, type, color, skill) -> None: # 重写父类的 init，就不会调用父类的 init 了
        super().__init__(type, color) # 所以这里要主动使用 super 关键字执行父类的构造函数
        self.skill = skill # 和 js 不同这里不要求一定要放在 super 后面

    def run(self):  # 这个 selft 必须声明  
        print('run faster')


my_cat = Cat('cat', 'white', 'clamp')
my_cat.bark('miao')
my_cat.run()
```

**同样的子类中可以重写父类中的属性和方法！！!**

类的编写也要遵循一些面向对象的原则：

- **类不宜过大**，如果一个类中有很多细节属性，可以将同类的创建一个新的子类来承载

***类就是用代码来描述实物！***  

## python 标准库

python 内置了许多有用的工具库被称为——python 标准库。

[内置库文档](https://pymotw.com/3/)：https://pymotw.com/3/

我们可以使用`import`导入这些标准库来帮我们实现一些操作。

下面举个🌰：

```python
from random import randint # 随机数生成标准库 

print(randint(1, 5)) #生成一个 1～5之间的随机整数
```

**常用的还有：**

*ps：`from random import choice` 用`random.choice`来表示*

- `random.choice` ：返回传入的列表或元祖中的一个元素

## python 文件系统

js 的运行时环境 node 提供了 fs 作为文件系统。python 作为偏服务端端语言不需要单独的运行时环境，内置了文件系统。

#### 读取文件

首先看代码🌰：

```python
with open('./chapter-03/files/pi_digits.txt') as file_object:
    contents = file_object.read()

print(contents)
```

这里有很多东西要注意，我们一点点说明：

- 如果文件不存在会直接报错

- `open`：open 函数是 py 提供的打开文件的方法，接收一个参数：要打开的文件名的名称。**这里要十分特别强烈的注意：这个文件所在的路径是相对于当前 py 的执行路径！！！而不是当前 py 文件所在的目录**。比如我在`# mrlyk @ Mrlyk-MacPro in ~/Study/Python/python-study`这个目录下直接使用`python my_python_file.py`，那么当前的执行路径就是：`~/Study/Python/python-study`

  就像我们在 node 中使用 fs 读取文件时，有时候要通过`process.cwd()`获取当前 node 的执行路径，因为相对路径都是相对于当前程序的执行路径！！！

  如果要像 node 一样查看当前 py 的执行路径如下：

  ```python
  import os
  
  current_directory = os.getcwd()
  ```

- `as`：open 执行后返回一个表示文件的对象，这里通过`as`关键字将其赋值给 `file_object`使用，后面就可以使用文件对象的方法来读取文件了

- `with`：最后说一说 with，with 会自动将我们打开的文件关闭。其实我们也可以像下面这样写：

  ```python
  file_object = open('./chapter-03/files/pi_digits.txt')
  print(file_object.read())
  file_object.close()
  ```

  **作用和上面的一样，但是如果我们忘记了 `close` 或者程序异常导致没有执行`close`可能会导致文件损坏！** 有了 with 之后 python 会决定什么时候去关闭它

*注意 读取文本文件时,Python将其中的所有文本都解读为字符串。如果读取的是数,并要将其作为数值使用,就必须使用函数int() 将其转换为整数或使用函数 float() 将其转换为浮点数。*

上面说了 `open` 最后会返回一个文件对象，我们这里调用了文件对象的`read`方法

##### 文件对象方法

- `read`：读取整个文件，速度慢，占用内存大

- `readline`：逐行读取

  ```python
  # 传统的逐行读取，不太推荐，写起来比较麻烦，除非只需要读取第一行
  file = open('./chapter-03/files/pi_digits.txt') 
  line = file.readline()
  print(line)
  line = file.readline()
  print(line)
  file.close()
  
  # 使用 with 配合 for 循环读取，比较推荐
  with open('./chapter-03/files/pi_digits.txt') as file:
      for line in file: # 注意这里直接遍历文件就像，不用 readline 方法
          print(line)
  ```

- `readlines`：读取所有行，并返回一个列表，将内容都存储在列表中

  ```python
  file = open('./chapter-03/files/pi_digits.txt') 
  line = file.readlines()
  print(line)
  file.close()  # ['3.141592653\n', '58979323846\n', '2643383279']
  
  # 也可以在 with 中读取
  with open('./chapter-03/files/pi_digits.txt') as file:
    contents = file.readlines()
  
  print(contents)
  ```
  



*对于可处理的数据量,Python没有任何限制。只要系统的内存足够多,你想处理多少数据都可以。*

#### 写入文件

了解了读取之后就要看看写入了。

**写入和读取用的方法是相同的，只不过要携带标志位参数**，标志位和 node 文件系统的标志位基本一致，只不过没那么多！

下面是个🌰：

```python
with open('./chapter-03/files/programing.txt', 'w') as file:
  file.write('I love programing1')
```

标志位有四个:

- 'r'：读取模式，也是默认的
- 'w'：写入模式，覆盖文件（**如果在一个 open 期间同时写入则也可以写入多行**），没有文件时会自动创建（没有文件夹不行）
- 'a'：附加模式，写入的内容附加到后面
- 'r+'：读写模式，有时候需要用到

在我们使用`open`方法并传入了`w`标志位后便可以往文件中写入内容了。

其中`write`方法接收要写入的内容，**写入的内容末尾不会像 node 那样自带换行符，需要自己手动输入**

#### 传入编码符号

除了传入标志位来决定是读取还是写入之外，也可以**像 js 那样传入编码符号来决定文件编码类型。**

```python
with open('./chapter-03/files/programing.txt', 'w', encoding='utf-8') as file: # 传入 encoding 编码
  file.write('I love programing1\n')
  file.write('I love programing2.2')
```

如上，虽然 `open`函数的第四个参数才是编码符号参数，但是 py 接收关键字实参，所以可以方便的在任意位置传入参数。而不用像 js 那样要时刻关注参数的顺序！歪瑞古德！

#### 其他常用文件方法

- `os.remove(path)`：删除文件
- `os.rmdir(path)`：删除目录
- `shut.rmtree(path)`：删除目录及下面的文件

## 异常

异常补货在程序中是很重要的，在 js 中我们使用 `try catch`捕获异常。python 中对异常的处理更加清晰，能够针对不同的异常类型提供不同的处理方式。

在 py 中使用`try excpet`来捕获异常。

下面举一个🌰：

```python
try:
  print(5/0) # 0 不能当除数
except SyntaxError:
  print('error')
except:
  print('other error') # 上面异常类型为 ZeroDivisionError，所以 SyntaxError 未发生，进入了这里，
```

可以看到使用方式和 js 中的异常捕获基本相同。

上面也说了 python 能够针对不同的异常类型提供不同的处理方式，还是以上面的例子为例。

例子中首先尝试捕获`SyntaxError`，然后再尝试捕获了所有其他类型的异常。由于 `print(5/0)`的**异常类型**为`ZeroDivisionError`，所以进入了下面其他类型的异常！

注意：

- 如果我们没有其他类型的异常捕获兜底，程序可能还是会异常中断
- **异常类型**如果不知道可以直接让异常发生看一下，终端上会输出这个异常类型。最好是有个捕获所有类型的捕获器兜底

#### else 代码块

在 js 中对于要正常执行的代码，我们一般直接写在 `try` 代码块中了，而在 python 中给异常捕获提供了`else`代码块。

**python 推荐在 `try`代码块中编写可能产生异常的代码，在`else`代码块中编写代码成功运行时才执行的代码！！！**下面是一个🌰

```python
while True:
    f_num = input('请输入除数:')
    if f_num == 'q':
        break
    s_num = input('请输入被除数:')
    if s_num == 'q':
        break
    try:
        result = int(f_num) / int(s_num)
        # print(f'结果是:{result}') # 不在这里编写正常运行的代码
    except ZeroDivisionError:
        print('被除数不能为 0')
    else:
        print(f'结果是:{result}')  #在 else 中编写成功运行时才执行的代码
```

这种做法能更好的帮助我们避免意料之外的语句带来的异常而导致的异常追踪困难！

#### 一些异常类型

上面我们知道了`SyntaxError`可以用来处理语法异常，`ZeroDivisionError` 用来处理 0 作为除数的异常。下面来看一些常见的异常类型。

##### FileNotFountError 文件未找到异常

当我们打开文件时，如果文件路径不存在则会抛出该异常！

```python
try:
    with open('./chapter-03/files/test.txt') as f: # 该路径不存在
        contents = f.read()
except FileNotFoundError: # 捕获到 文件不存在异常
    print('请检查文件路径')
else:
    print(contents)
```

这里要注意两点：

1. 异常是在 `open`函数调用时发生的，所以 `try` 要将 `open` 包裹起来
2. with 后面是一个代码块，`f` 文件对象是包含在代码块里的，所以不要试图将他们分开把`f`放入 else 中

#### 静默异常

在 js 中如果我们捕获到异常但不想做任何操作，那么把代码块空在那里就好了。

在 py 中由于缩进很重要所以不能捕获了异常之后什么都不做，py 会把后面的代码视作在代码块中。所以 **py 提供了一个特殊语句——`pass`来告诉编译器这里是什么都不做。**同时还充当了占位符的作用，语义化很神奇吧！

下面是一个🌰：

```python
try:
  print(5/0)
except ZeroDivisionError:
  # 什么都不写会报错，用 pass 不会
  pass

print('end')
```

**`pass` 不是 js 中的 return，在`pass`前后的语句还是会被执行。**

## 存储数据

在前端开发中我们如果要存储数据一般会选择框架提供的状态管理器或者存储在浏览器的 storage 中。**在 python 中内置了`json`模块来存储数据！**

`json`模块可以将简单的数据存储并且在 python 程序之间共享！

*存储是存储在确切路径的文件中！*

下面来看一个🌰：

```python
import json

content = [1, 2, 3]

try:
    with open('./chapter-03/files/json.json', 'w', encoding='utf-8') as f: # 通过写入模式打开文件
        json.dump(content, f) # 写入 json 数据
except FileNotFoundError:
    print('请检查文件路径')
else:
    print('存储成功')

```

在引入了`json`模块后，使用了`json.dump`方法来存储 json 数据！需要注意的是**如果存储的是中文字符，会被自动转换成 unicode 编码的**。当然取出的时候也会自动转回来。

- `json.dump`在这里接收了两个参数，第一个是要存储的 json 内容，第二个是文件对象
- 如果我们向同一个文件夹中多次写入，**文件内容不会被覆盖而是追加。同时追加的`json`内容因为也带有双引号最后会导致整个 json 文件的格式出错而无法读取**。所以不能往同一个文件中多次使用`dump`方法写入

可以看到前提是我们需要用`open`函数去访问这个文件才能往里写入内容。

#### 取出数据

上面存了，接下来就要取。取出数据的 api 是`json.load` 。同样依赖于`json`模块！

让我们把上面存储的数据取出来：

```python
import json

with open('./chapter-03/files/json.json') as f:
    jsons = json.load(f)

print(jsons) # [1, 2, 3]
```

- `json.load`接受一个参数，即要被取出的文件系统。**如果文件内容为空会报错！！！**

这样即可以取出之前存储的数据！

这里可能有人要问既然我用了`open`函数来打开文件，**那我直接使用文件对象的`read`方法可以吗？**

**答案是可以的**。这里只要注意两点：

1. 如果是数字或英文那么直接`read`结果是一样的。如果有中文的话就要注意了。**`json.load`加载时会把中文从 unicode 编码转回来，而 `read` 读到什么就是什么**
2. **`json` 模块的方法只能操作`json` 文件**，操作其他类型的文件会报错，很好理解！





