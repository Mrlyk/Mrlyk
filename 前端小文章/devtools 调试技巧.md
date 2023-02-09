# devtools 调试技巧

这里记录一些好用的调试相关技巧

[toc]

## 内存泄露排查

#### 1.确认问题，找出可疑操作

先确认是否真的存在内存泄漏：

1. 切换到Performance面板，开始记录（有必要从头记的话）
2. 开始记录 -> 操作 -> 停止记录 -> 分析 -> 重复确认
3. 确认存在内存泄漏的话，缩小范围，确定是什么交互操作引起的

也可以进一步通过Memory面板的内存分配时间轴来确认问题，Performance面板的*优势*是能看到DOM节点数和事件监听器的变化趋势，甚至在没有确定是内存问题拉低性能时，还可以通过Performance面板看网络响应速度、CPU使用率等因素

#### 2.分析堆快照，找出可疑对象

锁定可疑的交互操作后，通过内存快照进一步深入：

1. 切换到Memory面板，截快照1
2. 做一次可疑的交互操作，截快照2
3. 对比快照2和1，看数量`Delta`是否正常
4. 再做一次可疑的交互操作，截快照3
5. 对比3和2，看数量`Delta`是否正常，猜测`Delta`异常的对象数量变化趋势
6. 做10次可疑的交互操作，截快照4
7. 对比4和3，验证猜测，确定什么东西没有被按预期回收

#### 3.定位问题，找到原因

锁定可疑对象后，再进一步定位问题：

1. 该类型对象的`Distance`是否正常，大多数实例都是3级4级，个别到10级以上算异常
2. 看路径深度10级以上（或者明显比其它同类型实例深）的实例，什么东西引用着它

#### **术语说明**

参考文章：https://zhuanlan.zhihu.com/p/80792297

- `distance`：堆快照中有一个字段，表示从window出发的最短保留路径上的属性引用数量
- 

## 命令相关

#### 复制变量

还在为变量不好复制而发愁吗？还是`JSON.stringify()`吗？

devtools 提供了内置 `copy`函数，直接拷贝变量

```js
// res.data 
copy(res.data) // 直接拷贝到粘贴板
```

#### 引用上一次的执行结果

有时候我们想要以上次方法的执行结果来作为下一次的参数，很多时候是把上次的语句复制在执行一遍。

`$_`可以直接引用上一次的执行结果，即`return`的值

```js
function t(){return '1'}
t()
// '1'

$_ // '1'

```

#### 重新发起请求

不要再傻乎乎的每次都去点按钮了，善用这些功能😭

1. 选中`Network`
2. 点击`Fetch/XHR`
3. 选择要重新发送的请求
4. 右键选择`Replay XHR`

#### $$

$$ 符号是浏览器设计遗留的一个特殊方法，当时的JavaScript库Prototype.js使用用 $ 符号来表示 `document.getElementById`。后来为了处理选择任意元素，$ 已经被占用了，所以使用了 $$() 方法来充当选择器。

- `$$(".")` 相当于 `document.querySelectorAll('*')`
- `${"."}`相当于`document.querySelector(".")` 

**注意只能在控台使用（未实验）** 

#### 使用 Charles 抓包

映射生产环境的请求 / sourceMap 到本地

https://cloud.tencent.com/developer/article/1730948

#### Overrides 修改生产代码调试

如果要修改生产环境的代码并调试，在原来的代码文件上直接修改是不会生效的。

可以使用 Sources 面板中的 Overrides 功能。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211227110800268.png" alt="image-20211227110800268" style="zoom:50%;" align="left"/>

首先需要添加一个本地文件夹，然后将文件 / 请求 右键保存至 Overrides。再次执行时便会使用 Overrides 的代码。

#### 生产环境使用本地 SourceMap

首先本地打包产生 sourceMap 文件

然后添加 source map。**注意添加的 path 需要是一个服务地址，可以本地启一个静态资源服务**

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211227135502242.png" alt="image-20211227135502242" style="zoom:50%;" />

添加完成后便会在右侧边栏展示。可以断点进行调试。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211227135654815.png" alt="image-20211227135654815" style="zoom:50%;" />

#### queryObjects 查询当前执行环境的所有实例对象

这是一个可以在 devtools 中用于辅助调试的命令 API。他可以根据构造函数找到当前执行环境中的所有实例

```js
queryObjects(Object) // 会找到执行环境中所有的对象，比如其中肯定有我们的框架对象如 vue
queryObjects(Array) // 会找到所有的数组对象
queryObjects(Promise) // 会找到所有的 promise 对象
```

#### 使用 js 方法监听滚动找出产生滚动的元素

有时候滚动条太多，不知道是谁产生的滚动条，可以暴力一点直接监听滚动事件。**open your mind...** 

```js
function findScroller(element) {
    element.onscroll = function() { console.log(element)}

    Array.from(element.children).forEach(findScroller);
}

findScroller(document.body);
```

