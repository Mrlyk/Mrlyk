# react 工程配置

记录一下使用 react 需要的 eslint 和 babel 配置，webpack 后续再补充



#### eslint

eslint 主要是要配置好 react 的 extends 配置

```js
module.exports = {
  root: true,
  parser: '@babel/eslint-parser',
  parserOptions: {
    ecmaVersion: 2020,
    ecmaFeatures: { // 这个配置在 react 的 extends 里自带了，不配也可以
      jsx: true
    }
  },
  settings: {
    react: {
      version: 'detect'
    }
  },
  env: {
    browser: true,
    node: true,
    es2021: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended', // 这个是 react 必须的
    'standard',
    'plugin:react/jsx-runtime' // 这个是 react jsx 语法支持必须的
  ]
}
```

#### babel

babel 的配置直接使用预设就可以了，比较常规

```js
module.exports = {
  presets: [
    '@babel/preset-react'
  ]
}
```

