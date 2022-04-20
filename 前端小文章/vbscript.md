# vbscript

## CreateObject

CreateObject函数可以说是VBS中最强大的函数，没有了它，VBS只能用来算算数学题。我们都知道CreateObject函数可以创建对象，但是很多人并不知道其中的奥秘

曾经我也不明白为什么在CreateObject函数中传递不同的字符串就可以创建各种各样功能强大的对象。后来无意中看到UMU的《[UMU WSH 教程](9)CreateObject 过程》，才知道CreateObject函数创建的是COM对象，第一个参`数是COM对象的ProgID。再后来拜读了Jeff Glatt的《[COM in plain C](http://www.codeproject.com/Articles/13601/COM-in-plain-C)》，知道了如何用纯C语言编写COM组件。

```text
COM（组件对象模型）是遵循COM规范编写、以Win32动态链接库（DLL）或可执行文件（EXE）形式发布的可执行二进制代码，能够满足对组件架构的所有需求。
```

当然，作为VBSer，我们不需要去理解COM的原理或者本质。简单的说，COM就是别人写好的模块，我们要做的仅仅是调用它，而不必关心它的内部实现，这也是COM技术的一个初衷。ProgID可以认为是开发人员为COM对象起的一个名字，我们把COM对象的名字传递给CreateObject函数，告诉它我们想创建这个对象，CreateObject函数就会返回这个对象的指针给你。

例如我们可以用VB来编写一个COM组件，然后给它起个名字`demo.test`，那么注册该COM组件之后，就可以用CreateObject函数来创建了：

```vbscript
Set blog = CreateObject("demon.test")
blog.Open '假设我的COM对象实现了Open方法
```

我们常用的Scripting.FileSystemObject、WScript.Shell、ADODB.Stream等只不过是微软开发的系统自带的COM对象的名字罢了。

