# react 组件属性与通信

组件的属性和状态是不同的，前面我们看到状态是属于组件内部的。属性（`props`）则可以用来传递、通信。那么在 react 组件中如何传递属性，如何通信呢？

在 vue 中我们通过 props 来传递数据，在 react 中也是使用 props 属性（真巧啊😂），下面就来具体看看

[toc]

## props 属性

属性在组件中如何访问呢，很简单和 vue 还很想，直接使用 `this`访问。

```react
import React, { Component } from 'react'

export default class Navbar extends Component {
  render () {
    const { title } = this.props // 直接访问
    return (
      <div>{title}</div>
    )
  }
}

/**==========父组件==============**/
// ...
<Navbar title='首页'/>
// ...
```

可以看到和 vue 中不能说是一模一样吧，也可以说是毫无二致了。。。

#### 非字符串属性

在 vue 中我们要传对象或者布尔值，需要加一个`v-bind`指令，那么在 react 中怎么做呢？

很简单`{}`，在 react 的 jsx 中`{}`包含的就是 js 表达式，自然也可以是 js 变量。所以只需要用`{}`包裹即可

```react
<Navbar show={true}/> // {} 包裹
```

**在这里也能看到，父组件可以直接将要传递的属性写在子组件的标签上即可传递**

说明三点

1. **可以传递多个属性**，直接在标签上声明多个属性即可

2. 可以使用对象的形式将多个属性写在一个对象内一起传递，**记得使用扩展运算符，而且必须这么写**

   ```react
   <Navbar {...{title: '测试', leftShow: true }}/>
   ```
3. **如果传递相同的属性，后面的会覆盖前面的**

#### 属性验证（推荐）

在子组件中如果我们接收到父组件传过来的值直接就开始用会抛出警告，说缺少`props validation`，也就是需要对传递过来的属性做验证。这是 vue2 中所没有的（vue3 中也新增了可验证）

在**子组件中**对传过来`props`做验证有三个点要知道

- 验证的对象需要在子组件的类对象上，而不是类实例上，所以需要使用**static**静态属性来标识

- **验证对象属性固定使用`propTypes`来声明**。key 是接收的属性名，value 接收一个函数用来校验属性的类型

  ```react
  // ...
  static propTypes = {
    title: propTypes.string.isRequired,
    leftShow: propTypes.bool
  }
  // ...
  ```

- react 自身提供了`prop-types`来对常规的类型作验证

`prop-type`包哈你的校验规则可以查看官方文档：https://zh-hans.reactjs.org/docs/typechecking-with-proptypes.html

这里除了常规的基础类型外说明几个重要的：

- **props 必填** - `PropTypes.xxxxx.isRequired`，在任何类型后面加上一个 `isRequired`属性即可

- 联合类型

  ```js
  PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ])
  ```

- 指定数组元素组成 - `PropTypes.arrayOf(PropTypes.number)`

- **指定对象元素组成** 

  ```js
  PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  })
  ```

*可以看出很有 typescript 的味道，这也是为什么 react 对 ts 对支持比 vue 更好。*

##### 自定义的 array object 验证器

```js
// 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
// 它应该在验证失败时返回一个 Error 对象。
// 验证器将验证数组或对象中的每个值。验证器的前两个参数
// 第一个是数组或对象本身
// 第二个是他们当前的键。
customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
  if (!/matchme/.test(propValue[key])) {
    return new Error(
      'Invalid prop `' + propFullName + '` supplied to' +
      ' `' + componentName + '`. Validation failed.'
    );
  }
})
```

内置的 `propTypes ` 可以覆盖所有的数据类型，所以我们不需要也不能（待确认！）传入一个完全的自定义校验函数，复杂对象也可以使用上面的自定义验证器。

#### 默认的属性值

在  vue2 的组件中需要声明式的接收父组件传递过来的 props，还能使用`default`给 props 赋默认值。

在 react 中也能做到这一点，怎么做呢？和属性验证类似，在类对象上声明一个`defaultProps`属性并声明即可。

```react
static defaultProps = {
  title: '标题'
}
```

**声明了默认值之后`isRequired`将会永远为 true**

## 函数式组件的 props

在函数式组件中没有 this，props 会直接被作为第一个参数默认传入。**函数式组件一开始就是支持 props 传递的，但是不支持状态。从 react 16.8 开始引入了 hooks 之后才能使用状态** 

