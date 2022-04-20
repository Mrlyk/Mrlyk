# webpack Tapable 源码解读

> 本文深入原理，来探究一下 tapable 是怎样实现了一个**优雅**的 类发布/订阅模式。

[toc]

![tapable 源码解读](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/tapable%20%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB.jpg)  

通过最简单的`SyncHook`，深入源码，我们来一步步看看 tapable 是怎么实现的，示例代码如下

```js
const { SyncHook } = require("tapable");

const hook = new SyncHook(["test1", "test2", "test3"]);

hook.tap("event1", (...restArgs) => {
  console.log("event1：", restArgs);
});
hook.tap(
  {
    name: "event2",
  },
  (...restArgs) => {
    console.log("event2：", restArgs);
  }
);

hook.call(1, 2, 3);
```

通过代码的执行顺序，一步步在源码中走下去

## new 构造钩子实例

```js
// lib/SyncHook.js 略微打乱了一点声明顺序
class SyncHookCodeFactory extends HookCodeFactory {
	content({ onError, onDone, rethrowIfPossible }) {
		return this.callTapsSeries({
			onError: (i, err) => onError(err),
			onDone,
			rethrowIfPossible
		});
	}
}

const TAP_ASYNC = () => {
	throw new Error("tapAsync is not supported on a SyncHook");
};

const TAP_PROMISE = () => {
	throw new Error("tapPromise is not supported on a SyncHook");
};

const factory = new SyncHookCodeFactory(); // 实例化一个 SyncHookCodeFactory
const COMPILE = function(options) {
	factory.setup(this, options); // 在 SyncHookCodeFactory 源码解读中说明
	return factory.create(options);
};

function SyncHook(args = [], name = undefined) {
	const hook = new Hook(args, name); // 手动实现继承，继承自 Hook 类
	hook.constructor = SyncHook; // 将构造属性指向 SyncHook，以完成继承
	hook.tapAsync = TAP_ASYNC; // 同步钩子没有 tapAsync 方法，实现上直接抛异常
	hook.tapPromise = TAP_PROMISE; // 同步钩子没有 tapPromise 方法，实现上直接抛异常
	hook.compile = COMPILE;
	return hook;
}
```

查看`SyncHokk`的源码，发现它

1. 继承自 Hook 基类
2. `compile`属性继承自`HookCodeFactory`，在后面可以看到`compile`属性用于处理钩子事件的调用（call）

tapable 中有两个非常、及其、特别重要的基类，也是 tapable 的核心，**Hook**和**HookCodeFactory** 

我们一步步来，先看看 Hook 这个基类

## Hook 基类

#### 构造函数

`SyncHook`继承自 Hook 基类，那看一下 Hook 基类的构造函数做了什么

```js
class Hook {
	constructor(args = [], name = undefined) {
    // 构造函数
		this._args = args; // 存储传入的参数到实例的 _args 属性上 ['test1', 'test2', 'test3']
		this.name = name; // 存储名称 undefied 示例代码没传
		this.taps = []; // 声明一个事件注册列表
		this.interceptors = []; // 声明一个拦截器注册列表
    
    // 下面调用方法都实现了两遍，一般来说类中带有下划线开头的属性，我们认为是私有属性
    // 这里声明两遍的作用我们继续往下看就知道
		this._call = CALL_DELEGATE; // 实现了实例的 call 方法
		this.call = CALL_DELEGATE; // 这里也是实现 call 方法
		this._callAsync = CALL_ASYNC_DELEGATE; // 下面的同上
		this.callAsync = CALL_ASYNC_DELEGATE;
		this._promise = PROMISE_DELEGATE;
		this.promise = PROMISE_DELEGATE;
    
		this._x = undefined; // 声明了一个 _x 属性？？？后面说明
    /* 基类上的 compile 属性类似于抽象属性，只声明要有，不做具体实现。
    	 由子类自己实现自己的 compile 属性
    	 SyncHook 就通过 SyncHookCodeFactory 实现了自己的 compile 属性
     */
		this.compile = this.compile; 
		this.tap = this.tap; // 实现了三个调用方法
		this.tapAsync = this.tapAsync;
		this.tapPromise = this.tapPromise;
	}
}

Object.setPrototypeOf(Hook.prototype, null); // 将原型设置为 null，防止原型上属性的影响

module.exports = Hook;
```

