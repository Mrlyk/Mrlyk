# vue-cli 实现探究

> 本篇文章我们来探究一下 vue-cli 是怎么实现脚手架的，了解成熟的方案可以给我们更多思路。

先回忆一下我们是怎么使用 vue-cli 是怎么使用的

1. 首先我们需要安装 `@vue/cli`
2. 使用 `vue create xxx`命令启动创建
3. 根据命令行交互选择项目初始化信息
4. 完成创建，在当前目录输出初始化项目

我们就从使用入手，一步步看一下`@vue/cli`是如何运作的

[toc]

## 起步

安装我们就不赘述了，一般直接安装到全局即可。

根据我们的使用，我们首先输入到命令的是`vue create xxx`，所以我们第一步需要找到 `vue`命令的入口。在《从0开始的脚手架》(*后文中该文章里面的内容统称基础内容*)中，我们知道命令是定义在 package.json 的`bin`字段下的，所以我们先找到这个文件，看看入口是哪里。

![image-20220221170732165](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220221170732165.png?x-oss-process=image/resize,w_600,m_lfit)  

这样就找到了入口，接着进入该文件一探究竟。

#### bin/vue.js 构建入口命令

在该文件中定义了多个命令，除了我们使用`create`，还有`add`（给项目添加插件）、`serve`（启动项目）等等。文章篇幅关系，我们只说明主流程，也就是`create`。代码如下（经过精简）

```js
#!/usr/bin/env node

// 检查 node 版本，不符合条件的话直接退出
function checkNodeVersion (wanted, id) {
  // ...
}

checkNodeVersion(requiredVersion, '@vue/cli')

const fs = require('fs')
const path = require('path')
const minimist = require('minimist')
const program = require('commander') // 这里使用了 commander.js 来构建命令

// 
program
  .version(`@vue/cli ${require('../package').version}`)
  .usage('<command> [options]') // 在帮助信息的第一行中描述使用方法 

program
  .command('create <app-name>') // 构建 create 命令，下面是一系列参数
  .description('create a new project powered by vue-cli-service')
	// ...
  .option('-g, --git [message]', 'Force git initialization with initial commit message')
  .action((name, options) => {
  // 使用 minimist 来获取参数 _ 标识传入的非参数命令个数。可以看到 @vue/cli 只支持一个参数来作为项目名
  if (minimist(process.argv.slice(3))._.length > 1) {
    console.log(chalk.yellow('\n Info: You provided more than one argument. The first one will be used as the app\'s name, the rest are ignored.'))
  }
  // 是否自动初始化为 git 仓库
  if (process.argv.includes('-g') || process.argv.includes('--git')) {
    options.forceGit = true
  }
  // 最终调用 ../lib/create 方法，开始创建项目
  require('../lib/create')(name, options) 
})

/*
假设使用 vue create vue-yk -g cli:\ init 来创建
那么最后传入的参数 name: vue-yk, options: { git: 'cli: init', forceGit: true }
*/
```

在入口中即使用 commander.js 构建了`vue create xxx`命令，接下来进入`../lib/create`方法看一下后续如何处理。

#### lib/create.js 判断项目目录情况

既然上一步引入了该模块，那就先来看看这个模块是怎么导出的

```js
module.exports = (...args) => {
  return create(...args).catch(err => {
    // 把所有的异常情况都交给这里处理了，可以借鉴下，当我们调用一段复杂逻辑时最好有统一的入口。在最顶层捕获一场，防止异常抛出给用户了
  })
}
```

可以看到接收了我们所有的参数，调用了`create`方法，那 create 方法做了些什么呢？

