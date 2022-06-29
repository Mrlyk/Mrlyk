# pinia

vue 的新一代全局状态管理工具，支持 vue2 和 vue3 （在  vue2 中使用需要单独安装`@vue/composition-api`）

[toc]

## pinia vs vuex

| 区别                 | pinia                                                        | vuex                     |
| -------------------- | ------------------------------------------------------------ | ------------------------ |
| store                | 使用`defineStore`定义 store，支持多个 store -> stores。不存在 modules 概念 | 单个 store，使用 modules |
| state 定义           | state 可以为对象也可以是函数，函数时支持 SSR                 | state 为对象             |
| state 重置           | `store.$reset`✅                                              | ❌                        |
| state 订阅           | `store.$subcribe(mutation, state)=> { // dosth }`✅           | ❌                        |
| getter 传参          | `userInfo: id`；`getters: { userInfo: state => { return (id) => userInfo.find(e => e.id === id) }}`✅ | ❌                        |
| mutations            | 无 mutations                                                 | mutations 处理同步操作   |
| actions              | 既处理同步也处理异步                                         | 推荐只处理异步操作       |
| actions 订阅         | `store.$onAction(({ actionName, store, args}) => {})`✅       | ❌                        |
| maoState、mapActions | ✅                                                            | ✅                        |

## 从 vuex 迁移到 pinia

从 vuex 迁移到 pinia 中要注意以下几点：

1. pinia 使用的是多 store，vuex 是单 store，所以 pinia 没有 namespace 和 modules 的概念，但是可以模拟这样的效果，使用相同的语法



#### 语法&使用 区别

pinia 使用 `defineStore`来定义单个 store

```js
import { defineStore } from 'pinia'

export default defineStore({
  id: 'files', // 这个是必须的
  state: () => ({
    files: []
  })
})

```

##### state

修改state的5种方式

1. 直接用对象修改
2. `$patch(args)`
3. `$patch(() => args)`
4. `$state`
5. `actions`内的方法修改

下面是实例

```js
import { mapState } from 'pinia'
import { useUserStore } from '@/stores/user.js' // useUserStore 与 id 相同

export default {
  setup() {
    const store = useUserStore()
    store.name = 1 // 直接使用对象修改
    store.$path({ // 使用 $path 可以同时修改多个
      name: 1,
      age: 10
    })
    store.$state = {name: 1} // 使用 $state 直接替换
    store.setName('1') // 调用 actions 的 setName 方法修改
  },
  computed: {
    // 相当于 store.name
    ...mapState(useUserStore, ['name'])
    ...mapState(useUserStore, {
    myOwnName: 'doubleCounter',
    // 甚至可以写个方法计算
    double: store => store.doubleCount,
  }),
},
  // ...
}
```

##### getters

1. 可以在组件中使用`store.getter`直接访问
2. getter 和 state 不能重名，也不需要 getter 再声明了。在 vuex 中是为了计算属性缓存，在 pinia 的 state 中声明已经会这么做了，**pinia 中使用 getters 完全是为了基于状态的计算目的服务** 
3. 支持接收参数，只需返回一个函数即可
4. 可以通过`this`访问其他 getter

##### actions

通过 actions 改变 state 的值，actions 直接使用 this 访问当前 store，比 vuex 更贴近 vue 的用法

```js
import { defineStore } from 'pinia';

export default defineStore({
  // ...
  state: () => ({
    userProfile: {} // 用户配置
  }),
  actions: {
    setUserProfile (userProfile) {
      this.userProfile = userProfile;
    }
  }
});

```

#### 在组件外（js）中使用

在组件外使用和 vuex 基本相似，只不过更简单，更符合直观的感受

```js
import { useCommonStore } from '@/stores'; // 引入我们定义的 store

// 注意不要再全局函数中使用，因为 require 时 pinia 还未挂载
const commonStore = useCommonStore(); // 获取对象

// 获取 state
console.log(commonStore.name) // 输出 state 中的 name

commonStore.setName('lyk') // 调用 actions 中的 setName 方法
```

## 使用注意事项

在使用中可能会遇到一些报错或者疑惑的点，这里通过查到的一些资料做一个记录。

#### getActivePinia was called

这个异常一般是因为我们在`Vue.use(pinia)`前就使用了 pinia。

但是这种场景有时是不可避免的，比日我们要在全局路由守卫中使用，那么路由守卫最开的执行是比 pinia 初始化要快的？？？

官方给出了以下三种解决方案：

- Call the `useAuthStore()` function after `app.use(pinia)` (https://pinia.vuejs.org/core-concepts/outside-component-usage.html#single-page-applications)
- Pass the pinia instance to `useAuthStore(pinia)`. `pinia` being the one created with `const pinia = createPinia()`
- Call `setActivePinia(pinia)` somewhere before `useAuthStore()`

#### 解构会丢失响应性

和 vue3 解构对象会丢失响应性类似，解构 state 的值同样也会丢失响应性。需要使用`storeToRefs`包裹

```js
import { storeToRefs } from 'pinia'

const filesStore = useFilesStore()
const { files } = storeToRefs(filesStore)
```

