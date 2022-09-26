# react hooks

来了来了，被人狂吹的 react hooks，vue 从中学习的 hooks。来一睹 react hooks 的真容。

**为什么要有 react hooks？**

前面我们学习了普通的类组件，能感受到几个问题

1. 写起来麻烦
2. 生命周期用起来也没那么舒服，经常要注意重渲染的问题
3. 原有的函数式组件突然有状态化的需求了，需要重写

所以 react hooks 诞生了，它让我们能够在函数式组件中

- 保存状态
- 处理副作用（类似 vue watch 但不一样）
- ......

**hooks 也仅能在函数式组件中使用！** 

[toc]

## hooks API

#### useState 函数式组件中的状态

函数式组件最大的痛点是没有状态。react 通过提供`useState`这个 hooks 来提供了状态！

```react
import React, { useState } from 'react'

export default function UseStateComponent () {
  const [name, setName] = useState('123')

  console.log('重新执行')
  return (
    <div>
      UseStateComponent
      <button
        onClick={() => {
          setName('456')
        }}
      >click
      </button>
      <div>{name}</div>
    </div>
  )
}
```

**在函数式组件中，this 指向 undefined，因为函数式组件只被当作普通函数调用！**

在上面的例子我们能看到

- `useState`返回一个数组，我们可以使用数组解构获得两个对象。
  1. 第一个是我们使用`useState`时传入的初始值，也是函数式组件中的状态
  2. 第二个是一个函数，可以用来改变状态，**触发函数式组件的重新执行**

同样**我们也不能直接去给 `useState` 定义的状态赋值**，原因是和`setState`一样的！！！

**经试验发现，使用 `useState`多次改变状态时，第一次改变为相同的值时函数式组件会重新执行，第二次则不会。应该是 react 自身的优化**

举个🌰：上面的函数式组件使用`setName('456')`，click 第一次和第二次都会"重新执行"，第三次就不会了。如果是`setName('123')`即第一次都一样的话，那么一次“重新执行”都不会！

**总结一下 setState：**

1. 更新异步
2. **更新会触发整个函数式组件的重新执行，包括原来在这个组件里声明的对象、函数都已经不是原来的那一个了。因为他们都重新创建了。这一点要特别注意！**

##### 异步

在类组件中 `setState`是异步的，**这里`useState`返回的函数也是异步的**。

这就有一个问题：如何在实图更新完成后再进行操作？

类组件可以使用生命周期`componentDidUpdate`、可以使用`setState`的第二个参数作为更新后的回调函数。那函数式组件配合`useState`如何实现呢？

这里就要说到第二个 hook `useEffect`

**最后这里还有一个疑惑点：明明 `setXXX`会导致整个函数式组件重新执行，那为什么 state 的状态不会重新被赋成初始值呢？**

##### useState 存储状态简要说明

状态状态，`useState`的作用就是给没有状态的函数式组件以状态，所以记住这个状态的变更是它本身的职责。

我们需要理解是如何做到的：

> 在 react 中通过 currentRenderingFiber 来标识当前渲染节点，每个组件都有一个对应的 fiber 节点，用来保存组件的相关数据信息。
> 每次函数组件渲染时，currentRenderingFiber 就被赋值为当前组件对应的 fiber，所以实际上 hook 是通过currentRenderingFiber 来获取状态信息的。
>
> currentRenderingFiber.memorizedState 中保存一条hook对应数据的单向链表。
>
> useState的第二个参数是更新数据的方法（dispatchAction），每次调用此方法时就会创建一个update,当我们多次更新时，就会生成一个环式链表更新

*ps: currentRenderingFiber 挂在 window 对象上，保存在内存中（这是 react 内存占用多的原因？）*

#### useEffect（watch）、useLayoutEffect 处理副作用

副作用大家都知道指什么（未预期的要发生的情况），下面是几种典型的副作用情况

- 上面我改变`useState`定义的状态后去操作 dom，发现 dom 还没变。这就属于异步更新带来的副作用，我们期望更改状态的操作执行后状态就发生了改变，但实际上并没有。这就是一个典型的副作用！
- 定时器使用后需要手动清除才行，这样算是预期之外的事情也就是副作用！
- 我们在**函数式组件中写了个请求方法，每次`setXXX` 导致函数式组件重新执行都会重新请求，重新请求又导致了`setXXX`的重复调用，形成了死循环**。也属于副作用！

