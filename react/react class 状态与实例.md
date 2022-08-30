# react class 状态与实例

本文中将介绍 react 传统的 class 组件中的状态同时编写一个简单的实例！

[toc]

## 传统组件中的状态 state

> 定义：react 中的状态是组件描述某种现实情况的数据，由组件自己设置和更改。

react 中的 state 其实和 vue 的双向数据绑定有一点点相似，都不需要我们再像原生的那种方式一样去手动更新页面状态。但是 react 不是 vue 那种数据驱动的方式。不是我们更改变量就会自动去更新页面。而是**需要手动调用 react 提供给我们的方法去更新数据、更新状态**！

#### react 中 state 的定义

**react 中 state 定义的方式是在 react 组件中显示的声明`state`实例属性**。先忘记后面的 react hooks，传统的方式就是这样。

```react
import React, { Component } from 'react'

export default class StateComponent extends Component {
  state = { // 一定要是这个名字
    text: '收藏'
  }
  render () {
    return (
      <div>
        <h1>hello react</h1>
        <button onClick={ this.handleCollect }>{this.state.text}</button>
      </div>
    )
  }
}

/****========也可以放在类的构造函数，但是注意要先调用 super （相当于调用父类的构造函数），否则 react 会警告========***/
export default class StateComponent extends Component {
  constructor () {
    super()
    this.state = {
      text: '收藏'
    }
  }
}
```

#### 修改状态

如果学多了vue 直觉肯定是直接修改`this.state.xxx`的值页面就会更新了。

但要注意这是 react，**react 是没有自动的拦截 `state` 里声明的数据做双向绑定的。react 中一切都需要我们手动来操作**，打个比方 vue 就是自动挡，react 就是手动挡。前者更轻松，后者给你更多的操作空间！！

所以修改数据就需要我们调用从`React.component`继承来的方法`setState`

*这里提一下，虽然直接赋值不会对页面上的 dom 元素产生影响，但是数据确实是被改变了。非常不推荐这种做法，因为会对未来的数据变更改产生**副作用*** 

**修改状态**：

```js
//...
handleCollect = () => {
  this.setState({
    text: '取消收藏'
  })
}
//...
```

除了接收一个对象之外，react 也接收一个函数作为第一个参数，并用函数返回的对象作为要更新的状态！

~~react 会把`setState`传入的对象和原来的对象做比较~~。如果对象有更新 react 会重新调用`render`函数渲染。

**根据对源码的查看，react 并不会在数值层面比较新旧状态。而是到 vDom 那里使用 diff 算法来比较再判断是否更新**。所以也不需要好奇如果是很深的对象 react 怎么处理的。人家根本不处理这个数据层面的事情。

#### state 细节

- 状态不需要先声明也可以直接`setState`。再次强调这是他和 vue 的区别，vue 需要在 data 声明以初始化，而 react 更新完全是通过`setState`这个方法触发的
- **在合成事件和 react 的生命周期中，`setState`是异步的**。如果要获取修改后的结果，可以在`setState`的第二个**参数回调函数中获取结果**（回调函数无参数）。这一点和 vue 的考虑是相同的，都是为了性能！
- 在原生绑定的事件(`dom.addEventListener`)和`setTimeout`定时器中，`setState`可以看作是“同步“的。**因为 react 有一个同步检测更改操作的标志`isBatchingUpdates`。当操作本身变为异步后该标志就无法检测到。**
- 如果在一次更新中多次更改相同的一个状态属性，那么只有最后一次的会生效。因为 react 会对变更使用`Object.assign`进行合并，**也因为此我们在`setState`的时候只需要关注我们要变更的状态的值，其他未变更的值不会被覆盖掉** 
- **每次修改完成后都会执行 diff 算法对 vDom 进行更新，之后更新页面上的 dom ** 

## 实例

根据 react 的哲学：如无必、勿增实体，所以没有像 vue 一样提供很多 api。但是 vue 中的`v-for`真的很常用，在 react 中要怎么实现相同的功能呢？下面就以`v-for`为例来看一下在 react 中如何实现这类渲染。

#### 在 react 中“实现” v-for、v-show、v-if

其实要实现这个很简单，主要是亮点

- `{}` 可以解析 js 表达式
- jsx 是直接解析 html 标签

最终就如下

