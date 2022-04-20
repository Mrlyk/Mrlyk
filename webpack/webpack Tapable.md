# webpack Tapable

> webpack 中最核心的库

官方仓库地址：https://github.com/webpack/tapable

webpack 可以说是由插件构建起来的工具，而插件的实现就依赖于本文所说的 tapable。

[toc]

## 什么是 tapable？

直接查看 tapable 源码，看到最新的 tapable 导出了 10个 hook 的 class 类

```text
The tapable package expose many Hook classes, which can be used to create hooks for plugins.
```

官方介绍的第一句话也说明了，tapable 导出了许多的 hook 类，以供 plugins 调用。

在 webpack 的编译过程中，通过 tapable 实现了在编译过程中的一种“类发布订阅模式”（说是类发布订阅是因为发布订阅本质是一种松散耦合的设计模式，但 webpack 的钩子体系是一种强耦合架构，在触发钩子时会附带足够的上下文信息，让我们在回调中能与上下文数据结构交互，产生 side effect）。

总之 tapable 就是一个提供了各种钩子类，以让 webpack 和我们在各个 plugin 中注册 hooks 方法的工具。

最新的 tapable 提供了 10 个 hook 类如下

```js
const {
	SyncHook,
	SyncBailHook,
	SyncWaterfallHook,
	SyncLoopHook,
	AsyncParallelHook,
	AsyncParallelBailHook,
	AsyncSeriesHook,
	AsyncSeriesBailHook,
  AsyncSeriesLoopHook, // 比一般的说明文档又新增了一个这个
	AsyncSeriesWaterfallHook
 } = require("tapable");
```

他们都继承自 Hook 类（使用 ES5 做的继承，具体可以查看源码）。下面从使用入手看看这些钩子是如何使用的！

## tapable 使用

以最简单的`SyncHook`为例

```js
const { SyncHook } = require("tapable");
/**
 * SyncHook(args?: Array<any>, name?: string)
 * 构造函数接收两个参数
 * 第一个是参数数组，用于后面创建新的调用函数
 * 第二个是 name，在后面讲到拦截器的时候，可以通过 name 来区分事件
 *  */
const hook = new SyncHook(["test1", "test2", "test3"]);

/**
 * tap(options: string | { name: string }, fn)
 * 方法做三件事：
 * 1.合并参数，这里的第一个参数没有实际意义，仅仅作为一个标识
 * 2.注册拦截器： this._runRegisterInterceptors
 * 3.注册事件：_insert 将合并后的对象放入事件队列 this.taps
 */
hook.tap("event1", (...restArgs) => {
  console.log("event1：", restArgs);
});
hook.tap("event2", (...restArgs) => {
  console.log("event2：", restArgs);
});

/**
 * call()
 * 1.在 HookCodeFactory 调用 create，用 tap 传入的 fn 和初始化 hook 时传入的参数等构造了新的函数
 * 新的函数中，会把放入 taps 的 fn 全部放入函数体内一一调用（taps.map(t => t.fn)）
 * 2.调用
 * new SyncHook 时只传入了 3 个参数，所以新函数也只接收三个参数，多了不收
 */
hook.call(1, 2, 3);

```

以上就是 SyncHook 的使用，步骤很简单可以参照注释。钩子有点像 node 中的 EventEmitter，先注册再调用。其他钩子的基本使用方式也类似。

正如 hook 的名称中带有 sync 一样，tapable 中的钩子根据名称区分为同步钩子和异步钩子。

- 同步钩子：
- 异步钩子：

#### 同步钩子

> 注册的事件同步执行

同步钩子注册事件只有一个方法`tap`，调用则通过`call`

#### 异步钩子

> 异步回调，是异步串行还是并行取决于钩子的种类。串行的会等待前面的钩子回调，并行的则不会

异步钩子通过`tap`、`tapAsync`、`tapPromise`三种方式来注册（后两种注册方式在同步钩子中直接抛出异常），同样的通过对应的`call`、`callAsync`、`promise`三种方式调用

异步钩子还可以细分为

