# vue-loader

vue-loader 是编译 vue SFC 的 webpack loader。通过他我们可以对 vue 的编译统一进行处理，处理一些问题。比如覆盖内置指令，修改编译内容！



## 修改内置指令

内置的`v-html`指令：https://github.com/vuejs/vue/blob/8d3fce029f20a73d5d0b1ff10cbf6fa73c989e62/src/platforms/web/compiler/directives/html.js#L6982

通过下面的 vue-loader 配置来进行覆盖，为什么往 props 里推可以查看`addProp`方法（[点击这里](https://github.com/vuejs/vue/blob/8d3fce029f20a73d5d0b1ff10cbf6fa73c989e62/src/compiler/helpers.js#L23)）

```js
{
  loader: 'vue-loader',
  options: {
    // 处理 xss 注入问题
    compilerOptions: {
      directives: {
        html(node, directiveMeta) {
          (node.props || (node.props = [])).push({
            name: 'innerHTML',
            value: `xss(_s(${directiveMeta.value}))`
          });
        }
      }
    }
  }
}
```

