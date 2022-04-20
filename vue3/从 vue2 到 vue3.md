# 从 vue2 到 vue3  

[toc]

## 路由 vue-router@4  
### 一、创建 
```js
import { createRouter } from 'vue-router' 
const route = createRouter({
  routes: [
    {
      name: 'index'
      path: '/',
      component: component
    }
  ]
})
```
通过暴露出的 createRouter 方法进行创建，而不在是使用 new Router 实例。

### 二、路由模式 
```js
import { createWebHistory, createWebHashHistory } from 'vue-router'

createRouter({
  history: createWebHistory() // history 模式
  history: createWebHashHistory() // hash 模式
})
```

## 新功能 vue@3  
### 一、组合式 API （ Composition API ）
> 从选项式 API 到组合式 API，vue 暴露出了许多新的方法，如 reactive、ref 响应式，watch 监听等等，让我们可以在组件中按需引入和使用。  

#### 1. 处理的问题 
在 vue2 中 data、props、methods、watch 等方法过于庞大时，代码逻辑查找困难，维护也困难的问题。 

#### 2. 使用 setup(返回渲染上下文) 
1. 注册 setup API，他会在 M、D、C、W、所有生命周期、components 前，初始 prop 解析之后立刻调用，且无法访问 this 实例  
2. 在 setup 中声明变量或方法并返回，即可定义一个普通的对象或方法
```js
/* vue-template
  {{ a }}
  <button @click="getFn" />
*/
export default {
  // 接受两个参数 props，context
  // props： 父组件传递过来的参数对象
  // context：上下文对象，包括 attrs 、 emit 、 slot 。
  setup (props, context) {
    const a = 3
    const getFn = () => {
      console.log('getData')
    }
    return {    // 这里返回的任何内容都可以用于组件的其余部分，可以在实例中使用 this 访问
      a,
      getFn
    }
  }
}

```

#### 3. 在 setup 中定义响应式的对象，使用 C、W、LifeCycle 

**（1）响应式的对象**  
<hr/>

**使用 ref**  

在 vue3 中使用 ref 函数来定义响应式的变量，ref 会对我们传入对初始值进行一个包装；
在 js 中基本类型的变量使用是直接复制而不是引用，在 vue 中为了防止其失去响应性，对数据进行了一个**包装**（ref），实际 ref 就是对我们的值创建了一个响应式的引用。 

```js
/* vue-template
  {{ capacity }} // 这里会自动展开访问 .value 属性
*/

import { ref } from 'vue'
export default {
  setup (props, context) {
    const capacity = ref(1)  // ref(1): 含有 value 属性返回的是当前值，可以直接对value 进行操作。另外还具有 _shallow、_value、_rawValue、__v_isRef 属性
    console.log(capacity.value)
    capacity.value ++ 
    return {
      capacity
    }
  }
}
```

**使用 reactive**   

当访问一个被响应式的 Object 嵌套的对象时，Ref 实例会自动展开访问 .value ，当从 Array 或原生集合类型 Map 访问 ref 时不会进行展开，依然需要手动访问 .value 属性。 

```js
import { ref, reactive } from 'vue'

export default {
  setup () {
    const count = ref(0)
    const state = reactive({
      count
    })
    state.count ++ // 访问响应式对象的属性时，属性会自动展开 .value 
    
    const otherCount = ref(1)
    
    state.count = otherCount // state.count === 1;count.value === 0
    return {
      state
    }
  }
}
```
**ref 和 reactive 的区别** 

**reactive 主要用来包装复杂对象，其使用的是`proxy`对对象进行包装；基本类型的数据使用 ref 包装，其原理和 vue2 中响应式的原理相同：对 `getter`和`setter`进行劫持重写。如果 ref 包装引用类型对象，最终还是调用的 reactive。reactive 包装基本类型不会有任何效果** 。reacvie 包装后，会使用`proxy`代理对象，ref 则只是进行响应式的包装。

**响应式对象使用解构的情况**  

