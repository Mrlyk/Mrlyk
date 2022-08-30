# react 生命周期

vue 有 vue 实例的生命周期，react 也有 react 组件的生命周期。一个学习 react 却不了解 react 生命周期的人就像学前端不学 js，只能说明他的前端造诣和自我修养不足，他理解不了这种内在阳春白雪的高雅艺术。他只能看到外表的 html 堆砌，参不透 react 中深奥的精神内核。他的前端水平就卡在这里了，只能度过一个相对失败的技术生涯！

[toc]

首先来张和 vue 生命周期一样的图来帮助我们记忆！

![image-20220719101004438](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220719101004438.png?x-oss-process=image/resize,w_1200,m_lfit)

*基于 react 最新版本*

**react 16 之前生命周期的钩子会多几个**

- `componentWillMount` 在 `componentDidMount`之前触发（在上图的 render 之前）—— **首次挂载**时执行的钩子
- `componentWillUpdate`在 `componentDidUpdate`之前触发 —— **每次更新**时执行的钩子
- `componentWillReceivePorps` 在`componentWillUpdate`之前触发 —— **每次更新**时执行的钩子

**现在要使用这些钩子的话需要加上一个`UNSAFE_`前缀，表明你知道这些钩子是有害的，否则会抛出警告！**

举个🌰

```react
export default class LifeCycle extends Component {
  UNSAFE_componentWillMount () {
    console.log('UNSAFE_componentWillMount')
  }
}
```

react 17 之后除了强烈不推荐使用这些钩子，也新增了两个钩子。**注意删除的纵使新增了`UNSAFE_`前缀也不能和新增的这两个混用！** 

- `getDerivedStateFromProps` 
- `getSnapshotBeforeUpdate`

**为什么生命周期在 react 16.2 版本之后发生了这些变化？**

1. diff 算法是同步的，在组件数量特别多又大的情况下，效率还是不够高。会造成页面卡顿
2. 在 react 16.2 之后使用了`FIber`（纤维）技术，来实现“分片渲染”，**先执行优先级高的任务**
3. **`componentWillMount `这类生命周期钩子属于优先级低的任务**，时间分片耗尽后会看一下有没有紧急任务如 render，一看还真有就会执行这个 render ，而`componentWillMount`这类钩子方法会被直接扔掉不再执行。容易造成预期之外的错误

## 组件生命周期

首先说明一点 react 的生命周期都是说的类组件，**函数式组件只是一个函数没有生命周期**。只有到后面借助`hooks`才能实现一些具有副作用的类似生命周期的方法。

#### 组件首次挂载

1. `constructor` 执行组件构造函数（**相当于 vue beforeCreate**）
2. `static getDerivedStateFromProps`注意是**个静态对象**，替代了 react 16之前的 `compoentWillReceiveProps`。所以可以看出他的使用场景是在接收父组件传递过来的 `props` 之前。他的功能看名字可以看出来是：**在父组件传递过来的 props 改变时改变自身的 state** 
3. `render` 调用组件的 render 函数（**相当于 vue befroeMount，开始渲染 vDom ，可以获取状态但是不能获取 dom**）
4. `componentDidMount` 虚拟 dom 渲染成真实的 dom 并挂载到节点上（**相当于 vue mounted**）

#### 组件更新

组件更新有两种情形，一种是`props`改变引起的，一种是组件自身`state`改变引起的，**这两种更新所触发的生命周期是一致的**！

1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate` **如果返回 false 的话就不往后走了**，可以手动控制返回值
3. `render`
4. `getSnapShotBeforeUpdate` **获取 props 和 state 即将更新前的最后一次快照**
5. `componentDidUpdate`组件更新完成 （相当于 vue 的 updated），在 react 16 之前的 `componentWillUpdate`的时候是拿不到 dom 的，在这里才能拿得到

**另外注意**

 >如果是 es5 的语法 React.creatClass({}) 来创建组件，他们就没有 contructor 这个方法，取而代之的是使用 getDefaultProps 和 getInitialState 来初始化默认参数和状态

## 生命周期实践

#### 初始化时的 life hooks

##### componentDidMount 

```react
import React, { Component } from 'react'

