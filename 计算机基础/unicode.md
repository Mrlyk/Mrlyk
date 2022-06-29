# unicode 

unicode 是传说中的『万国码』，意图使用一个标准码表表意全世界所有的符号，解决乱码问题。例如『中日韩统一表意文字』的码位范围就在：4E00 - 9FFF。

- [Unicode 字符百科 ](https://unicode-table.com/cn/#cjk-unified-ideographs) 
- [Unicode特殊字符编码](https://blog.csdn.net/qq_41082953/article/details/104138667) 

大部分编码都有其固定作用，**但 unicode 留出了一个『私用区』**—— \E000 - \F8FF

所以你会看到 font awesome 的编码从 F000 开始，iconmoon 从 E900 开始，阿里 iconfont 从 E000 开始。

**所以我们可以定义这些『私用区』为自己的特殊图标甚至文字** 