当我们直接使用 ES6 解构来获取响应式的属性时，需要使用 ref 的一些 API 进行转换，否则会导致响应性丢失 
```js
import { reactive, toRefs } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: 'Mike',
      age: '23',
      sex: 'male'
    })
    let { name, age } = toRefs(state) // 使用 toRefs 方法将对象整个转换为响应式的，这样在解构之后的属性依然为响应式
  }
}
```
**响应式对象的只读限制**  

```js
import { reactive, readonly } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: 'Mike',
      age: '23',
      sex: 'male'
    })
    const copyState = readonly(state) // 使用 readonly 限制响应式对象只读，一般比如使用 provides 时可以用到，防止子组件更改
  }
}
```

**（2）setup 中的生命周期钩子** 
<hr/>

setup 中可以访问 beforeMount 到 updated 的生命周期钩子，包含他们本身。同时基于合并策略，setup 中的生命周期钩子回调的方法会比组件本身同一周期的方法更早执行。

**使用**  
使用 on 操作符访问，同时接受一个回调函数作为参数
```js
setup () {
  onMounted (() => console.log('setupMounted'))
}
```

**（3）setup 中的监听 watch**   

<hr/>

setup 中的 watch 与选项式 API 中的 watch 相同，同样接受三个参数：观察对象、回调函数、选项：onInvalidate  
```js
// 使用 watch 要先从 vue 中导入  
import { watch, ref } from 'vue'

export default {
  setup () {
    const r1 = ref('r1111')
    const r2 = ref('r2222')

    watch(r, (val, oldVal)=> {
      console.log(val)
    })
    // 在 vue3 中可以使用数组同时监听多个值
    watch(([r1, r2]), ([newR1, newR2], [oldR1, oldR2]) => { // todo: xxx
    })
  }
}
```
除了常规的 watch ，vue3 中还提供了一个 watchEffect 来观察响应式的变化。与计算属性有些类似，都是在依赖变更时调用，但是不需要返回一个值。
```js
import { watchEffect } from 'vue'

export default {
  setup () {
    const r = ref(0)
    watchEffect(() => console.log(r.value))
    // 同时 watchEffect 也返回一个函数可以用来手动终止监听（他也会在组件销毁时自动销毁）
    const stop = watchEffect(() => console.log(r.value + 1))
    stop()
    
    // watchEffect 还接受一个 onInvalidate 参数用来清除 Effect ，该函数会在监听被触发前和监听销毁前调用
    watchEffect((onInvalidate) => {
      console.log(r.value)
      onInvalidate(() => {
        console.log('onInvalidate')
      })
    })
    
    // watchEffect 还接受配置项 { flush: 'pre'/'post'/'sync'}，用来配置在组件更新前还是更新后或是同步调用，默认是组件更新前
  }
}
```
watchEffect 调试
onTrack 和 onTrigger 选项可用于调试侦听器的行为。且只在开发模式下生效  

- onTrack 将在响应式 property 或 ref 作为依赖项被追踪时被调用。
- onTrigger 将在依赖项变更导致副作用被触发时被调用。

**（4）setup 中的计算属性 computed**  
```js
import { computed, ref } from 'vue'

export default {
  setup () {
    const count = ref(2)
    // 默认接受一个 getter 函数，我们也可以显示的指定 get 和 set 函数
    const doubleCount = computed(() => count.value * 2)
    const tripleCount = computed({
      set: val => count.value = val + 1,
      get: () => count.value
    })
    return {
      count,
      doubleCount
    }
  }
}

```

#### 4、 `getCurrentInstance` 获取组件实例

`getCurrentInstance` **只能**在 setup 或生命周期钩子中调用

**除非在开发高阶组件，否则强烈不推荐使用这种方式来获取 this 的替代方案**

```js
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()
    internalInstance.appContext.config.globalProperties // 访问 globalProperties
  }
}
```