export default class LifeCycle extends Component {
  state = {
    name: 'test'
  }
  componentDidMount () {
    this.setState({ //
      name: 'Test'
    })
    console.log('componentDidMount')
  }

  componentDidUpdate () {
    console.log('componentDidUpdate')
  }
  
  // ...
}

```

在挂载中已经可以拿到真实的 dom 了，也可以修改初始化状态

修改初始化状态要注意

- 在 react 16 之前一般倾向于在`compoentWillMount`中初始化状态，相当于在 vue 的 created。现在 **react 推荐在`componentDidMount`中去做这件事**
- 如果在这里要修改状态还是要使用`setState`，**不能认为这里 dom 还没渲染，直接改数据也可以**。state 的状态在 constructor 的时候就已经初始化完成了，要更改状态还要用`setState`

这里最有用的一点还是能够拿到真实的 dom，让我们对 dom 进行一些操作！

#### 运行时的 life hooks

小括号内为执行顺序！

##### componentDidUpdate（3）

```react
import React, { Component } from 'react'

export default class LifeCycle1 extends Component {
  state = {
    text: '测试'
  }

  render () {
    console.log('render')
    return (
      <div>
        {this.state.text}
        <button onClick={() => {
            this.setState({
              text: '123'
            })
          }}>click</button>
      </div>
    )
  }

  UNSAFE_componentWillUpdate () {
    console.log('componentWillUpdate')
  }

  /**
   * @param {object} prevProps 更新前的 props
   * @param {object} prevState 更新前的 state
   */
  componentDidUpdate (prevProps, prevState) {
    console.log('componentDidUpdate')
  }
}
```

这里的执行顺序是 `componentWillUpdate` -> `render` -> `componentDidUpdate`

**因此走到 `componentDidUpdate` 这个 hook 的时候，组件已经更新完了**，获取到的状态也是更新后的了。不过`componentDidUpdate`提供两个参数，让我们能获取到更新前的状态！

还要注意运行时的状态是会多次触发的，不像 `componentDidMount `只有初始化的时候触发一次，`componentDidUpdate`每次更新都会触发，如果要写一些逻辑在里面要注意判断！

##### shouldComponentUpdate（2）

这个钩子主要用于逻辑优化，正常情况下上面的`componentDidUpdate`每次点击按钮都会执行（也会有 diff 算法的执行），但是后面的执行是没必要的。因为状态已经更新了，react 自身也不会自动去做判断（也不好判断）。这时候就可以使用`shouldComponentUpdate` 来进行一定的优化

```react
import React, { Component } from 'react'

export default class ShouldUpdate extends Component {
  state = {
    text: '测试'
  }

  render () {
    console.log('render')
    return (
      <div>
        {this.state.text}
        <button
          onClick={() => {
            this.setState({
              text: '123'
            })
          }}
          >
          click
        </button>
      </div>
    )
  }

  /**
   * @param {object} nextProps 更新后的 props
   * @param {object} nextState 更新后的 state
   * @returns
   */
  shouldComponentUpdate (nextProps, nextState) {
    // 增加逻辑判断
    if (this.state.text === nextState.text) return false
    return true
  }

  componentDidUpdate () {
    console.log('componentDidUpdate')
  }
}

