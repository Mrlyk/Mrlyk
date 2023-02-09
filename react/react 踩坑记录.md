# react 踩坑记录

1. `functions are not valid as a React child. This may happen if you return a Component instead of ...`

这个报错一般是想通过函数动态的获取样式、组件的时候，只声明了函数却没加`()`执行。虽然在`onClick`这类事件中我们牢记不能加`()`执行。**但是作为渲染时就要拿到结果的方法是一定要立即执行的。** 

2. `setState`会重新执行`render`函数，所以不要把一些只要执行一次的方法放在`render`函数中



## 工程相关

#### 热更新时报错

1. `process is not undefined`

这是 react 一个插件的问题，将`react-error-overlay@6.0.9`手动安装到 dev 依赖中即可解决。

这里很奇怪，查看 package-lock.json 发现安装的就是这个版本。可能是依赖层级没有扁平化到第一层导致无法找到的原因，手动安装就可有了。