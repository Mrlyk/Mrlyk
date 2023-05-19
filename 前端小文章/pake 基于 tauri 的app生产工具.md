# pake

官方文档：https://github.com/tw93/Pake

作用：将网页打包成本地的 App。

原理：基于 tauri 跨端框架。

## 安装

```shell
npm i pake-cli -g
```



## 使用

```shell
# 命令使用
pake url [OPTIONS]...
-i path # 指定图标
-h # {number} height 窗口高度，默认 780px
-w # {number} width
-t # --transparent 沉浸模式，头部会和网页同色

# 举例，首次由于安装环境会有些慢，后面就快了
pake https://weekly.tw93.fun --name Weekly --transparent
```

options 文档：https://github.com/tw93/Pake/blob/master/bin/README.md



## 打包后软件的快捷键

| Mac   | Windows/Linux | 功能               |
| ----- | ------------- | ------------------ |
| ⌘ + [ | Ctrl + ←      | 返回上一个页面     |
| ⌘ + ] | Ctrl + →      | 去下一个页面       |
| ⌘ + ↑ | Ctrl + ↑      | 自动滚动到页面顶部 |
| ⌘ + ↓ | Ctrl + ↓      | 自动滚动到页面底部 |
| ⌘ + r | Ctrl + r      | 刷新页面           |
| ⌘ + w | Ctrl + w      | 隐藏窗口，非退出   |
| ⌘ + - | Ctrl + -      | 缩小页面           |
| ⌘ + + | Ctrl + +      | 放大页面           |
| ⌘ + = | Ctrl + =      | 放大页面           |
| ⌘ + 0 | Ctrl + 0      | 重置页面缩放       |