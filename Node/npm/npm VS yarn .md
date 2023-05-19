# npm VS yarn

npm 作为 node 官方的包管理工具，其特点就是扁平化。同时也存在很多问题，比如

- 幽灵依赖：用户没有显示安装的依赖，但是却能使用。因为扁平化的结构将依赖的依赖也提升到了一级，让外部也能引用到了
- 体积巨大（node_modules hell）：由于 npm 的扁平化结构，本意是为了减少重复的依赖安装。但是由于依赖和依赖的依赖会存在使用相同依赖不同版本问题，只要最顶层有一个不同的版本了。后面的每一个版本都只能被重复的安装在下层中，即使下层中的依赖版本都相同。（扁平化的顺序依赖于安装的顺序）

在 npm5 之前还不存在 `package-lock.json`，导致每个人安装的依赖虽然大版本相同，但是小版本之间可能存在差异。最终项目在不同的机器运行结果不同。（npm3 之后用`npm shrink`，也能生成一个 npm-shrinkwrap.json，内容和 `pa ckage-lock.jso`相同）。

在 npm3 -> npm5 之间，yarn 出现了，其实他出现主要就是为了解决上面说的版本差异问题，用来控制更精确的版本。npm 就不在赘述了，这里详细介绍下 yarn

## yarn

yarn 由 facebook 在 2016.10 发布，最初发布的目标就是解决 npm 的问题。比如上面说的问题，同时带来了性能优化。

- yarn 2：2020 发布
- yarn 3：2021 发布

安装 yarn 可以直接使用 npm 全局安装

```shell
npm install -g yarn

# 可以设置不同的 yarn 版本
yarn set version berry
yarn set version latest # 设置为最新版
```

#### yarn vs npm 常用命令对比

| 命令     | npm                    | yarn                        |
| -------- | ---------------------- | --------------------------- |
| 安装     | `npm install`          | `yarn add`                  |
| 卸载     | `npm uninstall`        | `yarn remove`               |
| 更新     | `npm update`           | `yarn upgrade`              |
| 执行操作 | `npm run`              | `yarn run`                  |
| 运行脚本 | `npx vue create hello` | `yarn dlx vue create hello` |

- 全局操作：`yarn global add` 全局安装
- devDependecies 安装：`yarn add xxx -dev(-D)` 

## 性能对比

npm 安装包时是按顺序执行的，所以上一个包装完了才会装下一个。**yarn 是并行执行安装任务的**，所以性能比 npm 要高。

两者都有缓存机制，yarn 是将整个包都保存在磁盘上，所以可以做到下一次离线安装。npm 则不行，他会尝试联网去检查包的版本。

在最新的 npm 和 yarn 版本中，两者的性能差距并没有很大。

## 功能对比

#### 生产锁定文件

npm 5 之后会生成`package-lock.json`锁定文件，yarn 也会生产`yarn.lock`锁定文件。

在这一点上，**新版的 npm 做的更好。npm 不仅能锁定文件版本，并且可以保证每次安装的项目结构拓扑图相同**。yarn 则只保证版本相同。这一点在我们需要直接使用路径来访问`node_modules`中的包时显的很重要。

#### monorepo

单一仓库功能。

yarn 和 lerna 提供相同的 monorepo 方式：

1. 扁平化安装依赖，处理重复安装的依赖（也存在 npm 上面的问题）
2. 创建各个 package 的软连接到顶级目录的`node_modules`中
3. 创建各个 package 里 `node_modules`中 bin 软连接到顶级目录的`node_modules`中，保证每个 package 的命令能正常执行

当然这样也加剧了上面说的幽灵依赖和重复安装的问题

#### yarn 支持完全的离线安装（零安装）

上面已经说过了 yarn 会更好的利用缓存。yarn 将缓存存储在项目的`.yarn/cache`文件中。

#### 即插即用

当执行`yarn add`命令时会创建一个`.pnp.cjs`文件。

在过去，目录的解析工作交由 node 内置的 resolve 模块，一步步查找文件所在的目录。而 yarn 则提供了更优雅的方式：告诉 node 包在磁盘上的位置，并管理包之间的依赖关系（yarn > 2.0）。

`.pnp.cjs`文件包含了包和磁盘位置的映射关系，有了这张关系表，yarn 可以立即告诉 node 去哪里查找包。达到安装一个新的依赖时能立刻使用的效果（不用重启项目？）并且为优化提供了更多可能。

#### 按 scope 更新

yarn 支持按 scope 来更新或删除某一类包

```shell
yarn upgrade --scope @vue #更新所有 vue 开头的包
```

npm 则不支持这样的操作，需要使用另外的依赖包如 [npm-check-updates](https://www.npmjs.com/package/npm-check-updates) 

## 如何选择

无需纠结，这两个差距不是很大，习惯哪个就哪个！要么就选择更现代的`pnpm`、`tnpm`等