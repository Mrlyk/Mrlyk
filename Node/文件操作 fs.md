# 文件操作 FS

> Node.js 操作文件依赖内置的 fs 模块

[toc]

## 一、几种不同的操作方式

### callback

node.js 异步操作默认是采用回调的方式：`callback(err, returnValue)`

- err - 异常的错误信息，未发生异常时为`null`
- returnValue - 正常的回调方法

```js
const fs = require('fs')

fs.stat('.', (err, stats) => { // 第一个参数是文件路径
  if (err) { 
    // dosth 
    return;
  }
  // dosth
})
```

### promise

`fs.promise`提供了使用 promise 来处理异步操作的方法。API 可以通过 `require('fs').promises`或者`require('fs/promises')`（注意后者需要 node 在 v14 版本以上）

```js
const fsPromise = require('fs').promises

fsPromise.stat(path.join(__dirname, '../static/test.txt')).then(stats => {
  console.log(stats)
}).catch(err => {
  console.log('error:', err.message)
})
```

### promisify

在 promise 规范制定后，node 将该方法放入了内置的 util 模块内，**通过 util 提供的`util.promisify`方法可以把所有标准的 callback 方法转换成 promise 风格**

```js
const fsStat = util.promisify(fs.stat); // 使用 promisify 方法包装
fsStat(path.join(__dirname, "../static/test.txt"))
  .then((stats) => {
    console.log('promisify');
    console.log(stats);
  })
  .catch((err) => {
    console.log("error:", err.message);
  });
```

### 同步方法

fs 为大部分方法提供同步版本，一般规则是在后面加`Sync`后缀，如`fs.readFileSync()`

**stats**

上面举例的 stats 对象返回是文件的详细信息，还有一些方法

- stats.isDirectory()：判断文件是否是个文件夹
- stats.isFile()：判断文件是否是普通文件
- stats.isSymbolicLink()：判断文件是否是软链接
- stats.size：获取文件字节数（utf-8 一个中文三个字节，unicode 编码一个中文两个字节）

## 二、文件读取

### fs.readFile

`fs.readFile(path[, options], callback)`

```js
const fs = require('fs')
const path = require("path");

fs.readFile(path.join(__dirname, "../../static/test.txt"), (err, data) => {
  console.log(data)
}) // 未制定文件编码类型时默认返回 buffer，指定后根据编码类型返回。
```

**options**

配置选项对象，如果 options 是字符串，则指定文件的编码类型，如 utf-8。如果是对象则看配置。

- encoding - 编码类型属性
- flag - 文件的访问模式，默认'r'，以只读模式访问
- signal - 是否可中断读取

### fs.open、fs.read、fs.close

readFile 是非常简单的 api，会一次读取所有的数据，适合读取小文件。如果想精确的读取文件（指定读取的开始位置，大小等），需要使用 open、read、close 三个方法。

读取文件分三步走：

1. `fs.open()` 打开文件描述符
2. `fs.read()` 读取文件
3. `fs.close()`关闭文件描述符

```text
POSIX 系统（可移植操作系统接口），unix 的一种开发规范。其文件系统使用了一个数字作为文件描述符，文件系统使用描述符来追踪文件。分配后，文件系统可使用该标识符来读取文件，写入数据或者请求文件信息相关的 buffer。
```

#### fs.open(path[, flag[, mode]], callback)

- path - 路径，也可能是 buffer、url
- flag - 文件系统标志，默认 r
- mode - 文件模式，默认0o666（rw-）可读写
- callback - 回调 `(err, fd) => {}` 两个参数, **fd - fileDescription文件描述符**

#### fs.read(fd,[ options, ]callback)

- fd - 文件描述符

- options - 可选项，不设置默认下述值

  - buffer：数据（从 fd 读取）将被写入的缓冲区，默认会申请一个新的缓冲区 `Buffer.alloc(16384)`
  - length: **需要的字节数**，默认使用`buffer.length`
  - **position: 读取开始位置，如果 position 为 null，则从当前文件位置读取，并重置文件位置到上次位置；如果 position 是整数，则会从指定的字节开始，并且读取完成后文件位置重置到文件头部。默认值为 null**
  
- callback

  - err
  - bytesRead
  - buffer

```text
fs.read 还有一个需要把参数写全的重载 fs.read(fd, buffer, offset, length, position, callback)
```

buffer 可以使用对象上的`toString()`方法转换为文本，接受参数指定要转换的编码格式，默认 UTF-8。

#### fs.close(fs, callback)

`fs.close` 用于关闭文件描述符，大多数操作系统都会限制同时打开的文件描述符数量，因此当操作完成时关闭描述符非常重要。 如果不这样做将导致内存泄漏，最终导致应用程序崩溃。

### 流式读取

上面的两种都是直接读取的二进制数据。但是对于特别大的文件，一般使用流的方式读取。

#### fs.createReadStream

`fs.createReadStream(path[, options])`