```react
import React from 'react'

export default function SideBar (props) {
  const { bg } = props // 通过参数获取到传入的属性
  return (
    <div style={{ backgroundColor: bg, width: '200px' }}>
    </div>
  )
}

/**===========父组件==========**/
<SideBar bg='yellow'/>
```

那在函数式组件中和如何验证属性呢？

#### 函数式组件属性验证

首先明白 js 中类的本质只是一个语法糖，static 相当于是在类对象上直接添加一个属性。所以在函数式组件中我们直接在这个函数上添加这个属性即可，别忘了在 js 中函数是第一等公民，同时也是一个对象。

```react
import React from 'react'
import propTypes from 'prop-types'

export default function SideBar (props) {
  const { bg } = props
  
  // 直接加上验证即可
  SideBar.propTypes = {
    bg: propTypes.string
  }
  
  return (
    <div style={{ backgroundColor: bg, width: '200px' }}>
    </div>
  )
}
```

其实类也可以这么写，只不过 js 既然提供了`static`的语法糖，那就用更清晰的做法好了。

## 状态 state vs 属性 props

**相同点**

- 值改变都会触发 render 重新渲染

**不同点**

- 属性可以传递，状态只属于组件自己
- 属性可以由父组件修改，状态只属于自己（注意还是单向流的）

#### 在子组件中更改 props 会报错

如果尝试在子组件中给父组件穿过来的 props 直接赋值，react 会抛 TpeError，并告知**属性是只读的**。

这一点其实在 vue 中直接修改 props 传过来的值也会被警告，在 react 中更严重，直接就给异常了。

这里又有一个问题，**在 vue 中如果 props 传入的是一个对象，那么更改这个对象中属性的值 vue 是不会抛出任何警告的。当然这也不符合 vue props 的理年。那么在 react 中呢？**

**在 react 中是完完全全的单向数据流，修改对象属性虽然也不会报错，但是也不会有任何效果。**在 vue 中修改由于数据的响应式在初始化时就被包装了，所以修改了后大部分情况下页面还会重新渲染。但是在 react 中这样修改会没有任何反应。除非用另一个事件触发了子组件的 `render`方法页面才会重新渲染，表现就像 vue 中的数据失去了响应式一样。

**那子组件如何更改 props 呢？**

在 react 中的方案就是子组件通知父组件，由父组件来更改！！！具体看后面的实例。

**优化点**

和函数尽量少要参数的思想一样，参数越少函数越独立。**我们在编写 react 组件的时候应该尽量减少组件的状态**。状态越少，说明组件越独立。尽量多写无状态的组件，这样可以增加组件的可复用性！也就是我们常说的——**受控组件**（无状态组件），他的表现由父组件传过来的属性控制。

## 受控组件与非受控组件

上面提到了一个概念——受控组件，那么什么是受控组件呢？这里要区分为两类

- 表单中的受控与非受控组件
- react 中的受控与非受控组件

上面我们提到的受控组件实际上是指广义的 react 的受控组件

#### 表单中的受控与非受控（表单默认值）

**表单中的非受控组件**是指通过`ref`或原生操作来获取 dom 再获取表单数据的组件，或者专业一点是指状态不受 react state 控制的组件。如一个 input 元素没有 value 属性限制，用户的所有输入都会直接展示出来。要获取值只能通过 `ref`或原生的方式。比如下面这个

```react
<input type="text" ref={this.username} value="33"/>
```

如果我们要获取他的值需要使用 `this.usename.current.value` 来获取，和原生的差别不大。

非受控组件把**真实数据存储在 DOM** 节点中，使用 value 会把 dom 的值限制为 33，无论用户如何输入都是 33。react 也提供了 `defaultValue`来**设置默认值**而不限制用户输入！

```react
<input type="text" ref={this.username} defaultValue="33"/>
```

但是纵使可以设置默认值，如果我们要把这个表单上的值传递给子组件。我们仍然不能直接像下面这样传递。

```react
<Child value={this.usename.current.value}></Child>
```

因为非受控组件的真实数据存储在 dom 上，和原生一样**。改变 dom 元素的值不会重新触发`render`渲染**，传递给子组件的值也就不会同步变更。

