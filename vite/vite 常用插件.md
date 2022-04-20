# vite 常用插件

## 通用插件



## vue3 相关

#### vite-plugin-vue-setup-extend

在 vue3 中，如果使用 setup 标签语法，会无法设置组件的`name`。但是很多情况下需要组件有 `name`，就可以使用该`setup`编译增强插件

**使用**

使用时，直接给 setup 属性的`script`标签以`name` 属性即可

```js
// 配置
import { defineConfig } from 'vite'
import VueSetupExtend from 'vite-plugin-vue-setup-extend'

export default defineConfig({
  plugins: [
    VueSetupExtend()
  ]
})

// 使用
<script setup name='my-component'></script>
```

#### unplugin-auto-import API 自动导入

不熟悉 vue3 前不建议使用