# HTTP 服务

[toc]



## HTTP 协议

**请求报文**

- 报文头
- 空行
- 报文主体

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211220171425997.png" alt="image-20211220171425997" style="zoom:50%;" />

**响应报文**

- 报文头
- 空行
- 报文主体

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211220171459068.png" alt="image-20211220171459068" style="zoom:50%;" />

## 常用首部

在请求和响应报文中有些很常用的首部字段

| **报文**                  | **含义**                                  |
| ------------------------- | ----------------------------------------- |
| Transfer-Encoding:chunked | 服务器传输大量数据时候分块发送            |
| Accept                    | 客户端可以处理的媒体类型                  |
| Accept-Encoding           | 客户端能理解的编码方式（gzip\br\deflate） |
| Cache-Control             | 客户端缓存控制                            |
| User-Agent                | 客户端信息                                |
| Content-Length            | 返回内容字节数                            |
| Content-Type              | 返回内容的媒体类型（MIME）                |
| Cookie                    | 写入客户端的 Cookie                       |

## 创建 HTTP 服务

Node.js 内部提供了`http`模块，基于`net`模块。`net`模块是对 TCP 协议的 API 实现。

#### 创建 web server

##### `server.listen(handle[, backlog][, callback])`

- handle - 可以是端口，套接字（底层具有`_handle`成员的对象），具有有效文件描述符的文件对象

```js
const http = require('http')

const server = http.createServer((req ,res) => {
  res.write('Hello Web') // 标准的可写流
  res.end()
})

server.listen(7878, () => {
  console.log('http://localhost:7878')
})
```

**req**

本次的 http request，可读流。常用属性如下

- url - { String } 请求地址
- method - { String } 请求方法
- headers - { Object } 请求头

**res**

本次的 http response，可写流，常用属性如下

- `writeHead(StatusCode[, StatusMessage[, headers]])` - 返回状态码、状态信息、响应头
- `write(chunk)` - 响应体中写入数据
- `end()`- 向服务器发送响应结束信号，每个响应都需要调用一次
- `getHeader(name)` - 返回指定 name 的 header
- `getHeaders()` - 返回 header 信息
- `setHeader(name, value)` -  设置响应头，和`writeHead()`合并，有冲突时使用`writeHead()`
- `statusCode` - 设置响应 HTTP status

**返回音乐流文件例子**

```js
const http = require('http')
const path = require('path')
const fs = require('fs')

const server = http.createServer((req ,res) => {
  let { url } = req
  try{
    url = decodeURIComponent(url)
  }catch(e) {

  }
  const filePath = path.join(__dirname,'../../static', url)
  fs.readFile(filePath, (err, chunk) => {
    if (err) {
      console.log(err)
      res.writeHead(404, {
        'content-type': 'text/html; charset=utf-8'
      })
      res.end(`文件不存在 url:${url}`)
      return 
    }
    res.writeHead(200, {
      'content-type': 'audio/flac',
      'content-disposition': 'inline',
    })
    fs.createReadStream(filePath).pipe(res) // 使用可读流持续传输
  })
})

server.listen(7878, () => {
  console.log('http://localhost:7878')
})
```

## 参数 queryString

