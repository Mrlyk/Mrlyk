# Vue3 SFC setup

在《从 vue2 到 vue3》中，讲述了 SFC 中样式的相关变更。而在 vue3 中更重要的是函数式思想的引入。所以这里单独讲一讲 SFC 中的 `<script setup>` 

> SFC：single-file components 单文件组，就是我们平常写 .vue 文件组件

[toc]

**setup 属性可以看作是组合式 API 的语法糖**。我们都知道 vue3 最大的改变是组合式 API 的引入，让我们的代码逻辑结构更清晰，而不是一把梭。写什么声明式的 API 都像黑洞，不看文档都不知道哪来的。但是也同样带来了写法的繁杂性，setup 就像是提供了一个逃生通道。

同时 setup 带来的是一种函数式的编程思想？具有以下特点：

- 更少的代码：不需要一个个引入组合式的 API
- 更好的 TS 支持
- 更好的运行性能
- 更好的类型推断支持

## 一、基本语法

要使用 `setup `只需一步，将 `setup`属性添加到 `script`标签上

```vue
<template>
  <div class="border">{{ test }}</div>
</template>

<script setup>
const test = 'setup' // 声明的顶层属性会被直接暴露给模版
</script>
```

**其中的方法会被编译成组件 `setup() `函数的内容**，所以和一般组件 script 标签只在引入首次执行不同，这里的代码在每次**组件实例创建时都会执行** 

#### 顶层绑定直接暴露给模版

如例子中的`test`变量直接被模版使用一样，在`<script setup>`标签中**顶层声明**的属性包括：变量、函数、import 引入，都可以**直接在模版中使用** 

```vue
<template>
  <div>{{ capitalize('hello') }}</div>
</template>

<script setup>
import { capitalize } from './helpers'
</script>
```

## 二、响应式

在上面我们看到，顶层声明的属性能被模版直接使用。**但是要注意，他们不会被自动的响应式包装。**所以要使用响应式的变量还是需要手动进行响应式的包装。

```vue
<template>
  <div class="border">{{ test }}</div>
  <button @click="change">修改</button>
</template>

<script setup>
import { ref } from 'vue' // 需要手动响应式包装
let test = ref('setup')
function change() {
  test.value = 'change'
}
</script>
```

## 三 、使用组件

setup 中支持直接引入组件使用，组件会被当成变量所引用。

```vue
<template>
  <div class="border">{{ test }}</div>
  <SetupTestChild></SetupTestChild> <!-- 使用引入的组件，推荐 PascalCase 命名-->
</template>

<script setup>
import SetupTestChild from './setup-test-child.vue'
</script>
```

#### 动态组件

setup 中组件被当成变量引入，而不是通过字符键来注册。所以使用的时候（依然是`:is`），是直接绑定到对象这个变量上的。如下实例所示

```vue
<template>
  <div class="border">{{ test }}</div>
  <component :is="test ==='change' ? SetupTestChild2: SetupTestChild"></component>
	<!--SetupTestChild、SetupTestChild2 直接是变量而不是字符串-->
</template>

<script setup>
import { ref } from 'vue'
import SetupTestChild from './setup-test-child.vue'
import SetupTestChild2 from './setup-test-child-2.vue'
</script>
```

***注意***

- 如果将 vue 组件或者某些复杂的第三方类实例直接赋给响应式对象会带来像能问题
- vue3 提供诸如`markRaw`、`shallowRef`、`shallowReactive`等方法来不进行响应式包装，就像 vue2 中我经常使用`Object.freeze`来禁止一些常量声明响应式包装一样

不这么做的话开发时 vue 抛出警告

参见官方文档：https://v3.cn.vuejs.org/api/basic-reactivity.html#markraw

- `shallowReactive`：只处理对象最外层属性的响应式（浅层响应式）

- `shallowRef`：只处理基本数据类型响应式，不进行对象的响应式处理
- `markRaw`: 标记对象永远不会是响应式的，声明后。纵使后面使用`reactive`包装也不会生效

#### 递归组件

在 vue3 中 SFC 可以自己引用自己，但是要注意设置递归边界。目前还没看到应用场景

