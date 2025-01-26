# AbortController 中断一切

`AbortController` 这个 API 我们一贯认为是用来中断 http 请求的，其实它还能做很多事，几乎可以中断一切异步操作。

## 基础使用

通过 new 构造实例

```js
const controller = new AbortController();
```

- `contorller.signal` 属性，这是一个 `AbortSignal` 实例，我们可以将它传递给要中断的 API，来响应中断事件并进行相应处理，例如，传递给 `fetch()` 方法就可以终止这个请求了；
- `contorller.abort()` 方法，调用这个方法会触发 `signal` 上的中止事件，并将信号标记为已中止。

### 终止 fetch 请求

```js
const controller = new AbortController();

fetch('https://example.com', {
  method: 'get',
  signal: controller.signal,
});

// 终止请求  
controller.abort();
```

请求终止相当于 fetch 这个异步方法的状态被置为 reject 了，所以自然也不会进我们的正常业务处理逻辑而是会被异常捕获。

```js
fetch('https://example.com', {
  method: 'get',
  signal: controller.signal,
}).cahtch(error => {
  if (error.name === 'AbortError') {
    console.log('请求取消');
  }
});
```

**注意**：请求虽然被终止了，但是服务端可能已经收到了请求，数据处理已经完成。

对这种情况一个可能的方式是前端重新请求服务端告诉服务端用户终止了请求，需要服务端完成回退操作；

上述的这种场景比较少，目前我还没遇到过真的需要这么操作的场景。

## 终止事件监听

通常我们为了终止一个事件监听，需要将事件监听的处理方法的引用保存下来，方便我们通过`removeEventListener` 来清除这个监听。有时候处理起来就比较麻烦。常规方式如下

```js
function handleClick() {
  console.log('处理点击事件')
}

document.addEventListener('click', handleClick)

// 终止事件监听
document.removeEventListener('click', handleClick)
```

使用 AbortController 就简单多了，我们可以在任何时候直接调用 abort 方法就能终止事件监听。

```js
const controller = new AbortController();

document.addEventListener('click', () => {
  console.log('处理点击事件');
}, {
  signal: controller.signal
})

// 终止事件监听 接受一个参数作为终止的 reason
controller.abort('手动终止');
```

## 终止任何异步方法

同时我们可以使用 AbortController 方法来中止几乎任何异步方法，这得益于 controller.signal 属性可以挂载一个事件监听，感知到 abort 方法的执行。这样我们就可以在这个事件处理中来中断我们的异步方法。

```js
const controller = new AbortController();

controller.signal.addEventListener('abort', (e) => {
  console.log('事件成功中止', e, controller.signal.reason)
})
```

**中断 Promise 的示例:** 

```js
const controller = new AbortController();

new Promise((resolve, reject) => {
  controller.signal.addEventListener('abort', (e) => {
    reject('手动中止')
  })
  setTimeout(() => {
		resolve();
  }, [3000])
})
```

**中断读写流示例**

```js
const abortController = new AbortController();

const stream = new WritableStream({
  write(chunk, controller) {
    console.log('正在写入:', chunk);

    // 监听中止信号
    controller.signal.addEventListener('abort', () => {
      console.log('写操作被取消');
      // 处理流中止逻辑，例如清理资源或通知用户
    });
  },
  close() {
    console.log('写入完成');
  },
  abort(reason) {
    console.warn('写入中止:', reason);
  }
});
```

## 静态类方法

`AbortSignal` 类还具有一些静态方法，可以简化 JavaScript 中的请求处理。

### AbortSignal.timeout

`AbortSignal.timeout()` 可以设置一个超时事件，自动触发终止

```js
document.addEventListener('click', () => {
  console.log('处理点击事件');
}, {
  signal: AbortSignal.timeout(2000), // 2 秒后自动中止这个监听
})
```

### AbortSignal.any

`AbortSignal.any([signal1, signal2])` 类似 Promise.any ，接收一个AbortController signal list。当任何一个终止信号触发时，终止都会被触发。

```js
// 取消信号 1
const autoController = new AbortController()

setTimeout(() => {
  autoController.abort();
  console.log('5 秒自动中止监听');
}, 5000);

// 取消信号 2 手动中止
const manualController = new AbortController()

document.addEventListener('click', () => {
  console.log('处理点击事件');
}, {
  signal: AbortSignal.any([autoController.signal, manualController.signal]), // 2 秒后自动终止这个监听
})
```

## 兼容性

兼容性目前几乎不用担心，几乎兼容所有主流浏览器。

![Google Chrome 2024-10-17 10.22.13](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/Google%20Chrome%202024-10-17%2010.22.13.png) 