通过构造函数，我们知道

1. 存储了初始化钩子时的参数、name
2. 初始化了两个数组属性用于存储注册事件列表和拦截器列表
3. 注册事件的`tap/tapAsync/tapPromise`方法实现了
4. 调用的`call/callAsync/promise`也实现了

按照代码的执行顺序，我们接下来就探究一下 3、4 两点是具体如何实现的

#### tap 实现事件注册

这里先说明 tap 的实现，tapAsync 和 tapPromise 在后面说明它比普通的 tap 多做了什么

在示例代码中，我们是这样使用 tap 的

```js
hook.tap("event1", (...restArgs) => {
  console.log("event1：", restArgs);
});
```

源码中的，tap 方法同样接受两个参数，如下

```js
tap(options, fn) {
  /*
  	调用对象的 _tap 方法，同时写死第一个参数是 sync
  	options: 'event1',
  	fn: (...restArgs) => { console.log("event1：", restArgs); }
  */
  this._tap("sync", options, fn); 
}

/*
  	type: 'sync',
  	options: 'event1',
  	fn: (...restArgs) => { console.log("event1：", restArgs); }
  */
_tap(type, options, fn) {
  if (typeof options === "string") {
    options = { // 参数是字符串时转换为对象的形式 { name: 'event1 '}
      name: options.trim()
    };
  } else if (typeof options !== "object" || options === null) {
    throw new Error("Invalid tap options");
  }
  if (typeof options.name !== "string" || options.name === "") {
    throw new Error("Missing name for tap");
  }
  if (typeof options.context !== "undefined") {
    deprecateContext(); // 这个属性已经废弃了，没什么用，不用管他
  }
  // 合并参数
  // options: {fn: xxx, name: 'event1', type: 'sync' }
  options = Object.assign({ type, fn }, options); 
  options = this._runRegisterInterceptors(options); // 注册拦截器，示例代码中没有使用拦截器，所以这里什么也没做，拦截器的注册放到后面再说
  this._insert(options); // _insert 方法可以看作收集订阅者
}
```

在 tap 方法源码中可以看到 tapable

- 对参数进行了处理、合并
- 注册了拦截器
- _insert 收集订阅者

接下来就看看它是如何收集订阅者的

#### _insert 收集订阅者（事件注册）

```js
_resetCompilation() {
 // 每次注册事件的时候都对调用方法进行重置，为什么要重置呢？“懒编译”，后面说明调用事件的实现时会详细说明
  this.call = this._call;
  this.callAsync = this._callAsync;
  this.promise = this._promise;
}

/* item:
  	 type: 'sync',
  	 name: 'event1',
  	 fn: (...restArgs) => { console.log("event1：", restArgs); }
*/
_insert(item) {
  // 在 Hook 基类中，我们说到对所有的调用方法声明了两遍，为什么呢？ _resetCompilation 方法就用到了
  this._resetCompilation();
  let before;
  if (typeof item.before === "string") { // 处理我们的优先级配置
    before = new Set([item.before]); // 转成 set 类型
  } else if (Array.isArray(item.before)) { // 同时接收数组类型的 before 配置
    before = new Set(item.before);
  }
  let stage = 0; // 处理另一个优先级配置 stage
  if (typeof item.stage === "number") { // 只有 stage 是 number 时才生效，否则默认是 0 
    stage = item.stage;
  }
  let i = this.taps.length; // 在 Hook 基类中初始化为 [], 所以第一次进来时 i 是 0 
  while (i > 0) { // 第一次进来时 i 是 0，不会进这个循环，第二次注册时才会进来。里面的逻辑主要用于处理调用的优先级，即事件在 this.taps 中的顺序。这里不是核心，了解一下就好
    i--; // 每次处理完一个注册事件就 - 1
    const x = this.taps[i]; // 取到前一个注册的事件
    this.taps[i + 1] = x; // 将它复制到后面的位置
    const xStage = x.stage || 0; // 取到前一个注册事件的 stage
    if (before) { // 如果注册事件声明了 before 配置
      // 如果 before 配置的名字中存在前一个事件的名字，那当前这个事件确实应该在前一个事件之前执行
      if (before.has(x.name)) { 
        before.delete(x.name); // 把处理完的 name 删了
        continue; // 继续循环
      }
      if (before.size > 0) { // 如果 before 中没有前一个事件的名字，但是 before 还没删完，就继续找更前面的事件。就这样从后往前找
        continue;
      }
    }
    if (xStage > stage) { // 如果前一个事件的 stage > 当前的，那直接下一轮循环，大的话优先级低，就是要放在后面
      continue;
    }
    i++; // 这样一通操作下来，就把所有优先级低的事件放到自己的后面去了（这是一个很优秀的算法案例，O(n)的复杂度实现了排序）
    break;
  }
  /*
  this.taps: [
  	{fn: xxx, name: 'event1', type: 'sync' },
  	{fn: xxx, name: 'event2', type: 'sync' }
  ]
  最终将订阅者收集到 taps 数组中
  */
  this.taps[i] = item; 
}

```