- 异步串行钩子(`AsyncSeries`)：按照顺序连续调用的异步钩子
- 异步并行钩子(`AsyncParallel`)：并发调用的异步钩子

#### 钩子执行机制

钩子存在不同的执行机制

- Basic Hook：基本类型的钩子，仅仅执行注册的 fn，不关心 fn 的返回值
- Waterfall：瀑布类型的钩子，**会将上一个 fn 的返回值传给下一个 fn，并且替换掉下一个 fn 的第一个参数**，`${this._args[0]} = ${result};\n` 
- Bail：保险类型的钩子，当 fn **有返回值**（返回的不是`undefined`）时会终止后面的 fn 执行并直接结束
- Loop：循环类型的钩子，当 fn **有返回值**（返回的不是`undefined`）时会从头开始执行，直到所有的 fn 都返回`undefined`（相当于要把目标对象都处理完）

哪种钩子使用哪种机制看钩子的名称就知道了，比如`SyncWaterfallHook`就是一个同步的瀑布类型钩子。

下面列举几种钩子的小 demo

#### SyncWaterfallHook 同步瀑布钩子

```js
const { SyncWaterfallHook } = require("tapable");

const mySWH = new SyncWaterfallHook(["arg1", "arg2"]);

mySWH.tap("e1", (arg1, arg2) => {
  console.log("e1: ", arg1, arg2);
  return {name: 'lyk'}
});
mySWH.tap("e2", (...restArgs) => {
  console.log("e2: ", restArgs); // 将上一个 fn 的返回值作为这里的第一个参数 this.args[0] = result
});

mySWH.call('1', '2') 
// e1:  1 2
// e2:  [ { name: 'lyk' }, '2' ] 
```

#### SyncLoopHook 同步循环钩子

```js
const { SyncLoopHook } = require("tapable");

const mySLH = new SyncLoopHook(["arg1", "arg2"]);
let flag = 0;

mySLH.tap("e1", (arg1, arg2) => {
  console.log("e1: ", arg1, arg2);
  if (flag !== 2) {
    return flag++; // 有返回值时就从头开始执行
  }
});
mySLH.tap("e2", (...restArgs) => {
  console.log("e2: ", restArgs);
});

mySLH.call("1", "2");
/*
e1:  1 2
e1:  1 2
e1:  1 2
e2:  [ '1', '2' ]
*/
```

#### AsyncSeriesHook 异步串行钩子

异步：异步表示回调可以在稍后进行，后面的事件会在回调后触发

串行：顾名思义就是从前到后顺序执行，前面没走完不会走到后面

```js
const { AsyncSeriesHook } = require("tapable");

const myASH = new AsyncSeriesHook(["arg1", "arg2"]);

// tapAsync 携带第三个参数 callback，调用后继续或中断执行
myASH.tapAsync("e1", (arg1, arg2, callback) => { 
  console.log("e1: ", arg1, arg2);
  setTimeout(() => {
    callback(); // 回调方法和 node 中的事件类似，第一个参数是 Error 实例，如果 callback(err)则会报错，中断执行
  }, 1000);
});
// tapPromise 不需要传入回调方法，但是一定要返回一个 promise 对象，根据 promise 的状态来继续或中断执行
myASH.tapPromise("e2", (...restArgs) => { 
  console.log("e2: ", restArgs);
  return new Promise((res) => {
    setTimeout(() => {
      res();
    }, 1000);
  });
});

console.log("start");
// 普通的串行钩子不关心返回值，所以也没有任何参数
myASH.callAsync("1", "2", () => { // 串行执行完成周调用该方法（通过构造调用的新函数时 concat 进来的，allArgs.concat(after)），如果其中一个事件报错则会跳过后续事件执行，直接执行该方法
  console.log("This is callback");
});
console.log("end");
/*
start
e1:  1 2
end
// 等待 1 秒
e2:  [ '1', '2' ]
// 又等待 1 秒
This is callback
*/
```

可以看到钩子是一个个串行执行的，上一个执行完了再去执行下一个，有点类似 promise 的链式调用，最后总会调用`callAsync`传入的回调。

