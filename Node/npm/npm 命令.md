# npm 命令

[toc]

## 配置文件

**每个用户的配置文件**

`$HOME/.npmrc`（或`userconfig`参数，如果在环境或命令行中设置）

**全局配置文件**

`$PREFIX/etc/npmrc`（或`globalconfig`参数，如果在上面设置）：此文件是一个 ini 文件格式的`key = value`参数列表。环境变量可以如上替换。

**内置配置文件**

path/to/npm/itself/npmrc

## 命令

npm 命令执行也具有 hooks，调用我们在 package.json 中定义的 script 语句

**在执行 `install` 的时候会按生命周期顺序执行相应钩子：** 

- `NPM7`： `preinstall -> install -> postinstall -> prepublish -> preprepare -> prepare -> postprepare` 
- `NPM6`： `preinstall` -> `install` -> `postinstall`，同时也会触发 `prepublish`、`prepare` 

文档 

- [npm7 scripts](https://link.segmentfault.com/?enc=9zq0v54Tk7tWkRgiVlx0nA%3D%3D.dFVGvQ39ymMeixrbnKTWrXpsxRBy6ri7x%2B%2F6VS5nl0NRcY2mp23qrMEF2DrYy7i2) 
- 文档 [npm6 scripts](https://link.segmentfault.com/?enc=mDy03RZZCFx6PJe8uM4cxQ%3D%3D.gDzTZYOyM3e5TifQdHwYFJQXaMqLpVq9SSpeZlAMS0pGSqxJ2a572BbITboVQDrk) 

下面说明一些常用的命令

#### 一、常用命令

##### npx

`npx` 首先会去`/node_modules`中查找可执行的命令。**如果 node_modules 中没有会直接去下载（不安装）指定的包并执行**。

非常适合用来执行一些只需要一次的脚手架命令，比如`npx vue create hello world`

如果想让 npx 强制使用本地模块，可以使用`--no-install`选项，没有找到的话就会报错。

##### npm install [git_url]｜[local_file]

直接安装 git 仓库地址的包或本地文件，当然本地文件直接用`npm link`测试也可以

##### npm install 和 npm update

这里说一下`npm install`和`npm update`的区别

- **第一次安装** 

  - `npm update`不会默认安装 devDependencies 中的依赖，除非加上`--save-dev` 参数，`npm install`都会安装

- **对于已经安装的包** 

  - `npm install`会**直接重新安装一遍** dependencies 中指定的版本，支持模糊版本规则（也就是说会安装模糊版本控制支持的最新版）。并且更新 dependencies 中的版本号
  - `npm update`则会先检查 node_modules 中已经安装的依赖的版本是否低于模糊版本规则控制的版本。低于时升级来并更新 dependencies 中规定版本号，**否则什么也不做 **

  比如已经安装了`vue@^2.6.10`（最新 @2.6.11 版本），再次执行`npm install`和`npm update`效果是一样的，都会升级到 2.6.11 版。但是如果已经安装的是最新的 2.6.11 版本，那么`npm install`会再装一遍，`npm update`则什么也不做。

总的来说，直接使用`npm install`就行了。它对包其实也是执行的模糊版本规则的更新，`npm update`也只是一样的规则。(被垃圾文章误导了很久😭)

| 模糊版本控制符号 | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `^`，大版本控制  | 匹配 major 版本号下所有更新的版本，比如 ^2.0.0：2.1.0、2.2.0 ··· 2.9.0 ··· |
| ～，小版本控制   | 匹配 minor 版本号下所有更新的版本，比如 ~2.0.0：2.0.1、2.0.3 ··· |
| >=，大于等于版本 | >= 2.1.0，匹配大于等于 2.1.0 版本。包括 3.0                  |
| <=，小于等于版本 | 同上                                                         |
| 1.0.0 - 2.0.0    | 匹配1.0.0 - 2.0.0 之间的所有版本，包含 1.0.0 和 2.0.0        |



#### 二、不常用的一些命令

##### npm link

在本地开发npm模块的时候，我们可以使用npm link命令，将npm 模块链接到对应的运行项目中去，方便地对模块进行调试和测试。

**使用方法**

`npm link`

执行命令后会根据package.json上的配置，被链接到全局。被链接到的路径可以使用`npm config get prefix` 获取。

也可以指定要链接到的目录 `npm link my_project`，my_project 需要在 node 的全局环境能找到（比如被链接到了全局）

接着要去使用这个包的地方执行 `npm link [pkg_name]` 即可

**注意** 

npm link 存在一些问题，如果本地开发谨慎使用，**更推荐使用 link 工具**，简单点可以直接`npx link [path/pkgA]`  的方式。使用该工具如果要移除 link 过来的包，重新执行`npm install` 即可。

一些 `npm link` 的已知的问题如下：

- npm link 一个本地不存在的包时，会默认去服务上下载下载并 link 过去，有时候不容易注意到
- npm link 会组动执行包的 bin 字段中声明的命令，可能会带来意外的错误
- 多次使用 npm link 命令链接包时，会删除之前的。即 `npm link a`，之后 `npm link b`，最终只有 b 会被留下。要两个都加进来需要` npm link a b` ，这点有时也很迷惑

##### npm unlink

解除 npm link 的连接操作

---

##### npm view/info

查看包的信息

`npm view pkg versions` 查看包的所有远程版本

##### npm version [type] 升级当前项目版本

升级当前项目的版本并**创建 commit** 

1.1.2-1 其中 `-` 后面的是预发版本

| npm version | 功能                                                         |
| ----------- | ------------------------------------------------------------ |
| major       | - 升级大版本； - 如果有预发号则只删除预发号；                |
| minor       | - 升级中版本； - 如果有预发号则只删除预发号                  |
| patch       | - 如果没有预发布版本。直接升级小号，去掉预发布号； - 如果有预发布号：去掉预发布号，其他不动 |
| premajor    | - 直接升级大版本，中版本和小 版本置为0，增加预发布版本为0    |
| preminor    | - 直接升级中版本，小版本置为0，增加预发布版本为0             |
| prepatch    | - 直接升级小版本，增加预发布版本为0                          |
| prerelease  | - 如果没有预发布版本，增加小版本，增加预发布版本为0；  - 如果有预发布版本，则升级预发布版本 |

##### npm view [pkg] [options] 查看包信息

查看某个包的信息

- `npm view vue` 查看 vue 的相关信息，比如最新版本，发布日期等
- `npm view vue versions` 查看 vue 所有发布的版本

*还可以使用别名 `info`、`show`* 
