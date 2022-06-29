

# ESLint 源码探究

> ESLint 作为能自动查找、修复代码格式问题的工具，其核心流程是怎么样的呢？本文会做对其核心流程的源码做一个简要的说明。让我们更好的了解 ESLint，同时学习一下他的思路。

说到代码检测修复，又要回到老生常谈的 AST，可以这么说：**前端工具，无论是打包的 webpack、编译的 babel，还是这里要说明的 ESLint。只要涉及到源码层面的操作，肯定都是通过操作 AST 来处理的。**

AST 这里就不再赘述了，ESLint 使用的工具是 [espree](https://github.com/eslint/espree) 来生成 AST。

```text
espress 最早是从 esprima 上 fork 出来的。最新版本基于 Acorn，生成的 AST 结构和 esprima 是相同的，API 也相同。主要使用 parse() 和 tokenize() 这两个 API 来生成 AST 和词法单元（token）。
```

ESLint 整体的执行流程如下：

```mermaid
graph LR
A(读取配置)-->B(加载插件)
B-->C(解析获取 AST)
C-->D(遍历 AST 收集节点)
D-->E(注册规则, 注入 context)
E-->F(遍历收集到的节点, 获取所有 lint 问题)
F-->G(根据注释过滤)
G-->H(修复)
```

接下来就根据流程一步步看看 ESLint 做了什么

[toc]

## 读取配置

众所周知，ESLint 的配置有两种方式

1. `.eslintrc.js(on)?`文件，**优先级比 package.json 中的更高**
2. packge.json 中的`eslintConfig`字段

除了具体的规则配置之外，还要读取 cli 命令的参数，比如`eslint --fix`，就需要存储`fix`这个参数来决定后面的具体操作。cli 的参数读取借助`optionator`库（代码在`eslint/lib/cli.js`中）

ESLint 的第一步就是要读取用户配置，读取的规则很简单，主要遵循以下几点：

1. 从当前检测的文件目录开始，向父级查找离文件最近的配置文件，一直到根目录。最终合并成一个配置，**离检测的文件最近的配置文件优先级更高**
2. `root: true`如果配置了该项，那么就会在这一层停止寻找

#### 相关代码如下

**下面的代码不在 eslint 的源码中，而是`eslintrc`这个依赖中**。这个依赖专门用来给 eslint 处理配置

```js
try {
  configArray = configArrayFactory.loadInDirectory(directoryPath);
} catch (error) {
  throw error;
}

// 这里如果添加了 root 字段将会中断向外层遍历的操作
if (configArray.length > 0 && configArray.isRoot()) {
  configArray.unshift(...baseConfigArray);
  return this._cacheConfig(directoryPath, configArray);
}

// ....
const configFilenames = [
  .eslintrc.js ,
  .eslintrc.cjs ,
  .eslintrc.yaml ,
  .eslintrc.yml ,
  .eslintrc.json ,
  .eslintrc ,
  package.json 
];

loadInDirectory(directoryPath, { basePath, name } = {}) {
  const slots = internalSlotsMap.get(this);
  // 这里是以 configFilenames 数组中元素的顺序决定优先级的，所以优先级是 js 的最高 
  for (const filename of configFilenames) {
    const ctx = createContext();
    if (fs.existsSync(ctx.filePath) && fs.statSync(ctx.filePath).isFile()) {
      let configData;
      try {
        configData = loadConfigFile(ctx.filePath);
      } catch (error) {
      }
      if (configData) { // 一旦匹配到一个就直接结束
        return new ConfigArray(
          ...this._normalizeConfigData(configData, ctx)
        );
      }
    }
  }
  return new ConfigArray(); // 否则返回一个空的配置
}
```

其中还会有一些细节，如转换配置差异：在单独的配置文件中`globals`是对象，在 packages.json 中`globals`是数组，所以其中还会有一些抹平这种差异的操作。 

## 加载插件

在读取配置的源码中，关键的一个方法是`_normalizeConfigData`，这个方法里面是一系列的递归调用

`_normalizeConfigData`（校验参数合法性） -> `_normalizeObjectConfigData`（标准化对象配置） -> `_normalizeObjectConfigDataBody`（**加载 extends、parser、plugins，都是从前往后遍历，最终再合并**）

#### extends

源码如下

```js
_loadExtends(extendName, ctx) {
  debug("Loading {extends:%j} relative to %s", extendName, ctx.filePath);
  try {
    if (extendName.startsWith("eslint:")) { // 首先判断是不是以 eslint 开头的内建的，比如 eslint:recommend
      return this._loadExtendedBuiltInConfig(extendName, ctx);
    }
    // 加载 plugin 格式的，比如 plugin:vue/essential，同时会取出 vue 作为 pluginName，essential 作为 configName
    if (extendName.startsWith("plugin:")) { 
      return this._loadExtendedPluginConfig(extendName, ctx);
    }
    // 最后会去加载 eslint 的插件，比如 eslint-config-standard，如果只写了 standard 会自动拼上 eslint-config-
    return this._loadExtendedShareableConfig(extendName, ctx);
  } catch (error) {
    error.message += `\nReferenced from: ${ctx.filePath || ctx.name}`;
    throw error;
  }
}
```

**`_loadExtendedShareableConfig` 同时要特别说明这个方法**

我们知道 extends 其实就是别人封装好的 eslint 配置文件，所以我们需要像读取我们的配置文件一样去读取其中的配置。

在`_loadExtendedShareableConfig`中最后会调用`loadConfigData`方法，而这个方法最终又回到了我们递归调用的开头`_normalizeConfigData`这个方法，达成了递归。直到所有的配置都被读取完毕。

这里有一个疑问就是如果这样读取，最后 extends 的配置会覆盖我们的？**因为最终的 ConfigArray 是 [extends 的配置，我们的配置] 这样？**

实际在 eslint 命令执行的时候会调用`ConfigArrat`中的的`extractConfig`合并配置

`ConfigArray`继承自`Array`

```js
extractConfig(filePath) {
  const { cache } = internalSlotsMap.get(this);
  const indices = getMatchedIndices(this, filePath);
  const cacheKey = indices.join(",");

  if (!cache.has(cacheKey)) { // 已经读取过的配置不再重复读取并且创建缓存
    cache.set(cacheKey, createConfig(this, indices)); // 最终在 createConfig 中完成合并
  }

  return cache.get(cacheKey);
}

function getMatchedIndices(elements, filePath) {
    const indices = [];
    for (let i = elements.length - 1; i >= 0; --i) { // 这里是逆序遍历的，而我们的配置在最后，所以我们的配置会被最先放进去，优先级也最高
        const element = elements[i];
        if (!element.criteria || (filePath && element.criteria.test(filePath))) {
            indices.push(i);
        }
    }
    return indices;
}
```

最终使用`createConfig`方法完成合并

- 对于 parser 和 processor 字段，后面的配置文件会覆盖前面的配置文件。

- 对于 env，globals，parserOptions，settings 字段则会合并在一起，但是这里注意，只有当后面的配置里存在前面没有的字段时，这个字段才会被合并进来，如果前面已经有了这个字段，那后面的相同字段会被摒弃。

- - 例如 [{a: 1, b: 2}, {c: 3, b: 4}] 这个数组的合并结果则是 {a: 2, b: 2, c: 3}。

- 对于 rules 字段，同样是前面的配置优先级高于后面的，但是如果某个已存在的 rule 里带了参数，那么 rule 的参数会被合并。

上面两步都是读取配置的过程，到这里真正读取所有配置的流程结束

**总结一下读取配置流程**

1. 读取当前命令执行目录下的配置文件
2. 读取配置文件中的 extends
3. 完成当前目录下的配置读取
4. 查询 root 是否 true，是的话读取完成，不是的话继续读取外层配置

接下来是执行检验和修复的过程吗，依然按照流程图一步步来

## 解析获取 AST

到检验开始就要对我们的源码进行操作了，上面主要是依赖于`eslintrc`这个依赖，这里就回到了我们`eslint`本身。

获取 AST 要分两步走

1. 使用预处理器处理非 js 文件（如果需要的话）
2. 确定解析器

*源码就是 Eslint 处理的当前文件*

#### 预处理器

插件可以提供处理器。处理器可以从其他类型的文件中提取 JavaScript 代码，然后让 ESLint 对 JavaScript 代码进行 lint，或者处理器可以出于某种目的在预处理中转换 JavaScript 代码。

通常我们**使用 preprocess 从非 js 文件里提取出需要被检验的部分 js 代码**，使得非 js 文件也可以被 ESLint 检验。而 postprocess 则是可以在文件被检验完之后对所有的 lint problem 进行统一处理。

#### 确定解析器

ESLint 默认使用 espree，但是我们也可以手动指定解析器。其源码如下

```js
let parser = espree;

if (typeof config.parser === "object" && config.parser !== null) {
  parserName = config.parser.filePath;
  parser = config.parser.definition;
} else if (typeof config.parser === "string") {
  if (!slots.parserMap.has(config.parser)) {
    return [{
      ruleId: null,
      fatal: true,
      severity: 2,
      message: `Configured parser '${config.parser}' was not found.`,
      line: 0,
      column: 0
    }];
  }
  parserName = config.parser;
  parser = slots.parserMap.get(config.parser);
}
```

确定了解析器之后，就会调用`parse`方法将源码转换为 AST

```js
const parseResult = parse(
  text,
  languageOptions,
  options.filename
);

// parse 方法中
const ast = parseResult.ast;
```

之后就进入了正式的 lint 流程

## 遍历 AST 收集节点

lint 开始会先去获取配置中的 rules

```js
let lintingProblems;

try {
  lintingProblems = runRules(
    sourceCode,
    configuredRules,
    ruleId => getRule(slots, ruleId), // 获取配置中 rules
    parserName,
    languageOptions,
    settings,
    options.filename,
    options.disableFixes,
    slots.cwd,
    providedOptions.physicalFilename
  );
} catch(e) {}
```

最终执行的是` runRules `方法，在这个方法中遍历 AST 收集了所有的 node

```js
Traverser.traverse(sourceCode.ast, {
  enter(node, parent) {
    node.parent = parent;
    nodeQueue.push({ isEntering: true, node }); // 收集所有的 node ，后面遍历处理
  },
  leave(node) {
    nodeQueue.push({ isEntering: false, node });
  },
  visitorKeys: sourceCode.visitorKeys
});
```

收集到节点之后就可以对不同规则使用不同的处理方式

## 注册 rules

在`runRules`中遍历完 AST 之后，就会遍历所有的 rules，并且绑定 rule 类的`create`方法，是个命令模式和发布订阅模式的混合

```js
const ruleListeners = createRuleListeners(rule, ruleContext); // 注册所有配置了的 rule 的监听，调用规则的 create 方法检查

// 收集订阅者
Object.keys(ruleListeners).forEach(selector => {
  const ruleListener = timing.enabled
  ? timing.time(ruleId, ruleListeners[selector]) // 计算性能的，不重要
  : ruleListeners[selector]; // 真正的监听

  emitter.on(
    selector,
    addRuleErrorHandler(ruleListener)
  );
});
```

## 触发 emit 获取 lint 问题

上面的一系列准备工作完成后，就回去遍历开头收集到的源码的所有 AST 节点，emit 出问题

```js
nodeQueue.forEach(traversalInfo => {
  currentNode = traversalInfo.node;

  try {
    if (traversalInfo.isEntering) {
      eventGenerator.enterNode(currentNode); // 触发 applySelector
    } else {
      eventGenerator.leaveNode(currentNode); // 触发 applySelector
    }
  } catch (err) {
    err.currentNode = currentNode;
    throw err;
  }
});

applySelector(node, selector) {
  if (esquery.matches(node, selector.parsedSelector, this.currentAncestry, this.esqueryOptions)) {
    this.emitter.emit(selector.rawSelector, node); // 最终将问题抛出去
  }
}
```

至于怎么定性问题，eslint 对不同的问题定义了不同的方法，可以查看` ESLINT/lib/rules` 下面的具体定义。

下面随便举一个规则的例子`no-new-object`禁止用 new 操作符号声明 Object 对象

```js
create(context) {
  return {
    NewExpression(node) { // ast new 表达式的 visitor
      const variable = astUtils.getVariableByName(
        context.getScope(),
        node.callee.name
      );

      if (variable && variable.identifiers.length > 0) {
        return;
      }

      if (node.callee.name === "Object") { // 如果表达式的名字是 Object 则 report 问题
        context.report({
          node,
          messageId: "preferLiteral"
        });
      }
    }
  };
}
```

根据注释过滤就不细述了，就是个简单的逻辑判断而已，具体在`applyDisableDirectives`这个方法中。

**至此所有`verify`（校验）的操作就结束了，接下来是`fix`** 

## 修复

获取到所有的问题之后就可以进行修复了，修复也是让我最疑惑的点。通过 AST 怎么判断`=`号前是否有空格呢？根据索引吗？

看看 eslint 源码中是如何实现的

**入口**

```js
messages = this.verify(currentText, config, options); // 上面遍历 AST 等一系列操作都在这个方法中

fixedResult = SourceCodeFixer.applyFixes(currentText, messages, shouldFix); // 这里就开始修复了
```

入口方法还是`verifyAndFix`方法中，先`verify`再`fix`

其实最后`applyFixes`的方法很简单，比我们想的要简单。ESLint 直接对不符合的 rules 能处理的就直接执行**替换操作**。

**核心**

```js
function attemptFix(problem) {
  const fix = problem.fix; // problem.fix 描述修复命令的对象 { range: Number [], test: String }
  const start = fix.range[0];
  const end = fix.range[1];

  // 防止两处修复有重叠范围，有的话就暂存等后面再修
  if (lastPos >= start || start > end) {
    remainingMessages.push(problem);
    return false;
  }

  // 这里的 BOM 是值 Byte Order Mark 字节顺序标记，如果有的话会把他移除，utf-8 编码不需要他
  if ((start < 0 && end >= 0) || (start === 0 && fix.text.startsWith(BOM))) {
    output = "";
  }

  // Make output to this fix.
  output += text.slice(Math.max(0, lastPos), Math.max(0, start));
  output += fix.text; // 直接用 fix.text 替换
  lastPos = end;
  return true;
}
```

最终的修复方式是由各个 rules 定义的，他们会返回修复对象。

在 rules 的`meta`中一般会有`fixable`标识是否可修复，可以话在 rule 的`create`方法中也会返回这个 fix 对象

以 newline-after-var 举例

```js
context.report({
  node,
  messageId: "unexpected",
  data: { identifier: node.name },
  fix(fixer) {
    const linesBetween = sourceCode.getText().slice(lastToken.range[1], nextToken.range[0]).split(astUtils.LINEBREAK_MATCHER);

    return fixer.replaceTextRange([lastToken.range[1], nextToken.range[0]], `${linesBetween.slice(0, -1).join("")}\n${linesBetween[linesBetween.length - 1]}`); // 最终返回修复方式
  }
});
```

如果我们要自己编写 eslint 的规则插件的话并且要修复的话就要以这种方式来处理。

## 总结

整个 ESlint 的修复流程总的来说就是三步

1. 收集合并配置，当前目录配置文件的，外层目录配置文件的....
2. 源码转 AST ，收集所有 AST 节点
3. 遍历收集的节点，使用 rules 中定义的规则来直接替换以修复源码
