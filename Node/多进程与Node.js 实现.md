# 多进程与 Node.js 实现

[toc]

## 进程与线程

进程和线程的诞生要从多任务谈起，多任务是指操作系统可以在同一时间内运行多个应用程序，CPU 按顺序执行代码，在同一时间内只能处理一个任务，而在单核时代主流操作系统都有了多任务能力，主要靠快速在多个任务之间切换，让人感觉多个任务同时执行

进程是指操作系统正在运行的应用程序，而一个进程内部可能有多个并发的子任务，这就是线程

## Web 服务器模型

Web 服务器需要同时处理多个用户的请求，返回给用户响应内容，有几种不同的服务器模型实现多任务

#### 多进程单线程

这种服务模型通过进程复制实现同时响应多个请求，每个请求使用一个单独的进程处理，但操作系统复制进程需要复制进程内部状态，这样相同的状态在内存中存在多份，对**内存有一定的开销，可以同时处理的请求数和内存大小正相关**

#### 单进程多线程

为了避免复制多进程带来的内存浪费问题，多线程被引入 Web 服务器模型，一个线程响应一个用户请求，线程**可以共享进程的内存，不会造成内存浪费**，同时线程相对于进程的内存开销要小得多。但每个线程有自己的独立堆栈，需要占据一定的内存空间，因此只是缓解了多进程带来的资源浪费问题

另外操作系统在切换线程的同时需要切换线程的 context，**当线程数量过多时 CPU 会被耗在 context 切换中。同时一个线程的崩溃可能会导致整个进程 crash，为服务器带来了相当程度的稳定性风险**

#### 多进程多线程

顾名思义多进程多线程模型就是启用多个进程，在每个进程内启用多个线程来解决高并发问题，集成了多进程和多线程模型的好处，但当用户量足够大的时候也同时拥有了另外两种模型的缺陷

当并发数达到千万级内存问题就会暴露出来，这就是著名的 C10k 问题，C10k 问题的本质在于：**为了处理高并发创建的进程线程太多，数据拷贝频繁、进程/线程上下文切换消耗大， 导致操作系统崩溃**

#### 事件驱动

为了解决 Web 高并发问题 Nginx 使用了事件驱动的模型，在一个 CPU 上使用单进程、单线程来响应用户请求，把最耗时的阻塞任务 I/O 任务异步化，处理完成后通过事件通知主进程给用户响应，在等待 I/O 任务的时候处理下一个请求

这样的模型性能取决于 CPU 的运算能力，但不受多进程、多线程模式中资源上限的影响，非常适合 Web I/O 密集的特征，成了现在 Web 服务器的主流模型

## master-worker 模式

Node.js 本身是事件驱动模型，为了处理多核使用不足的问题，**可以按照 cpu 核心数启动多个进程**，理想情况下每个进程使用一个核心。

Node.js 提供了`child_progress`模块支持多进程，通过`child_process.fork(modulePath)`可以调用指定模块，衍生新的子进程。

**主进程负责管理工作进程，工作进程负责具体业务和逻辑处理**

```js
// work.js 工作进程
const http = require('http');
const randomPort = parseInt(Math.random() * 10000)；
http.createServer((req, res) => {
  res.end('Hello world')
}).listen(randomPort);


// master.js 主进程
const { fork } = require('child_process');
const os = require('os');

for (let i = 0, len = os.cpus().length; i < len; i++) {
  fork('./worker.js'); // 按照核心数启动子进程进行管理
}
```

## 进程通信

#### WebWorker

主进程和工作进程之间通过 WebWroker 通信，和浏览器上的很相似

```js
// worker.js
const http = require('http')

http.createServer((req, res) => {
  res.end('Hello World')
}).listen('9898')

process.on('message', msg => { // 子进程通过全局的 process 对象通信
  console.log('worker获得msg:', msg)
})
process.send('9898 端口 启动')

// master.js
const { fork } = require('child_process')
const worker = fork('./worker.js')

worker.on('message', msg => { // 父进程通过 fork 的子进程通信
  console.log('master获得msg:', msg)
})
worker.send('ok');
```

*在 windows 上，客户端和服务端通信是基于端口的。服务器监听某一个端口，客户端向某一个端口发送请求。而在 Linux/Unix 上，服务端可以监听端口也可以监听文件，将文件也视为“端口”*

## 稳定性

#### 句柄传递

>  句柄 -windows 上标识资源的引用，内部包含了指向对象的文件描述符，也可以看作是一类文件描述符。

在 node 创建服务时，通常情况下如果多个服务启动在一个端口上会报错，提示端口已经被占用。但是如果我们**想多个服务监听一个端口分别做不同的操作呢？**