**如果是通过 hook 的 promise 调用** 

```js
myASH.promise("1", "2", () => {
  console.log("This is callback");
});
// 也可以成功调用，但是不需要第三个回调方法，传入了也没用（传入不会报错，但是 callAsync 就必传该回调）
```

#### AsyncSeriesBailHook 异步串行保险钩子

保险：存在返回值时会终止执行

异步的可以通过`promise.resolve()`或者`callback(null, result)`的第二个参数返回值

```js
const { AsyncSeriesBailHook } = require("tapable");

const myASB = new AsyncSeriesBailHook(["arg1", "arg2"]);

myASB.tapAsync("e1", (arg1, arg2, callback) => {
  console.log("e1: ", arg1, arg2);
  setTimeout(() => {
    callback(null, '111'); // 返回了值且不是 undefined，在 callback 调用后，直接调用 callAsync 传入的回调，之后结束
  }, 1000);
});

myASB.tapPromise("e2", (...restArgs) => {
  console.log("e2: ", restArgs);
  return new Promise((res, rej) => {
    setTimeout(() => {
      res('222');
    }, 1000);
  });
});


console.log("start");
myASB.callAsync("1", "2", (..restArgs) => {
  console.log("This is callback");
  console.log(restArgs); // [null, '111']，如果存在第一个参数（不是 null | undefined）则只会返回第一个参数，否则会返回如上参数。通过 promise.resolve 返回参数相当于第一个参数是 null。返参和上面相同
});
console.log("end");
/*
start
e1:  1 2
end
// 等待一秒后
[ null, '111' ]
This is callback // 由于第一个事件有返回值，这里直接调用，跳过了第二个事件的执行
*/
```

#### AsyncParallelHook 异步并行钩子

并行：两个注册事件同时执行，就像同时发送了异步请求一样。但是等到所有请求都返回了才会执行最后的回调。也不关心回调参数。类似于`promise.all` 

```js
const { AsyncParallelHook } = require("tapable");

const myAPH = new AsyncParallelHook(["arg1", "arg2"]);

myAPH.tapAsync("e1", (arg1, arg2, callback) => {
  console.log("e1: ", arg1, arg2);
  setTimeout(() => {
    callback();
  }, 500);
});

myAPH.tapPromise("e2", (...restArgs) => {
  console.log("e2: ", restArgs);
  return new Promise((res, rej) => {
    setTimeout(() => {
      res('123');
    }, 2000);
  });
});


console.log("start");
myAPH.callAsync("1", "2", (...restArgs) => {
  console.log(restArgs);
  console.log("This is callback");
});
console.log("end");
/*
start
e1:  1 2
e2:  [ '1', '2' ]
end
// 等待2秒后 （最晚的平行钩子返回了，类似于 promise.all）
[]
This is callback
*/
```

#### AsyncParallelBailHook 异步并行保险钩子

异步并行的保险钩子和普通钩子区别就是并行的一定会调用所有事件，所以每个事件中的同步行为都会被调用，保险的存在会让某个事件存在返回值后直接就去调用结束的回调。类似于`promise.race` 

```js
const { AsyncParallelBailHook } = require("tapable");

const myAPBH = new AsyncParallelBailHook(["arg1", "arg2"]);

myAPBH.tapAsync("e1", (arg1, arg2, callback) => {
  console.log("e1: ", arg1, arg2);
  setTimeout(() => {
    callback(null, '111');
  }, 200);
});

myAPBH.tapPromise("e2", (...restArgs) => {
  console.log("e2: ", restArgs);
  return new Promise((res, rej) => {
    setTimeout(() => {
      res('222');
    }, 2000);
  });
});

console.log("start");
myAPBH.callAsync("1", "2", (...restArgs) => {
  console.log(restArgs);
  console.log("This is callback");
});
console.log("end");
/*
start
e1:  1 2
e2:  [ '1', '2' ]  // 两个同步事件都调用了
end
// 等待 200 ms 后
[ null, '111' ]
This is callback
// 继续等待 1800 ms 后结束进程
*/
```

其他钩子的用法可以类推。

#### 异步类型钩子调用的返回值

