# @commitlint

commitlint 是规范化 git 提交信息的工具，内置了一套规则



## 安装

```sh
npm install @commitlint/cli @commitlint/config-conventional --save-dev
```



## 使用

一般我们配合 git husky 使用

在`commit-msg.sh`脚本中

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx --no-install commitlint --edit $1
```

**历史版本 $1 就是 `HUSKY_GIT_PARAMS` 表示 ** 

由于在`husky`中的指定的`commit-msg`钩子命令并不是git直接执行的，因此只能通过`husky`间接暴露的变量`HUSKY_GIT_PARAMS`来获取临时文件的地址 

```js
const param = process.argv[process.argv.length - 1]  // 获取git commit消息临时存放文件地址
```

**默认遵守规范 ** 

- `type`: 用于说明 commit 的类型。一般有以下几种:

```
feat:        A new feature(新增feature)
fix:         A bug fix(修复bug)
docs:        Documentation only changes(仅文档更改,如README.md)
refactor:    A code change that neither fixes a bug nor adds a feature(代码重构，没有新增功能或修复bug)
perf:        A code change that improves performance(优化相关，如提升性能、用户体验等)
test:        Adding missing tests or correcting existing tests(测试用例，包括单元测试、集成测试)
build:       Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)(影响构建系统或外部依赖关系的更改（示例范围：gulp、broccoli、npm）)
chore:       Other changes that don't modify src or test files(其他不修改src或测试文件的更改)
style:       Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)(不影响代码含义的更改（空格、格式、缺少分号等）)
ci:          Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)(对ci配置文件和脚本的更改)
revert:      Reverts a previous commit(还原以前的提交)
```

#### 配置文件

[官方文档](https://commitlint.js.org/#/reference-configuration)

通常无特殊需求使用官方推荐的配置即可，如下

```js
module.exports = {
  defaultIgnores: true,
  extends: ['@commitlint/config-conventional']
};

```

