# ArrayBuffer

> ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区，是最基础的原始**数据容器**。

官方的定义不是很明了，通俗的讲，ArrayBuffer 是一串**有固定长度的、连续的内存地址**，存储了一段二进制数据。

有人说计算机上什么不是二进制的？确实，计算机上的数据本质都是二进制的。但是 ArrayBuffer 作为一个浏览器提供的对象，具有上面说的长度固定且连续的特性，给我们操作这类数据提供了便利。

ArrayBuffer 本身是不能读写的，只能通过视图（Typed Array View | Data View）来读写；所以要想在前端操作二进制数据，就需要依赖这两个视图提供的方法。最经典的例子就是将 dataUrl 转成 blob 对象再下载。

##  ArrayBuffer 和 blob 的关系

这里提到了 blob 对象，顺便说一下 ArrayBuffer 和 blob 的关系

> `Blob` 对象表示一个不可变、原始数据的**类文件对象**。

blob 对象可以从字符或者 **ArrayBuffer** 创建而来，成为我们想要的格式的对象，比如`image/png`、`application/pdf`。我们也可以通过 `FileReader` 这个 api 将 blob 转回 ArrayBuffer！

我们可以通过视图来操作 ArrayBuffer 但是却不能操作 Blob 对象，他是一个不可变的的的类文件对象，只有一个 `slice` 方法。从这个角度而言 ArrayBuffer 是一个底层的数据对象！

回到 ArrayBuffer 对象本身，我们说了 ArrayBuffer 本身是不可读写的，只能通过视图来操作。那么我们就来看看两种试图分别如何处理！

## Typed Array

Typed Array 是一系列的视图集合，包括

`Int8Array`：8位有符号整数，长度1个字节，范围是`[-128 ~ 128]`。

`Uint8Array`：8位无符号整数，长度1个字节，范围是`[0 ~ 255)`。

`Uint8ClampedArray`：8位无符号整数，长度1个字节，溢出处理不同，是针对 Canvas 元素的专有类型。

`Int16Array`：16位有符号整数，长度2个字节，范围是`[-32768 ~ 32767]`。

`Uint16Array`：16位无符号整数，长度2个字节，范围是`[0 ~ 65536)`。

`Int32Array`：32位有符号整数，长度4个字节。

`Uint32Array`：32位无符号整数，长度4个字节。

`Float32Array`：32位浮点数，长度4个字节。

`Float64Array`：64位浮点数，长度8个字节。

*ps: 不要把 U int 看成 Unit，int 是整数，Uint 是无符号整数！*

看到定义可能很多人还是不明白表达的是什么意思，我们以 Uint8Array 做说明。

#### Uint8Array 到底是什么

首先我们知道在计算机中，一个字节有 8 位:

形象表示：`[0] [0] [0] [0] [0] [0] [0] [0]`  我们将这一个8位的连续内存当作一个“基本长度”。

下面我们创建一个 Uint8Array：

```js
const u8 = new Uint8Array(5) // 创建一个 Uint8Array 实例，长度是 5，未赋值之前 5 位都是 0，具有 5 个“基本长度”

u8[0] = 1 // 给第一位赋值
u8[1] = 2 // 给第二位赋值

// 如果赋超过 255 的值，会被进位后舍去。比如 520 -> 255 进位 * 2 -> 520 - 255 * 2 -> 10，最后就是 10
```

创建完成后（二进制的）：

- 第一位形象表示：`[0] [0] [0] [0] [0] [0] [0] [1]`
- 第二位形象表示：`[0] [0] [0] [0] [0] [0] [1] [0]`

所以 Uint8Array 就是这样在内存中的一串二进制数据。那他有什么用呢？

我们说了计算机中的所有数据都是二进制的，Uint8Array 就是计算机中的数据，而且可以是任意类型的数据，可以是普通的数字，也可以是文件，更多的我们用它来处理文件。普通数字、字符有自己的字面标识符不需要使用到它。

回到开头我们说的一个常见示例：将 dataUrl 转成 blob 对象，一般我们会将编码取出转换成二进制数据

```js
const [binaryType, base64Str] = dataUrl.split(',')

const binaryStr = window.atob(base64Str)
```

ps:

- `window.btoa`：binary to ascii，二进制数据用 ascii 码编码——编码
- `window.atob`：ascii to binary，ascii 码编码的数据解析回原始的二进制数据——解码

要想变回 blob 对象，我们需要通过`Blob`构造函数来处理，`Blob`构造函数，接收 Typed Array 作为参数，那么问题来了，**这么多类型的 Typed Array，我们用哪一个呢？**

