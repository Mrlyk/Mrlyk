# Stream

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/Stream-2.png" alt="Stream-2" style="zoom:50%;" />

[toc]

Node.js 最初是为了处理 Web 端 I/O 密集的性能问题，要处理这些问题免不了和文件系统打交道，最长使用的两个模块就是文件系统和网格系统。这两个系统都非常依赖流式控制。

```text
A stream is an abstract interface for working with streaming data in Node.js. The stream module provides an API for implementing the stream interface.
```

Node.js 中内置了`Stream`模块，提供了用于实现流接口的对象（就是用`Steam`来描述`Stream`）。我们平常使用的例子有 linux 中的连字符"|"。把前一个命令的结果作为流式数据传给后一个处理器。

**流是对输入输出设备的抽象，是一组有序的、有起点和终点的字节数据传输手段**

## 初识 Stream

#### Node.js stream 类型

- 设备流向程序，可读流 - readable
- 程序流向设备，可写流 - writable
- 双向流动，双工流 - duplex、transform

node.js 关于流的操作封装到了 Stream 模块，这个模块也被多个核心模块饮用。按照 unix 的哲学：一切皆文件。在 no de.js 中对文件的处理多数用流来完成。

- 普通文件（txt、jpg、mp4）
- 设备文件（stdin、stdout）
- 网络文件（http、net）

#### Steam 流转

假设读取一个配置文件`package.json`，就是

- 数据 - package.json 的内容
- 方向 - 设备（物理磁盘） -> node 程序

所以要创建一个可读流。

接着我们要把读取出来的数据放入我们的目标位置，这时候就需要一个可写流，将数据从 node 程序写入设备。

两个流都创建好之后就需要将可读流到可写流，怎么流呢？就是使用"管道"，在 node 中就是 `pipe`方法

```js
const fs = require('fs') // fs 文件系统内置了 Stream 模块
const path = require('path')

const readStream = fs.createReadStream(path.join(__dirname, '../../static/test3.txt'))
const writeStream = fs.createWriteStream(path.join(__dirname, '../../static/test4.txt'))

readStream.pipe(writeStream)
```

这样就实现了一个完整数据流转。**注意流只能从上往下，也就是从可读流流入到可写流。**

#### 加工数据

假如有这样一个需求，要把`package.json`的字符都转成小写并保存到`package-lower.json`。那么就需要一个**双向流**。

```js
const fs = require('fs') // fs 文件系统内置了 Stream 模块
const path = require('path')

const readStream = fs.createReadStream(path.join(__dirname, '../../static/test3.txt'))
const writeStream = fs.createWriteStream(path.join(__dirname, '../../static/test4.txt'))

readStream.pipe(lowercase).pipe(writeStream) // lowercase 是处理方法，这里只举例，具体如何定义看后面详细介绍
```

如上所示。先将可读流的数据流入了 lowercase 这个加工器，lowercase 作为了下游。之后又将处理后的流，流向 writeSteam。这时候 lowercase 就是上游。所以 lowercase 必须是双向的流。当然还可以向其中接入更多双向流，做更多次加工。

#### 为什么需要 Stream

大文件不可能一次返回给数据，网络带宽吃不消，用户内存吃不消。所以需要采用流的形式，把文件一点点放入内存中，一点点返回给用户。（**利用 http 协议的 Transfer-Encoding: chunked; 分段传输的特性**）

```js
const http = require('http');
const fs = require('fs');

http.createServer((req, res) => {
  fs.createReadStream(moviePath).pipe(res);
}).listen(8080);
```

同时在回传之前我们还可以对数据进行处理，比如压缩数据，只需要 pipe 一个新的处理方法即可，逻辑清晰。

## 可读流

**可读流是生产数据用来供程序使用的流**。常见的生产数据的方法有读取磁盘上的文件、读取网络请求等，如下：

```js
const fs = require('fs')
const rs = fs.createReadSteam(path)
```