**总结**  
组合式 API 似乎只是使用了一个 setup 选项来将数据定义，生命周期访问，计算，监听等放到了一起，而他也确实是这么做的。我们可以使用这种方法来将同类逻辑组合提取到一个单独的文件上，让后续维护更简单。

### 二、Teleport 组件（传输组件） 

Teleport 组件用于将 dom 元素渲染到指定位置。常见的业务场景就是模态框，我们希望他是全屏的、位于 app 之外的。但是我们知道 vue2 渲染是绑定了根节点 `#app `的。所以组件内的 dom 渲染都被限制在了`#app`这个元素之内。

为了处理这个问题，vue3 提供了`Teleport`组件。使用起来也很简单，使用该组件包裹要渲染的元素即可

```vue
<div class="header">header</div>

<div class="body">
  <!-- to 指向要移动到的目标元素，接收一个 disabled 参数用于禁用移动 -->
  <teleport to=".header" disabled> 
    <span>我在哪</span>
  </teleport>
</div>
```

- 如果 teleport 组件中包含的是其他 vue 组件，该组件的逻辑父组件仍然是原来的父组件，不会迁移到新的
- teleport 组件的移动属于真正的 DOM 节点移动，而**不是**销毁和重新创建。所以使用它可以保证 DOM 的元素状态
- to 参数和 `document.querySelector`的`id`、`class`差不多，当然也接收变量
- 多个组件被 `to`到相同节点时，按顺序在后面追究
- **to 的目标元素需要在 teleport 组件前渲染完成** 

### 三、多片段（单组件支持多节点）

在 vue 2 中，一个组件只能有一个根节点，如果存在多个 vue 2 会报错。

```vue
<template>
	<div id='app'></div>
</template>
```

在 vue 3 中，组合式的 api 支持使用组合式命令指定渲染节点支持了多节点。实现方式和 react 的类似，创建了一个虚拟的根节点用来包裹用户创建的元素。具体实现待读书笔记...

当然这会带来一点影响：

- **多节点时，`$attrs`需要我们手动绑定**。vue 会自动帮我们把父组件传过来的，但是子组件不作为`props`使用的属性放在`$attrs`中。如果是单节点，vue 或自动将`$attrs`添加到该单节点中。但是存在多节点的情况下就需要我们手动指定他传到哪，如下

  ```vue
  <template>
    <test-a></test-a>
    <test-b v-bind="$attrs"></test-b>
  </template>
  ```

### 四、SFC（单文件组件）内样式

#### 1、支持 v-bind

心心念念的更直观的动态 css 方式来来，不用再去繁杂的操作类名来改变样式。直接使用`v-bind`来绑定变量

```vue
<template>
  <div class="text">异步展示的组件</div>
</template>

<script>
export default {
  name: 'test',
  data(){
    return {
      textColor: 'red'
    }
  }
}
</script>
<style>
.text {
  /* 好用，非常好用 */
  color: v-bind(textColor) 
}
</style>
```

**其实现是通过 css 中原生的`var()`函数，访问定义在 DOM style 属性中的样式变量**。DOM 的 style 属性中的值在渲染 SFC 时生成。 

注意由于浏览器带来的 CSS 选择器的差异，实现上会产生性能损失。**原来的那种方案**还是比较好，虽然写起来麻烦，但是**性能更好**。 

#### 2、保留 scoped 

scoped 仍然被保留下来，用以限制 SFC 内的样式范围。其实现依赖于`PostCSS`，将样式和对应的 DOM 元素生成随机的`data-xxxx标识（和 vue2 相同）。

只要注意现在深度选择器`/deep`已经不被支持了，需要使用`:deep`

```vue
<style scoped>
.text {
  color: v-bind(textColor)
}
 /* 原理还是和 vue2 一样，加上该标签后，样式会被编译为 
  .text[data-xxxx] .text-b { }
  否则是编译为 .text .text-b[data-xxxxx] { }
  */
