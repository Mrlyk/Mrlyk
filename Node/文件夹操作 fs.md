# 文件夹操作

> 文件夹也是通过 fs 模块来进行管理，这里列举常用的一些操作

[toc]

## 文件夹读写创建与删除

### fs.openDir

`fs.openDir(path[, options], callback)`打开一个目录，**返回一个 `fs.Dir`对象**。迭代完 dir 对象后，资源会被关闭。

**fs.Dir & fs.Dirent**

fs.Dir 是可迭代的目录流的类，fs.Dirent 是遍历 fs.Dir 获得的目录项，可以是文件或者子目录。

fs.Dir

- dir.path - 目录的只读路径
- dir.read() - 不传入 callabck 函数则返回 Promise，读取迭代器下**一个目录项**，返回一个 Promise，resolve 后得到 fs.Dirent 或 null（如果没有更多的目录项要读取）
- dir.close() - 不传入 callabck 函数则返回 Promise，关闭目录的底层资源句柄

fs.Dirent

- dirent.name
- dirent.isDirectory()
- dirent.isFile()
- dirent.isSymbolicLink()

```js
const fs = require('fs')
const path = require("path");

fs.opendir(path.join(__dirname, "../../static"), async (err, dir) => {
  console.log(err) // 没有报错的时候返回 null
  console.log(dir) // 返回 dir 对象
  // dir 对象的的迭代需要使用异步迭代器，因为这边的读取是异步的？同步迭代器无法遍历对象
  // 注意这里迭代完后资源句柄会被关闭，后续如果再读会报错
  for await (const dirent of dir){
    console.log(dirent.name)
  }
  // 也可以使用 dir.read() 读取，但是只会读取一个目录项且顺序不固定，再读取期间添加或删除的条目也不会被读取到
  // dir.read().then(dirent => {
  //   console.log(dirent.name)
  // })
})
```

### fs.readdir

`fs.readdir(path[, options], callback)`读取目录的内容，回调有两个参数`(err, files) => {}`。files 是目录下**文件名**的数组

**options**

- encoding - { String } 编码默认 utf-8，如果设置'buffer'则返回 Buffer 对象
- withFileTypes - { Boolean } 默认 false，设置为 true 后回调函数数组会变为 fs.Dirent 对象`files:[{"name":"test.txt"},{"name":"test2.txt"},{"name":"test3.txt"}]`

```js
const fs = require('fs')
const path = require("path");

fs.readdir(path.join(__dirname, '../../static'), (err, files) => {
  if (err) {
    console.log(err)
    return null
  }
  console.log('files:%j', files)
}) // files:["test.txt","test2.txt","test3.txt"]
```

### fs.mkdir

`fs.mkdir(path[, options], callback])`创建目录

**options**

- rescurive - { Boolean } 默认 false，设置为 true 的时候会把不存在的目录创建，存在的不做操作
- mode - { String } 0o777 (**0o(Octonary) - 表示8进制**) 这里表示给所有人所有权限，windows 不支持

```js
fs.mkdir(
  path.join(__dirname, "../../static/img"), // 创建 static 下的 img 目录
  {
    recursive: true
  },
  err => {
    if (err) {
      console.log(err);
      return null;
    }
  }
)
```

### fs.rmdir

`fs.rmdir(path[, options], callback)` 删除文件夹

**options**

- recursive - { Boolean } 默认为 false，是否递归删除。开启后如果 path 不存在也不会报错
- retryDelay - { Number } 默认100毫秒，异常重试延迟，resursive 不开启则不生效
- maxRetries - { Number } 默认为0，resursive 不开启则不生效。如果遇到 EBUSY、 EMFILE、 ENFILE、 ENOTEMPTY 或 EPERM 错误，则 Node.js 将会在每次尝试时以 retryDelay 毫秒的线性回退来重试该操作

```js
fs.rmdir(path.join(__dirname, "../../static/img"), (err) => { // 删除 img 文件夹
  if (err) {
    console.log(err);
    return null;
  }
});
```

## 文件变化监听

fs.FSWatcher 继承自 EventEmitter，调用后返回一个 fs.FSWatcher 实例。每当指定文件被修改时会触发回调。

