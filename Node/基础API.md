# 基础 API

[toc]

## 一、Path

> 路径处理，node.js 通过内置的 path 模块提供了路径处理的基础 API。

### 在 Windows 上与 POSIX(Linux）上

- windows 风格: 是以双反斜杠`\\`作为路径，如`C:\\user\\test`
- POSIX 风格：以`/`作为路径，如`/user/test`

要注意路径区分

### 内置属性

#### path.parse() - 解析路径，返回元信息

```js
const path = require('path')
path.parse('/usr/local/nginx/nginx.conf');
// 返回
{
  root: '/',
  dir: '/usr/local/nginx',
  base: 'nginx.conf',
  ext: '.conf',
  name: 'nginx'
}
```

#### path.format() - 和上面的相反，从对象返回路径

```js
path.format({
  root: '/',
  dir: '/usr/local/nginx',
  base: 'nginx.conf'
}) 
// 返回 '/usr/local/nginx/nginx.conf'
```

#### path.normalize() - 规范化 path

```js
path.normalize('/usr/local//test/my/..') // 返回 /usr/local/test
```

#### path.join() - 使用系统分隔符连接路径

注意这里是直接将路径连接起来

#### path.resolve(from, to) - 返回 from 到 to 到相对路径

```js
path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');
// 返回: '../../impl/bbb'
```

### 获取基本信息

1. path.basename: 返回 path 最后一部分
2. path.delimiter: 返回操作系统路径界定符，Windows 返回 `;` POSIX 返回 `:` 
3. path.dirname: 返回文件目录名
4. path.extname: 返回路径的拓展名（jquery.min.js 拓展名是 .js）
5. path.isAbsolute 检测路径是否是绝对路径
6. path.sep: 返回路径分隔符，Windows 返回 `\` POSIX 返回 `/`

## 二、事件

和浏览器上的事件有些相似之处，比如浏览器上可以用`addEventListener`添加事件，node 上也有类似的方法。

### 事件监听

Node.js 的事件监听和 jQuery 的有点像，`emitter.on(eventName, listener)`

```js
const EventEmitter = require('events')

const e = new EventEmitter()
e.on('customEvent1', () => { console.log('监听到了') })
```

1. EventEmitter 实例维护一个 listener 数组，每次新的 listener 都会被添加数组尾部。事件按顺序触发，不会被同步阻塞，像是 forEach 的循环调用，不关心事件内部的行为，只负责调用。但是**整体的线程会被异步事件占用，直到事件完成才释放。**
2. listener 不会检测重复性去覆盖，而是多次添加。

### 事件触发

```js
const e = new EventEmitter()

e.emit('customEvent1') 
e.emit('customEvent1'， {name: 'asd', age: 123}, '参数2') // 支持传递参数，可以传递多个

e.on('customEvent1', (param1, param2) => { console.log('监听到了') }) // 可以接受多个参数
```

### 事件卸载

**off / removeListener**

```js 
const EventEmitter = require('events')

const e = new EventEmitter()
const callback = () => { console.log('监听到了') }
e.on('customEvent1', callback)

e.off('customEvent1', callback) // 移除单个事件，两个参数都必填。 off 是 removeListener 的别名

