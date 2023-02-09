# gitattributes

在项目中有时会看到 .gitattributes 文件，他的作用是什么呢？

我们以 windwos 和 macos 中一个常见的 git 提交问题为例。

在 macos、linux 等操作系统中中文件的结尾通常是 LF（Line Feed）换行符。而在 windows 上是 CRLF（Carriage Return Line Feed），表示先换行再回车，就像老式的电传打字机一样。

所以如果从 windows 上拷贝过来的代码在 mac 上提交的时候 git 总会提示这个换行问题。

在有了 .gitattributes 之后，我们可以针对文件类型来配置不同的值，如下：

```text
*.js    eol=lf
*.jsx   eol=lf
*.json  eol=lf
# 如果使用 !xxx 可以覆盖其他地方的设置为未设置，相当于 css 的 unset
```

eol（end of line）使用 LF 换行符。

他还有很多其他属性，比如更改 git 对文件的语言识别，可以手动声明将什么文件识别成什么语言的：

```text
.jsx linguist-language=Vue
```

如上项目中的 jsx 文件会被 git 当成 Vue。

## 重置 GitAttributes

```text
git rm --cached -r
git reset --hard
```



## 参考文章

什么是 .gitattributes ？：https://zhuanlan.zhihu.com/p/108266134

官方文档：https://www.git-scm.com/docs/gitattributes