### fs.watch()

`fs.watch(filename[, options][, listener])`

- filename - { String } 文件或文件夹路径

- options

  - encoding - { String } 默认 utf8，传递给监视器的**文件名**的字符编码

  - recursive - { Boolean } 默认 false，是否监视所有子目录，仅在 macOS 和 Windows 上支持
  - persistent - { Boolean } 默认 true，指示如果文件已经被监视，是否仍然启动本监视进程
  - listener - { Function }`(eventType, filename) => {}` 回调函数
    - eventType 主要是 `rename`和`change`。在大多数平台上，文件出现或消失触发`rename`事件。在 windows 上则不会触发任何事件，并且被监视的文件被删除时，则报告`EPERM`错误

```js
const fs = require("fs");
const path = require("path");

fs.watch(
  path.join(__dirname, "../../static/test2.txt"),
  (eventType, filename) => {
    console.log(eventType);
    console.log(filename);
  }
);
```

 **fs.watch() 依赖操作系统的实现，在不同平台上表现会有差异**

1. Linux 操作系统使用 inotify
2. 在 macOS 系统使用 FSEvents
3. 在 windows 系统使用 ReadDirectoryChangesW

### fs.watchFile

`fs.watchFile(filename[, options], listener)`监听文件变化的具体内容，采用轮询的方式。上一个是监听整体文件是否变化了，这个方法则是监听文件变化的具体时间、大小信息，但是效率比`fs.watch()`低。

- options
  - bigInt - { Boolean } 默认 false，指定回调函数中 stat 中的数值是否为 bigint 类型
  - persistant - { Boolean } 默认 true，指示如果文件已经被监视，是否仍然启动本监视进程
  - interval - { Number } 默认 5007，指定轮询频率（ms）
- listener - { Function } `(currentStats, previousStats) => {}`回调函数，参数是当前 stat 对象和之前的 stat 对象

```js
const fs = require("fs");
const path = require("path");

fs.watchFile(
  path.join(__dirname, "../../static/test2.txt"),
  (cur, prev) => {
    console.log('cur:%j', cur.mtime)
    console.log('prev:%j', prev.mtime)
  }
)
```

### fs.unwatchFile

`fs.unwatchFile(filename[, listener])`移除监听器

如果指定了 listener 则移除特定的监听器，否则移除所有的监听器。

```js
const fs = require("fs");
const path = require("path");

fs.unwatchFile(path.join(__dirname, "../../static/test2.txt"));
```

### 社区的监听方案

`fs.watchFile()`轮询的性能问题和`fs.watch()`在不同平台表现的差异问题不尽如人意。

> Node.js `fs.watch`:
>
> - MacOS 有时候不提供 `filename`
> - 在部分场景不触发修改事件（MacOS Sublime）
> - 经常一次修改两次触发事件
> - 大部分文件变化 eventType 都是 `rename`.
> - 未提供简单的监视文件树方式
>
> Node.js `fs.watchFile`:
>
> - 事件处理问题和 fs.watch 一样烂
> - 没有嵌套监听
> - CPU 消耗大

日常在监视文件变化可以选择社区的优秀方案