从磁盘路径上读取一个文件，生产数据。

`process.stdin`也是一个可读流，读取用户的输入。`process.stdin.pipe(process.stdout)`，将用户的输入流向输出（打印出来）

#### 自定义可读流

除了系统提供的 `fs.createReadStream`，使用 gulp（基于流的自动化构建工具）或者 vinyl-fs（基于流的文件处理依赖）提供的 src 方法时也在使用可读流

```js
gulp.src(['*.js', 'dist/**/*.scss'])
```

#### 使用特定方式生产数据

1. 继承 Stream 模块的 **Readable**类
2. 重写`_read`方法，调用`this.push`将生产数据放入待读取队列

```js
const Readable = require('stream').Readable

class RandomNumberStream extends Readable { // 继承 Readable 类
  constructor() {
    super()
  }
  _read () {
    const ctx = this
    setTimeout(() => {
      const randomNumber = parseInt(Math.random() * 10000);
      ctx.push(`${randomNumber} - `) // 只能 push 字符串或 Buffer
    }, 2000) // 每 2000 毫秒 push 一个字符到缓冲区
  }
}
const rns = new RandomNumberStream()
rns.pipe(process.stdout) // 流向 shell 输出
// 1777 - 8099 - 7241 - 4133 - 9675 - 2441 - 5274 - 效果是在 shell 持续输出
```

**如何停下来**

**向缓冲区 push 一个 null 即可**

```js
ctx.push(null) // 停止流传输
```

#### 流的工作方式

上面的例子中，可以发现我们使用的是`setTimeout`而不是`setInteval`，但是结果依然按我们所想的执行了。这是因为流有两种工作方式

- 流动模式 - 数据由底层系统独处，并尽可能快的提供给程序
- 暂停模式 - 必须显示调用`read()`方法来读取若干数据块

流在**默认情况下是处于暂停模式的**，也就是需要程序显式调用`_read()`方法。上面的例子并没有显示的调用`_read()`方法是因为**pipe 会把流切换为流动模式**，这样`_read()`方法会被反复调用直到数据读取完毕。

**流动模式/暂停模式 切换**

从暂停切换到流动

1. 通过添加 data 时间监听器来启动数据监听
2. 调用`resume()`方法启动数据流
3. 调用`pipe()`方法将数据转接到另一个可写流

从流动切换到暂停

1. 在流没有`pipe()`时，调用`pause()`方法
2. 在`pipe()`时移除所有 data 事件的监听，再调用`unpipe()`方法

#### data 事件 / _read() 方法

使用了 pipe() 方法后数据就从可读流进入了可写流，但对用户是个黑盒，如何才能监听到具体的数据流动情况呢。切换流动模式和暂停模式的时候有两个重要的名词

1. 流动模式对应的 data 事件
2. 暂停模式对应的 `_read()` 方法

**data 事件**

一旦监听了可读流的 data 事件，流就进入了流动模式。

继续使用上面流的例子

```js
rns.on('data', chunk => { // 当可读流产生可供消费的数据后就会触发 data 事件
  console.log(chunk) // 每次流过来的小块数据，Buffer 对象
  console.log(chunk.toString())
})

rns.on('end', err => { // 流动完全结束后触发 end 事件
  if (!err) console.log('done')
})
```

**read(size) 方法**

流在暂停模式下需要显示调用`read()`方法才能得到数据，若传入`size`则会返回指定字节数的数据，超出时返回 null，不传则会返回所有的数据。

当然我们很多情况下不确定什么时候产生了数据，如果采用轮询的方式会显得效率很低。所以 nodejs 提供了一个 **readable **事件，在可读流数据准备好后触发。

```js
rns.on('readable', () => {
  let chunk
  while ((chunk = rns.read()) !== null) { // 注意这里不能把读取事件写在上面，不然永远只会读到第一次流进来的数据，导致循环无限触发。需要每次都重新取调一下 read 方法
    console.log(chunk.toString())
  }
})
```