.text :deep .text-b { 
  color: blue;
}
</style>
```

#### 3、支持全局选择器

之前我们想在 SFC 中创建一个全局组件一般是另起一个`style`标签，vue 3 提供了 `:global`选择器。

```vue
<style scoped>
:global(.text-b) {
  color: pink;
}
</style>
```

相当于将`.text-b`的样式直接放入了全局样式表中

#### 4、CSS modules 规范支持

style 中的 module 标签会被编译为 CSS Modules，并且生成的 CSS 类作为`$style`对象暴露。同时对生成的类做了 hash 计算避免了冲突，所以不需要额外加`scoped`属性

```vue
<template>
  <div :class="$style.green">哈哈哈</div> <!-- 对象中可以获取到 module 中的类 -->
</template>

<style module>
.green {
  color: green;
}
</style>
```

也可以显示的声明 CSS modules 的名称，不一定是用`$style`

```vue
<template>
  <div :class="text.green">哈哈哈</div>
</template>

<style module='text'>
.green {
  color: green;
}
</style>
```

**在组合式 API 中，可以使用如下两个方法获取到 CSS module 模块** 

```js
// 默认, 返回 <style module> 中的类
useCssModule()

// 命名, 返回 <style module="text"> 中的类
useCssModule('text')
```



### 五、Suspense 异步组件处理方案

**suspense 目前仍然是一个实验性的标签**，其作用是为异步组件的渲染提供另一种方案。通常我们会有这种业务场景：请求一个接口，接口回传后再显示组件。之前我们的处理方式是为每一个这种组件写一个 loading 事件，重复的写这个东西还是很烦的。suspense 的出现则提供了一种更简洁的方案。

suspense 还可以放在 router-view 层上，来处理全局的路由跳转事件。可以说非常方便了。

使用方法如下：

```vue
<template>
  <suspense>
      <template #fallback> <!-- 这里不用 #default 插槽，用其他组件也可以 -->
        还没加载完成哦
      </template>
      <template #default>
        <test></test>
      </template>
    </suspense>
</template>

<script>
import { defineAsyncComponent } from "vue"
  
export default {
  components: {
    Test: defineAsyncComponent(() => import("./test.vue")) // 加载异步组件，也是发送异步请求加载文件
  }
}
</script>
```

如果要**嵌套** `keep-alive`，`transition`使用，需要注意**嵌套顺序**：transition -> keep-alive -> suspense -> router-view

另一种使用 suspense 的方法是从 `setup` 函数中返回一个 Promise，如下：

```vue
<script>
export default {
  async setup() {
    // 在 setup 内使用 await 的话要注意：
    // 大部分组合式 API 只会在第一次 await 之前工作
    return {
      // ...
    }
  }
}
</script>
```

**注意**

- 异步组件和 vue router 提供的组件懒加载不是一个东西。并且组件懒加载不会触发`suspense`。

### 六、createRenderer 创建自定义渲染器

我们可以继承官方的渲染器，来创建自定义的渲染器。在处理某些全局性的事件时会有用！

更多说明待读书笔记...

```js
import { createRenderer } from '@vue/runtime-core'

const { render, createApp } = createRenderer({
  patchProp,
  insert,
  remove,
  createElement
  // ...
})

// `render` is the low-level API
// `createApp` returns an app instance
export { render, createApp }

// re-export Vue core APIs
export * from '@vue/runtime-core'
```

## 兼容性变更

这里记录一些兼容性的变更，可以不改变原有的 vue 2 写法，也可以改变。看心情决定

#### 自定义事件

现在支持在组件上声明`emits`选项，以定义事件，并且如果是原生事件（click 等），会用组件中的事件覆盖原生事件。

**当然不声明也不影响`$emit`的使用，只不过声明了能让组件更加清晰，推荐声明**。

自定义事件选项还有另一个作用，验证事件

```vue
<script>
export default {
  name: 'HelloWorld',
  // emits: ['test']
  emits: {
    test: (param) => {
      if (!param) return false
      return true // 返回 true 表示事件有效
    }
  }
}
</script>
```