那如果要让他变成一个受控组件呢

**表单中的受控组件**

react 中的受控组件则是指由 react state 来管理的组件，赋值取值都要依赖于状态。

```react
import React, { Component } from 'react'

export default class ControlForm extends Component {
  state = {
    username: ''
  }

  render () {
    return (
      <div>
        <h1>登陆页</h1>
        {/* 将 value 绑定到状态 */}
        <input
          type="text"
          value={this.state.username}
          onChange={(e) => {
            this.setState({
              username: e.target.value
            })
          }}
        />
        <button
          onClick={() => {
            console.log(this.state.username)
          }}
        >登陆
        </button>
      </div>
    )
  }
}

```

将表单的值绑定到状态之后就实现了 react 中的受控组件。

##### 受控组件5点注意

1. `value`和`defaultValue`不能同时声明，想想就知道这么做没意义
2. 上面的例子中是一个 `input` 输入框，可能会有点好奇**为什么不是监听`input`事件而是`change`事件**来更改状态？**这是因为 react 为了统一各类表单比如单选框的改变而做了一层封装，统一暴露成了`change`事件**。所以我们监听`change`事件做更改就可以了。当然如果坚持对`input`框监听`onInput`事件也是完全可以的，区别就是`onChange`只会在值改变的时候触发，`onInput`只要输入就会触发。而且`onChange`行为更加统一！
3. `change`事件的回调函数中 `e` 也是被包装过的，但是和原生对象基本一致。可以通过`e.target.value`来获取用户输入的值
4. 不同类型的表单虽然改变的事件被包装的相同的了，但是**绑定的值还是和原生的一样**：文本输入框对应 value，单选复选框对应 checked ...
5. **如果 value 绑定的状态是 undefind ，react 仍然会将其视为非受控组件，和没绑定 value 是一样的** 

**对于渲染一个列表的情况** 

react 这里比 vue 用起来没那么方便的一点就是 vue 帮你做好了绑定关系，渲染列表时，在 vue 中直接 `v-model`绑定列表某项的值就可以了。但是**在 react 中需要手动在`change`事件中找到变更的那一项然后调用`setState`手动更改**。感觉就是 vue 帮你照顾好好的但是有时候突然不响应了会有点懵，react 用起来没那么方便但是让你一切尽在掌握！

#### 受控组件的实例

有了受控组件之后我们可以更加轻松的处理一些场景。我们知道**每次状态更新都会引发 render 函数的重新渲染**，而受控组件每次值改变都会触发状态更新。那这有什么用呢？

假设有这样一个例子：有一个列表，支持用户搜索过滤。如果使用常规思路我们需要两个数组

- 一个存储列表所有数据
- 一个列表存储过滤后的数据用于展示

这样带来的问题是额外的内存占用，不够优雅。那在受控组件中如何处理呢？

我们知道列表是通过 `render` 渲染的，那我们每**次改变表单的值的时候重新去过滤并渲染**一下不就可以了吗！其中渲染 react 已经自动帮我们做了，我们只需要在原来的地方加上过滤的逻辑即可。

```react
// ...
render () {
  // 更改状态触发重新渲染过滤，这样也不会改变原数组的值，而是更改渲染结果
  const cinemasList = this.state.cinemas.filter(e => {
    return e.name.includes(this.state.keyword)
  }).map((item) => {
    return <li key={item.cinemaId}>{item.name}</li>
  })
  return (
    <div>
      <input
        value={this.state.keyword}
        {/* 值改变时更改状态 */}
        onChange={(e) => {
          this.setState({
            keyword: e.target.value
          })
        }}
        />
      <ul>
        {cinemasList}
      </ul>
    </div>
  )
}
// ...
```

#### 受控组件的优点

经过上面的实例我们可以总结一下受控组件的优点

- 状态完全交由用户控制
- 状态更新带来 dom 的更新，有点贴近 vue 那种双向绑定的感觉了
- 状态更新带来属性传递的自动更新，props 更方便

所以推荐在 react 中使用受控组件

#### react 广义的受控组件

上面说的组件都是比较狭义的表单组件，下面来说一说 react 自己定义的组件。

> 简单的定义：**定义的组件完全交给 react 的 props 和 state 控制渲染，这样就是受控组件，否则就是非受控组件！**