#### 数据不会遗漏

在 nodejs 中，生产数据和事件监听都是异步操作。而 on 监听事件使用了`process.nextTick`会保证在数据生产之前被绑定好。所以在理论上不存在数据丢失的可能

## 可写流

可写流是对数据流向设备的抽象，通过可写流把数据写入设备，如磁盘文件、TCP、HTTP 等网络响应。

```js
process.stdin.pipe(process.stdout);
```

如上，`process.stdout`就是一个可写流，把数据写入标准的输出设备。流就是有方向的数据，其中可读流是数据源，可写流是目的地，中间的管道环节是双向流。

#### 可写流的使用

`write(chunk[, encoding[, callback]])`调用可写流实例的**`write()`**方法把数据写入可写流

- chunk - { String | Buffer } 要写入的数据
- encoding - 编码，默认 utf-8
- callback - 写入数据后的回调

方法返回一个布尔值：**如果流希望调用代码在继续写入其他数据之前等待 `'drain'` 事件被触发，则为 `false`；否则为 `true`。**（意思是一次没处理完这些数据，超出 highWaterMark了，这时候应该根据返回的 false 参数暂停可读流的读取。直到可写流的`drain` 事件触发）

```js
const fs = require('fs')
const path = require('path')

const crs = fs.createReadStream(path.join(__dirname, '../../static/test3.txt'))
const wrs = fs.createWriteStream(path.join(__dirname, '../../static/test4.txt'))

crs.setEncoding('utf-8')
crs.on('data', chunk => {
  if (chunk === '123ABC') { // 对数据进行一些操作
    chunk = Buffer.from('ABC'.toLowerCase(), 'utf-8')
  }
  wrs.write(chunk) // 写入数据
})
```

#### 自定义可写流

和自定义可读流类似，也需要做两个操作

1. 继承`stream`模块的`Writable`类
2. 实现`_write()`方法

```js
const Writable = require('stream').Writable // 引入 Writable 类

class OutputStream extends Writable {
  constructor() {
    super()
  }
  _write (chunk, encode, done) { // 自定义可写流
    process.stdout.write('custom:' + chunk.toString().toUpperCase()) // 使用标准输出的可写流并且将可读流数据转换成大写
    process.nextTick(done) // 这里是写入完一次的回调，是手动调用 write 方法时传入的（这里不严谨，应该在每次写入完成后才调用）
  }
}

const ops = new OutputStream()
new RandomNumberStream(3).on('data', chunk => {
  ops.write(chunk, err => { // chunk - 传入的自定义可写流的数据，callbcak - 传入的回调
    if (err) {
      console.log(err)
    }
    console.log('一次写入完成')
  })
})
/** 执行结果
3880 - 
custom:3491 - 一次写入完成
4748 - 
custom:7217 - 一次写入完成
9450 - 
custom:68 - 一次写入完成
*/
```

由上面的例子可以看出和最终可写流暴露出来的`write()`方法一样，`_write()`方法有三个参数，作用也相似

- chunk  - { Buffer | String } 写入的数据
- encoding - { String } 编码，传入的是字符串的时候才可设置
- callback - { Function } 回调，可以用来通知下一个数据传入

#### 实例化可写流时可传入的 options

上面我们在自定义可写流并实例化的时候没有传入任何 options ，但实际上有几个非常重要的 options

- objectMode - { Boolean } 默认 false，设置成 true 后 `writable.write()` 方法将可以写入任意 JS 对象（null 除外）
- highWaterMark - { Number } 每次写入的数据块大小，Buffer 时默认 16kb，objectMode 时默认 16？
- decodeStrings - { Boolean } 默认 true，将传入的参数转化为 Buffer

#### 可写流事件

- pipe - 当可读流调用`pipe()`方法向可写流传入数据时，会触发可写流的 pipe 事件（实验没触发，待确认）
- unpipe - 数据传输切断或结束的时候会触发该事件