1. [node-watch](https://www.npmjs.com/package/node-watch)
2. [chokidar](https://www.npmjs.com/package/chokidar)

```js
const chokidar = require('chokidar');
 
// One-liner for current directory
chokidar.watch('.').on('all', (event, path) => {
  console.log(event, path);
});

const watcher = chokidar.watch('file, dir, glob, or array', {
  ignored: /(^|[\/\\])\../, // ignore dotfiles
  persistent: true
});
 
// Something to use when events are received.
const log = console.log.bind(console);
// Add event listeners.
watcher
  .on('add', path => log(`File ${path} has been added`))
  .on('change', path => log(`File ${path} has been changed`))
  .on('unlink', path => log(`File ${path} has been removed`));
 
// More possible events.
watcher
  .on('addDir', path => log(`Directory ${path} has been added`))
  .on('unlinkDir', path => log(`Directory ${path} has been removed`))
  .on('error', error => log(`Watcher error: ${error}`))
  .on('ready', () => log('Initial scan complete. Ready for changes'))
  .on('raw', (event, path, details) => { // internal
    log('Raw event info:', event, path, details);
  });
```

## 其他常用 API

`fs.constants` node 文件操作系统的常量存放对象

### fs.exitsSync 

`fs.exitsSync(path)` 判断路径是否存在，存在返回`true`，不存在返回`false`

```js
const fs = require('fs')
const path = require('path')

console.log(fs.existsSync(path.join(__dirname, '../../static/'))) // true
```

*注：不建议在在对文件操作前使用 fs.exists() 检查文件是否存在，这样做会引入竞态条件，因为其他进程可能会在两次调用之间更改文件的状态。相反应该直接对文件进行操作，如果文件不存在则处理引发的错误*

### fs.access

`fs.access(path[, mode], callback)` 测试用户对 path 指定的文件或目录的权限。

mode 可选值

| mode 可选值 | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| `F_OK`      | 方法默认值，表明文件对调用进程可见，可用于判断文件是否存在   |
| `R_OK`      | 表明调用进程可以读取文件                                     |
| `W_OK`      | 表明调用进程可以写入文件                                     |
| `X_OK`      | 表明调用进程可以执行文件，在 Windows 上无效（表现等同 `fs.constants.F_OK` |

```js
const fs = require('fs')
const path = require('path')

console.log(fs.existsSync(path.join(__dirname, '../../static/')))

fs.access(path.join(__dirname, '../../static/'), fs.constants.F_OK, err => {
  console.log('F_OK:', err) // 无错误返回表示可访问，文件存在
})

fs.access(path.join(__dirname, '../../static/'), fs.constants.W_OK, err => {
  console.log('W_OK:', err) // 无错误返回表示可写
})
```

### fs.copyFile 复制文件

`fs.copyFile(source, target[, mode], callback)` 将 source文件 拷贝到 target 目标文件下。 默认情况下，如果 target 已经存在则会覆盖，可以通过 mode 参数（[文件 copy 常量](http://nodejs.cn/api/fs.html#fs_file_copy_constants)）修改其行为。

Node.js 不保证拷贝操作的原子性， 如果在打开 dest 文件用于写入后发生错误，则 Node.js 将尝试删除 dest 文件。

| mode 可选值              | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `COPYFILE_EXCL`          | 如果 `target` 已经存在，则复制操作将失败                     |
| `COPYFILE_FICLONE`       | 复制操作将尝试创建写时复制引用链接。 如果平台不支持写时复制，则使用后备复制机制。 |
| `COPYFILE_FICLONE_FORCE` | 复制操作将尝试创建写时复制引用链接。 如果平台不支持写时复制，则该操作将失败。 |

```js
const fs = require('fs')
const path = require('path')

fs.copyFile(path.join(__dirname, '../../static/test.txt'), path.join(__dirname, '../../static/test4.txt'), err => {
  if (!err) console.log('复制成功')
})
```

### fs.rename 文件重命名

`fs.rename(oldPath, newPath, callback)` 将 oldPath 上的文件重命名为 newPath 提供的路径名，如果 newPath 已存在，则覆盖它

```js
const fs = require('fs')
const path = require('path')

fs.rename(path.join(__dirname, '../../static/test4.txt'), path.join(__dirname, '../../static/test44.txt'), err => {
  if (!err) console.log('改名成功')
})
```

### fs.unlink 删除文件

`fs.unlink(path, callback)` 删除常规文件或软链接，不能用于目录。指定文件不存在会报错。

```js
const fs = require('fs')
const path = require('path')

fs.unlink(path.join(__dirname, '../../static/test44.txt'), err => {
  if (!err) console.log('删除成功')
})
```
### fs.chmod 修改文件模式

`fs.chmod(path, mode, callback)`mode 使用八进制表示 `0o765` (0o 表示数字是八进制)，和 linux 文件权限规则一致

```js
const fs = require('fs')
const path = require('path')

fs.chmod(path.join(__dirname, '../../static/test2.txt'), 0o741, err => {
  console.log('chmod:', err)
})
```