react 提供了 `useEffect`、`useLayoutEffect` 来处理副作用！

##### useEffect

接收两个参数

1. 回调函数，**可以返回一个函数，在组件销毁时执行**
2. 依赖的对象数组

```react
useEffect(() => {
  console.log('useEffect')
}, [])
```

当依赖的对象**改变时**就会触发回调函数，**如果没有任何依赖就只会执行一次。也要注意 `useEffect` 的回调是异步的，有点像 vue 的 `$nextTick`**  

下面举个🌰：

每次 `text` 改变时将首字母置为大写

```react
const [text, setText] = useState('name')

useEffect(() => {
  setText(text.charAt(0).toUpperCase() + text.slice(1)) // 这里调用 setText 又会使整个函数式组件重新执行，导致第二遍进这里（注意不是因为检测到 text 变更再进的，有个顺序先后问题。setText 之后函数式组件会先整个重新执行）
}, [text])

return (
  <div>
    <h1>app-{text}</h1>
    <button onClick={() => { // 每次 text 改变，useEffect 都会触发
        setText('xiao')
      }}>click</button>
  </div>
)
```

这里还有一个问题就是，对依赖的监听是“深”还是“浅”——答案是**浅**

**即 `useEffect` 在依赖是对象的情况下，会不停的重复触发**。但是就像 vue watch 支持键路径一样，`useEffect`的依赖也支持键路径！

```react
useEffect(() => {
  setObjS({ name: 'mike'})
}, [objS.name]) // 支持键路径
```

有人就会问了，这里 `useEffect` 的依赖是`objsS.name` ，那如果我改变`objS.age`，`useEffect` 会触发吗？

答案是会的。

```react
const [objS, setObjS] = useState(obj)

useEffect(() => {
  setObjS({ age: 20 })
}, [objS.name])
```

看改变状态的方式和`setState`类似，都是接收一个参数。但是和 `setState` 不同的是:

- `setState` 会对对象使用`Object.assign`进行合并
- **`useState` 则是直接覆盖**，上面相当于只有一个`age`属性了

所以当我只改变`objS.age`时，`objS.name`也一定变了（被删了也是变）

*ps：这里 react 使用`Object.is`来判断值是否相等！* 

开头说了`useEffect`的第一个回调函数可以再返回一个函数，**可以取代`componentWillUnMount`在销毁时执行一些处理副作用的操作！**

```react
useEffect(() => {
  const timer = setInterval(() => {
    console.log(1111)
  }, 500)
  return () => {
    console.log('销毁')
    clearInterval(timer)
  }
}, [])
```

但是要注意，**如果有依赖的 state**，在 state 发生变更时，也会触发这个返回的函数。因为`setXXX` 带来的函数式组件重新执行（~~自然也包含销毁的过程~~），会导致 `useEffect` 也重新执行，上面的定时器虽然被销毁了，但是随后又重新创建了个新的，没有达成真正的销毁目的！

所以如果只是为了处理组件销毁时的状态，可以单独写一个`useEffect`并且不依赖任何 `state`！！！

*ps: 这里有一点奇怪，`setXXX` 理论上会触发组件销毁，但是当我写多个 `useEffect` 的时候通过`setXXX` 却没有触发其他（没有依赖这个`XXX`状态）的`useEffcet` 的 return 函数*	

上面的问题经过实验得出：`setXXX` 虽然会导致整个函数式组件重新执行，但没有依赖`XXX`状态的`useEffect` 只会在第一次时执行。即**`setXXX`只会影响对`XXX`状态有依赖的`useEffect`声明，对那些无依赖的 `useEffect` 不会产生影响（包括销毁也不会执行）**

> react 会使用 depends 做比较，给有变更的打上 hasEffect 的标记，这样表明这个 useEffect 被影响了应该重新执行。否则不会做任何操作！

所以我们要抓住`useEffect`  的本质：