`writeable.write()` 方法有一个 bool 返回值，前面提到了 ***highWaterMark***，当要求写入的数据大于可写流的 highWaterMark 的时候，数据不会被一次写入，有一部分数据被滞留，这时候`writeable.write()`就会返回 **false**，如果可以处理完就会返回 true

- drain - 当之前滞留的数据（`writable.write()`返回 false），处理完成后，可以继续写入新数据的时候触发该事件

除了`write()`方法之外，**可写流还有一个`end()`方法**，参数和`write()`相同，允许用户在关闭数据流之前写入最后一个额外的数据块。但也可以不传参数表示没有其他数据需要写入。需要在可写流完成后再调用，在`end()`方法后再调用`write()`方法会报错。

- finish - 调用`writable.end()`方法完成后，会触发`finish`事件。如果没有调用`end`方法不会触发该事件。
- error - 错误调用 error 事件

```js
const ops = new OutputStream()
let i = 0
new RandomNumberStream(3).on('data', chunk => {
  ops.write(chunk, err => {
    if (err) {
      console.log(err)
    }
    console.log('一次写入完成')
    i += 1
    if (i === 3) {
      ops.end('自定义数据', e => { // 自定义数据会被写入可写流
        // dosth
      })
    }
  })
})

// new RandomNumberStream(3).pipe(ops)
// ops.on('pipe', chunk => {
//   console.log('pipe') // 不知道为什么没触发
// })

ops.on('unpipe', err => {
  console.log('unpipe')
})
```

#### back pressure

背压问题，我们知道数据的读取速度是远大于写入速度的，这与硬件设备的特性有关。在我们使用流的方式读取和写入数据时，如果保持高速的读取，同时原本逻辑中包含复杂的运算，那么从源头读取的数据就会积压下来变得很庞大，依然可能挤爆存储。为了解决这个问题，node 提供了内置的`pipe()`方法，他的原理就和下面的例子相同。

```js
const ops = new OutputStream({
  highWaterMark: 1
})
const rns = new RandomNumberStream(100)
rns.on('data', chunk => {
  const writeStatus = ops.write(chunk, err => {
    if (err) {
      console.log(err)
    }
    console.log('一次写入完成')
  })
  if (writeStatus === false) { // 当数据块超出 highWaterMark 水位或写入队列处于繁忙状态，返回 false
    console.log('pause')
    rns.pause() // 暂停可读流的读取
  }
})

ops.on('drain', () => { // 积压数据处理完成后触发 drain 事件
  console.log('drain')
  rns.resume() // 恢复数据读取
})
```

## 双工流

双工流就是同时实现了 readable 和 writable 的流，既可以作为上有生产数据，又可以作为下游消费数据。这样就可以处于数据流动管道的中间部分

```js
rs.pipe(rws1).pipe(rws2).pipe(rws3).pipe(ws)
```

在 node 中更有两种双工流

- Duplex
- Transform

#### Duplex

**实现 Duplex**

实现 Duplex 很简单，上面我们分别实现了自定义的可读流和可写流。要实现 Duplex 这种双工流，就是把他们结合起来，同时实现`_read()`和`_write()`方法。但是 Node 不支持多继承，所以我们需要继承 Duplex 类。

```js
const { Duplex } = require('stream')
const kSource = Symbol('source')

class MyDulex extends Duplex {
  constructor(source, options) {
    super(options)
    this[kSource] = source
  }
  _write(chunk, encoding, callback) {
    // dosth
    if (Buffer.isBuffer(chunk))
      chunk = chunk.toString();
    this[kSource].writeSomeData(chunk); // 消费数据
    callback();
  }
  _read(size) {
    // dosth
    this[kSource].fetchSomeData(size, (data, encoding) => {
      this.push(Buffer.from(data, encoding)); // 生产数据
    });
  }
}
```

**构造函数参数**

