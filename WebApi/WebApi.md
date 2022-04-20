# WebApi

> 经常忘记的一些 web api 的知识点

## Event

> DOM 中出现的事件

官方文档：[`Event`](https://developer.mozilla.org/zh-CN/docs/Web/API/Event) 

#### traget 和 currentTarget

两个都是 Event 接口的只读属性，区别在于

- currentTarget：总是指向事件绑定的元素，如下会指向 button 这个元素
- target：指向真正触发事件的元素，如下会指向 span 这个元素

```html
<button onclik="test()"><span>test<span></button>
```