异步类型的钩子，对于最终`tapAsync`、`promise`调用时注入的回调函数的返回值要注意 

- 一般的钩子不关心返回值，在最终的调用中传入的返回值也就是空的
- **只有 bail 类型的钩子会给最终调用传入参数，且第一个永远是` null`**  

## 拦截器

上面了解了 hook 及其用法，tapable 还对所有的 hook 都支持拦截器的注入，类似于 axios 的拦截器。**我们可以通过拦截器对钩子的发布(call)/订阅(tap)进行监听**，从而触发对应的逻辑。

示例如下

```js
const { SyncLoopHook } = require("tapable");

const hook = new SyncLoopHook(["test1", "test2", "test3"]);

hook.intercept({
  /**
   * 每次调用 hook.tap 注册时间时都会触发该拦截器，可以对事件进行修改
   * @param {*} tap 注册的事件 tap: { type: 'sync', fn: [Function (anonymous)], name: 'event1' }
   */
  register: (tap) => {
    console.log("register-tap:", tap);
    if (tap.name === "event1") {
      tap.fn = (...restArgs) => {
        console.log("event1-intercept：", restArgs);
      };
    }
  },
  /**
   * 每次 hook.call 调用前触发，仅触发一次，可以获取到 call 的参数
   * @param  {...any} restArgs hook.call 调用时传入的参数
   */
  call: (...restArgs) => {
    console.log('call：', restArgs)
  },
  /**
   * 每个注册的事件调用前触发，不能再更改事件了，仅能查看
   * @param {*} tap : { type: 'sync', fn: [Function (anonymous)], name: 'event1' }
   */
  tap: (tap) => {
    console.log("tap-tap:", tap);
  },
  /**
   * 在 loop 类型的钩子中才会生效
   * 每轮循环开始时都会触发
   * @param  {...any} args
   */
  loop: (...args) => {
    console.log("loop args:", args);
  },
});


let i = 0;
hook.tap("event1", (...restArgs) => {
  console.log("event1：", restArgs);
  if (i < 2) {
    i++;
    return i;
  }
});
hook.tap("event2", (...restArgs) => {
  console.log("event2：", restArgs);
});

console.log("start");
hook.call(1, 2, 3);
console.log("end");
/*
register-tap: { type: 'sync', fn: [Function (anonymous)], name: 'event1' }
register-tap: { type: 'sync', fn: [Function (anonymous)], name: 'event2' }
start
call： [ 1, 2, 3 ]
loop args: [ 1, 2, 3 ]
tap-tap: { type: 'sync', fn: [Function (anonymous)], name: 'event1' }
event1-intercept： [ 1, 2, 3 ]
tap-tap: { type: 'sync', fn: [Function (anonymous)], name: 'event2' }
event2： [ 1, 2, 3 ]
end
*/

```

透过拦截器，我们可以

- 在事件注册时，获取触发的 tap，**修改 tap 的属性** 
- 在事件调用时，获取事件的参数
- 在事件触发时，获取触发的 tap
- 在事件循环调用时，获取事件调用的参数（和重新循环 return 的值无关）

拦截器的使用如上，**实现上 tabable 同样是在构建新函数时将拦截器调用注入函数体内**，以实现目标。具体看后面的源码解读。

除了上面的事件注册（tap），调用（call），拦截（intercept）外，tapable 还支持一些注册事件时的配置，在构建新函数时会读取这些配置进行操作。

## Before & stage

#### before

在调用`tap`注册时间时，第一个参数支持传入对象，同时接收一个`before`属性，如下

```js
hook.tap({
  name: 'event2',
  before: 'event1' // before 接收一个字符串 | 数组，函数会在指定事件名的事件之前触发
}, (...restArgs) => {
  console.log("event2：", restArgs);
});
/*
start
event2： [ 1, 2, 3 ]
event1： [ 1, 2, 3 ]
end
*/
```

通过 before 指定了`event1`后，`event2`一定会在 `event1`之前触发。

#### stage