比如下面这个 `com` 组件，他的状态就是完全交由 react state 控制的，就是一个受控组件。我们可以在`setState`之前做一系列逻辑处理，最终决定展示内容！

```react
state = {
  value: ''
}

<com value={value} onChange={e => this.setState({value: e.target.value})} />
```

## 组件通信

上文的 props 说明中已经看到了很多父传子通信的例子，react 也是单向数据流的。那么就没有子传父了吗？子传父也是一个很常见的场景。在 react 中子传父如何做呢？

#### 子传父通信

在 vue 中我们可以通过`$emit`事件通知父组件来更新数据，甚至对于对象类型的数据在子中直接更改都可以（不推荐）。

那么在 react 如何子传父让父更新数据呢？

其实思路也是一样的，**在 react 中我们可以通过 props 传递数据，这个数据类型是没有限制的。那我们直接传递一个函数给子组件**，让子组件需要的时候执行，同时函数上下文（`this`）指向父组件不就可以了吗？

```react
/**===========父组件==========**/
class Parent extends Component {
  state = {
    isShow: true
  }

  render () {
    return (
      <div>
        {/* 传递一个 event 函数给子组件，名字随意 */}
        {/* 这里是箭头函数，所以 this 在函数创建时就已经确认了会一直指向父组件，这也也能调用父组件的 setState 来更新状态 */}
        <Navbar event={() => {
          console.log(this)
          this.setState({
            isShow: !this.state.isShow
          })
        }} />
        {this.state.isShow && <SideBar/>}
      </div>
    )
  }
}

/**================子组件===============**/
class Navbar extends Component {
  static propTypes = {
    event: propTypes.func
  }

  render () {
    return (
      <div style={{ backgroundColor: 'red' }}>
        <button onClick={this.handleClick}>click</button>
        <span>navbar</span>
      </div>
    )
  }

  handleClick = () => {
    const { event } = this.props
    event() // 调用父传过来的事件
  }
}
```

**总结一下**

在 vue 中把传递数据和监听事件分开了，传递数据用`props`，发送事件使用了`$emit`。当然在 vue 中也可以使用`props`传递函数，但是在 vue 中会有`this`的指向问题，除非传递的时候不用箭头函数并且使用`bind`这种显示绑定`this`的方法让`this`指向父才行。而且这样还会让项目难以维护，因为不熟悉项目时不清楚哪些子组件会在什么时候更改数据造成混乱，所以在 vue 中提供了**`$emit`事件机制**，不仅方便了子传父通信，还能方便的实现兄弟组件之间的通信、爷孙组件之间的通信，**解耦了父子组件**。

在 react 中`this`的处理虽然不用这么麻烦，但是没有`$emit`事件机制导致也没有那么灵活，数据只能一级一级的传递。优点也是有的

- 数据处理更加清晰，避免混乱在大型项目中是很重要的
- 可以获取到传入的函数的返回值，某些情况下很有用

#### 兄弟组件通信

在 vue 中兄弟组件通信一般依赖于父组件，兄 1 传父，父传兄 2 这样。或者使用 event bus。

那么在 react 中兄弟组件如何通信呢？

##### 第一种方式 —— ref

还是和 vue 中的第一种一样（vue 确实“借鉴”了 react 不少东西）通过`ref`来调用组件的方法

```react
export default class TabCard extends Component {
  tabbarRef = React.createRef()

  render () {
    return (
      <div className="tab-card">
        <Navbar event={() => {
            // this.tabbarRef.current 就可以拿到子组件实例，之后就可以调用子组件上的方法来进行通信
            this.tabbarRef.current.handleTabClick({ id: '3' })
          }}/>
        <Tabbar ref={this.tabbarRef} setCurrentCard={this.setCurrentCard}/>
      </div>
    )
  }
}
```

这种方式的好处是简单直接，对表单中的数据更加方便。

##### 第二种方式 —— 受控组件

上面已经说明了什么是 react 中的受控组件，即自身表现交由父组件传递的 props 控制，**自身没有状态**。

所以我们可以**将子组件的状态剥离出来，放到父组件上，实现受控组件**

