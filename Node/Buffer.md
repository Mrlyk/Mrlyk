# Buffers

[toc]

## 二进制数据

计算机只认识 0 和1，在初期 js 没有引入 TypedArray 之前没办法操作二进制数据。后来 js 中引入了 TypedArray 类型化数组来描述一个底层的二进制数据缓冲区（binary data buffer）的一个类数组视图。TypedArray 是指代一类类型化的数组，可能是下面的其中之一，它本身并不是一个构造函数。

```
// TypedArray 指的是以下的其中之一：

Int8Array();
Uint8Array();
Uint8ClampedArray();
Int16Array();
Uint16Array();
Int32Array();
Uint32Array();
Float32Array();
Float64Array();
```

有了这个类型化的描述数组视图，我们就可以用它提供的 API 来操作 TCP 流、文件操作系统等要处理二进制字节的场景。

**Node.js 中基于 Buffer 类构建了 Unit8Array API，以供 node 使用。**

**Byte 与 bite**

- 1Byte = 8bit 即一个字节等于8个字符。bit 是二进制最小信息单位，64位的 cpu 一次能处理2^64 的数组
- Byte 是**计量**存储或者传输流量的单位，一个英文字符是一个字节也就是 1B。中文通常是 2B，取决于编码类型。由于 node.js 默认 utf-8 编码，所以一个中文占 3 个字节。

Buffer 处理的字节是 2^8 = 256，从 0 开始计数，Buffer 中的 255 标识的是一个每位都是 1 的字节

## Buffer 特性

Buffer 构建的实例类似于 0 到 255 之间的整型数组（其他整数通过 & 255（截断以符合范围 0–255，取模运算，超出后256 又变成了 00，257 -> 01）**数组采用16进制，每项对应一个 ASC2 编码**） 操作强制转换到此范围）。**Buffer 是一个 js 和 C++ 结合的模块，对象不经 V8 分配，而是由 C++ 申请，js 分配。缓冲区的大小在创建时确定，不能调整**

Buffer 对象过于常用所以直接被放置到了全局变量中。

## 实例化 Buffer

在 node.js V6 之前是通过调用构造函数的方式实例化 Buffer，根据参数返回不同结果。后来由于安全原因，在 v6 后改为提供了三个 API 来处理。

### Buffer.from

Buffer.from 支持四种参数类型

- Buffer.from(string [, encoding])：返回一个包含给定字符串的 Buffer
- Buffer.from(buffer)：返回给定 Buffer 的一个副本 Buffer
- Buffer.from(array)：返回一个内容包含所提供的字节副本的 Buffer，数组中每一项是一个表示八位字节的数字，所以值必须在 0 ~ 255 之间，否则会取模(取两个数相除的余数)
- Buffer.from(arrayBuffer)：返回一个与给定的 ArrayBuffer 共享内存的新 Buffer
- Buffer.from(object[, offsetOrEncoding[, length]])：取 object 的 valueOf 或 Symbol.toPrimitive 初始化 Buffer

```js
const buf1 = Buffer.from('test', 'utf-8')
const buf2 = Buffer.from(buf1) // 创建副本，修改 buf1 不会影响 buf2
const buf3 = Buffer.from([0, 257, 1,16])
const arr = new Uint16Array(2) // 创建长度为 2 的 unit16 类数组，超出长度会被截断
arr[0] = 10
arr[1] = 20
const arr2 = Buffer.from(arr.buffer) // 创建和 arr 共享内存的 buffer2，修改其中一个另一个也会被影响

const buf = Buffer.from(new String('test')) // 从对象的 valueOf 创建一个 Buffer实例

console.log(buf1)
console.log(buf2)
console.log(buf3)
console.log(arr)
console.log(buf)
```

### Buffer.alloc

`Buffer.alloc(size[, fill [, encoding]])`分配一个`size`字节大小的新 Buffer，如果`fill`为 undefined 则用 0 填充。

- size - { Integer } 新 Buffer 的长度
- fill - { String | Buffer | Unit8Array | Integer } 预填充 Buffer 的值，默认 0 
- encoding - { String } fill 是字符串时这一选项作为编码，默认 utf-8

```js
const bufAlloc = Buffer.alloc(2, 1)
console.log(bufAlloc) // <Buffer 01 01>
```

`Buffer.alloc`永远不会使用内部的 Buffer 池（**`Buffer.poolSize`，内部分配的内存池大小，默认8Kb**）。

### Buffer.allocUnsafe

和`Buffer.alloc`类似，执行速度更快，但是不安全。在 size 小于 `Buffer.poolSize`的一半时会使用内部的 Buffer 池。分配一个尚未初始化（归零）的空间，速度虽然快，但是可能存在旧数据。**不推荐使用。**

### Buffer.allocUnSafeSlow

和上述两个方法也类似，但是是直接通过 C++ 进行内存分配。不会进行旧值填充，`size`小于`Buffer.poolSize`的一半（4Kb）时会直接使用预分配的 Buffer，速度比`Buffer.alloc`要快，大于的时候无差距。

### 编码

Buffer 目前支持以下几种编码格式

- ascii
- utf8
- utf16le
- base64
- binary
- hex

### Buffer 和 String 转换

`buf.toString([encoding[, start, end]])`

- encoding - { String } 编码
- start - { Number } 开始位置
- end - { Number } 结束位置

```js
const buf = Buffer.from(new String('test')) // 从对象的 valueOf 创建一个 Buffer实例

buf.toString() // test
```

### Buffer.concat

`Buffer.concat(list[, totalLength])`拼接多个 Buffer 为一个新的实例。新实例与老的不共享内存

```js
const bufConcat = Buffer.concat([buf1, buf3])
```

### StringDecoder

在 Node.js 中一个汉字由三个字节表示，如果处理中文的时候字节不是 3 的倍数就会乱码

```js
for (let i = 0; i < bufChinese.length; i+= 5) {
  let b = Buffer.alloc(5)
  bufChinese.copy(b, 0, i)
  console.log(b.toString())
}
/*
中�
�字�
��串
*/
```

Node.js 提供了内置的 StringDecoder 模块来处理这个问题，StringDecoder 在拿到编码后，在处理末尾不全的字节时会保留到第二次`write()`。**目前只能处理 utf-8、Base64 和 ucs-2/utf-16le**

```js
const bufChinese = Buffer.from('中文字符串')
const StringDecoder = require('string_decoder').StringDecoder // 拿到构造函数
const decoder = new StringDecoder('utf-8') // 传入编码，构造解码实例

for (let i = 0; i < bufChinese.length; i+= 5) {
  let b = Buffer.alloc(5)
  bufChinese.copy(b, 0, i)
  console.log(decoder.write(b)) // write 方法输出
}
```

### Buffer 其他常用 API

- `Buffer.isBuffer()` - { Boolean } 判断是否 Buffer 对象
- `Buffer.isEncoding(encoding)` - { Boolean } 判断 Buffer 对象是否支持该编码
- `buf.length` - Buffer 实例字节数，**申请时的并不是实际内容的**
- `buf.indexOf()` - 和数组的类似，返回位置
- `buf.copy(target[, targetStart[, souurcreStart[, sourceEnd]]])` - 将 buf 实例的内容复制到目标 buf