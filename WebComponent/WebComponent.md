# WebComponent

什么是`WebComponent`呢？官方定时是 HTML5 提供的一套自定义元素的接口，允许创建可重用的定制元素（它们的功能封装在您的代码之外）并且在您的 web 应用中使用它们。

有以下三大要素

- **Custom elements（自定义元素）：** 一组 JavaScript API，允许您定义 custom elements 及其行为，然后可以在您的用户界面中按照需要使用它们。
- **Shadow DOM（影子 DOM）** ：一组 JavaScript API，用于将封装的“影子”DOM 树附加到元素（与主文档 DOM 分开呈现）并控制其关联的功能。通过这种方式，您可以保持元素的功能私有，这样它们就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。
- **HTML templates（HTML 模板）：** `<template>` 和 `<slot>` 元素使您可以编写不在呈现页面中显示的标记模板。然后它们可以作为自定义元素结构的基础被多次重用。

像`<video>`标签就是一个天然的`WebComponent`