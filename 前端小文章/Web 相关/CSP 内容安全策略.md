# CSP

> CSP：Content Security Policy 内容安全策略

[toc]

## 基本介绍

网站的安全模式基于**同源策略**，理论上为我们的网站做了安全隔离。但是仍然有一些方法来突破这个策略，比如 XSS 脚本攻击。为了阻止这类问题，google 提出了 CSP，进一步增强浏览器中的安全问题。

#### 相关内容

- 使用白名单告诉客户端允许加载和不允许加载的内容
- 可使用哪些指令，这些指令接收哪些关键字（iframe - sandbox）
- **内联代码**和**`eval()`**被认为是不安全的
- 向服务器举报违规行为，以避免执行

## 白名单

浏览器自身无法区分哪些来源的代码是可信的，哪些是不可信的，所以 CSP 提供了 `Content-Security-Policy`响应头。

![image-20220116172721101](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220116172721101.png?x-oss-process=image/resize,w_600,m_lfit)

`Content-Security-Policy`用于指示浏览器创建白名单，而不是盲目信任服务器提供的内容

```text
Content-Security-Policy: script-src 'self' https://apis.xxx.com
```

除了通过响应头外，通过 meta 标签也可以指定该策略（不能配置`frame-ancestors`、`report-uri`或` sandbox`。）

```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self' https://apis.xxx.com;">
```

*meta 标签 http-equiv 的作用*：https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta#attr-http-equiv

**多个值采用空格分隔，如果重复声明则只有第一次声明的有效**

#### 策略配置

**外部资源相关**

- **script-src** 

  限制外部脚本文件的访问。

  - `self`表示允许自己的域名访问
  - `none`表示不允许任何外部脚本
  - url 地址时，如`https://apis.xxx.com`，匹配对应 url 来源的访问，支持通配符
  - `unsafe-inline`：允许内联脚本的访问，不能单独设置，除非也配置了`nonce`
  - `unsafe-eval`：允许 eval、setTimeout、Function 等执行
  - `nonce`：http 返回一个 token，内嵌脚本必须有这个 token 才会执行
  - `hash`：列出允许脚本访问的 hash 值，内嵌脚本 hash 符合的情况下才会执行

- **style-src** ：外部样式表加载，选项和`script-src`相同

- **img-src**  ：图像来源

- **media-src**：媒体文件来源

- **font-src**：字体文件来源

- **child-src**：嵌入的工作线程和 frame 的来源，如`child-src https://youtube.com`将允许`youtube`视频内容的嵌入。替代已经废弃的`frame-src` 

- **object-src**：内嵌脚本的来源，比如 flash 插件、`<object src=>""`src 来源

- **connect-src** 

  允许的连接请求

  - XHR
  - WebSockets
  - EventSource

- **worker-src**：worker 脚本来源

- **manifest-src**：manifest 文件来源

**default-src**

`default-src`用于配置上面各选项的默认值

**注意如果没配置`default-src`，那么`script-src`和`object-src`是必须的** 

**URL 限制**

- **form-action** 用于限制可从 `<form#action>` 标记提交的有效地址
- **base-uri** ：限制在页面的 `<base#href>` 元素中配置的地址
- **frame-ancestors**： 用于指定可嵌入当前页面的来源。此指令适用于 `<frame>`、`<iframe>`、`<embed>` 和 `<applet>` 标记。此指令不能在 `<meta>` 标签中使用，并仅适用于非 HTML 资源
- **upgrade-insecure-requests**: **指示 User Agent 将 HTTP 更改为 HTTPS，重写网址架构。 该指令适用于具有大量旧网址（需要重写）的网站** 

例如，有一个从 cdn（例如，`https://cdn.example.net`）加载所有资源的应用，并清楚的知道不需要任何 frame 内容或插件，则白名单配置可能如下:

```
Content-Security-Policy: default-src https://cdn.example.net; child-src 'none'; object-src 'non
```

#### 选项值

- 主机名：`example.org`，`https://example.com:443`
- 路径名：`example.org/resources/js/`
- 通配符：`*.example.org`，`*://*.example.com:*`（表示任意协议、任意子域名、任意端口）
- 协议名：`https:`、`data:`
- 关键字：`'self'`：当前域名，需要加引号
- 关键字：`'none'`：禁止加载任何外部资源，需要加引号

在网络上的各种教程可能会看到 `X-WebKit-CSP` 和 `X-Content-Security-Policy` 标头。 这些都是比较老的标准，可以直接忽略掉。

## sandbox 指令

上面的`Content-Security-Policy`限制的是外部文件来源，`sandbox`则用来限制哪些指令是可以执行的，就是 iframe 标签上的`sandbox`一样的。

iframe sandbox 参考：https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe

**如果`sandbox`指令存在则将此页面视为使用`sandbox`属性在 iframe 标签内部加载的。** 

## 内联脚本 & `eval()`

#### 内联脚本

有了白名单之后可以明确指定接受的资源，但是 XSS 攻击最大的威胁是内联脚本注入。而 CSP 解决此问题的**唯一有效方式**就是完全禁止内联脚本。

此禁止规则不仅包括在 `script` 标记中直接嵌入的脚本，也包括内联事件处理程序和 `javascript:URL` 

要想不被阻止就只有两种方法：

1. 手动开启配置允许内联脚本的执行（不安全）
2. 将内联的脚本文件改写为外部 js 并通过 src 属性引入（更好的做法）

**如果一定要使用它 ...**

CSP Level 2 可为内联脚本提供向后兼容性，即允许使用一个加密随机数（数字仅使用一次）或一个哈希值将特定内联脚本列入白名单。尽管这可能很麻烦，但它在紧急情况下很有用。

要使用随机数，请为您的 script 标记提供一个随机数属性。该值必须与信任的来源列表中的某个值匹配。 例如:

```html
<script nonce=EDNnf03nceIOfn39fn3e9h3sdfa> // 示例写死，用的时候要随机生成
  //Some inline code I cant remove yet, but need to asap.
</script>
```

将随机数添加到已追加到 `nonce-` 关键字的 `script-src` 指令。

```text
Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
```

#### `eval()`

因为`eval`、`new Function`、`setTimeout([string], ...)`等方法会直接执行传入的字符串，所以他们是很危险的。CSP 的策略是完全阻止这类方法的执行。

## report-uri 向浏览器报告违规行为

`report-uri`用于告诉浏览器将此类行为报告给哪个地址

```text
Content-Security-Policy: report-uri /may-xss-collection;
```

浏览器会发送如下 JSON 对象

```json
{
  "csp-report": {
    "document-uri": "http://example.org/page.html",
    "referrer": "http://evil.example.com/",
    "blocked-uri": "http://evil.example.com/evil.js",
    "violated-directive": "script-src 'self' https://apis.google.com", // 违反的指令
    "original-policy": "script-src 'self' https://apis.google.com; report-uri http://example.org/may-xss-collection"
  }
}
```

## 参考文章

Google CSP：https://developers.google.com/web/fundamentals/security/csp 

Content-Security-Policy：https://www.ruanyifeng.com/blog/2016/09/csp.html