至此我们示例代码已经走过了大半，钩子实例化完了，事件注册完了（订阅者收集完了）最后就只剩下调用了。接下来就继续看看调用

#### call 事件调用（发布）

在实例化 Hook 基类时，可以看到将`call`属性指向了一个函数表达式`CALL_DELEGATE`，调用委托（同样的`callAsync`和`promise`的差异在后面说明）

那这个`CALL_DELEGATE`做了什么呢？源码如下：

```js
const CALL_DELEGATE = function(...args) {
	this.call = this._createCall("sync"); // 可以看到它调用了_createCall 方法创建了要调用的方法
	return this.call(...args); // 最后调用这个方法
};
```

所以核心是`_createCall`方法，**这也是 tapable 实现的核心**

**`_createCall`** 创建新的调用函数

```js
_createCall(type) {
  return this.compile({ // 后面 HookCodeFactory 的 this.options
    taps: this.taps,
    interceptors: this.interceptors,
    args: this._args,
    type: type
  });
}
```

查看`_createCall`的源码发现它自身很简单，只是调用了`compile`方法。那这个`compile`方法又是哪来的呢？

回到最开头，在 new 构造钩子实例过程中我们就能看到，`SyncHook`在自己的构造函数中实现了 Hook 基类中的抽象`compile`方法。

```js
class SyncHookCodeFactory extends HookCodeFactory { // 继承 HookCodeFactory
	content({ onError, onDone, rethrowIfPossible }) { // 实现自己的 content 方法
		return this.callTapsSeries({
			onError: (i, err) => onError(err),
			onDone,
			rethrowIfPossible
		});
	}
}

const factory = new SyncHookCodeFactory(); // 实例化一个 SyncHookCodeFactory
const COMPILE = function(options) {
	factory.setup(this, options);
	return factory.create(options);
};
```

这里就又涉及到另一个重要的基类 **HookCodeFactory **

## HookCodeFactory 基类

#### 构造函数

HookCodeFactory 基类的构造函数特别简单，初始化了三个参数

```js
class HookCodeFactory {
  constructor(config) {
    this.config = config; // config 这个参数在初始化时并没有任何一个钩子有传入参数进来，这里应该是未删除的遗留代码？？？
    this.options = undefined;
    this._args = undefined;
  }
}
```

我们继续按照示例代码的步骤走下去，在`COMPILE`方法中，调用了`SyncHookCodeFactory`实例的`setup`方法，这个方法`SyncHookCodeFactory`自身没有实现，所以肯定是用的`HookCodeFactory`继承过来的。下面来看看`setup`这个方法

#### factory.setup 挂载响应事件到 _.x

`setup`方法传递了两个参数

- this：当前`SyncHook`实例作为函数上下文
- options：`_createCall`调用时传入的参数对象 

**options 如下** 

```js
{
  /* taps:
    [
      {fn: xxx, name: 'event1', type: 'sync' },
      {fn: xxx, name: 'event2', type: 'sync' }
    ]
 */
  taps: this.taps, 
  interceptors: this.interceptors, // []
  args: this._args, // ['test1', 'test2', 'test3']
  type: type // 'sync'
}
```

在`setup`中也只做了一件很简单的事