- 首次渲染和**依赖变更时**执行，如果依赖没有变更，那就不会**再**执行！！！
- 上面我们写的 `useEffect` 都加了第二个参数一个空数组`[]`，这样就相当于没有任何依赖！只在组件**第一次加载**和销毁时会执行相关方法
- **如果我们忽略第二个参数，那么 `useEffect` 会在每轮组件渲染完成后都执行！！！**什么意思呢？就是说`setXXX`带来的函数式组件整体重新执行不会执行有`[]`依赖的`useEffect`但是会执行没有第二个依赖参数的`useEffect`！（当然依赖`XXX`的肯定也会执行）

**总结一下**：

1. `useEffect`在没有依赖的时候（`[]`）只会执行一次

2. `useEffect`回调也是异步，React 会等待浏览器完成画面渲染之后才会延迟调用 `useEffect` 

3. `useEffect`依赖比较执行的是浅比较

4. `useEffect`可以多次声明，回调函数都会执行

5. `useEffect` 回调函数可以返回一个函数，在函数销毁时执行。但要注意1、`useEffect`的本质；2、如果有依赖会在依赖变更时也执行！

6. `useEffect` 第二个参数传`[]`和不传的效果时完全不同的！！

7. **`useEffect`可以替代类组件中的`componentDidUpdate`，实现真正的类似于 vue watch 的方法**。在类组件中要初始化时获取数据又要在更新时获取数据，就需要使用多个生命周期。有了这个钩子之后她一个人就可以干这个事情

   ```react
   useEffect(() => {
     type === 'coming' && getComingSoonFilms() // type 变更时发送不同的请求，再配合 useState 完成重渲染
     type === 'release' && getReleaseFilms()
   }, [props.type]) // 依赖 props 的 type
   ```
8. **调用 `setXXX` 会让整个函数式组件重新执行，这也会让在其中声明的一些定时器方法无限触发。如果 `useEffect`又恰好依赖了定时器中的状态来改变某个状态，可能会导致无限循环** 

##### useLayoutEffect

下面来讲一讲另一个副作用处理 hook。**他和 `useEffect` 最大的区别是调用时机不同**

- `useEffect` 会在页面渲染完后（CSSOM + DOM 结合成渲染树）才会执行，这时候用户已经能看到页面了，不会阻塞页面的渲染
- `useLayoutEffect`则执行的更早，会在 DOM 更新完成后立刻就同步执行，阻塞页面渲染

但是`useLayoutEffect`也有自己的作用——避免布局抖动，即对用户可见的 DOM 不会因为我们的 DOM 操作突然闪一下，给用户体验造成影响。因为`useLayoutEffect`中执行 DOM 操作是同步的，并且 react 会对其进行优化，合并更改！

所以**大部分情况下都还是使用`useEffect`**，只有在修改 DOM 且任务耗时不长的时候才使用这个 hook，看他的名字也能看出来。主要是用来处理布局相关的副作用的！

#### useCallback 记忆函数

上面我们使用 `useState` 会记住状态，在函数式组件重新渲染时不会被重新初始化！但是上面记住的是状态，如果我们想记住函数式组件中的其他声明呢？

比如声明`const add = (a, b) =>  (a+b)` ，当函数式组件重新渲染时这个函数表达式又会被重新声明一遍，这显然是没有必要的。`useCallback`的作用就是缓存这些声明，只有他依赖的状态变化了才去重新声明这个函数！

下面是个🌰：

```react
import React, { useState, useCallback } from 'react'

export default function UseCallbackCom () {
  const [count, setcount] = useState(0)
  const callback = useCallback(
    () => {
      console.log('count 不会改变，每次触发时都是原来的值', count)
    },
    []
  )
  console.log('每次渲染都改变的 count', count)

  return (
    <div>
    {count}
    <button onClick={() => { setcount(count + 1)}}>count++</button>
    <button onClick={callback}>useCallback</button>
    </div>
  )
}
```

上面的例子中，使用`useCallback`声明的 `callback` 方法，无论在 count 的值是多少的时候调用该方法，他的值都是初始的 0。**因为他是“记忆函数”，且没有任何依赖，整个函数的状态一直都是历史的**

该方法主要用来优化函数式组件，让一些特别大的方法不要重复声明影响性能！

那他是怎么做到的呢？实际上就两个字——**闭包**

通过闭包在自己的函数作用域中保存了自己的外部变量，只要依赖不改变就不会去重新执行这个函数，变量就会一直是自己闭包中的值！

