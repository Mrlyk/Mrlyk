# pnpm 现代包管理工具

[toc]

特点：

- 符合依赖的目录结构
- 使用硬连接的方式相同的依赖只安装一遍，节省磁盘空间
- 因为只安装一遍依赖且不需要扁平化，所以快（还解决了幽灵依赖的问题）
- 工作区可以更好的在微应用之间共享包

使用 pnpm 安装，pnpm 会将依赖存储在位于 `~/.pnpm-store` 目录下。只要你在同一机器下，下次安装依赖的时候 pnpm 会先检查 store 目录，如果有你需要的依赖则会通过一个硬链接丢到到你的项目中去，而不是重新安装依赖。

也可以手动配置该目录`pnpm config set store-dir /path/to/.pnpm-store`

## 常用命令

| npm 命令        | pnpm 对应的命令                               |
| --------------- | --------------------------------------------- |
| `npm install`   | [`pnpm install`](https://pnpm.io/cli/install) |
| `npm i <pkg>`   | [`pnpm add `](https://pnpm.io/cli/add)        |
| `npm run <cmd>` | [`pnpm `](https://pnpm.io/cli/run)            |
| `npm update`    | [`pnpm update`](https://pnpm.io/cli/update)   |
| `npm uninstall` | [`pnpm remove`](https://pnpm.io/cli/remove)   |

#### 其他命令

- `pnpm start`： 如果未在 script 中声明 start，默认执行`node server.js`
- `pnpm run`: 和 npm 类似执行 script 语句，但是支持简写即不使用`run`关键字也可以
- `pnpm test`: 执行 script 中的 test 命令
- `pnpm import`: 可以从 `package-lock.json`或者`yarn.lock`生成一个自己的版本管理文件`ppm-lock.yaml`
- `pnpm prune`: 自动删除未使用的包

#### ppm --filter

filter 命令时 monorepo 解决方案的关键命令，允许用户指定一个 scope 来执行命令

`pnpm --filter <package_selector> <command>`

**举例说明** 

```shell
pnpm --filter "@babel/core" test # 执行 @babel/core 中的 test 命令

pnpm --filter "foo^..." test # 执行 foo 的所有依赖的 test 命令
pnpm --filter "...^foo" test # 执行所有依赖 foo 的依赖的 test 命令

pnpm --filter ...{<directory>} <cmd> # 还可以使用具体的文件夹地址

pnpm --filter=!./lib <cmd> # 执行非 lib 文件下的命令
```

通过这个命令我们就可以只安装某个文件夹下的依赖，而不是每次都全量安装

同时由于`pnpm`默认使用一个`store`文件夹来存储依赖，所以对 monorepo 有天然的支持

## 带来的一些问题

老项目如果直接切换为 pnpm 反而会带来一些之前没有的问题

#### 幽灵依赖没有反而导致的问题

以普通项目为例，比如我们在项目中要使用`async`，那么为了兼容低版本的浏览器，需要`regenerator-runtime`这个包。

这个包很多地方都要用，比如 vue-cli 自己就用了，加上 npm 原本的扁平化安装方式。让我们即使没有安装也不会报错。但是在使用`pnpm`就需要单独的显示安装了。

## 从 npm/yarn 迁移

`pnpm import` 命令可以帮助我们从另一个软件包管理器的 lock 文件生成 `pnpm-lock.yaml`。 支持的源文件：

- `package-lock.json`
- `npm-shrinkwrap.json`
- `yarn.lock` (v6.14.0 起)

迁移一般分四步：

1. 删除原来的 `node_modules`
2. `pnpm import package-lock.json(yarn.lock) `
3. 修改原来的 `script`命令
4. 处理一下原来的幽灵以来问题，build 看一下有哪些幽灵依赖就行