我们可以使用 master 主进程监听目标端口，同时将**句柄发送给工作进程。**

```js
subprocess.send(message[, sendHandle[, options]][, callback])
```

> `handle` 对象可以是服务器、套接字（任何具有底层 `_handle` 成员的对象），也可以是具有有效文件描述符的 `fd` 成员的对象。

将句柄直接发送给工作进程还有个好处就是我们不需要额外创建一个 master 与 worker 的 socket 连接。文件描述符的总数是有限的，创建新的 socket 连接会消耗描述符。直接将句柄发送后便关闭连接可以节省一倍的资源。

实例代码如下

```js
//master.js 
const { fork } = require('child_process')
const net =  require('net')
const os = require('os')
const path = require('path')

const workers = []
for(let i = 0;i < os.cpus().length; i++) {
  const worker = fork(path.join(__dirname, './http-worker.js')) // 有几个 cpu 核心就启动几个子进程
  workers.push(worker) // 暂存这些子进程
}

const server = net.createServer()
server.listen(9988, () => { // 监听目标端口
  workers.forEach(worker => {
    worker.send('tcp', server) // 将句柄发送给 worker 
  })
  server.close() // 发送完成后就直接关掉，不占用资源
})


// worker.js
const http = require('http')
const httpServer = http.createServer((req, res) => { // 创建一个 http 服务，但是不监听任何端口
  res.end(`Hello ${process.pid}\n`)
})
/**
 * @param { String } msg tcp发送的请求信息
 * @param { net } tcpServer 主进程启动的tcp服务
 */
process.on('message', (msg, tcpServer) => { // 接收信息
  if (msg === 'tcp') { // 如果是 msg 发过来的 tcp 服务的句柄
    tcpServer.on('connection', socket => { // tcp 连接建立后，将服务交给 httpServer 处理
      httpServer.emit('connection', socket)
    })
  }
})
```

**文件描述符在每个进程中都不同，所以监听相同端口号会失败**，node 中有`SO_REUSEADDR`选项，允许不同进程监听相同端口号。因为上面的 worker 都是用 master 发送的 socket，所以他们用的文件描述符是相同的，所以可以监听。

注意：虽然启动了多个进程来监听同一个端口，但是文件描述符同一时间只能被一个进程占用，所以每次请求过来也只有一个进程可以抢到对请求提供服务。

#### 自动重启

一般情况下，我们可以在 master 进程中监听`exit`事件，来自动重启`worker`服务以保证稳定性。

```js
const workers = {}
for(let i = 0;i < os.cpus().length; i++) {
  const worker = fork(path.join(__dirname, './http-worker.js')) // 有几个 cpu 核心就启动几个子进程
  worker.on('exit', () => {
    // worker 进程退出了。重启
    delete workers[worker.pid]
    // todo: 重启 fork
  })
  workers[worker.pid] = worker // 暂存这些子进程
}
```

在 master 进程结束的时候，也可以中调用`kill()`方法主动结束子进程

```js
// master.js
process.on('exit', () => { // 主进程使用全局的 process
  for (const pid in workers) {
    workers[pid].kill();
  }
});
```

#### cluster 集群模块

> Node.js 的单个实例在单个线程中运行。 为了利用多核系统，可以启动 Node.js 进程的集群来处理负载。

cluster 集群其实就是我们上面所说的 master 管理 worker 的 node 实现。

**使用 cluster 创建集群**

```js
const cluster = require('cluster')
const http = require('http')
const numCPUs = require('os').cpus().length

if (cluster.isMaster) { // 判断进程是否是主进程，由 process.env.NODE_UNIQUE_ID 决定 如果 process.env.NODE_UNIQUE_ID 未定义，则 isMaster 为 true。
  console.log(`主进程${process.pid}正在运行`)
  // 初始化工作进程
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork()
  }
  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程${worker.process.pid}退出`)
  })
  console.log(`所有工作进程hash数组${cluster.workers}`)
} else {
  // 工作进程之间可以共享 tcp 连接
  // 不是主进程则启动 http 服务
  http.createServer((req, res) => {
    res.writeHead(200)
    res.end('Hello World!')
  }).listen(8080)
}

```

**cluster 事件**

- disconnet - 工作进程的 IPC 管道断开后触发
- exit - 工作进程关闭时触发
- fork - 新工作进程被衍生时触发
- listening - 工作进程调用`listen()`后触发
- message - master 收到任何工作进程的消息时触发
- online - 当衍生一个新的工作进程后，工作进程应当响应一个上线消息。 master 收到上线消息后触发此事件
- setup - master 进程`setupMaster()`方法调用时触发