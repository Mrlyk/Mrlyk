# vue3 最佳实践

## 

## SFC 结构

经过对 github 上项目的观察，除了 vue2 迁移的项目，为了减轻工作量仍然保留了声明式的语法。纯 vue3 项目都已经放弃了 vue2 中的声明式语法。基本就下面两个方式：

- setup 一把梭
- 使用 setup 语法糖一把梭

仍然存在 vue2 种逻辑太多的问题，需要进行详细的逻辑拆分。

与 vue2 相比的优点：

- vue2 拆分逻辑一般使用 mixin，mixin 存在重名和不够灵活的问题（不能通过参数改变逻辑）
- 逻辑拆分更加清晰，组合式的 API 也让每一处逻辑有迹可循

## 逻辑拆分

vue3 相比 vue2 最大的变更就是 composition-api 的存在，也正是因为他的存在我们可以更好的拆分逻辑。将独立的逻辑放在独立的文件中维护，不再是一个 vue 文件几千行，也没有了 mixin 带来的混乱。

以父组件获取数据打开弹窗展示这种常规的业务逻辑为例。

```vue
<template>
	<a-dialog :visible="visible" :data="data"></a-dialog>
</template>
<script setup>
  import dialogHandler from './dialogHandler.js'
  
  const { data, visible } = dialogHandler()
</script>
```

在`dialogHandler`中

```js
/* dialogHandler */
import { ref } from 'vue'

export function () {
  const data = ref({})
  const visible = ref(false)
  
  getData() {
    // .... res
    data.value = res.data
    visible.value = true
  }
  return {
    data,
    visible
  }
}
```

可以看到组件中弹窗这块的逻辑只有两行引入代码，其余逻辑完全独立。

这样整体项目更加清晰，易于维护，再也不用一个 vue 组件方法跳来跳去看几千行了。属实泪目了T-T
