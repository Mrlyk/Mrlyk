# react hooks

来了来了，被人狂吹的 react hooks，vue 从中学习的 hooks。来一睹 react hooks 的真容。

**为什么要有 react hooks？**

前面我们学习了普通的类组件，能感受到几个问题

1. 写起来麻烦
2. 生命周期用起来也没那么舒服，经常要注意重渲染的问题
3. 原有的函数式组件突然有状态化的需求了，需要重写

所以 react@16.8 （2018 年底）hooks 正式上线，它让我们能够在函数式组件中

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

3. 连续多次调用时结果会被合并，只有最后一次生效，为了避免这种情况**可以传入一个函数**来接收上一次的值

   ```ts
   // loadPicInDOM 会在循环中被多次调用
   const loadPicInDOM = (src) => {
     if (!src) return;
     const image = new Image();
     image.src = src;
     image.onload = () => {
       setImgLoaded((prev) => {
         console.log('prev', prev); // 上一次的参数
         return {
           ...prev,
           [src]: true,
         };
       });
     };
   };
   ```

   

##### 异步

在类组件中 `setState`是异步的，**这里`useState`返回的函数也是异步的**。

这就有一个问题：如何在视图更新完成后再进行操作？

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

⚠️⚠️⚠️：**useEffect 会捕获执行时的 state 和 props，如果 useEffect 没有重新执行，那么里面的 props 和 state 就是老的** 

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

6. `useEffect` 第二个参数传`[]`和不传的效果是完全不同的！！

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

#### useImperativeHandle

上面使用 useRef 可以获取到子组件的 DOM，可以存储值的状态。但是函数式组件和类组件有一个很大的不同，类组件式真的有类实例在的，我们可以通过类实例直接调用组件实例上的方法。

但是函数式组件不是实例，函数式组件中的方法也不会挂在函数上，所以 useRef 在函数式组件中只能用来访问 DOM，不能访问函数式组件的方法（这点和 vue 很不一样）。那么如果我真的想**调用子组件的方法**怎么办呢？使用`useImperativeHandle` 暴露子组件的方法。

`useImperativeHandle(ref, handler, dependencies?)`

接受 3 个参数

-  ref 对象，一般是使用`forwardRef` 转发过来的 ref
- handler，要要暴露的处理方法
- depencies，和其他钩子一样，决定什么时候重新创建的依赖

下面是一个例子

```tsx
/*===========子组件=============*/
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});

/*=============父组件==============*/
import { useRef } from 'react';
import MyInput from './MyInput.js';

const ref = useRef(null);

function handleClick() {
	ref.current.focus();
	// This won't work because the DOM node isn't exposed:
	// ref.current.style.opacity = 0.5;
}

<MyInput placeholder="Enter your name" ref={ref} />
<button type="button" onClick={handleClick}>
  Edit
</button>
```



#### useReducer、useContext 全局状态管理

##### useContext

在 react 类组件中使用 `React.createContext` 创建了一个全局上下文对象用来通信。这里则是对应的指令式的写法。

