# python 测试函数

在任何程序中测试代码都是很重要的。js 项目中想要编写单元测试需要引入第三方的库比如`jest`来帮助我们。python 内置了`unittest`模块来编写测试代码。

[toc]

## 编写测试用例

#### 单元测试

单元测试：用于核实函数的某个方面没有问题。

要编写单元测试需要三步走：

1. 导入`unittest`模块
2. 创建一个继承自`unittest.TestCase`的测试类
3. 编写具体的测试方法

下面我们看一个🌰：

```python
import unittest # 1.导入模块
from module import get_formatted_name


class NameTestCase(unittest.TestCase): # 2.继承单元测试类
    def test_first_name(self):
        formatted_name = get_formatted_name('liao', 'yk') # 3.调用方法
        self.assertEqual(formatted_name, 'Liao Yk') # 判断输出结果是否符合预期
        
    # 如果需要多个测试用例，定义多个实例方法即可
   def test_full_name(self):
        formatted_name = get_formatted_name('liao', 'keji', 'yk')
        self.assertEqual(formatted_name, 'Liao Keji Yk')

        
if __name__ == '__main__':
    unittest.main()
```

其中

- `self.assertEqual`被称为**断言**，其他语言里也这么称呼。**断言方法核实得到的结果是否与期望的结果一致。**
- **实例方法名必须以`test_`开**头，这样在运行测试用例时才会被执行

最后面的`__name__`可能有人看不懂。**`__name__`是一个特殊变量，这个特殊变量时在程序执行时设置的。如果这个文件作为主程序执行。`__name__`的值就是`main`。**（这种双下划线包裹的变量可以看作 python 运行时的环境变量！）

所以这里的意思是**如果程序作为主程序执行，就调用`unittest.main()`方法来执行测试程序！**

下面是`unittest`模块中常用的 6 个断言方法：

![image-20220910235949835](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220910235949835.png?x-oss-process=image/resize,w_800,m_lfit) 

## 测试未通过？

测试未通过时，我们千万不能去修改测试用例来让测试通过，而是去修改我们的代码，让其兼容老的程序。

这才符合代码变更的要求。

## 测试类

在开头我们编写了针对单个功能的单元测试，现在来编写针对类的测试。

首先我们创建一个问卷收集类来作为测试目标，问卷收集类如下：

```python
# 定义一个匿名问卷收集类
class AnonymousSurvey:
    def __init__(self, question) -> None:
        self.question = question
        self.response = []

    def show_question(self):
        print(self.question)

    def store_res(self, res):
        self.response.append(res)

    def show_results(self):
        for res in self.response:
            print(f'——{res}')
```

针对类的测试其实也是针对类的行为的验证。

接下来编写一个验证用户的**单个**回答能否被正确收集的测试类！

```python
import unittest # 1.引入 unittest 类
from module import AnonymousSurvey


class TestAnonymousSurvey(unittest.TestCase): # 2.继承单元测试类
    def test_store_single_response(self):
        question = 'How old are u?'
        my_survey = AnonymousSurvey(question)  
        my_survey.store_res('13') 
        self.assertIn('13', my_survey.response) # 3.断言结果是否符合


if __name__ == '__main__':
    unittest.main()
```

这里我们编写了一个测试用例，使用了`assertIn`来判断回答是否被正确手机到 response 列表中。

接下来添加一个测试三个回答是否被正确收集的测试用例：

```python
def test_store_three_response(self):
        question = 'How old are u?'
        my_survey = AnonymousSurvey(question)
        responses = [13, 14, 15]
        for res in responses: # 通过循环来存储
            my_survey.store_res(res)

        for res in responses: # 通过循环来判断
            self.assertIn(res, my_survey.response)
```

这样就完成了！

可以看到测试单个回答和测试三个回答有相同的地方，比如都创建了一个实例。那有没有办法只创建一个实例呢？

在 js 中编写这类测试的时候我们都只会在开头创建一个实例用于后面的测试。python 的 unittest 模块也提供了方法来创建一个在每个测试用例中都能访问的对象——`setUp`

#### setUp 测试用例公用对象

因为测试用例也是一个类，所以可以很自然的想到在初始化类的时候将要测试的类对象放到测试类的实例上。

但这样是不行的，子类的`__init__`构造函数会重写父类的，重写了父类的之后父类的就不会调用。如果我们使用`super`关键字主动调用又不知道如何传参...

所以 python 规定了一个`setUp`实例属性，并且会在单元测试实例执行前调用他。

下面是一个🌰：

```python
import unittest
from module import AnonymousSurvey


class TestAnonymousSurvey(unittest.TestCase):
    '''报错'''
    # def __init__(self) -> None:
    #     super().__init__()
    #     question = 'How old are u?'
    #     self.my_survey = AnonymousSurvey(question)
    #     self.response = [13, 14, 15]

    def setUp(self) -> None: # 定义 setUP 属性
        question = 'How old are u?'
        self.my_survey = AnonymousSurvey(question) # 在其中挂载后面要用的公共属性到实例上
        self.response = [13, 14, 15]

    def test_store_single_response(self):
        self.my_survey.store_res(self.response[0]) # 直接使用实例上的，不用再重复实例化了
        self.assertIn(13, self.my_survey.response)


if __name__ == '__main__':
    unittest.main()
```

**注意** 

运行测试用例时,每完成一个单元测试,Python都打印一个字符:**测试通过时打印一个句点**,测试引发错误时打印一个E ,而测试导致断言失败时则打印一个F

![image-20220911112502912](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220911112502912.png?x-oss-process=image/resize,w_800,m_lfit) 

![image-20220911112658690](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220911112658690.png?x-oss-process=image/resize,w_800,m_lfit) 