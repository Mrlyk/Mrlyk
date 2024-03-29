# vue router 鉴权

前端处理权限问题的场景也有很多。这里对几个经典场景的权限处理方案做个记录！



## 用户通过地址栏输入地址跳转

不同角色有不同权限，用户如果直接在地址栏输入没权限的菜单，在没有做限制的情况下也会跳转过去。针对这种情况我们有几种方案，总的来说都是要借助 vue router 的路由守卫来处理，只是处理的细节不同。

1. 每次跳转前给路由参数中加入一个特殊参数，如果跳转时能获取到这个特殊参数表示是正常跳转，否则为用户手动输入地址跳转；
2. 借助浏览器提供的 api——`window.performance.navigation.type`，如果为 0 表示是手动输入链接跳转的；（官方文档表示未来可能会废弃该特性）

```text
performance.navigation.type 该属性返回一个整数值，表示网页的加载来源，可能有以下4种情况：

- 0：网页通过点击链接、地址栏输入、表单提交、脚本操作等方式加载
- 1：网页通过“重新加载”按钮或者location.reload()方法加载
- 2：网页通过“前进”或“后退”按钮加载
- 255：任何其他来源的加载
```