```react
class TabCard extends Component {
  tabbarRef = React.createRef()
  state = {
    currentCard: '1'
  }

  render () {
    return (
      <div className="tab-card">
        {/* 只需要在父组件上操作状态，就可以控制子组件 */}
        <Navbar event={() => {
            this.setState({
              currentCard: '3'
            })
          }}
        />
        <Tabbar event={(id) => {
            // 将 currentCard 这个状态剥离出来，不要放到子组件上，保证组件的受控性
            this.setState({
              currentCard: id
            })
          }}
          currentCard={this.state.currentCard}
        />
      </div>
    )
  }
}
```

这样做也存在两个问题：

1. **组件不能完全独立**，一定要求外部的 props。这也是为什么  react 自身就有一个`propTypes`验证。就是为了在编写这种受控组件的时候提前对 props 进行验证，保证代码在编译阶段是可用的，减少风险
2. **子组件需要的 props 太多会造成父组件状态的臃肿**，这一点需要我们在编写组件的时候把握好粒度，粒度越低这种情况会少。但是过低的粒度会消耗大量的时间也不是所有时候都是必要的

react 总的来说是推荐这种受控组件的写法的，**受控组件将状态交给父组件这个中间人来处理，可以方便的将数据分发给下面的兄弟组件。**

这种方式在 react 中又称为**状态提升**，将状态提升到父组件来处理！

当然具体使用这两种的哪一种还得**结合业务场景来**！

*受控组件也能非常轻松的改为完函数式组件，在 recat 16.8 之前函数式组件就是没有状态的，也推荐使用函数式组件来编写受控组件，会更加简洁。*

##### **第三种方式——采用函数式编程的设计思想来手动完成**

在 vue 中可以通过 eventBus 事件系统来处理。react 没有帮我们封装好这个系统，但是我们**可以手动实现一个发布订阅模式来实现状态的更新**，这也是 react 的优势，可以很方便的使用各类设计模式！

```react
 /**===========父组件===========**/
import React, { Component } from 'react'

// 定义一个简单的发布订阅对象
export const eventListener = {
  listener: [],
  subscribe (callback) {
    this.listener.push(callback)
  },
  publish (film) {
    this.listener.forEach(callback => callback && callback(film))
  }
}

export default class Film extends Component {
  // ...
}


/**===========FilmDetail===========*/
import React, { Component } from 'react'
import { eventListener } from './film'

export default class FilmDetail extends Component {
  formDetailRef = React.createRef()

  state = {
    selectFilm: null,
    showDetail: true
  }

  constructor () {
    super()
    // 在构造函数中订阅事件，当然这里需要自己有一个状态来承载
    eventListener.subscribe((film) => {
      this.setState({
        selectFilm: film,
        showDetail: true
      })
    })
  }
  // ...下略
}

/**===========FilmList==========*/
import { eventListener } from './film'

export default class FilmList extends Component {
  render () {
    const filmList = this.state.films.map(film => (
      <li key={film.filmId} onClick={() => {
        this.publishSelectFilm(film)
      }}>{film.name}</li>
    ))
    // ...
  }

  publishSelectFilm = (film) => {
    eventListener.publish(film) // 点击的时候发布事件
  }
}
```

可以看到 react 和原生的设计模式真的非常契合，拿进来就可以直接用。不过要注意

1. 这种数据是存在全局的，也就是内存里的
2. 最终数据的展示还是需要通过自身的状态来承载

通过这种方式不仅能实现兄弟组件之间的通信，**爷孙之间的、嵌套兄弟之间的都可以**。只要有订阅，有发布就可以了

这是完全由我们自己实现的一种方式，react 官方也提供了一种生产者、消费者的模式来实现这种复杂一些的通信的——context 通信

##### 第四种方式 —— context 通信

`React.createContext` 可以创建一个对象，包含一个`Provider` react 内置组件，被他**包裹**的组件可以**消费**他 provide 的**数据**!

```react
/**============生产者 GlobalContext.provider 提供数据================**/
export const GlobalContext = React.createContext()

export default class Film extends Component {
  render () {
    return (
      <GlobalContext.Provider value={{
          test: '数据'
        }}>
        {/* ... */}
      </GlobalContext.Provider>
    )
  }
}

/**============消费者 GlobalContext.Consumer 获取数据================**/
import { GlobalContext } from './xxxx'

render () {
  const { selectFilm } = this.state
  this.formDetailRef.current && (this.formDetailRef.current.scrollTop = 0)

  return (
    <GlobalContext.Consumer>
      {(value) => {
        // 在 Consumenr 中使用一个回调函数来接收传递过来的数据
        console.log('value', value)
        // 最终这个函数也需要 return 回要渲染的 jsx
        return (<div>{value}</div>)
      }}
    </GlobalContext.Consumer>
  )
}
```

