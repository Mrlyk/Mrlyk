# husky

[官方文档](https://typicode.github.io/husky/#/?id=install)

> husky 是 git hooks 的简易配置工具。能让我们轻松的配置 git 在提交前的各种钩子，让我们在提交前执行代码检查，单元测试......

## git 钩子

### 原生的方法安装一个钩子

钩子都被存储在 Git 目录下的 `hooks` 子目录中。 也即绝大部分项目中的 `.git/hooks` 。 当你用 `git init` 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。 这些脚本除了本身可以被调用外，它们还透露了被触发时所传入的参数。

把一个正确命名（不带扩展名）且可执行的文件放入 `.git` 目录下的 `hooks` 子目录中，即可激活该钩子脚本。 这样一来，它就能被 Git 调用。接下来，我们会讲解常用的钩子脚本类型。

### 常用的钩子

**`pre-commit`**

 钩子在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非零值退出，Git 将放弃此次提交，不过可以用 `git commit --no-verify` 来绕过这个环节。

**`prepare-commit-msg`** 

钩子在启动提交信息编辑器之前，默认信息被创建之后运行。 它允许你编辑提交者所看到的默认信息。 该钩子接收一些选项：存有当前提交信息的文件的路径、提交类型和修补提交的提交的 SHA-1 校验。 

**`commit-msg`** 

钩子接收一个参数，此参数即上文提到的，存有当前提交信息的临时文件的路径。 如果该钩子脚本以非零值退出，Git 将放弃提交，因此，可以用来在提交通过前验证项目状态或提交信息。

**`post-commit`** 

钩子在整个提交过程完成后运行。 它不接收任何参数，但你可以很容易地通过运行 `git log -1 HEAD` 来获得最后一次的提交信息。 该钩子一般用于通知之类的事情。

**`pre-receive`**

`pre-receive`是服务端的钩子， 每次有用户执行 `git push` 来推送提交到仓库中时都会被执行。其仅能存在于那些作为推送目标的 **远端仓库**，而非在原始仓库。

## husky 

### 安装

`npm install husky -D`

### 使用

```shell
npx husky install # 本地安装 husky 命令
npx husky add .husky/pre-commit "cmd 命令" # 添加 husky 命令

npm uninstall husky && git config --unset core.hooksPath # 卸载
```

添加命令后，需要去 sh 文件中新增要执行的 shell 语句，在到达这一钩子时便会执行，比如配合`lint-staged`执行代码 eslint 格式化

```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged # 执行 lint-staged
npx eslint --fix src/**/*.js # 执行 eslint 格式化
```

为了防止有的用户本地没有初始化 husky，一般我们还会在 package.json 的 `prepare`脚本中手动安装

```json
{
  "scripts": {
    "prepare": "npx husky install"
  }
}
```



## 其他说明

#### package.json 中的 husky 配置？

在 husky 更新到 6.0 后有破坏性的变更，不再支持以前那样在 package.json 中写配置的做法，而必须使用 sh 脚本来执行。

但是在网上很多教程都比较老，没提到这一点。加上大家抄来抄去，不实践就扔到网上来，哎～

**总之使用上面的方法中的 sh 脚本配置即可！** 