```js
const fs = require('fs-extra')
const path = require('path')
const inquirer = require('inquirer')
const Creator = require('./Creator')
const { getPromptModules } = require('./util/createTools')
const { chalk, error, stopSpinner, exit } = require('@vue/cli-shared-utils')
const validateProjectName = require('validate-npm-package-name')

async function create (projectName, options) {
  if (options.proxy) { // 首先判断是否配置了代理参数，配置了的话就在环境变量上加上。通过代理来完成后面的网络请求，正常网络情况下是不需要了
    process.env.HTTP_PROXY = options.proxy
  }

  const cwd = options.cwd || process.cwd() // 获取当前执行路径
  // 如果传入的项目名是 '.' 的话就取当前目录名为项目名，使用 name 变量存储项目名
  const inCurrent = projectName === '.' 
  const name = inCurrent ? path.relative('../', cwd) : projectName
  // 变量存储项目创建的目标文件夹
  const targetDir = path.resolve(cwd, projectName || '.')

  const result = validateProjectName(name) // 使用 npm 的包命名检查工具检查项目名称是否合规
  if (!result.validForNewPackages) {
    // ... 不合规退出
  }

  if (fs.existsSync(targetDir) && !options.merge) { // 如果目标目录存在且不是合并文件夹的情况进入
    if (options.force) {
      await fs.remove(targetDir) // 强制删除
    } else {
      // ... 
      if (inCurrent) {
        // 如果是在当前文件夹创建项目的情况下，询问用户是否同意
        const { ok } = await inquirer.prompt([
          {
            name: 'ok',
            type: 'confirm',
            message: `Generate project in current directory?`
          }
        ])
        if (!ok) {
          return
        }
      } else {
        // 目录存在的情况下是覆盖还是合并，交由用户选择
        const { action } = await inquirer.prompt([
          {
            name: 'action',
            type: 'list',
            message: `Target directory ${chalk.cyan(targetDir)} already exists. Pick an action:`,
            choices: [
              { name: 'Overwrite', value: 'overwrite' },
              { name: 'Merge', value: 'merge' },
              { name: 'Cancel', value: false }
            ]
          }
        ])
        if (!action) {
          return
        } else if (action === 'overwrite') {
          console.log(`\nRemoving ${chalk.cyan(targetDir)}...`)
          await fs.remove(targetDir) // 覆盖的话就直接删除目标目录
        }
      }
    }
  }

  const creator = new Creator(name, targetDir, getPromptModules()) // getPromptModules 获取所有的需要交互选择的模块，比如是否配置 ts，是否配置 babel
  await creator.create(options)
}
// ...
}
```

可以看到在这里前面的一系列操作都是判断项目创建的目录情况，到最后才创建 creator 实例来进行下一步一创建。这里完成了整个起步阶段的准备工作。

下一步之后就要开始和用户交互来定制项目的信息了，比如创建 vue2 还是 vue3 项目，是否配置 typescript 等等。

## 交互（插件机制）

继续上面的步骤，先来看看  Creator 这个类的构造函数

#### lib/Creator.js constructor

构造函数肯定首先是初始化一些参数，其中有三个比较重要

- resolveIntroPrompts：创建开始前的交互提示，比如要装 vue2 还是 vue3 版本。同时会去读取 .vuerc 中配置的选项，**接收 PromptModuleAPI 实例注入的选项** 
- resolveOutroPrompts：输出文件前的交互提示，比如 babel 等配置文件是单独创建配置文件还是放在 package.json 中
- **PromptModuleAPI：这个比较重要，相当于提供了一个插件机制，让我们可以不断的扩展交互逻辑，下面重点说明，可以学习** 

```js
module.exports = class Creator extends EventEmitter {
  //接收项目名，目标文件夹，交互模块三个参数
  constructor (name, context, promptModules) {
    super()

    this.name = name
    this.context = process.env.VUE_CLI_CONTEXT = context
    const { presetPrompt, featurePrompt } = this.resolveIntroPrompts()

    this.presetPrompt = presetPrompt
    this.featurePrompt = featurePrompt
    this.outroPrompts = this.resolveOutroPrompts()
    this.injectedPrompts = []
    this.promptCompleteCbs = []
    this.afterInvokeCbs = []
    this.afterAnyInvokeCbs = []

    this.run = this.run.bind(this)

    const promptAPI = new PromptModuleAPI(this)
    promptModules.forEach(m => m(promptAPI))
  }
  // ...
}
```

**PromptModuleAPI** 

PromptModuleAPI 自身的代码并不多，可以看到他的操作都是在 creator 这个对象上。而 creator 就是 Creator 类的实例