该 Hook 会触发重渲染，并使用最新传递给 `MyContext` provider 的 context `value` 值。**即使祖先使用 [`React.memo`](https://zh-hans.reactjs.org/docs/react-api.html#reactmemo) 或 [`shouldComponentUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，也会在组件本身使用 `useContext` 时重新渲染。**

`useContext` 主要是在消费者的修改，生产者还是用组件式的写法。

```react
import React, { useState, useEffect, useContext } from 'react'
import { GlobalContext } from './film'

export default function FilmList () {
  const [films, setFilms] = useState([])
  
  // ...

  const { setSelectFilm } = useContext(GlobalContext) // 这里接收生产者提供的整个 context 对象
  const filmList = (setSelectFilm) =>
    films.map((film) => (
      <li
        key={film.filmId}
        onClick={() => {
          setSelectFilm(film)
        }}
      >
        {film.name}
      </li>
    ))

  return ( // 渲染这里就简单多了，不用再使用 GloabalContext.Consumer 来包裹一层了，减少了层级
    <div>
      <ul className="film-list">{filmList(setSelectFilm)}</ul>
    </div>
  )
}

```

**ps：组件式的`React.createContext()`创建`context`的方法在函数式的组件中也可用。只是 hooks 的写法会让“消费者”写起来更简单。**

##### useReducer

`useReducer`的理念来源于 redux 状态管理，通过统一的状态管理来降低耦合度。**他是`useState`的替代方案。**和`useState`的区别是他将组件状态放到组件之外，可以传递到其他组件方便管理。就像一个小的 redux。

`useReducer`接收一个形如 `(state, action) => newState` 的 **reducer** 和**初始化状态**作为参数。同时返回一个 state 和一个 disatch 方法（和 vuex 中的 state、disatch 类似。）

如果要更改状态，需要调用 **`dispatch`方法，这个方法会调用状态管理的入口方法**，也就是我们声明的 reducer。

```react
import React, { useReducer } from 'react'

// 接收两个参数，之前的状态和 dispatch 调用时传入的参数
const reducer = (prevState, action) => {
  const nextState = { ...prevState } // 注意不能直接修改历史状态，需要用一个新的状态来承载
  switch (action.type) {
    case 'minus':
      nextState.count--
      break
    case 'plus':
      nextState.count++
      break
  }
  return nextState
}

// 在组件外部来管理状态
const initialState = {
  count: 0
}

export default function UseReducerComponent () {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <div>ReducerComponent: {state.count}
    <button onClick={() => {
      dispatch({
        type: 'minus'
      })
    }}>dispatch - </button>
    <button onClick={() => {
      dispatch({
        type: 'plus'
      })
    }}>dispatch + </button>
    </div>
  )
}
```

只要我们使用一个抽离的 store.js 来保存状态并且在需要的是机会引入，就能实现状态共享了。和我们的状态共享工具的核心理念是相同的。只不过功能没有那么强大。

**`useReducer`还可以接收第三个参数**

有时候状态的初始值需要依赖于外部状态如 props 进行动态计算，这时候可以传入第三个参数（一个函数）作为惰性初始化函数。这时候初始状态的实际值为`init(initState)`。

举个🌰：

```react

const initState = {
  child2Text: 'child2',
  child3Text: 'child3'
}

const reducer = (prevState, action) => {
  // ...
}

const init = (initState) => { // 这里计算后返回的参数才是真正的初始状态
  return { ...initState, child3Text: 'child3-init' }
}


const [state, dispatch] = useReducer(reducer, initState, init)
```



**什么时候使用`useReducer`**

官方将`useReducer`作为`useState`的替代品，声明的使用场景如下：

-  state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 
- 使用 `useReducer` 还能给那些会触发深更新的组件做性能优化



**配合`useContext`使用**：

既然`useReducer`是想剥离组件与状态之间的耦合性，那么还需要将状态分发到任意的组件。

如果要分发的组件都是子组件，那么我们直接使用 props 传递即可。但是都用到了`useReducer`就是想任意组件之间的通信。所以我们可以配合`useContext`一起，在任意组件中获取状态。



**使用 `useReducer` 使用要注意的有三点：**

1. 声明`useReducer` 时传入的参数需要在全局声明，而不是放在组件内部
2. 在 reducer 中更改状态时，不能直接更改原状态，需要使用一个新的状态并返回这个状态作为新状态
3. 一般配合 `useContext`使用

#### 自定义 hook

自定义 hook 是 react > 16.8 之后新增的。在编写代码时我们通常会提取公共的逻辑的写成公共函数。而函数式组件和 hook 都是函数，所以我们可以把 react 中的公共逻辑提取成一个 hook，在需要时调用。

**要求：必须以`use`开头，一部分是出于语义化目的，另一部分是因为 react 会对 use 开头的函数进行检查，判断其是否符合 hook 规则。**  

**自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。**

既然目的都是提取公共逻辑，那么自定义 hook 和直接提取公共逻辑的函数有什么区别呢？

1. 自定义 hook 中如果没有再调用其他 hook ，只是普通的函数逻辑，那么和普通的提取公共逻辑的函数没有区别。如果有调用其他 hook，react 则会对其进行检查，判断是否符合 hook 规则。比如是否在非 react 函数中调用了。
2. 自定义 hook 如果其中有 `setState`之类的方法，那么自定义 hook 也会**使调用他的函数式组件重新执行。**



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

#### hook 规则

- **只在最顶层使用 hook。不要再循环、条件、嵌套函数使用调用 hook**
- **只在 react 中使用 hook，不要在普通的 js 函数中使用**