```js
setup(instance, options) {
  instance._x = options.taps.map(t => t.fn); // 将所有注册的事件，放到 SyncHook 的 _x 属性上
}
```

此时的`SyncHook`实例对象的属性就如下图：

![image-20220209182322625](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220209182322625.png?x-oss-process=image/resize,w_800,m_lfit) 

这里就明白了一个问题，**为什么在 Hook 基类的构造函数中声明了一个`_x`属性，就是给这里用的**。那又为什么要这么做呢？我们继续往下看

`setup`方法调用完成后接着调用了 create 方法

#### factory.create 创建调用函数

`create`方法是 tapable 核心中的核心，用于根据我们前面初始化的一系列参数，构建出一个最终供我们调用的函数

```js
init(options) {
  this.options = options; // 将参数作为自己的实例化属性，方便后面取值
  this._args = options.args.slice(); // 创建一个传入的参数副本
}

// 接收 options 参数，和 setup 的 options 参数相同
create(options) {
  this.init(options); // 初始化参数，将参数作为自己的实例化属性
  let fn;
  switch (this.options.type) { // 不同的注册方式构建新函数的方法不同
    case "sync":
      fn = new Function( // 使用函数构造器来创建新的函数
        ...
        })
      );
      break;
    case "async":
      ...
      break;
    case "promise":
      ...
      break;
  }
  this.deinit(); // 重置参数
  return fn; // 返回构建出来的新函数
}
```

在`create`方法中可以看到，tapable 最终使用函数的构造器`new Function`来构造供我们调用的最终函数。

`new Funciton(arguments, body)`接收两个参数，第一个参数作为构造出来的函数的参数，第二个参数作为函数体

`SyncHook`是同步调用函数，所以进入了同步函数的构造

```js
fn = new Function(
  this.args(), // 参数
  '"use strict";\n' + // 启用严格模式
  this.header() + // 函数体的一些变量声明，比如所有的注册事件的 fn 都在这里声明
  this.contentWithInterceptors({ // 函数体内容（包括拦截器一起），真正要执行的核心内容
    onError: err => `throw ${err};\n`,
    onResult: result => `return ${result};\n`,
    resultReturns: true,
    onDone: () => "",
    rethrowIfPossible: true
  })
);
```

在函数的构造方法中，可以看到主要通过三个方法来构造这个新的函数

- this.args：创建参数
- this.hader：创建函数体的头部内容
- this.contentWithInterceptors：创建函数体的内容，同时携带上拦截器

#### this.args 获取钩子实例参数

```js
args({ before, after } = {}) {
  let allArgs = this._args; // 在 create 一进来就 initOptions 了 _args 的值 ['test1', 'test2', 'test3']
  // Sync 没有传入 before 或者 after
  // 如果是 tapAsync 的话就会有 after，携带异步返回的 _callback 方法
  if (before) allArgs = [before].concat(allArgs); 
  if (after) allArgs = allArgs.concat(after);
  if (allArgs.length === 0) { // 没有的话返回一个空串
    return "";
  } else {
    // 有的话把数组转字符串，所以初始化的传入的参数必须是数组，不能是字符串
    return allArgs.join(", "); // "test1, test2, test3"
  }
}
```

#### this.header 创建调用函数体的头部内容

header 中主要是包含声明内容

- _context：声明上下文对象（未赋值）意义不明？？？
- _x：声明响应事件

```js
header() {
  let code = "";
  if (this.needContext()) { // 这个是将要废弃的属性，判断 tap 是否配置了 context。现在都不会配
    code += "var _context = {};\n";
  } else {
    code += "var _context;\n"; // 所以进入了这里 code: "var _context;"
  }
  code += "var _x = this._x;\n"; // _x 的作用在这里体现出来了 
  if (this.options.interceptors.length > 0) { // 判断是否有拦截器
    code += "var _taps = this.taps;\n";
    code += "var _interceptors = this.interceptors;\n";
  }
  return code;
}
```

所以最终 header 返回了如下 code

```js
var _context;
var _x = this._x;
```

而在前面我们就知道了`_x`属性中，存储的是所有注册的事件（订阅者的响应方法）

#### this.contentWithInterceptors