- path
- options 常用的
  - fd: 文件描述符，**如果指定了描述符则会忽略 path**
  - autoClose: 默认 true，是否读取完成或异常时自动关闭文件描述符
  - start：读取开始位置
  - end：读取结束位置
  - highWaterMark：默认 64*1024，普通可读流一般 16k，单位是字节

```text
highWaterMark: fs.createReadStream() 是基于 fs.read() 的。一开始，会先在内存中准备一段buffer，然后用fs.read()逐步将磁盘的字节copy到这段buffer里。完成一次读取时，会通过Buffer的slice方法取出部分数据作为一个小Buffer对象，再通过data事件传递给调用方。理想情况下每次读取的长度就是 highWaterMark。
```

#### 流各个阶段的事件

```js
const fs = require('fs')
const path = require("path");

const rs = fs.createReadStream(path.join(__dirname, "../../static/test.txt"), {
  highWaterMark: 6 // 每个 chunk 6个字节
})

rs.on('open', fd => {
  console.log('文件描述符:', fd)
})

rs.on('ready', (e) => {
  console.log('文件准备好')
})

rs.on('data', chunk => { // 会根据 highWaterMark 的大小，递归的触发该事件
  console.log('当前chunk 数据:', chunk.toString())
})

rs.on('end', () => {
  console.log('读取完成')
})

rs.on('close', () => {
  console.log('文件关闭')
})

rs.on('error', (err) => {
  console.log('读取异常', err)
})
```

## 三、文件写入

写入的操作基本与读取对应

### fs.writeFileSync

`fs.writeFile(file, data[, options], callback)`

1. file：文件名或文件描述符
2. data：常用的主要是 string 和 buffer
3. options
   - encodding: 编码 默认 utf-8
   - mode: 权限 默认 0o666
   - flag：文件系统标志 默认 w
3. callback(err)

当 `file` 是文件名时，则异步地写入数据到文件。文件不存在则创建文件，文件已存在，则**覆盖文件内容**。（如果父文件夹都不存在会报错）

```js
const fs = require('fs')
const path = require('path')

fs.writeFileSync(path.join(__dirname, "../../static/test2.txt"), '测试', err => {
  console.log('写入异常:', err)
})
```

### fs.appendFile

`fs.appendFile(path, data[, options], callback)` 将数据追加到文件尾部，如果文件不存在则创建该文件

```js
const fs = require('fs')
const path = require('path')

fs.appendFile(path.join(__dirname, "../../static/test2.txt"), 'Hello Node', err => {
  err && console.log('append error:', err) // append 没有异常也会进入回调，但是 err 的值是 null
})
```

### fs.createWriteStream

`fs.createWriteStream(path[, options])`创建可写文件流

options（比较常用的有）

- fd: 默认值 null，如果指定了 fd，则会忽略 path 参数，使用指定的文件描述符（不会再次触发 open 事件）
- mode：默认值 0o666（rw-）可读写
- autoClose: 默认值: true，当 'error' 或 'finish' 事件时，文件描述符会被自动地关闭
- start: 开始写入文件的位置，不设置默认覆盖

可写流也具有 `open`和`ready`事件。

```js
const fs = require('fs')
const path = require('path')

const ws = fs.createWriteStream(path.join(__dirname, "../../static/test2.txt"), {
  start: 6
})

ws.on('open', fd => {
  console.log('open fd:', fd)
})

ws.on('ready', () => {
  console.log('ready')
})

ws.write('Hello Stream') // 通过 write 方法将数据写入流
```




## 文件系统标志

以下标志在 `flag` 选项接受字符串的任何地方可用。

- `'a'`: 打开文件进行追加。 如果文件不存在，则创建该文件。

- `'ax'`: 类似于 `'a'` 但如果路径存在则失败。

- `'a+'`: 打开文件进行读取和追加。 如果文件不存在，则创建该文件。

- `'ax+'`: 类似于 `'a+'` 但如果路径存在则失败。

- `'as'`: 以同步模式打开文件进行追加。 如果文件不存在，则创建该文件。

- `'as+'`: 以同步模式打开文件进行读取和追加。 如果文件不存在，则创建该文件。

- `'r'`: 打开文件进行读取。 如果文件不存在，则会发生异常。

- `'r+'`: 打开文件进行读写。 如果文件不存在，则会发生异常。

- `'rs+'`: 以同步模式打开文件进行读写。 指示操作系统绕过本地文件系统缓存。

  这主要用于在 NFS 挂载上打开文件，因为它允许跳过可能过时的本地缓存。 它对 I/O 性能有非常实际的影响，因此除非需要，否则不建议使用此标志。

  这不会将 `fs.open()` 或 `fsPromises.open()` 变成同步阻塞调用。 如果需要同步操作，应该使用类似 `fs.openSync()` 的东西。

- `'w'`: 打开文件进行写入。 创建（如果它不存在）或截断（如果它存在）该文件。

- `'wx'`: 类似于 `'w'` 但如果路径存在则失败。

- `'w+'`: 打开文件进行读写。 创建（如果它不存在）或截断（如果它存在）该文件。

- `'wx+'`: 类似于 `'w+'` 但如果路径存在则失败。