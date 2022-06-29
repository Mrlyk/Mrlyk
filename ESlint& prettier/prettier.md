# prettier

eslint 是一个代码语法风格化的检查工具，而 prettier 是一个纯粹的**代码格式风格化**的检查工具。

在我们有代码规范化要求（整洁强迫症）的时候可以使用它，他可以格式化

- CSS（当然也可以使用更专一的 stylelint ）
- HTML
- JS / TS
- vue
- markdown
- ...

等等绝大部分前端用到的格式

[toc]

## 安装

```shell
npm install prettier -D

npm install eslint-config-prettier -D # 为了防止和 eslint 冲突，还需要安装这个插件 
```





## 使用

如果使用了 eslint 需要配置插件

```js
// eslintrc.js
module.exports = {
  rules:{
    "prettier/prettier":"error" 
  },
  plugins: ["prettier"]，
}

/*=====也可以直接使用扩展=====*/
module.exports = {
  extends:{
    "eslint:recommended",
    "plugin:prettier/recommended"
  }
}
```



## 配置文件

- .prettierrc.[js|json|yaml] 配置文件
- .prettierignore 配置要忽略的文件

#### 配置规则

```js
module.exports = {
  // 一行最多 120 字符
  printWidth: 120,
  // 使用 2 个空格缩进
  tabWidth: 2,
  // 不使用缩进符，而使用空格
  useTabs: false,
  // 行尾需要有分号
  semi: true,
  // 使用单引号
  singleQuote: true,
  // 对象的 key 仅在必要时用引号
  quoteProps: 'as-needed',
  // jsx 不使用单引号，而使用双引号
  jsxSingleQuote: false,
  // 末尾需要有逗号
  trailingComma: 'all',
  // 大括号内的首尾需要空格
  bracketSpacing: true,
  // jsx 标签的反尖括号需要换行
  jsxBracketSameLine: false,
  // 箭头函数，只有一个参数的时候，也需要括号
  arrowParens: 'always',
  // 每个文件格式化的范围是文件的全部内容
  rangeStart: 0,
  rangeEnd: null,
  // 不需要写文件开头的 @prettier
  requirePragma: false,
  // 不需要自动在文件开头插入 @prettier
  insertPragma: false,
  // 使用默认的折行标准
  proseWrap: 'preserve',
  // 根据显示样式决定 html 要不要折行
  htmlWhitespaceSensitivity: 'css',
  // vue 文件中的 script 和 style 内不用缩进
  vueIndentScriptAndStyle: false,
  // 换行符使用 lf
  endOfLine: 'lf',
  // 格式化内嵌代码
  embeddedLanguageFormatting: 'auto',
};
```