在上面的一系列操作后，我们拿到了最终要构建的函数的参数，拿到了所有订阅者的响应方法。接下来就是真正要执行的 content了。这里是 tapable 核心中的核心中的核心逻辑，我们最终调用的函数体也是在这里构建。

```js
/*
options：
{
	onError: err => `throw ${err};\n`,
	onResult: result => `return ${result};\n`,
	resultReturns: true,
	onDone: () => "",
	rethrowIfPossible: true
}
*/
contentWithInterceptors(options) {
  // 区分 this.options 和 options，this.options 是 call 事件调用时传入的钩子实例相关的属性
  if (this.options.interceptors.length > 0) { // 首先处理拦截器
    const onError = options.onError; // err => `throw ${err};\n`,
    const onResult = options.onResult; // result => `return ${result};\n`,
    const onDone = options.onDone; // () => ""
    let code = "";
    for (let i = 0; i < this.options.interceptors.length; i++) {
      const interceptor = this.options.interceptors[i];
      if (interceptor.call) {
        code += `${this.getInterceptor(i)}.call(${this.args({
          before: interceptor.context ? "_context" : undefined
        })});\n`;
      }
    }
    code += this.content(
      Object.assign(options, {
        onError:
        onError &&
        (err => {
          let code = "";
          for (let i = 0; i < this.options.interceptors.length; i++) {
            const interceptor = this.options.interceptors[i];
            if (interceptor.error) {
              code += `${this.getInterceptor(i)}.error(${err});\n`;
            }
          }
          code += onError(err);
          return code;
        }),
        onResult:
        onResult &&
        (result => {
          let code = "";
          for (let i = 0; i < this.options.interceptors.length; i++) {
            const interceptor = this.options.interceptors[i];
            if (interceptor.result) {
              code += `${this.getInterceptor(i)}.result(${result});\n`;
            }
          }
          code += onResult(result);
          return code;
        }),
        onDone:
        onDone &&
        (() => {
          let code = "";
          for (let i = 0; i < this.options.interceptors.length; i++) {
            const interceptor = this.options.interceptors[i];
            if (interceptor.done) {
              code += `${this.getInterceptor(i)}.done();\n`;
            }
          }
          code += onDone();
          return code;
        })
      })
    );
    return code;
  } else {
    // 没有拦截器则直接通过 SyncHook 类的 cotent 方法处理
    // content 方法则不在 HookCodeFactory 中实现，而是交由各个钩子类自己实现，所以这里就要返回到各个钩子的实现中查看
    return this.content(options); 
  }
}
```

可以看到在`contentWithInterceptors`中

- 公共的处理拦截器的相关逻辑
- 将钩子自身的调用逻辑交给钩子自己实现

继续往下走，我们回到具体的钩子类，去看看具体的钩子类做了什么

## 钩子实现（SyncHook 为例）

又回到最初的起点，最终函数体的构建交还到了各钩子自己手中。查看源码会发现，不同类型的钩子又会去调用不同的方法来实现`content`，主要三种

- 串行：`this.callTapsSeries`
- 并行：`this.callTapsParallel`
- 循环：`this.callTapsLooping`

**这里就和套娃一样，HookCodeFactory 类将实现交给了具体钩子，具体钩子则只是对调用的处理事件进行了定义，最终又回到了 HookCodeFactory** 

上面三种不同类型的钩子都依赖于基础的`this.callTap`实现（非常优雅，遵守开放封闭原则），我们还是按照代码的执行顺序，先来看看`this.callTapSeries`

#### this.callTapsSeries 串行函数实现

示例代码中的 SyncHook 最终就是由串行函数的方法来实现`content`的

```js
class SyncHookCodeFactory extends HookCodeFactory {
  /* 首先取到了 sync 类型钩子的事件处理方法
  onError：err => `throw ${err};\n`
  onDone：() => ""
  rethrowIfPossible：true
  */
  content({ onError, onDone, rethrowIfPossible }) { 
    return this.callTapsSeries({ // 调用 callTapsSeries
      onError: (i, err) => onError(err),
      onDone,
      rethrowIfPossible
    });
  }
}
```

继续进入`callTapsSeries`的源码（各注释值以示例代码为例）