```js
module.exports = class PromptModuleAPI {
  constructor (creator) {
    this.creator = creator
  }

  injectFeature (feature) { // 给实例的 featurePrompt choices 传入选项，featurePrompt 是预设好的特性，有默认的比如 babel。也可以自己传入新的
    this.creator.featurePrompt.choices.push(feature)
  }

  injectPrompt (prompt) { // 给实例的 injectedPrompts 传入选项，featurePrompt 是第一级，这里就是第二级。通过 inquirer.js 的 when 选项来控制
    this.creator.injectedPrompts.push(prompt)
  }

  injectOptionForPrompt (name, option) { // 给上面注入的选择类型的交互注入选项
    this.creator.injectedPrompts.find(f => {
      return f.name === name
    }).choices.push(option)
  }

  onPromptComplete (cb) { // 在交互结束后执行的回调
    this.creator.promptCompleteCbs.push(cb)
  }
}
```

有了这个扩展交互的方法之后，@vue/cli 通过这个方式来扩展配置。我们以 babel 举例。

#### 扩展配置

在 Creator 的构造函数中，有这样一段

```js
const promptAPI = new PromptModuleAPI(this)
promptModules.forEach(m => m(promptAPI))
```

上面我们了解了 PromptModuleAPI 是什么。这个 promptModules 又是什么呢？很简单，就是我们提示用户要配置的模块。babel 就是其中一个，看看 babel 这个 promptModule 里面是什么样的

```js
module.exports = cli => { // cli 就是 PromptModuleAPI 实例
  cli.injectFeature({ // 注入特性，babel 是自动配置的，当然最后会更具用户的选择来配置是否用 babel 插件
    name: 'Babel',
    value: 'babel',
    short: 'Babel',
    description: 'Transpile modern JavaScript to older versions (for compatibility)',
    link: 'https://babeljs.io/',
    checked: true
  })

  cli.onPromptComplete((answers, options) => { 
    if (answers.features.includes('ts')) {
      if (!answers.useTsWithBabel) {
        return
      }
    } else if (!answers.features.includes('babel')) {
      return
    }
    options.plugins['@vue/cli-plugin-babel'] = {}
  })
}
```

*这里我们还可以看到，这个类继承自 node 的 事件系统，所以我们可以猜测这个类还可以用来分发事件和通信。（事实上是在使用 vue ui 创建项目时，会创建一个服务端来接收信息，触发事件，这里就不展开说了）* 

## 创建

回到主流程上，在起步的 create.js 中调用了 Creator 实例的 create 方法。所以继续来看看这个 create 方法做了什么。

#### lib/Creator.js create

同样代码经过精简和注释

```js
/**
* create 接收两个参数，在使用 vue create 的时候第二个参数 null 
* @param { object } cliOptions - vue create 时的选项
* @param { * } preset vue cli 预设，一个包含创建项目所需要的插件、配置等的 JSON 对象。用户无需在命令提示中选择他们
*/
async create (cliOptions = {}, preset = null) { 
  const { run, name, context, afterInvokeCbs, afterAnyInvokeCbs } = this

  if (!preset) { // 创建时没有配置预设，所以会进这里
    if (cliOptions.preset) { // vue create xxx -p
      // 加载预设，比如 vue2 vue3 的选择
      preset = await this.resolvePreset(cliOptions.preset, cliOptions.clone)
    } else if (cliOptions.default) { // vue create xxx --default
      // 默认配置 vue3 的预设
      preset = defaults.presets['Default (Vue 3)']
    } else if (cliOptions.inlinePreset) { // vue create foo --inlinePreset {...}
      try {
        // 手动配置行内预设
        preset = JSON.parse(cliOptions.inlinePreset)
      } catch (e) {
        error(`CLI inline preset is not valid JSON: ${cliOptions.inlinePreset}`)
        exit(1)
      }
    } else {
      // 核心1: 插件机制的执行入口，通过和用户交互来生成预设配置
      preset = await this.promptAndResolvePreset()
    }
  }
  // 改变指向，不影原对象
  preset = cloneDeep(preset)
  // 自动配置开发服务插件
  preset.plugins['@vue/cli-service'] = Object.assign({
    projectName: name
  }, preset)

  // ...根据选项配置一些插件，如 @vue/cli-plugin-router

  // 配置默认的包管理工具
  const packageManager = (
    cliOptions.packageManager ||
    loadOptions().packageManager ||
    (hasYarn() ? 'yarn' : null) ||
    (hasPnpm3OrLater() ? 'pnpm' : 'npm')
  )

  await clearConsole()
  const pm = new PackageManager({ context, forcePackageManager: packageManager })

  // ...

  // 生产默认的 package.json 配置文件
  const pkg = {
    name,
    version: '0.1.0',
    private: true,
    devDependencies: {},
    ...resolvePkg(context)
  }
  const deps = Object.keys(preset.plugins)
  // ...将预设插件写入 package.json

  // 之后通过 fs 写入硬盘
  await writeFileTree(context, {
    'package.json': JSON.stringify(pkg, null, 2)
  })

  // ...

  // 通过上面的包管理工具安装依赖
  await pm.install()

  // 核心2：通过 Generator 类实例生产最终配置文件
  const plugins = await this.resolvePlugins(preset.plugins, pkg)
  const generator = new Generator(context, {
    pkg,
    plugins,
    afterInvokeCbs,
    afterAnyInvokeCbs
  })
  // 到 generate 执行完成，整个项目就已经生成了
  await generator.generate({
    extractConfigFiles: preset.useConfigFiles
  })
  // 再次执行安装
  await pm.install()


  // 最后执行插件绑定的一些回调
  for (const cb of afterInvokeCbs) {
    await cb()
  }
  for (const cb of afterAnyInvokeCbs) {
    await cb()
  }
  // ...


  generator.printExitLogs()
}
```