```

如上我们在`shouldComponentUpdate` 增加逻辑判断，如果状态已经更新后面就不用在执行了，返回 `false`

在这个 hook 中 state 还没有更新完成，**所以 hook 提供了两个参数让我们能够获取更新后的状态与现在的状态进行对比**！

**`shouldComponentUpdate`渲染优化实例** 

```react
shouldComponentUpdate (nextProps) {
  const nextRenderBox = nextProps.selectBox === this.props.id // 比对下一次传入的所选择的盒子判断当前盒子是否要渲染
  const prevRenderBox = this.props.selectBox === this.props.id // 比对当前是否是上一次所选择的盒子判断是否要渲染
  // 只有下一个要渲染的盒子和上一个要清除渲染历史的盒子才需要重新渲染
  if (prevRenderBox || nextRenderBox) return true
  return false
}
```

Q ：这里提一点，为什么前面说不使用`setState`而直接使用`this.state.xxxx = xxx` 修改状态会有问题呢？

一个原因就是这里的 `nextProps`、`nextState`实际就是指向的`state`这个对象，如果通过上面的方式去修改那么`nextProps`、`nextState`会被直接改变，导致`shouldComponentUpdate`中的比对失效！

##### componentWillReceiveProps（1）

这也是 react 16.9 之前的一个生命周期，现已被弃用。老的项目中可能互用，放在这里记录一下！

该生命周期的最大特点是

- **最早**接收父组件传过来的 props（作为第一个参数）

```react
UNSAFE_componentWillReceiveProps (nextProps) {
  // nextProps 接下来更新的 props
  // this.props 当前 props
  console.log('1', nextProps, this.props)
}
```

所以我们**可以在这里”监听“父组件传过来的属性的变更然后通过 ajax 或其他方式获取数据！**

当然这个钩子有一个问题

- 每次 props 传过来都会执行，即使本次 props 的更改和`componentWillReceiveProps`中的渲染逻辑不相关，这一次的`componentWillReceiveProps`也会执行。如果里面没有写很好的逻辑判断，那么每次都会发起耗时的一些请求操作！

这也是为什么 react 放弃了这个钩子而推荐下面的`static componentDerivedStateFromProps` 

##### static getDerivedStateFromProps（1，首次挂载也会执行）

react 16.2 新增的钩子，首先要注意是**静态属性**，以下是其他一些要注意的点

- 不能和已经删除的老的钩子混用
- **首次挂载**和**组件更新**都会触发该钩子
- 返回值会做为组件的新的`state` **并且与已经声明的做合并`Object.assign('组件的', '钩子返回的')`** 
- **在这里`this`访问不到组件实例**，因为很明显他是一个静态属性...`this`不会指向组件实例！
- 这个钩子是**同步的**，所以我们无法在这里请求数据再返回初始化状态
- **这个钩子执行的特别频繁**，除了父组件传过来的 props 改变时会执行，组件自身的 state 改变时也会执行。所以可以取代原来`componentWillReceiveProps`和`compoentDidMount`的组合（分别处理每次更新和首次加载）。当然也需要配置`compoentDidUpdate`来判断当前状态（**因为这个钩子返回新的状态**）—— 实际上写起来更麻烦了😭但是也减少了上面`componentWillReceiveProps`的副作用

特别注意最后一点，**自己的状态改变也会触发这个钩子，所以在`componentDidUpdate`中执行逻辑的时候也需要比对老的状态进行判断！！！**

根据这个钩子的这种特性，我们可以对初始状态做一些校验和初始化操作！

```react
// 接收两个参数和 shouldComponentUpdate 想同
static getDerivedStateFromProps (nextProps, nextState) {
  console.log('getDerivedStateFromProps', nextProps, nextState)
  return {
    text: '111111'
  }
}
```

但是这里是获取不到 dom 的，要注意！！！

##### getSnapShotBeforeUpdate

react 16.2 新增的钩子，可以用来替代原来的`componentWillUpdate`， 只有组件更新时才会调用，首次挂载不会！

**执行顺序：render -> `getSnapShotBeforeUpdate` -> `componentDidUpdate`**

相比`componentWillUpdate`有以下优点

- 该钩子之后会连续触发`componentDidUpdate`，而`componentWillUpdate`和`componentWillUpdate`中间会先触发 render，可能会带来 dom 的改变

**`getSnapShotBeforeUpdate`必须配合`componentDidUpdate`**一起使用，自身接收两个参数

变更前的 props 和 变更前的 state，和`componentDidUpdate`是一样的

```react
getSnapshotBeforeUpdate (prevProps, prevState) {
  console.log('getSnapshotBeforeUpdate', prevProps, prevState)
  return '返回给 componentDidUpdate 用'
}
```

但是他必须要有一个返回值，**这个返回值会传递给`componentDidUpdate`作为第三个参数** 。所以我们可以获取到更新前的状态快照，这也是这个生命周期的意义！

ps：这里的状态除了 state 中的状态，还有组件渲染的 dom 的状态！

```react
getSnapshotBeforeUpdate (prevProps, prevState) {
  const scrollTop = this.refDiv.current.scrollTop // 所以我们可以在这里获取到未渲染前的 dom 状态
  return {
    scrollTop // 传给 didUpdate，didUpdate 已经渲染完了真实的 dom，可以再操作 dom了
  }
}
```

到这个钩子的时候虽然 render 已经执行过了，但是还未执行 vDom 到 dom 的渲染。所以拿到的真实 dom 状态仍然是渲染前的！

#### 销毁时的 life hooks

##### compontentWillUnmount

类似 vue 中的 `beforeDestroy`，销毁组件时 react 提供了 `compontentWillUnmount` 。该 hook 无参数，主要是用来释放定时器、监听器、全局变量等

```react
render () {
  return (
    <div>UnmountLife
      <button onClick={() => {
          this.setState({
            show: false
          })
        }}>click</button>
      { this.state.show && <Child />}
    </div>
  )
}