*ps: 和 useEffect 一样，如果没有传第二个参数，那么在组件每次重新渲染时都会重新声明！*

#### useMemo 记忆组件（computed）

**`useMemo`和`useCallback`的区别**：`useMemo`会执行第一个函数，并且将 return 值返回并记忆。

所以在我们需要**事先进行一些计算的时候使用`useMemo`**，否则使用`useCallback`即可！

`useMemo` 其实是对结果有一个缓存功能，相当于 vue 中的 `computed`计算属性！

下面是个🌰：

```react
import React, { useMemo, useState, useEffect } from 'react'

export default function UseMemoCom () {
  const [cinemas, setCinemas] = useState([])
  const [keyword, setkeyword] = useState('')

  useEffect(() => {
    console.log('useEffect')
    // ... 获取电影院列表
    setCinemas(data?.cinemas || [])
    // ...
  }, [])

  const cinemasList = useMemo(
    () => {
      console.log('usememo')
      return cinemas
        .filter((e) => {
          return e.name.includes(keyword)
        })
        .map((item) => {
          return <li key={item.cinemaId}>{item.name}</li>
        })
    }
    ,
    [keyword, cinemas] // 在 keyword 和 cinemas 改变时执行上面的过滤方法，并将结果返回
  )

  const search = (e) => {
    setkeyword(e.target.value)
  }

  return (
    <div>
      <input value={keyword} onChange={search} />
      <ul>{cinemasList}</ul>
    </div>
  )
}

```

*ps: useMemo 比 useEffect 执行的更早！*

上面有了记忆函数、记忆组件，但是有一个问题：如果组件中函数很多难道我要一个个全部用 `useCallback` 包裹一遍吗？那太麻烦了，所以 React 提供了`React.memo`来包裹整个函数式组件，以达到优化的目的。这个后面再详细说明！

#### useRef

在类组件中写了很多`React.createRef`来创建 DOM 对象的引用，在函数式组件中**依然**可以继使用这个方法来。react hooks 也提供了一个钩子来创建这个对象——`useRef`

他的作用和`React.createRef`的作用是一模一样的！但是也具有两点特性：

1. `useRef` 接收一个初始值作为`ref.current`的值。即`const ref = useRef(10)` ——> `ref.current === 10`
2. `useRef`的值会被缓存，所以**在函数式组件中，除了使用`useState`还可以用`useRef`来缓存状态**（这与 react hook 的实现机制有关）

下面举个🌰：

```react
import React, { useState, useRef, useCallback } from 'react'

export default function RefTodoList () {
  const ref = useRef()
  const count = useRef(0)
  const [stateCount, setstateCount] = useState(0)
  
  // ...

  return (
    <div>
      { count.current } - {stateCount}
      <button onClick={() => {
        count.current += 1 // 值会累加，不会因为组件重新渲染就导致值被重置
        setstateCount(count.current + 1) // 但是要手动通过其他方式触发组件重新渲染，直接使用 useRef 改值不会重新渲染组件
      }}>count++</button>
      <br />
      <input
        ref={ref}
      />
      <button
        onClick={() => {
          setList([...list, ref.current.value])
          ref.current.value = ''
          ref.current.focus()
        }}
      >
        add
      </button>
      { /*...*/ }
    </div>
  )
}

```

**需要注意的是 `useRef`的值改变 不会触发组件的重新渲染！！！**

#### useReducer、useContext 减少组件层级

##### useContext

在 react 类组件中使用 `React.createContext` 创建了一个全局上下文对象用来通信。



## 补充点

#### 手写 useState

```js
import ReactDOM from 'react-dom'

let _state //全局_state用来存储state的值，避免重新渲染的时候被myUseState重置为初始值
const myUseState = initialValue => {
  _state = _state === undefined ? initialValue : _state // 没有初值的时候赋初值
  const setState = (newValue) => {
    _state = newValue
    render() // 调用重新渲染
  }
  return [_state, setState]
}

const render = () => {
  ReactDOM.render(<App1 />, document.getElementById('root'))
}

function App1() {
  const [n, setN] = myUseState(0)
  return (
    <div className='App'>
    <p>{n}</p>
		<button onClick={() => setN(n + 1)}>+1</button>
		</div>
		)
}
```

