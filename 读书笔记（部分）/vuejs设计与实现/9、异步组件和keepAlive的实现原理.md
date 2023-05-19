# 异步组件和 keepAlive 的实现原理

[toc]

# 异步组件

实现异步组件并不需要框架层面的支持，但是我们可以给用户提供一个高阶组件，对异步组件的异常、加载状态、超时状态进行统一处理。比如vue3 提供的 `defineAsyncComponent`

下面是一个`defineAsyncComponent`的最小实现：

```js
function defineAsyncComponent(loader) {
  let InnerComp = null
  return {
    name: 'AsyncComponentWrapper',
    setup() {
      const loaded = ref(false)
      loader().then(c => {
        InnerComp = c
        loaded.value = true
      })
      return () => {
        return loaded.value ? { type: InnerComp } : { type: Text, children: '' }
      }
    }
  }
}
```

可以看到其实就是对要渲染的组件包了一层。

本身组件被编译后会具有 `render`方法，返回一个vnode进行渲染。

## keepAlive

keepAlive的原理是创建一个隐藏的元素，用来缓存组件的 vnode。元素隐藏时，将被包裹的组件移动到隐藏的容器元素上而不销毁他们，当元素展示时再移动到真正的容器中去。

其实现和渲染器紧密相关，渲染器在渲染时会做两件事：

1. 生成一个被KeepAlive包裹的内部组件实例，它不会渲染除了被包裹组件以外的内容
2. 会将一些标记和缓存的vnode挂载在这个实例上
