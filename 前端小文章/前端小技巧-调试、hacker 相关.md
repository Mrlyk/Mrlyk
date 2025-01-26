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

#### (0, obj.fn)() 的作用

在看某些源码库时经常看到这样的代码，它的作用是什么呢？

这样的写法称为**逗号运算符**，它会对每个操作数求值。像`(0, obj.fn)`，最终会返回 `obj.fn` 的值，类似于 `fn = obj.fn` 。这样一下就能看出它的作用了：**改变 this 的指向**。正常我们直接调用`obj.fn` ，fn 中的 this 指向 obj 对象，而通过逗号操作符这种方式来调用，可以让 this 指向全局对象。这在某些场景下非常简洁有效。

#### setZeroTimeout 

使用 `setTimeout(fn, 0)` 在各个浏览器上都会有最短时间限制，比如 chrome 现在是 2ms，还不够快。

我们可以使用 postMessage 来模拟并达到接近这个最短时间的场景。执行100次该方法在 webkit 内核的浏览器上大约需要4～6ms，而 `setTimeout(fn, 0)` 的形式会话费近 1 秒的时间。

他们都达到了不阻塞 Main Thread 的目的，让任务异步执行。

```js
(function() {
  var timeouts = [];
  var messageName = "zero-timeout-message";

  // Like setTimeout, but only takes a function argument.  There's
  // no time argument (always zero) and no arguments (you have to
  // use a closure).
  function setZeroTimeout(fn) {
      timeouts.push(fn);
      window.postMessage(messageName, "*");
  }

  function handleMessage(event) {
      if (event.source == window && event.data == messageName) {
          event.stopPropagation();
          if (timeouts.length > 0) {
              var fn = timeouts.shift();
              fn();
          }
      }
  }

  window.addEventListener("message", handleMessage, true);

  // Add the one thing we want added to the window object.
  window.setZeroTimeout = setZeroTimeout;
})();
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
