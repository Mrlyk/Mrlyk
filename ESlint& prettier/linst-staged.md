# lint-staged

> 这个工具和名称一样，只对 git stage 的代码进行检查，而不是检查所有的代码。可以提升检查效率。

*lint-staged 和 eslint 没什么关系，只是使用 eslint 提交前检查的时候总是会用到它，所以放在一起记录一下*

## 安装

```shell
npm install lint-staged -D
```



## 使用

一般 lint-staged 要配合 git hooks 一起使用，毕竟是对 git stage 的代码进行检查。如果不是在`git commit`时配合使用，就没那味。

**lint-staged 的配置支持在`package.json`中和`lint-staged.config.js`两种** ，一般用法如下（在 package.json 中）

```json
{
  "name": "my-project"
  "lint-staged": {
    "*.{vue,js,ts}": ["eslint --fix", "git add"]
  },
}
```

支持执行 npm script

```json
{
  "*.js": "npm run my-custom-script --"
  // "*.js": ["cross-env NODE_ENV=test jest --bail --findRelatedTests"]
}
```

支持`ignore`配置

```json
{
  "lint-staged": {
    "linters": {
      "*.{js,vue}": [
        "eslint --fix",
        "git add"
      ],
      "*.{vue,css,less}": [
        "stylelint --fix",
        "git add"
      ]
    },
    "ignore": [
      "**/*.min.js",
      "weaver.common.js"
    ]
  }
}
```