stage 作用和 before 类似，用以提升事件优先级，数字越小，优先级越高，执行的就越早。支持传入负数且默认为 0 。用法和 before 相似，也是在注册时传入该属性。

**注意：before 具有更高的优先级** 

```js
hook.tap({
  name: 'event2',
  stage: -1,
}, (...restArgs) => {
  console.log("event2：", restArgs);
});
/*
start
event2： [ 1, 2, 3 ]
event1： [ 1, 2, 3 ]
end
*/
```

两个属性都用于调整优先级，由于属性会冲突，所以最好不要混用以免造成混乱。

在 tabable 导出的对象中，除了钩子之外，还有两个辅助对象 HookMap 和 MultiHook ，下面对这两个辅助对象的使用也做一个说明。

## HookMap & MultiHook

#### HookMap

HookMap 用于更好的创建和管理 hook。

举个例子，假如我需要创建多个`SyncHook`，那一般可能就如下

```js
const hook1 = new SyncHook(["args"]);
const hook2 = new SyncHook(["args"]);
const hook3 = new SyncHook(["args"]);

hook1.tap('xxxx')
hook1.tap('xxxx')
hook2.tap('xxxx')
hook2.tap('xxxx')
hook3.tap('xxxx')
```

这样写不仅繁琐，逻辑复杂的时候可能自己都忘了自己创建过这样的 hook了。

HookMap 的作用就像名字一样，通过一个 map 类似的结构来存储我们创建的钩子，如下

```js
const { SyncHook, HookMap } = require("tapable");

const keyHook = new HookMap(() => new SyncHook(['args']));

// 相当于 map.set('key1', hook1)
keyHook.for("key1").tap("key1-event1", function (...restArgs) {
  console.log("key1-event1: ", restArgs);
});
// 在 key1 这个 hook 上注册第二个事件
keyHook.for("key1").tap("key1-event2", (...restArgs) => {
  console.log("key1-event2: ", restArgs);
});
// 相当于 map.set('key2', hook2)
keyHook.for("key2").tap("event2", (...restArgs) => {
  console.log("event2: ", restArgs);
});
// 获取要调用的 hook
const hook = keyHook.get("key1");

console.log("start");
hook.call(1);
console.log("end");
/*
start
key1-event1:  [ 1 ]
key1-event2:  [ 1 ]
end
*/
```

#### MultiHook

MultiHook 也用于辅助 hook 的编写，他可以同时给多个 hook 注册事件，拦截器，新增参数等

```js
const { SyncHook, HookMap, MultiHook } = require("tapable");

const keyHook = new HookMap(() => new SyncHook(["args"]));

keyHook.for("key1").tap("key1-event1", function (...restArgs) {
  console.log("key1-event1: ", restArgs);
});;
keyHook.for("key2");

const hook1 = keyHook.get("key1");
const hook2 = keyHook.get("key2");

const hook = new MultiHook([hook1, hook2]); // 传入多个 hook
hook.tap("event1", (...restArgs) => { // 同时给 hook1、hook2 注册 event1 事件
  console.log("event:", restArgs);
});

console.log("start");
console.log("hook-useed:", hook.isUsed());
hook1.call(1);
hook2.call(1);
console.log("end");
/*
start
hook-useed: true
key1-event1:  [ 1 ]
event: [ 1 ]
event: [ 1 ]
end
*/
```

以上是所有 tapable 的使用相关和略微一点原理，了解了 tapable 的使用对我们编写 webpack plugin 有很大的帮助，到这里已经可以去稍微了解一下 webpack plugin 然后实现一个了。 tapable 源码请看另一篇 《tapable 源码解读》。

## tapable 与 webpack 的关系

webpack 在初始化两个核心对象 Compiler 和 Compilation 时会创建一系列响应的 Hook 作为属性保存在各实例对象中。

在后续的构建过程中，调用`hook.call`来处理我们注册的各种事件，最终完成构建。具体可以查看 《webpack plugin》

## 参考文章

Webpack 核心库 Tapable 的使用与原理解析：https://zhuanlan.zhihu.com/p/100974318

Tapable 看这一篇就够了：https://juejin.cn/post/7040982789650382855