```vue
// test.vue
<template>
  <div class="border">我是 child {{ number }}</div>
  <Test></Test> <!-- 可以自己引用自己，注意爆栈 -->
</template>
```

#### 命名空间组件

当我们使用单文件导出多个组件时。在组件使用处，可以使用命名空间

```vue
// namsspace.js
import SetupTestChild from './setup-test-child';
import SetupTestChild2 from './setup-test-child-2';
export { SetupTestChild, SetupTestChild2}

// parent.vue
<template>
  <Child.SetupTestChild></Child.SetupTestChild> <!--使用命名空间使用-->
</template>

<script setup>
import * as Child from './namespace-component' // 使用命名空间引入
</script>

```

## 四、声明 props 和 emit

在 `<script setup>` 中必须使用 `defineProps` 和 `defineEmits` API 来声明 `props` 和 `emits`

```vue

<template>
  <div class="border">{{ test }}</div>
  <button @click="$emit('change')">emit</button>
</template>

<script setup>
import { defineProps } from 'vue';
defineProps({ // 返回 props 对象
  test: String
})
const emit = defineEmits(['change']) // 返回 emit 对象
emit('change') // 触发 change 事件
</script>
```

- `defineProps` 和 `defineEmits` 都是只在 `<script setup>` 中才能使用的**编译器宏 **
- `defineProps` 不声明无法接收传递进来的变量
-  `defineEmits` 不声明依然可以触发事件，但是 vue 会给出警告

## 五、暴露自身属性

 `<script setup>` 的组件是**默认关闭**的，即无法通过模版`ref`或者`$parent`获取到组件中声明的变量。但是 vue3 提供手动暴露的方法

```vue
/*===========parent.vue===========*/
<template>
  <SetupTestChild ref="child" :test="test" @change="change"></SetupTestChild>
</template>

<script setup>
import { ref } from 'vue'
const child = ref(null) // $refs 的替代方案，需要先声明属性，再在模版中绑定

function getChild () {
  alert(child.value.number) // child 就是子组件实例，child.value 才能获取到子组件暴露的变量
}
</script>

/*===========child.vue============*/
<template>
  <div class="border">我是 child }</div>
</template>

<script setup>
import { defineExpose } from 'vue';

const number = 1
defineExpose({ // 暴露 setup 中可访问的变量
  number
})
</script>
```

## 六、$slots、$attrs

一般很少需要在 setup 中获取，如果要获取使用下面两个方法

```vue
<scipt setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</scipt>
```

## 七、setup 和普通的 script 一起使用

setup 和普通的 script 可以同时使用。主要场景时：

- **导出组件名 **，比如`keep-alive`就需要组件名，这时候就可以使用这种混用
- 声明`inheriAtters`：不继承父组件传递的 DOM 元素上的 prop 属性（不影响 $attrs 取值，只影响渲染）
- 运行副作用或者创建只需要执行一次的对象
- **如果同时使用这两个标签，会不支持`render`函数**。这种场景下使用一个普通的 `<script>` 结合 `setup` 选项来代替

## 八、顶层 await

```vue
<script setup>
const post = await fetch(`/api/post/1`).then(r => r.json()) // 顶层直接使用 await
</script>
```

因为 setup 块最终会被编译成 SFC 中的 setup 方法：`async setup`。

*`async setup`必须与`suspense`组合使用* 

## 最佳实践

- **setup 还是声明式语法？ **

setup 具有函数式编程的优点，也更加简洁。如果是**新的 vue3 项目推荐使用 setup 语法。**

同时也观察过 github 上一些高 star 项目，也是以 setup 为主。只有从 vue2 迁移到 vue3 的项目为了兼容性，偏向于声明式语法！



## 其他

#### `<script setup>` & `<script>`中的 setup

测试发现，当存在 `<script setup>` 时，会覆盖`<script>`中的 setup（但是可以逃过代码检查）

#### `<script setup>` & `<script src>`

由于模块执行语义的差异，`<script setup>` 中的代码依赖单文件组件的上下文。当将其移动到外部的 `.js` 或者 `.ts` 文件中的时候，对于开发者和工具来说都会感到混乱。**因而 `<script setup>` 不能和 `src` attribute 一起使用。** 