```js
callTapsSeries({ // 各类处理事件交由钩子自己实现
  onError,
  onResult,
  resultReturns,
  onDone,
  doneReturns,
  rethrowIfPossible
}) {
  // this.options 还是在 Hook 己类 call 事件调用时传入的。在 factory.create 中初始化的
  if (this.options.taps.length === 0) return onDone();
  const firstAsync = this.options.taps.findIndex(t => t.type !== "sync"); // -1
  const somethingReturns = resultReturns || doneReturns; // undefined
  let code = "";
  let current = onDone; // () => ""
  let unrollCounter = 0;
  for (let j = this.options.taps.length - 1; j >= 0; j--) { // 从后往前遍历
    const i = j; // 1
    const unroll =
          current !== onDone &&
          (this.options.taps[i].type !== "sync" || unrollCounter++ > 20); // false
    if (unroll) {
      unrollCounter = 0;
      code += `function _next${i}() {\n`;
      code += current();
      code += `}\n`;
      current = () => `${somethingReturns ? "return " : ""}_next${i}();\n`;
    }
    const done = current; // () => ""
    const doneBreak = skipDone => { // 声明一个方法用来跳过 onDone 事件的执行
      if (skipDone) return "";
      return onDone();
    };
    const content = this.callTap(i, { // 这里就进入了下面的 callTap 方法
      onError: error => onError(i, error, done, doneBreak),
      onResult: // onResult 不存在时不覆写
      onResult &&
      (result => {
        return onResult(i, result, done, doneBreak);
      }),
      onDone: !onResult && done, // 不存在 onResult 时调用 done，注意这里 done 的值在上面被替换为了 current，而 current 是上一次订阅者响应时间处理后返回的函数体内容。bail 和 waterfall 类型的钩子才会对 onResult 进行实现，具体可以看每个钩子自身的实现。
      rethrowIfPossible: // sync 传进来的是 true，异步都是直接 false。这里是处理异步的逻辑。如果没有异步事件或者当前事件在异步事件之前也直接传 true，作用可以看上面的 this.callTap
      rethrowIfPossible && (firstAsync < 0 || i < firstAsync)
    });
    current = () => content; // 最终获得了要构建的函数的函数体
  }
  code += current();
  return code;
}
```

在上面的 code 生产过程中，进入到了`this.callTap`。`this.callTapSeries`则只对 next 要响应的事件进行处理，真正的 code 执行内容构建在`this.callTap`中。

#### this.callTap 

