# vue2 到 vue3 的破坏性变更

[toc]

## 被移除的 API

- $on、$off 和 $once 实例方法
- filter 过滤器被移除，官方推荐使用方法或计算属性进行替代
- `>>>  /deep/ ::v-deep` 选择器被弃用（3.2 版本仍可使用，但是未来会被放弃），可以使用`:deep(.clas-name)`替代

## v-xx 各自指令

- **v-model** 不再默认传递`value`和响应`input`事件。现在默认传递`modelValue`和响应`update:modelValue`。如果需要自定义值可以`v-model:title="title"`这样。因**此也移除了`.sync`修饰符和组件的`model`选项** 
- 

## 组件相关



## 生命周期相关

- `destroyed` 生命周期选项被重命名为 `unmounted`
- `beforeDestroy` 生命周期选项被重命名为 `beforeUnmount`