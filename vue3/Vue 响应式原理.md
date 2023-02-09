# Vue 源码浅析

> 根据源码来理清 vue 响应式原理与其核心实现

## vue2 响应式原理  

#### 对象初始化

总的来说是通过 Object.defineProperty 这个方法，对 data 中所有的属性的 get 和 set 重新定义，拦截属性的获取进行依赖收集。拦截属性的更新进行通知。 
**大概过程**：  
initData() 方法初始化(defineReactive)用户传入的参数 ->  
使用 Observer 构造函数包装对象观测数据，同时该函数会对对象和数组分别处理 -> 
使用 observe 方法处理需要被监听的对象和数组，同时对根实例上对数据进行标记 ->  
如果数据是一个对象类型就会调用 this.walk(value) 进行处理 -> 
this.walk 内部使用 defineReactive 循环处理对象属性，定义响应式变化

**深入具体（部分要点）**：  
最开始调用对是 this._init 方法, 在该方法中执行 initState(vm)，initState(vm) 中分别对 props, methods, data, computed, watch 进行了初始化。

- defineReactive: 该方法的作用就是把一个对象变成响应式的对象。核心就是重新定义了对象的 get 和 set方法。该方法首先创建了一个订阅者收集 dep，每当读取该对象时，就会触发 dep.depend()方法，收集订阅者。每当重新设置该值时就会触发 dep.notify() 通知订阅者执行更新。
- new Observer: 构造函数，把构造添加到数据 value 的 \_\_ob__ 属性上同时创建了一个收集器。之后分别对数组和对象类型进行了处理。 
- observe: 该方法首先判断对象是否含有 \_\_ob__ 属性，如果有，说明是被 new Observer 实例化的对象（被监听了）不再操作。否则做一些边界情况的判断并且判断是否是数组或对象，如果是则使用 Observer 函数重新构造该对象。然后如果是根数据，给vmCount 属性 ++ ，作为一个标记。表示这是一个根数据。vmCount 属性在 Observer 构造函数的属性。
- this.walk: 该方法中使用 Object.keys 对对象进行遍历，分别对每一个属性使用执行 defineReactive 方法，进行监听。
- 嵌套监听：若 data 中的 value 是个对象或数组，在 defineReactive 中会对非“浅层”数据调用 observe 方法进行处理。在 observe 函数中进行数组和对象的以及一些边界情况的判断后又会执行 new Observer 构造函数，在 Observer 函数中又会调用 defineReactive 对数据进行响应式定义，就这样形成了一个完美的递归调用判断来处理嵌套数据。
- Vue.set ：该方法会把传进来的数据，调用 defineReactive 方法变为一个响应式的对象，同时手动触发ob 父对象的 notify() 事件，来保证变化被监听到。

#### 订阅者的收集 

在上一节中，我们知道响应式的监听和通知，都是通过 new Dep() 创建一个订阅者来对数据对变化进行检测。他的实现要点如下：
首先 vue 数据的监听基于**发布订阅者模式**，与观察者模式不同的是，观察者模式直接观察到数据的变化作出响应的响应。而发布订阅者模式则是通过第三方来通知观察者对象的改变。对象的改变会通知到这个第三方，而第三方会通知订阅了该对象的观察者。 

在将 vue 对象挂载到根节点 dom 对象时，最后会指向一个 mountComponent 方法。在该方法中会实例化一个 Watcher 对象，而这个实例化对象就是订阅者。 
在 Vue 中 Watcher 分为三种类型：

- render Watcher
- computed Watcher
- user Watcher (vm.$watch)

##### Watcher对象 

拥有几个重要属性 this.deps、this.newDeps、this.depIds、this.newDepIds。 
Dep 构造函数:

```js
var uid = 0
var Dep = function Dep() {
  this.id = uid++
  this.subs = []
}
```

- **render Watcher**
  在 Watcher 的构造函数中，最后会执行一个 this.get 方法。-> 
  this.get 方法调用 vm._render() 方法生成 vnode，这个过程中会读取 data 中的数据 -> 
  在上节中说到，defineReactive 方法中，重写了对象的 get 方法，读取对象时会触发 get 方法，get 方法中又会触发 dep.depend()方法。 ->
  dep.depend() 方法中包含一个 Dep.target.addDep 方法，该方法将对象（发布者）的收集器 Dep 添加到 Watcher 对象的 this.newDeps 中。 ->  
  然后如果该对象在 this.deps 中也没有，则调用 Dep 的 addSub 方法，收集订阅者放到收集器 Dep 的 subs 中。-> 
  这样就完成了订阅者的收集。最后还会执行 finally 中的所有方法，清除无用的发布者。

*为什么 Dep.target 就是当前这个 Watcher 对象？*
在 get 方法中有一个 pushTarget(this) 方法。 
在 pushTarget 方法中，就会把 Dep.target 指向这个 this，也就是这个 Watcher 对象。这样就保证了同一时间只有一个 Watcher 被收集。之后会维护一个栈，这个栈存了所有的 Watcher，每执行完一个就把下一个 Watcher 赋值给 Dep.target。