在处理参数信息时，node 提供了`URL`模块来处理。URL 由不同的字段组成，`url` 模块提供了两套 API 来处理 URL：一个是旧版本遗留的 API，一个是实现了 [WHATWG标准](https://url.spec.whatwg.org/)的新 API。为了避免混淆，下面基于目前通用的 WHATWG 标准

```text
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              
```

#### URL 类

Node 中的 URL 类和浏览器的完全兼容

```js
URL === require('url').URL // true

new URL(input[, base]) // 实例化一个 URL 对象， base 是基础路径

url.password // 密码
```

还可以结合 path 模块中的方法进行解析

**url.searchParam**

返回 url 中的参数

- `url.searchParams.get(name)` - 获取参数
- `url.searchParams.set(name, value)` - 设置参数
- `url.searchParams.delete(name)` - 删除
- `url.searchParams.append(name, value)` - 追加参数
- `url.searchParams.forEach(fn[, thisArg])` - 遍历参数

```js
const myURL = new URL('https://example.org/?abc=123');
console.log(myURL.searchParams.get('abc')); // 123

myURL.searchParams.append('abc', 'xyz');
console.log(myURL.href); // https://example.org/?abc=123&abc=xyz

myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b');
console.log(myURL.href); // https://example.org/?a=b
```

## response 内容压缩

根据 http 协议，服务器会尝试把 response 内容进行压缩，提升传输速度，原理如下：

1. 浏览器发送请求，携带请求头`accept-encoding`标识压缩格式
2. 服务端选择压缩，并在响应头的`content-encoding`中指明压缩的格式
3. 浏览器得到相应后，一句`content-encoding`解压

#### NodeJs 压缩

node 内置`zlib`模块实现，提供了几种压缩方法，和浏览器的对应关系如下

| accept-encoding | zlib                        |
| --------------- | --------------------------- |
| gzip            | `zlib.createGzip()`         |
| deflate         | `zlib.createDeflate()`      |
| br              | `zlib.createBrotliCompress` |

zlib 创建的最终都是 Transform 双工流

```js
const { createGzip } = require('zlib')
const path = require('path')
const fs = require('fs')

const ws = fs.createWriteStream(path.join(__dirname, '../../static/xxx.flac.gz'))

fs.createReadStream(path.join(__dirname, '../../static/video/xxx.flac')).pipe(createGzip()).pipe(ws)
```

**判断是否需要压缩**

压缩会消耗 cpu，所以不是所有情况都需要压缩，比如 jpeg 图片。可以使用社区提供的`compressible`模块判断

```js
const compressible = require('compressible');

// 参数是 MIME
compressible('text/html'); // true
compressible('image/png'); // false
```

## 缓存

#### 客户端强缓存

- HTTP 1.0 - 使用`Expires`指定绝对时间，但由于不同地区的时区差异，会有问题
- HTTP 1.1 - 使用`Cache-Control`指定相对时间（秒， no-store：每次都重新下载内容；no-cache：每次都让服务器使用协商缓存重新校验）

**注意请求头和响应头里都有 cache-control、expires 字段。区别是**

1. 请求头里的决定这一次请求怎么走缓存，即使max-age 设置为 1000，也只影响这一次请求。下一次又回重新去判断，所以请求头里设置成1000 和 0 没有区别
2. 响应头里的则影响后面的请求怎么走缓存，才是真正的缓存方法

#### 协商缓存

如果请求时强缓存到期，但是服务器资源没变化，就可以使用协商缓存来高速浏览器使用本地资源（返回 304 状态码，不返回具体内容）

- HTTP 1.0 - 在第一次请求时响应头中写入`Last-Modified`，浏览器会在下次请求的请求头的`If-Modified-Since`中携带这个时间，服务器可以比对这两个时间来判断是否返回 304。缺点是1）文件修改但被撤销的情况下，修改时间会变，导致不使用缓存；2）文件周期性被修改但内容不变，修改时间也会变化
- HTTP 1.1 - 使用文件内容来判断缓存是否可用，使用 hash 处理城一个固定长度字符串，第一次请求时被写入` ETag`，浏览器第二次请求时放在请求头`If-None-Match`中。缺点是1）ETag 生成有开销；2）客户端对文件变化内容不敏感的时候，不需要这么精准的判断

**[ETag](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Etag)** 

`W/` 可选

`'W/'`(大小写敏感) 表示使用[弱验证器](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#Weak_validation)。 弱验证器很容易生成，但不利于比较。 强验证器是比较的理想选择，但很难有效地生成。 相同资源的两个弱`Etag`值可能语义等同，但不是每个字节都相同。

**ETag 一般是每次请求的时候都会重新计算生成，有些服务为了避免性能损耗可能也会选择缓存一段时间而导致一些问题。** 

不同的服务生成策略可能不太一样，nginx 使用 last-modified + content-length 计算生成`W/"627b3fe1-2dfff"`。627b3fe1 转 16 进制 * 1000，转换成日期就是 last-modified 时间。2dfff 直接转 16 进制就是 content-length。

**注意几种缓存的使用条件：**

1. 新打开一个标签页：从强缓存到协商缓存，一步步判断
2. 点击刷新按钮：js 文件依然从强缓存开始判断，other（如 html）则会跳过强缓存，走协商缓存，并且会在请求头中添加`cache-control: max-age=0`（请求头里的表示按照协商缓存的规则走）
3. ctrl+F5 强刷：跳过强缓存，并且在请求头中添加`cache-control: no-cache`和`pragma: no-cache`

**实际请求中会先判断 ETag 是否存在，不存在的情况下再去判断`Last-Modified`**

```js
// ETag
const etag = require('etag');

const reqEtag = headers['if-none-match'];
// 为了方便演示，使用同步方法读取响应内容，计算 etag，正常逻辑不会这么处理
const resEtag = etag(fs.readFileSync(filePath));
res.setHeader('ETag', resEtag); // 结合文件内容标记 Etag
res.statusCode = reqEtag === resEtag ? 304 : 200;

// Last-Modified
const lastModified = headers['if-modified-since'];
const mtime = fs.statSync(filePath).mtime.toUTCString();
res.setHeader('Last-Modified', mtime);
res.statusCode = lastModified === mtime ? 304 : 200;
```