这里有四点要注意：

- `Provider` 传递参数的属性只能是 `value` 
- **只有被`<Context.Provider>`包裹的子组件才能使用`<Context.Consumer>`进行消费** 
- **`Consumer`内的回调函数最终也要 return 要渲染的内容，否则什么也不会渲染** 
- 如果`Provider`组件已销毁，那么所有内容都会被销毁，纵使 `GlobalContext` 还存在！

虽然使用了Context 但是还是在父组件上要新增一个状态，所以如果只有单个子组件的时候直接使用受控组件就好了。**这个内置组件主要的应用场景是父传多个子组件的情况。**

还有一点是上面的例子中我们虽然也可以通过子传父的形式由子组件通知父组件来改变 state，但**既然使用了Context 就可以让这些数据、事件全部由 Context 来提供**：一个是可以保证所有子组件上都能获得该事件，一个是为了行为的统一！

其实看上面的内置组件可以发现我们在内置组件内部写了 jsx，很容易就能联想到 vue 中的 slot，接下来就来看看 react 中的 “slot”！

## 插槽

在 vue 中有多类插槽而且很有用，特别是作用域插槽还可以用来子传父传递数据。

react 中也存在插槽的概念，也可以用来传递数据和渲染用。还是遵循 react 的哲学：如无必要，勿增实体。所以 react 没有像 vue 一样单独给了一个 slot 组件，而是通过 `props`传递，固定存放在`children`对象上

```react
import React, { Component } from 'react'
import PropTypes from 'prop-types'

class Child extends Component {
  static propTypes = {
    children: PropTypes.object 
  }

  render () {
    return (
      <div>
        <h2>child</h2>
        {/* 插槽 */}
        {this.props.children}
      </div>
    )
  }
}

export default class SlotComponent extends Component {
  render () {
    return (
      <div>
        <h1>SlotComponent</h1>
        <Child>
          <p>1111</p>
        </Child>
      </div>
    )
  }
}
```

这里要注意一下`this.props.children`的类型

- 渲染的是纯文本无 html 标签，那么就是`string`
- 渲染的是带 html 标签的**单个**元素，那么就是 `object`，而且是一个 react 的 vDom（`React.element`）
- 渲染**多个**元素时则是一个数组，**且可以使用索引访问来改变渲染的顺序，相当于 vue 中的具名插槽** 
  - `this.props.children[0]` 这样

**和 vue 一样 slot 的数据是属于父作用域的** ，在一些简单的展示渲染中还是比较有用的！

上面已经有了匿名插槽和具名插槽，那**作用域插槽**呢？这里还是要想起来两点：

1. react 中的 props 传递可以是任意的 js 变量，当然也包括对象
2. js 中函数是一等公民，也是对象

所以我们可以**在父组件中写一个函数渲染，然后在子组件中将子组件的状态传递出来**

```react
/**=========父组件===========**/
<Child>
  {
    v => (<div>{v}</div>)
  }
</Child>



/**============子组件============**/
state = {
  name: 'child state'
}

render () {
  return (
    <div>
      <h1>Children</h1>
      {
        // 直接传递状态 state.name
        this.props.children[1](this.state.name) 
      }
    </div>
  )
}
```

## 总结

至此 react 的 props 和通信就学习的差不多了，其实和 vue 有很多想通之处

- props 对应 vue 的 props，传递数据时要注意 react 是完全的单向流。vue 中传递对象时还能更改（虽然不推荐）
- ref 对应 vue 的 $refs
- this.props.children 对应 vue 的 slot

最大的区别还是数据状态的管理，vue 在初始化的时候自动做了双向绑定。帮我们省了很多事，react 中每一次的状态更新，vDom 重新渲染都需要通过 `setState`来完成。

同时在 react 中还能更好的使用函数式编程的思想，我们可以直接写一个发布订阅模式来帮助我们通信，写一个代理模式来帮我们处理懒加载。因为组件就是 js ， 使用起来更加得心应手！
