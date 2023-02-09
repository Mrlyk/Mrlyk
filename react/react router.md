# react router

react 路由，vue 有自己的官方路由 vue router，react 官方并未推出组件。目前社区的流行方案就是 react-router。一般工程中使用的是 react-router-dom。

1. [react-router](https://github.com/remix-run/react-router)：实现了路由的核心功能
2. [react-router-dom](https://github.com/remix-run/react-router/tree/main/packages/react-router-dom)：react-router-dom 基于 react-router，react-router-dom中很多组件都是直接从react-dom 中直接导出的。所以安装了 react-router-dom 后就不用再安装 react-router

react-router-dom 在 react-router 的基础上，额外提供了 BrowserRouter、HashRouter、Link、NavLink 组件，可以用于路由跳转。其中 BrowserRouter、HashRouter 用于替代 react-router 中的 Router 组件；Link、NavLink 的作用类似于a标签；

两个东西其实是一个，只不过 react-router-dom 提供了更多扩展功能。目前（20221104）最新版本是 6.4.3。文章以 5 版本为说明，变更部分后面补充。

*ps: 还有一个 react-router-native，都是一家人*

```text
目前官方从5开始已经放弃原有的react-router库，统一命名为react-router-dom
```

[toc]

## 安装

`npm install react-router-dom -S`

直接安装 react-router-dom 即可。

## 路由模式

vue-router 有两种路由模式，默认的 hash mode 和常用的 history mode。

react-router 也有两种路由模式，表现和 vue-router 的两种模式也对应。

- BrowserRouter 对应 vue 的 history 模式，基于 h5 的 `history.pushState`等新 api 实现
- HashRouter 对应 vue 的 hash 模式，基于`locatuon.hash`等 api 实现，会带有一个 # 号标志

## 路由配置

react-router-dom 引入后，实际上他也是一个 react 组件，类似于 vue 中的`<router-view>`。同时他通过插槽的方式接收`Route`组件，这个`Route`组件作用类似于 vue 中的 `<router-link>`（但实际不同，`Route`组件不会被渲染）。

#### HashRouter 

先看最基础的使用方法：

```react
import React from 'react'
import { HashRouter as Router, Route } from 'react-router-dom' // 引入 router
import Film from './views/film'
import Cinema from './views/cinema'
import Mine from './views/mine'

export default function App () {
  return (
    <div>
      <Router>
        <Route path='/films' component={Film} />
        <Route path='/cinemas' component={Cinema} />
        <Route path='/center' component={Mine} />
      </Router>
    </div>
  )
}
```

当然我们可以将 Router 组件再封装一层，使项目结构更清晰

```react
import Router from './router'

export default function App () {
  return (
    <div>
      App
      <Router></Router> // 这样就有 vue 的味道了
    </div>
  )
}
```

- 路由默认从上到下匹配，匹配到的最后一个为最终的跳转路径。即如果有两个`/films`路径对应不同的组件，则会渲染后面的那个组件。（下面有方法可以渲染第一个）
- 路由的`path`是默认是模糊匹配，即只要包含该路径就会匹配上
- **通过 component 属性传入的组件会作为 `Route` 组件的子组件，一起实例化。同时 `Route` 会给子组件传入一些属性、方法，具体使用看编程式导航**  

#### BrowserRouter

BrowserRouter 的使用和 HashRouter 没有区别，两者可以直接替换。（注意原有的跳转逻辑）

主要是实现方式的不同，以及在服务器部署的时候需要配置默认页面（和 vue 一样）。

这里就不再讲一遍了。

#### 路由重定向

在 vue 中可以声明`redirect` 属性来声明当前路由要重定向的地址。在 react 中也有 redirect，不过他不是一个属性而是一个组件。

***在 react 中万物皆组件！和 vue2 这种声明式的框架就有很大的不同。*** 

下面是个🌰：

```react
import React from 'react'
import { HashRouter, Route, Redirect, Switch } from 'react-router-dom'
import Film from '../views/film'
import Cinema from '../views/cinema'
import Mine from '../views/mine'

export default function Router () {
  return (
    <HashRouter>
      <Switch>
        <Route path="/films" component={Film} />
        <Route path="/cinemas" component={Cinema} />
        <Route path="/center" component={Mine} />
        {/* 因为 from 为 / 可以匹配到所有路由，所以在 版本5 之前直接下面这样写会报错 */}
        <Redirect from="/" to="/films" />
      </Switch>
    </HashRouter>
  )
}
```

1. 引入了新的 `Redirect` 和 `Switch` 组件
2. `Redirect` 组件**在模糊匹配时必须放在路由组件的最后面，用于在上面的路由匹配不到的时候进行重定向**。其中 `from` 是可选的，默认就是匹配到所有组件，即上面的都没匹配到那么就匹配到这一个。当然也可以指定明确的来源
3. **`Switch` 组件**。react-router 默认采用的是从前往后全匹配的形势，会以最后一个匹配到的结果作为最终跳转页面。**如果没有 `Switch`组件，那么在任何页面刷新最后都会跳转到 film 页面。当有了 `Switch` 组件之后，则在匹配到第一个符合的之后就直接 break（和 switch 条件语句的语法一样），不会在出现前面的情况。** 

##### 404 匹配

`Route`组件可以不声明具体的 `path` 或者声明`path: *`，实现类似于 vue `path: *`的作用，即未匹配到任何一个时就匹配这个。

```react
export default function Router () {
  return (
    <HashRouter>
      <Switch>
        <Route path="/films" component={Film} />
        <Route path="/cinemas" component={Cinema} />
        <Route path="/center" component={Mine} />
        {/* 因为 from 为 / 可以匹配到所有路由，所以在 版本5 之前下面这样写会报错 */}
        <Redirect from="/" to="/films" />
        <Route path="*" component={NotFound} />
      </Switch>
    </HashRouter>
  )
}
```

**如果我们像上面这样写，那么永远都不会进 Not Found，为什么呢？**

因为 react-router 是自上向下匹配的，在有`Switch`组件包裹的情况下，匹配到后就跳出。所以这里会优先匹配到`Redirect`组件。

那如何才能匹配到最后一个路由呢？很简单，让`Redirect`组件为精确匹配（只能匹配到 `/`）的时候才匹配到即可，这样就可以继续往后匹配了。

实现方式也很简单，给`Redirect`组件加一个`exact`属性即可。

```react
<Redirect from="/" to="/films" exact />
```

#### 嵌套路由（子模块渲染）

在 vue 中我们可以使用嵌套路由配合`router-view`组件实现嵌套路由的模块渲染。

react 中的方法差不多，在 react 中使用`Route`组件来实现，**同时要注意 react 中路由的匹配方式**

```react
import { Route, Redirect, Switch } from 'react-router-dom' 

/*======== 父组件中依然使用正常路由匹配到子组件 ========*/
<Route path="/films" component={Film} />

/*======== 子组件 film ========*/
return (
  <GlobalContext.Provider
    >
    <div
      style={{
        backgroundColor: '#888',
          height: '200px'
      }}
      >
      Banner
    </div>
    <div style={{ height: '100%', overflow: 'auto' }}>
      {/* 嵌套路由 */}
      <Switch>
        <Route path="/films" component={FilmList} exact /> {/* 默认还是列表组件，默认模糊匹配，要使用 exact 属性来限制 */}
        <Route path="/films/nowplaying" component={FilmNowPlaying} /> {/* 子路由匹配到嵌套组件 */}
        <Redirect from="/films" to="/films/nowplaying" /> {/* 进入到该页面时重定向到嵌套组件 */}
      </Switch>
      {/* <FilmList /> */}
      <FilmDetail showDetail={showDetail} />
    </div>
  </GlobalContext.Provider>
)
```

**嵌套子路由的父路由一定要是模糊匹配的，否则父路由都进不去，就不会匹配到子路由了！！！** 

react 路由的匹配规则要注意！

react 路由的匹配规则要注意！

react 路由到匹配规则要注意！

## 路由 api

#### 声明式导航

类似`<a>`标签的这种形式，声明一个跳转链接。

在 vue 中有内置的`<router-link>`组件，在 react 中该如何处理呢？

首先我们可以使用原生的方式，`a` 标签。vue 中的 `<router-link>` 本质上也是 `a` 标签。react 中直接用 `a` 标签是完全没有问题的。

```react
<a href="#/films">电影页</a> // 注意原生的方式 HashRouter # 号也要带上
```

但是如果想要让跳转的 dom 自动实现一些交互效果，在用原生的情况下我们可能就要去从`window.hashchange`方法下手了，比较麻烦。`react-router-dom`内置了导航组件`NavLink`来帮我们实现，不需要重复造轮子了。

```react
import { NavLink } from 'react-router-dom'

<ul>
  {this.state.tabbar.map((e) => (
    <li
      style={{
        display: 'inline',
        marginRight: '40px'
      }}
      key={e.label}
      >
      <NavLink to={e.href} activeClassName='active-name'>{e.label}</NavLink> {/* 使用 NavLink */}
    </li>
  ))}
</ul>
```

注意：

- **`<NavLink>`组件只能放在`<Router>`组件中去用，要不然 react-router-dom 也不知道去哪找**，我们可以通过插槽的形式将他传到`<Router>`中。
- **在 vue 中路由路径前不加`/`就表示是当前路径的子路径，react-router-dom 则都会当作绝对路径来处理（注意在 V6 版本之后支持相对路径了...）**

最终他其实还是一个`a`标签，只是内部帮我们捕捉了`window.hashchange`来动态的给标签加上样式我们传入的`activeClassName`，默认是`active`。

##### Link 

上面我们使用的`<NavLink>`是`<Link>` 的一个特定版本, 会在匹配上当前 URL 的时候会给已经渲染的元素添加样式参数。其他的和。`<Link>` 组件则是朴实无华的跳转。

#### 编程式导航

编程式导航则是通过 api 的形式来跳转。

vue 中有`this.$router.push`，原生的可以用`location.href`来跳。

react 中的编程式跳转则一般有如下三种方法。

1. 首先可以使用原生方案`location.href`来跳
2. **上面说了在`Route`的 componet 属性中传入的组件会作为 `Route`的子组件，同时传入一些属性。其中`history`属性用于路由跳转，也同样是`push`方法**

```react
export default function FilmList (props) {
  FilmList.propTypes = {
    history: PropTypes.object
  }
  
  const navToFilmDetail = ({ filmId }) => {
    props.history.push(`/detailId/${filmId}`)
  }
}
```

3. **使用 react-router-dom 提供的 hooks**

```react
import { useHistory } from 'react-router-dom'

const history = useHistory() // hooks 记得不要在普通 js 函数内用
const navToFilmDetail = ({ filmId }) => {
  history.push(`/detailId/${filmId}`)
}
```

   **注意：只有在 `Router` 中才能使用，否则只会返回 `undefined`！！！**

## 动态路由

在 vue 中我们可以使用`/detail/:id`的形式，传递动态路由参数。在 vue 组件中通过 props 或者直接通过`this.$route`来获取。

在 react 中如果传递了动态参数要如何获取呢？

首先当然可以原生的 `location.href` 来自己截取，当然这就失去了使用路由组件的意义。主要还是使用以下两种方法：

1. 和 vue 一样通过`:id`的（占位符）形式来传递，在组件的 `props.match.params`对象中获取

   ```react
   <Route path="/detail/:id" component={FilmDetail} />
   
   /*==========获取============*/
   const id = props.match.params.id
   ```

2. 和 vue 一样，在路由跳转的时候通过`props.history.push({pathname: '/detail', query: {id: 'xxx'}})`或者`props.histroy.push({pathname: '/detail', state: {id: 'xxxx;}})`

   ```react
   const navToFilmDetail = ({ filmId }) => {
     // history.push({ pathname: `/detail/${filmId}`, query: { filmId } })
     history.push({ pathname: `/detail/${filmId}`, state: { filmId } })
   }
   
   /*==========获取============*/
   const filmId = props.location.state.filmId // query 的话就是 query.filmId
   ```

第二种方法要注意，在 vue 中我们通过 query 来传参数，vue 会帮我们将参数拼在浏览器路由地址后面。这样我们刷新参数也不会丢失。

**但是 react 中不会这么做，数据只会在内存，刷新页面传递的参数都会丢失。那要怎么处理呢？**

两种方案

1. **朴实无华的使用动态路由的`:/id`的形式...**
2. 在跳转的时候通过`history.push(/detail?filmId=${filmId}`的形式将参数拼接，在组件通过原生的`location.search`解析的方式获取参数



## 路由拦截

来了来了，路由非常重要的部分之路由拦截。项目中我们依靠路由拦截来实现前端权限的管理，在 vue 中提供了全面的路由钩子供我们使用，在 react 中又是如何处理的呢。

这要回到 react 中路由的声明，在上面的🌰中我们是直接通过`component` 属性传入了固定的组件。其实还可以通过`render`属性传入一个函数，来动态的渲染组件（**可以替代 vue 中的 `:is` 动态组件**）

```react
const isLogin = () => {
  return localStorage.getItem('token')
}

export default function Router (props) {
  return (
    <HashRouter>
      {props.children}
      <Switch>
        <Route path="/login" component={Login} />
        {/* 权限 */}
        <Route path="/center" render={() => {
            return isLogin() ? <Mine /> : <Redirect to='/login' />
          }}/>

        <Route path="*" component={NotFound} />
      </Switch>
    </HashRouter>
  )
}
```

如上面的例子我们就实现了路由拦截。这里有几个注意点

1. 为什么不直接 `isLogin() ? <Mine /> : <Login />` 这样写呢而要用 Redirect 呢？

   因为直接这样写路由地址不会变，明明是 mine 的路由地址却渲染了  login 组件，很奇怪。

2. 为什么要用 `Redirect` 而不使用编程式的(`useHistory`)跳转呢？

   因为`useHistory` 只能写在`Router`中，而这里就是声明路由的地方，就是根。没办法使用这个 hook！

在真实的工程中我们一般采用遍历的形式来动态配置路由，同时配合上面的路由拦截来处理权限问题！

优秀案例参考：https://github.com/yezihaohao/react-admin/blob/master/src/routes/index.tsx



## 使用注意点

在使用 react-router-dom 的时候有一些坑，这里我们记录一些可能会遇到的问题。

1. 为什么组件在`Router`下面还是无法使用`useHistory`这个 hook？

   首先要确认组件是通过`compoent`直接传入的还是`render`动态获取的。如果是通过`render`动态声明，那么传入的实际是一个函数，`Route`这个父组件的 props 实际是传递给了这个函数。需要再做一遍透传！ 

   ```react
   <Route
     path="/center"
     render={(props) => {
       console.log(props)
       return isLogin()
         ? (
         <Mine {...props} /> 
       )
       : (
         <Redirect to="/login" />
       )
     }}
     />
   ```

   如果里面还有子组件，可以继续传下去。

但是这样很麻烦，如果层级比较深的话需要传很多层。所以 react-router-dom 提供了一个高阶组件来给我们传递这些属性——`withRouter`

#### withRouter

```react
import { Route, withRouter } from 'react-router-dom'

const WithMine = withRouter(Mine) // 使用 withRouter 这个高阶组件包一层

<Route
	path="/center"
	render={() => {
  	return isLogin() ? <WithMine /> : <Redirect to="/login" />
}}
/>
```

使用 `withRouter` 这个高阶组件之后，他会将路由的属性传进去。

```text
高阶组件是 react 中复用并提升组件能力的常用方式。
```



## 声明式路由配置

在上面我们通过引入组件的形式，将我们需要的路由一个个写好。那么问题来了，如何像 vue 一样，使用声明式的配置呢？

声明式的配置更方便动态的路由配置，写起来也更清晰。在 react 中这需要借助一个库`react-router-config`