在 create 中主要是两个东西要说一下

- 插件机制的执行过程
- Generator 类

#### 插件机制的实现

其实这个应该是上一节的内容，但是在 create 中才真正执行，所以放到这里来说

![image-20220224155629313](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220224155629313.png?x-oss-process=image/resize,w_1200,m_lfit)  

**vue cli 将交互类型分为四类，通过插件机制（`PromptModuleAPI`）暴露出去。在 `Creator` 实例化时插入到不同类型的 prompt 中，并且注册各自的回调函数。这样的设计极大的解耦了各项 prompt，后期可以随意的扩展或删除插件。** 

#### lib/Generator.js 

构造函数本身就不赘述了，依然是初始化一些实例属性

`generate`本身逻辑很短，将需要做的事都拆开放在外面了，我们一步步来看一看。

```js
async generate ({
  extractConfigFiles = false,
  checkExisting = false
} = {}) {
  /** 这个方法主要是对每个插件注入了一个`GeneratorAPI`实例
  * 1、生成该 plugins 的配置文件并通过`GeneratorAPI`中的`extendPackage`写入 pkg 对象。
  * 2、有的插件写进去之后可能需要执行一些初始化的方法，也在这里配置进去，在后面安装完插件后执行
  */
  await this.initPlugins()

  // 初始化要写入的文件
  const initialFiles = Object.assign({}, this.files)
  // 将上面生成的配置抽取到单独的配置文件中（如果配置了的话）
  this.extractConfigFiles(extractConfigFiles, checkExisting)
  // 通过 ejs 渲染要输出的文件数据，使用了 GeneratorAPI 中的 render 方法读取 generator 模版（模版文件和 cli 工具同级）
  await this.resolveFiles()
  this.sortPkg()
  this.files['package.json'] = JSON.stringify(this.pkg, null, 2) + '\n'
  // 根据上面渲染的文件模版，将文件写入硬盘，到这一步其实就已经基本完成了
  await writeFileTree(this.context, this.files, initialFiles, this.filesModifyRecord)
}
```

其中的核心在 `GeneratorAPI`和`resolveFiles`中

一个用来处理插件的配置，一个用来处理文件的生成。

## 感受

1. 充分考虑用户意图，比如用户输入`.`的情况虽然可以自动帮用户在当前目录创建，但是防止用户不明白后果提示用户更好
2. 在起步阶段要考虑目标文件夹的各类情况
2. 插件架构能很好的扩展我们的工具能力，在写工具时可以考虑在什么时候可以暴露出方法以方便扩展

## 参考文章

剖析 Vue CLI 实现原理：https://cloud.tencent.com/developer/article/1781202