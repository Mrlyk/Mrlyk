# 前端小技巧

> 一些日常开发可用的小技巧

[toc]

## JS 相关

#### 不使用 delete 来删除属性

存在的问题

- 性能
- 不可配置的属性会报 TypeError
- 删除不存在的属性时也会返回 true，造成误判
- 不能删除原型链上的属性，只作用在当前对象

更好的方式

```js
let obj = { a: 1, b: 2 }

const removePropertyFn = (target, key) => {
  const { [key]: _, ...newTarget } = target
  return newTarget
}
obj = removePropertyFn(obj, 'a')
```

## Web API 相关

#### storage 事件通信

[官方文档](https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event)

同域下的不同标签页之间可以使用该事件，注意触发时机：在**非当前**页面的**同域**（host 完全相同）页面下调用`localStorage`的相关方法时才会触发。

能监听到的事件类型：

- `localStorage.setItem()` 
- `localStorage.removeItem()` 

可以在回调方法的 `event`参数中，根据 key 和 newValue / oldvalue 获取具体的参数信息

具体实践可以看看这篇：https://segmentfault.com/a/1190000015443064?utm_source=tag-newest

#### Navigator.sendBeacon() 浏览器关闭的情况下发送异常信息

`navigator.sendBeacon()` 方法可用于通过[HTTP](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP)将少量数据异步传输到Web服务器。

https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator/sendBeacon