这取决于**要构造的数据他们能表示的最大值**（比如英文单字符，一个字节，8位就能表示完，最多只到 127，用 Uint8Array 足以）和他们匹配上即可，示例中的 binaryStr 最大能表示 255 个字符，所以我们只需要取无符号的 8 位整数，也就是 Uint8Array。（这里就明白了为什么一些工具方法中总是把 dataUrl 的数据通过 Uint8Array 来构造了吧！）

**小知识点：**

> 在 js 中，binary string 是一种字符集，类似于 ascii、unicode 字符集，用来表示二进制数据。
>
> 在 ascii 中，对于单字节字符，一个字节是 8 位，除去符号位最多 127 个字符。
>
> 在 binary string 中，没有符号位，所以能表示 255 个字符。(2^8 - 1)
>
> UTF-8、UTF-16、UTF-32 中的 "UTF" 是 "Unicode Transformation Format" 的缩写，意思是"Unicode 转换格式"，后面的数字表明至少使用多少个比特位来存储字符, 比如：UTF-8 最少需要8个比特位也就是 1 个字节来存储，对应的， UTF-16 和 UTF-32 分别需要最少 2 个字节 和 4 个字节来存储

**可能有人好奇那我不能用 Uint16Array 来处理吗？**答案是可以的，只不过 Uint16Array 一个字符会用两个字节来表示，纵使一个字节就能表示完，另一个字节也会以 0 的形式存在内存中，导致内存占用翻倍。

下面继续 dataUrl 到 blob 的转换：

```js
const u8a = new Uint8Array(asciiStr.length)

for(let i = 0; i++; i <= asciiStr.length) {
  u8a[i] = asciiStr.charCodeAt(i) // charCodeAt 给定字符对应的 玛点，直接给字符是不行的，Typed Array 中的每一位都是数字，记住他是一个整数型数组，给非数字会被视为 0 
}

const blob = new Blob([u8a], { binaryType })
```

*ps: 玛点，二维表中行与列交汇的点，码点值(即码点编号)通常来说就是其所对应的字符的编号。如果想知道具体指可以百度一下对应的编码表😄*

至此就解开了我们的疑惑，为什么 dataUrl to blob 总是用 Uint8Array 作为中间人？因为他是最合适的。

## Data view

DataView 可以理解为对 TypedArray 的封装，更简单易用一些。

#### 创建一个Data View

```arduino
const buffer = new ArrayBuffer(8);
const view = new DataView(buffer);
```

原型上有以下方法用于写入和读取，

- getInt8/setInt8： 
- getUint8/setUint8
- getInt16/setInt16
- getUint16/setUint16
- getInt32/setInt32
- getUint32/setUint32
- getFloat32/setFloat32
- getFloat64/setFloat64

**示例代码如下：**

```javascript
const buffer = new ArrayBuffer(16);
const view = new DataView(buffer);

// 赋值
view.setInt8(0, 1);
view.setInt16(1, 32767);

// 取值
view.getInt8(0); // 1
view.getInt16(1); // 256 
view.getInt16(1); // 32767 用二进制表示就是 0111 1111 1111 1111，第一位为符号位
view.getInt8(1); // 127 如果用 8 位来获取 16 位的整数的数据，多余的部分会被舍去 -> 0111 1111 
```

这里说一下 `getInt8` 和 `getInt16` 之间的转换关系。

在上面的示例代码中，我们给 8 位的整型数组第一个值赋值为了 1，再通过 8 位去取的时候也仍然是 1。但是当我们通过 16 位去取的时候值是 256，为什么不是 1 呢？这里涉及到一个概念“字节序”！

>字节序，或字节顺序（"Endian"、"endianness" 或 "byte-order"），描述了计算机如何组织字节，组成对应的数字。
>
>大部分需占用多个字节的数字排序方式是 **little-endian**（译者注：可称小字节序、低字节序，即低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。与之对应的 big-endian 排列方式相反，可称大字节序、高字节序），所有的英特尔处理器都使用 little-endian。

所以在 8 位中，1 用二进制表示就是 `0000 0001`

而在 16 位中，由于我们使用低字节序，1 用二进制表示就是`0000 0001 0000 0000`，低位的在前面，转换成十进制就是 256 了！

## 总结

ArrayBuffer 是一个可以操作的，更接近底层的二进制数据容器，使用它我们可以对文件数据进行一些转换和处理。计算机数据的本质就是二进制的，ArrayBuffer 让我们感受到了这种本质。我们更多的是在开发中要知道什么时候，怎么样合适的去访问和操作他！

## 参考文章

10分钟带你了解什么是ArrayBuffer？：https://juejin.cn/post/7176079047699464253