*user Watcher 和 computed Watcher 后续再补充。*


#### 订阅者的响应 

在第一节中，defineReactive 方法中重新定义了 set 方法。该方法在数据变化后会执行一个 dep.notify() 方法通知订阅者。  
**dep.notify() 方法**：

```js
Dep.prototype.notify = function notify() {
  var subs = this.subs.slice()
  if (!config.async) {
    sub.sort(function(a, b) {
      return a.id - b.id
    })
  }
  for (var i = 0, l = sub.length; i < l; i++) {
    subs[i].update()
  }
}
```

在 dep.notify() 方法中，对每个订阅者调用 update() 方法。

**sub[i].update()** 

```js
Watcher.prototype.update = function update() {
  if (this.lazy) {  // lazy 是计算属性订阅者 computedWatcher 的标志
    this.dirty = true // 在计算属性的订阅者中 如果 dirty 为true 才会重新触发计算（在其中执行 this.get 方法）, 所以只有在订阅者订阅的对象（发布者）变化时才会将 dirty 设置为 true， 这也就是 computed 缓存的原理
  } else if (this.sync) { // this.sync 默认为false
    this.run()
  } else {
    queueWatcher(this) // render Watcher 最后执行该方法
  }
}
```

**queueWatcher 简单实现如下：**

```js
var queue = []
var has = {}
var waiting = false
var flushing = false
var index = 0
function queueWatcher (watcher) {
  var id = watcher.id
  if (has[id] == null) { // 首先判断该 watcher 是否已经被添加到响应式队列中了，去重
    has[id] = true
    if (!flushing) {
      queue.push(watcher)  // vue 中的响应不是即时的，而是在nextTick 中执行，这就是为了处理重复的订阅者响应
    } else {
      var i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i --
      }
      queue.splice(i + 1, 0, watcher)
    }
    if (!waiting) { // 通过 waiting 确保一个队列中只会执行一次 nextTick(flushSchedulerQueue)
      waiting = true
      if (!config.async) {
        flushSchedulerQueue()
        return 
      }
      nextTick(flushSchedulerQueue)
    } 
  }
}
```

接着在 **flushSchedulerQueue** 这个方法中使用排序，将订阅者从小到大排序。（订阅者在创建时 id 也是自增 1 的）  
排序的原因：

- Vue 中组件更新从父刀子，所以组件中响应的顺序也要先父后子。
- 用户自定义的订阅者（vm.\$watcher）比渲染订阅者 render watcher 的创建要早。因为 init 方法先于 vm.\$mount 调用，所以用户自定义的订阅者要先于渲染订阅者。

然后 flushSchedulerQueue 方法中会遍历当前的 queue 队列，取出队列中的 watcher ，分别执行 watcher.run() 方法。之后为了能继续通知这个订阅者，会把 has[id] 置为 null。  

在执行 watcher.run() 的过程中，可能会导致其他被订阅对象（发布者）发生变化。然后又去调用 queueWatcher 函数。所以在 flushSchedulerQueue 方法中，将 flushing 设置为了 true 。执行 queueWatcher 中的 else 方法，将最大的订阅也就是最新添加的订阅者放到 queue 队列的最后一个位置上。最后为了防止循环响应次数过多，使用 circular 对象记录了一个 watcher 的响应次数，并设置最大限制。

**watcher.run() 方法:** 
首先判断实例是否被销毁 -> 
如果没有被销毁则继续执行 value = this.get() 方法，就是渲染订阅者的响应 -> 
因为在 this.get 方法中，执行了 value = this.getter.call(vm, vm)。这里的 this.getter 是在 Watcher 的构造函数中被赋值的

```js
if (typeof expOrFUn === 'function') {
  this.getter = expOrFn
}
```

这个 **expOrFn** 又是在 new Watcher 这个构造器的时候传入的 **updateComponent**  

```js
var updateComponent = function () {
  vm._update(vm._render(), hydrating)
}
```

这里的 vm._update 就是在 vNode 渲染成真实的 DOM，所以这里就是在响应了。 
最后如果是 vm.$watch 这种用户自定义的监听，那还会判断值是否改变或者是否深度监听，如果是则调用用户传入的回调方法。

**总结**

触发对象的 setter 函数 -> 
调用 Dep 实例方法 notify 通知订阅者 Watcher ->
执行 Watcher 的 update 实例方法 ->  
将 Watcher 的实例放入一个队列中并去重，然后在下一个事件循环中遍历这个队列 ->  
执行 Watcher 的 run 方法，其中：  

- 渲染订阅者是通过执行其实例方法 get 重新求值完成响应；
- 计算属性则是通过实例对象的 dirty 属性作为标志，进行缓存，最后也是执行 get 方法完成响应；
- 用户自定义的的订阅者则是通过回调函数来完成响应。