/*=========Child=======*/
componentWillUnmount () {
  console.log('componentWillUnmount', this.timer)
  clearInterval(this.timer)
}
```

如上状态`show`变为 false 之后`<Child />`组件就会被销毁（和 v-if 的效果是一样一样的），这时候就会执行销毁的生命周期！

## computed 和 watch（1）

vue 中的 computed 和 watch 我们用的非常多，react 还是一贯如哲学的：如无必要，勿增实体！所以 react 中没有。

在生命周期这一章节中我们能学到 react 和 vue 很大不同的一点就是 react 把所有的都交给开发来处理，在 vue 中如果不需要更新的组件不会触发任何操作！

**在 react 中，只要调用`setState`那么组件及其子组件就会执行完整的`update`时的生命周期（`shouldComponentUpdate`为 true）的情况下。**

再回忆一下 vue 中使用 computed 和 watch 的场景，两个都是在状态变更时需要执行一些操作！而 reat 只要状态变更就会触发上面这些运行时的生命周期，**所以我们完全可以使用 react 提供的生命周期钩子完成 vue 中 computed 和 watch 的使命！**当然实际写起来会麻烦一些。

**如果要实现首次挂载和更新时都执行的方法，有时需要在`componentDidMount`和`componentDidUpdate`中写相同的逻辑，存在这样的重复性**！

有人说那函数式组件咋办，人家没有生命周期钩子！！**在后面学习了 react 提供的新 hooks 之后，也有原生的方式来实现类似的效果** 

## 性能优化（1）

上面说了这么多生命周期钩子，其中一些与性能优化相关。这里说明几点和生命周期相关的性能优化点！

#### ShouldComponentUpdate

这里优化很好理解，我们不是所有时候都需要重新进行 diff 比对，重新执行 state 赋值。

纵使 nextState 和 prevState 完全相同，只要执行了`setState`子组件都会重新渲染，这是完全不必要的。react 也没有像 vue 一样贴心的为我们做这些优化，所以我们需要好好利用这个钩子来进行渲染优化！

*ps：react 的重渲染是一个很大的优化点！* 

#### PureComponent

上面我们用 shouldComponent 判断是否需要更新的时候是我们手动写逻辑来判断。其实 react 也提供了一种方式来**自动比对！**

**`PureComponent` 是 react 一个内置的类，继承自他的组件会在每次更新时自动进行`shallowEqual`比对！**

```react
import React, { PureComponent } from 'react'

class ShouldUpdatePure extends PureComponent {
  // 改变 state 或者接收的 props 改变
}
```

`PureComponent`会自动比对组件的 props 和 state 是否改变**来决定是否调用`render`重新渲染**

- **比对是浅比对**，简单来说就是不会对对象状态内容的改变进行比对，如果对象内容被重新赋值就会重新渲染，纵使赋相同的值
- 比对也会耗时，如果确认组件状态每次都会被改变，那么就不要继承自改对象！举个🌰：我用循坏渲染一堆 box，然后给他们传入当前选择的盒子 id，这个 id 每次一变，所有盒子的 props 都会变。而我只需要当前选择的盒子的 id 是当前盒子 id 的盒子改变，这是 `PureComponent`不会起作用，所有盒子都会重渲染！

综上`PureComponent`虽好，但是用的场景并不多，很多时候还是要结合`ShouldComponentUpdate`来使用！
