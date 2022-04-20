# vite

> 新一代打包工具

[toc]

## 介绍



#### 特点

- vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM。



## 重要配置说明

#### base 打包后的静态资源目录

```js
export default {
  base: '/', // 打包后静态资源的目录，相对于 ng 上的 location root 路径
}
```

#### optimizeDeps 依赖预构建

vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM。

默认的依赖项发现为启发式可能并不总是可取的。在想要显式地从列表中包含/排除依赖项的情况下, 可以使用 [`optimizeDeps` 配置项](https://cn.vitejs.dev/config/#dep-optimization-options)。

```js
export default {
  optimizeDeps: {
    include: ['@liaoyk/cookie-util'] // 手动添加预构建的依赖，比如 npm link 放进来的可能检测不到，就需要手动指定
  }
}
```