e.removeAllListeners('customEvent1') //移除所有名称为 customEvent1 的 listener，不传的话则移除所有 listener
```

### this 指向

监听回调中的 this 指向 EventEmitter 实例。注意箭头函数的情况下 this 指向和父级作用域一致（测试是个空对象）。

可以使用 this.eventNames() 获取到现在所有注册的事件。

### <font color="red">异步调用</font>

Node.js 提供以下方法执行异步事件，前两个是 node 独有的，没有第二个参数。后两个和 web 相同

- `setImmediate()`
- `process.nextTick()`
- `setTimeout()`
- `setInterval()`

`setImmediate()` 、 `setTimeout(() => {}, 0)`（传入 0 毫秒的超时）、`process.nextTick()` 的**不同点**：

传给 `process.nextTick()` 的函数会在事件循环的当前迭代中（同步代码执行完成后）被执行。 这意味着它会始终在 `setTimeout` 和 `setImmediate` 之前执行。（也会在 promise.resolve 之前。因为这些事件都是在下一次事件循环中触发，但是 nextTick 会在下一次循环开始前执行）

延迟 0 毫秒的 `setTimeout()` 回调与 `setImmediate()` 非常相似。 执行顺序取决于各种因素，但是它们都会在事件循环的下一个迭代中运行。(测试是 setTimeout 更早一点)。

**注意**

`process.nextTick()`会阻止 I/O 的回调执行，纵使 I/O 已经执行完成。`setImmediate`则是被设计出来防止这个问题的，官方更推荐使用`setImmediate`。

### 其他事件属性

- **prependListener**

该方法可以把监听事件放入头部。

```js
const e = new EventEmitter();
e.prependListener('customEvent1', () => console.log('监听到了'));
```

- **once**

控制事件只触发一次

```js
const e = new EventEmitter();
e.once('customEvent1', () => console.log('监听到了'));
```

## process

Node 的全局变量，也是一个 Eventemitter 实例，提供了当前 node.js 的进程信息和操作方法

### 系统信息

常用的系统信息 process

- title - 进程名，默认 node
- pid - 进度 pid
- ppid - 父进程 pid
- platform - 运行进程的操作系统（aix、drawin、freebsd、linux、openbsd、sunos、win32）
- version - node 版本
- env - 当前 shell 的环境变量
- **execPath** - 当前 node 执行文件所在的绝对路径
- **argv**: - 返回一个数组，第一个是 node 执行文件所在的绝对路径，后面跟着可执行文件、脚本名称或脚本名称后面的任何选项
- **execArgv** - 返回一个数组，里面是执行命令时加入的参数，如 ['--inspect']，这些参数不会出现在 argv 中

**argv 和 execArgv 的区别示例** 

```shell
$ node --harmony script.js --version
# `process.execArgv` 的结果：
['--harmony']
#`process.argv` 的结果
['/usr/local/bin/node', 'script.js', '--version']

$ webpack --mode=development
# `process.execArgv` 的结果：
# 没有任何直接参数，所以是空
[]
#`process.argv` 的结果
# node 路径								# 脚本															# 脚本后面的选项
['/usr/local/bin/node', '/xxx/xx/node_modules/.bin/webpack', '--mode=development']
```

### stdin & stdout 

Node.js 的标准输入、输出对象通过 process 提供

```js
process.stdin.pipe(process.stdout)
```

上面一行代码可以把控制台的输入在控制台再输出出来（类似于按一下等于按了两下）

## 常用操作方法

- process.chdir() - 切换工作目录到指定目录
- process-.cwd() - 返回当前脚本的执行目录
- process.exit() - 退出当前进程
- process.memoryUsage() - 返回进程内存使用情况

## 进程事件

process 作为 EventEmitter 的实例，可以监听系统中一些核心的事件。

### exit

当进程意外退出或是事件循环不再需要执行任何事件时（包括异步事件）则触发该事件，并且 exit 事件执行完成后，node.js 进程终止。

```js
process.on('exit', () => { //dosth })
```

### uncaughtException

进程抛出未被捕获的异常时会触发该事件

```js
process.on('uncaughtException', (err) => {
  console.log(err.stack)
})
```

### beforeExit

在事件循环不再需要执行任何时间时，会先于 exit 触发该事件。可以在这里注册监听器让进程继续

```js
process.on('beforeExit', () => { //dosth })
```

### message

通过 IPC 通道 **fork**子进程（fork 是 node 子流程模块提供的执行外部应用的方法），子进程收到父进程使用`childprocess.send()`发送的消息就会触父`message`事件

```js
process.on('message', (m) => { console.log('收到的消息', m) })
```

### process.nextTick(callback)

process.nextTick() 方法将 callback 添加到当前主线程的末尾执行。
