# react 踩坑记录

1. `functions are not valid as a React child. This may happen if you return a Component instead of ...`

这个报错一般是想通过函数动态的获取样式、组件的时候，只声明了函数却没加`()`执行。虽然在`onClick`这类事件中我们牢记不能加`()`执行。**但是作为渲染时就要拿到结果的方法是一定要立即执行的。** 

2. `setState`会重新执行`render`函数，所以不要把一些只要执行一次的方法放在`render`函数中