- readableObjectMode - { Boolean } 默认 false，设置可读流为 ObjectMode
- writableObjectModel - { Boolean } 默认 false，设置可写流为 ObjectMode
- readableHighWaterMark/writableHighWaterMark - { Number } 读和写的“水位”
- allowHalfOpen- { Boolean } 默认 true，可写流结束时，自定结束可读流

可以看到 Duplex 就是这样既能生产数据又能消费数据，node 中常见的 Duplex 流有：

- TCP Socket
- Zlib - 压缩
- Crypto - 加密

#### Transform

首先 Transform 也是一种 Duplex 流，但是相比于 Duplex

- Transform 的可读流进过一定处理会自定进入可写流，而 Duplex 两者相对独立。**Duplex 是拿到数据后单独走自己的逻辑，不需要基于这段数据去处理。transform 则是基于数据做一些转换操作**
- Transform 内部继承了 Duplex 并自己实现了`writable.write()`和`readable.read()`方法

之所以叫 transform 流，就是因为他主要用来对数据进行”变形”，比如压缩数据。也就是说输出数据和输入的数据之间有个映射关系。

**实现 Transform**

```js
const { Transform } = require('stream')

class MyTransform extends Transform {
  constructor(options){
    super(options)
  }
  
  _transform(chunk, encoding, done) { // 和 read、write 的参数相同，必须实现
    this.push(chunk)
    callback();
  }

  _flush(callback) { // 不必须实现的方法，在可读流的 end 事件前调用，允许我们最后写入一点数据;当没有更多的写入数据被消耗时，但在触发 'end' 事件以表示 Readable 流结束之前，将调用此方法。
  }
}
```

**transform 事件**

- 继承自可写流的`finish`
- 继承自可读流的`end`

当调用`transform.end()`方法并且数据被`_transform()`处理完后会触发`finish`事件，在调用`_flush`后，所有数据输出完毕，触发`end`事件。end()方法 -> finish 事件 -> flush() 方法 -> end 事件

## through2

日常我们会频繁的使用 transform 处理数据，社区流性的方案是使用  [through2](https://www.npmjs.com/package/through2) 来处理。through2 封装了一个 transform 流。

`through2([options, ][ transformFunction][, flushFunctions])`

- options  - 其中 objectMode 设置为 true 后表示使用对象模式，也可以使用`through2.obj()`达到相同效果
- transformFunction - 接收三个参数，和 write、read 方法相同

```js
fs.createReadStream('example.txt').pipe(
	through2(chunk, encoding, callback) {
  	for (let i = 0; i < chunk.length; i++) {
  		if (chunk[i] === 97) chunk[i] = 122 // 把 'a' 转换为 'z'
		}
		this.push(chunk) // 生产数据
		callback()
  }
).pipe(fs.createWriteStream('out.txt'))
 .on('finish', () => { 
	// dosth
})
```

## pipeline

pipeline 方法用于链式调用`pipe()`方法，在管道内链式传输多个流，在管道任务结束后回调

`pipline(source[, ...transforms], destination, callback)`

- source - 可读路
- ...transforms - 双工流
- destination - 可写流
- callback - 回调

```js
const { Transform } = require('stream')
const fs = require('fs')
const path = require('path')
const { pipeline } = require('stream')

class MyTransform extends Transform { // 自定义 transform 双工流
  constructor(options){
    super(options)
  }
  
  _transform(chunk, encoding, done) {
    chunk = chunk.toString().toUpperCase() // 转换数据
    this.push(chunk)
  }

  _flush(callback) {
    console.log('flush') // 未触发，待确定原因？？？
  }
}

pipeline(
  fs.createReadStream(path.join(__dirname, '../../static/test3.txt')),
  new MyTransform(), // 双工流 transform 处理
  fs.createWriteStream(path.join(__dirname, '../../static/test4.txt')),
  (err) => {
    if (err) {
      return console.log('传输失败:', err)
    }
    console.log('传输成功')
  }
)
```