```js
callTap(tapIndex, { onError, onResult, onDone, rethrowIfPossible }) {
  let code = "";
  let hasTapCached = false;
  for (let i = 0; i < this.options.interceptors.length; i++) { // 处理拦截器
    const interceptor = this.options.interceptors[i];
    if (interceptor.tap) {
      if (!hasTapCached) {
        code += `var _tap${tapIndex} = ${this.getTap(tapIndex)};\n`;
        hasTapCached = true;
      }
      code += `${this.getInterceptor(i)}.tap(${
      interceptor.context ? "_context, " : ""
    }_tap${tapIndex});\n`;
    }
  }
  // tapIndex: 1, 从后往前处理; getTapFn 返回`_x[tapIndex]`所以这里
  // code: var _fn1 = _x[1];
  code += `var _fn${tapIndex} = ${this.getTapFn(tapIndex)};\n`; // 真正的函数执行内容
  
  
  const tap = this.options.taps[tapIndex]; // 获取当前的 tap 订阅者
  switch (tap.type) {
    case "sync": // 同步的
      if (!rethrowIfPossible) { // 如果存在异步 reject 的情况，则会加上 try catch，SyncHook 没有
        code += `var _hasError${tapIndex} = false;\n`;
        code += "try {\n";
      }
      if (onResult) {
        code += `var _result${tapIndex} = _fn${tapIndex}(${this.args({
          before: tap.context ? "_context" : undefined
        })});\n`;
      } else {
        /* code: var _fn1 = _x[1];
								_fn1(test1, test2, test3); // 给订阅者的响应事件传参
					*/
        code += `_fn${tapIndex}(${this.args({
          before: tap.context ? "_context" : undefined
        })});\n`;
      }
      if (!rethrowIfPossible) { // 异步的 try catch 并交给 hasError 标记
        code += "} catch(_err) {\n";
        code += `_hasError${tapIndex} = true;\n`;
        code += onError("_err");
        code += "}\n";
        code += `if(!_hasError${tapIndex}) {\n`;
      }
      if (onResult) { // bail、waterfall 类型的钩子将结果传递给 onResult 处理
        code += onResult(`_result${tapIndex}`);
      }
      if (onDone) {
        /* 
        获取 onDone 结果，SyncHook onDone 在第一遍进入的时候 onDone() 是空
        但是在第二次进入的时候，会将 onDone 替换为 () => content，这个 content 是上一次订阅者响应事件的处理内容。再加上 callTapSeries 是倒序处理的，自然而然的就把后面的代码放在了后面（+= 拼接）
        async 会调用 _callback()，promise 则会调用 _resolve()
        */
        code += onDone(); 
      }
      if (!rethrowIfPossible) {
        code += "}\n";
      }
      break;
    case "async":
      let cbCode = "";
      if (onResult)
        cbCode += `(function(_err${tapIndex}, _result${tapIndex}) {\n`;
      else cbCode += `(function(_err${tapIndex}) {\n`;
      cbCode += `if(_err${tapIndex}) {\n`;
      cbCode += onError(`_err${tapIndex}`);
      cbCode += "} else {\n";
      if (onResult) {
        cbCode += onResult(`_result${tapIndex}`);
      }
      if (onDone) {
        cbCode += onDone();
      }
      cbCode += "}\n";
      cbCode += "})";
      code += `_fn${tapIndex}(${this.args({
        before: tap.context ? "_context" : undefined,
        after: cbCode
      })});\n`;
      break;
    case "promise":
      code += `var _hasResult${tapIndex} = false;\n`;
      code += `var _promise${tapIndex} = _fn${tapIndex}(${this.args({
        before: tap.context ? "_context" : undefined
      })});\n`;
      code += `if (!_promise${tapIndex} || !_promise${tapIndex}.then)\n`;
      code += `  throw new Error('Tap function (tapPromise) did not return promise (returned ' + _promise${tapIndex} + ')');\n`;
      code += `_promise${tapIndex}.then((function(_result${tapIndex}) {\n`;
      code += `_hasResult${tapIndex} = true;\n`;
      if (onResult) {
        code += onResult(`_result${tapIndex}`);
      }
      if (onDone) {
        code += onDone();
      }
      code += `}), function(_err${tapIndex}) {\n`;
      code += `if(_hasResult${tapIndex}) throw _err${tapIndex};\n`;
      code += onError(`_err${tapIndex}`);
      code += "});\n";
      break;
  }
  /* 至此第一个事件就处理好了，code
  			var _fn1 = _x[1];
				_fn1(test1, test2, test3);
		 第二个事件也处理好后，code
			  var _fn0 = _x[0];
			 _fn0(test1, test2, test3);
			 var _fn1 = _x[1];
			 _fn1(test1, test2, test3);
  */
  return code;
}
```

到这一步整个`contentWithInterceptors`就处理好了，加上`use strict;`和`this.header`就是最终产生的执行函数，如下

#### 最终产物（SyncHook）

```js
var fn =  function () {
  "use strict";
  var _context;
  var _x = this._x;
  var _fn0 = _x[0];
  _fn0(test1, test2, test3);
  var _fn1 = _x[1];
  _fn1(test1, test2, test3);
}
```

所以我们使用`hook.call`最终调用的就是上面的函数。

#### 懒编译

**上面有提到为什么每次 tap 时都要重置 `call`等调用方法**，当时没有答案，现在知道了。因为我们最终调用的 fn 是调用时才产生的，而不是一开始就声明好的。这也就是一种懒编译方法。

如果我们不在每次注册事件时都重置一次`call`方法，假设又注册了一个新的 tap，那么最终生成的 fn 中是不会有这个新的 tap 的，所以每次注册都需要重新生成最终的 fn。

待说明：

- tapAsync \ tapPromise 区别
- 拦截器如何注册
- 拦截器在 contentWithInterceptors 中的实现
- 异步的实现后面有精力再补充吧