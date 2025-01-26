# react 最佳实践

入门一门框架简单，但要做到最佳实践则需要大量经验积累。下面是一些技术博主推荐的最佳实践操作，也需要带入自己的一些思考，推荐的不一定是最好的，同时想想为什么这样做更好。

[toc]

## 组件相关

现在都推荐使用函数式组件而不是传统的类组件。

#### 组织好工具类函数

对于只处理数据逻辑的工具函数，不要放在函数组件中，放在外面声明它。这会让组件更加清晰。

下面是一个例子，我们将`valid`函数放到外面声明，因为它不依赖任何状态，只是一个纯校验工具函数，没有必要放在组件函数中声明。

```react
export default function Component() {
  const [value, setValue] = useState('')

  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={validateInput}
      />
      <button onClick={valid(value)}>valid</button>
    </>
  )
}

function valid(value) {} 
```

#### 不要对重复项硬编码

对重复项硬编码会产生很多重复代码，并且使整个组件变的冗余，我们要避免下面这种情况

```react
function Filters({ onFilterClick }) {
  return (
    <>
      <p>Book Genres</p>
      <ul>
        <li>
          <div onClick={() => onFilterClick('fiction')}>Fiction</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('classics')}> Classics</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('fantasy')}>Fantasy</div>
        </li>
      </ul>
    </>
  )
}
```

对这种情况我们可以使用一个配置好的对象，并使用数组映射的方式来处理。

#### 管理好组件体积

- 一个 react  组件只应该获取 props，返回 markup
- 如果组件做了太多事情那么这个组件中的部分逻辑应该被提取出来拆分成更小的组件
- 对循环和条件渲染， 不要犹豫，提取出来
- 提取的标准不是代码行数，而是逻辑的独立性

#### 使用一个捕获异常的高阶组件

参考官网：https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

这可以避免因为一个组件异常而导致整个页面白屏的问题。有时候异常的组件不可见也不会对整体产生多大影响，但是如果因为这一个异常导致整个页面不可见，那么体验就很差了。

#### 管理好 props 数量

- props 数量越多，意味着这个组件要做的事情越多，也意味着我们应该拆分这个组件了
- 如果超过 5 个 props 我们就要考虑这种情况了
- 可以使用传递一个对象的方式，而不是一个一个的传递对象中属性，这也有助于减少 props 数量

⚠️：**props 数量越多，重渲染的可能性越大** 

#### 处理条件渲染

使用三元运算符而不是短路运算符。

有时候我们会这样写

```react
function Component() {
  const [count, setCount] = useState(0)
  return (
  	{count && <h1>Count Is {count}</h1>}
  )
}
```

但是上面这样写会出现一个`0`展示在页面上，这肯定不是我们想要的。

使用三元运算符就可以避免这种情况的发生。`count ? <h1>Hello</h1> : null`。

三元运算符虽好，但也不要滥用，像下面这种情况要避免，这会让代码的意图不明显，很难理解。

```react
// 👎 
isSubscribed ? (
  <ArticleRecommendations />
) : isRegistered ? (
  <SubscribeCallToAction />
) : (
  <RegisterCallToAction />
)
```

取而代之的是下面这样，让代码意图更清晰。

```react
// 👍 
function CallToActionWidget({ subscribed, registered }) {
  if (subscribed) {
    return <ArticleRecommendations />
  }

  if (registered) {
    return <SubscribeCallToAction />
  }

  return <RegisterCallToAction />
}
```

#### 将列表分离到一个单独的组件中

渲染列表时，我们一般使用`map`方法来完成。但是如果组件本身就比较大了，再渲染这样一个列表带来的缩进、语法问题会不利于阅读代码。

更好的方式是列表渲染单独提取成一个组件。只有一种情况例外：这个组件的主要职责就是渲染这样一个列表。

下面是一个例子：

```react
// 👎 
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      {articles.map((article) => (
        <div>
          <h3>{article.title}</h3>
          <p>{article.teaser}</p>
          <img src={article.image} />
        </div>
      ))}
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}

// 👍 
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      <ArticlesList articles={articles} />
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}
```

#### 解构 props 时分配默认值

在类组件中一般使用`defaultProps`来声明`props`的默认值，在函数式组件中更推荐在解构`props` 时赋默认值。

这会让从上到下阅读代码变得更容易，而无需跳跃，并将定义和值保持在一起。

####  避免嵌套渲染函数

我们要知道一个组件只是一个函数，在组件中声明另一个渲染函数会使这个渲染函数可以访问所有父组件的属性。这会使代码更加难以理解，我们需要将它封装到一个单独的组件中。

**同时这会带来一个潜在的 bug，就是每次父组件更新时，子组件一定会被重新创建，而不是复用。给用户的感觉就是组件闪了一下。因为子组件是在父组件内创建的，父组件 re-render 子组件相当于被重新声明了一遍。** 

```react
// 👎 
function Component() {
  function renderHeader() {
    return <header>...</header>
  }
  return <div>{renderHeader()}</div>
}

// 👍
import Header from '@/components/Header'

function Component() {
  return (
    <div>
      <Header />
    </div>
  )
}
```

## 状态管理

#### useReducers

在简单的状态管理中更推荐使用官方提供的 `useReducers`，它已经能帮助我们做大部分的状态管理工作。当然它也有一些缺点，比如子组件不必要的重渲染。

#### 使用数据请求库

传统的我们使用 axios 或者 xhr 的形式来每次请求数据并重新渲染。

现代的数据库像 [react query](https://tanstack.com/query/v3/) 提供了更强大的功能，如数据缓存，必要时才重新请求，可以极大的减少重渲染和帮助我们管理数据。

## 项目结构

- 遵循基于路由管理模块的形式，而不是按照功能区分

## 十个要避免的设计

1. Big Components

大组件，耦合性高，逻辑多，不利于复用...等等缺点太多了就不赘述了。

2. Nesting Gotcha

过多的嵌套组件，显然大组件是不应该的，但是过度的细化会导致多重嵌套，也不提倡。

3. 昂贵的计算

如果一个方法涉及到重复的昂贵计算，使用`useMemo`缓存它。

4. 不必要的DOM

React 组件和 Vue2 一样根元素只能有一个，有时候我们为了暴露两个同级元素而在最外层套一个 div。

React 提供了 `React.Fragment` 来应对这种情况，没有必要创建一个额外的 div。

5. 混乱的文件结构

对子组件创建一个文件夹来承载。

6. 过大的打包文件

如果想做到懒加载可以使用`React.lazy`来实现。

7. 过深的属性传递

不要用 prop 一层一层传，使用 redux 或者 `useContext` 来管理。

8. 过多的单个属性传递

要传递很多 prop 时不要一个个的传，直接使用一个对象`{...prop}`的形式。

9. 使用 currying 优化多参数传递

10. 使用自定义 hook 处理数据团

## 风格推荐

- **减少使用类组件而是使用更新的函数式组件**

1. 类组件有一些历史包袱，函数上下文（this）的处理也比较混乱
2. 函数式组件更简洁

- **避免使用内联事件监听器** 

1. 代码混乱，渲染逻辑和业务逻辑混在一起，就像不推荐直接写 style 一样
2. 不利于逻辑复用
3. 不利于维护和扩展

- **保持一致的代码风格，函数式就都函数式，类组件就都类组件，不要混着来** 