```react
import React, { Component } from 'react'

export default class ArrayState extends Component {
  state = {
    list: ['1111', '2222', '333']
  }

  render () {
    return (
      <div>
        <ul>
          {
            this.state.list.map(item => <li key={item}>{item}</li>)
          }
        </ul>
      </div>
    )
  }
}
```

直接在`{}`中将数组映射成 jsx 认识的标签返回。

注意：

1. **jsx 语法不需要`""`**，有引号就变成字符串了
2. 所有的 js 变量都需要用`{}`包裹，否则也会被认为是字符串
3. **html 元素上也需要 key**。和 vue 中的作用相同，都用于 diff 算法比较的时候

有了这个启发之后我们也可以用同样的思路实现`v-show`、`v-if`之类的的功能了。

**v-show 模拟实现** 

```react
// ...
state = {
  show: false
}

render () {
  return (
    // ...
    { <li style={{ display: this.state.show ? 'block' : 'none' }}>展示</li> }
  )
}
```

当然真实项目中我们不能这样内联的写，最好是提取出来放在外面～

**v-if模拟实现**

```react
// ...
state = {
  show: false
}

render () {
  return (
    // 或者 this.state.show && <div>条件渲染</div>
    { this.state.show ? <div>条件渲染</div> : null}
  )
}
```

#### 浅析 diff key

在上面我们用到了 key。这里就先简单的说明下 react diff 算法中的 key。

基于之前对 vue diff 算法的学习，两者非常相似。key 都用于在 vDom **比较**的时候

- **提高效率**，key 值相同可以决定用来复用还是删除

这也决定了为什么 react 和 vue 都不推荐用 index 索引做 key，因为如果后续对数组中间的数据进行删除或修改，会影响后续元素的 key，那么后面元素的 key 也就失去了意义。

具体的可以看《react diff 算法》

#### react 中的 v-html

有时候我们有将字符串当成 html 标签解析的需求，比如动态给文本上色。在 vue 中我们可以使用`v-html`，那在 react 中如何处理呢？

关键字：**`dangerouslySetInnerHTML`** 

看名字就知道，**和 vue 一样这样写都会有 xss 脚本注入攻击的风险**，所以要谨慎使用。具体使用方式看下面的例子

```react
import React, { Component } from 'react'

export default class TodoList extends Component {
  // ...
  render () {
    return (
      <div>
        <ul>
          {this.state.list.map((item) => (
            <li key={item.id}>
              {/* v-html */}
              <span dangerouslySetInnerHTML={{
                __html: item.text
              }}></span>
            </li>
          ))}
        </ul>
      </div>
    )
  }
}

```

**需要两点** 

1. 在 html 标签上添加`dangerouslySetInnerHTML` 属性，接收一个对象
2. `dangerouslySetInnerHTML` 的对象中使用`__html`属性来接收要解析的 html 字符串

#### tab 选项卡组件实例

下面是一个 tab 选项卡实例，融合了上面所有的知识点

```react
import React, { Component } from 'react'
import './css/02-card.css'
import Film from './cardComponents/film'
import Cinema from './cardComponents/cinema'
import Mine from './cardComponents/mine'

export default class TabCard extends Component {
  state = {
    cardContent: '电影',
    currentCard: '1',
    tabs: [
      {
        id: '1',
        text: '电影'
      },
      {
        id: '2',
        text: '影院'
      },
      {
        id: '3',
        text: '我的'
      }
    ]
  }

  render () {
    const { tabs } = this.state
    const tabsRender = tabs.map((e) => (
      <li style={this.getActiveTabStyle(e)} key={e.id} onClick={() => this.handleTabClick(e)}>
        {e.text}
      </li>
    ))
    return (
      <div className="tab-card">
        <div className='tab-card__content'>{this.getCurrentContent()}</div>
        <ul className="tab-card__label">{tabsRender}</ul>
      </div>
    )
  }

  handleTabClick ({ id }) {
    const { currentCard, tabs } = this.state
    if (currentCard !== id) {
      const cardContent = tabs.find(item => item.id === id)?.text || ''
      this.setState({
        currentCard: id,
        cardContent
      })
    }
  }

  getActiveTabStyle = ({ id = '1' }) => {
    if (id === this.state.currentCard) return { color: 'orange' }
  }

  getCurrentContent () {
    const contents = {
      1: <Film />,
      2: <Cinema />,
      3: <Mine />
    }
    return contents[this.state.currentCard] || null
  }
}
```

