# TCP 

> TCP - Transmission Control Protocol 传输控制协议，是一种面向连接的、可靠的、基于字节流的传输层通信协议。

我们日常和 HTTP 打交道，都知道 HTTP 是建立在 TCP 基础上的协议。所以我们有必要深入了解一下 TCP，以排查日常中的各类奇葩网络问题。

[toc]

## 基本介绍

TCP 是端口到端口到传输协议，下面是 TCP 报文组成

![image-20220524135215302](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524135215302.png?x-oss-process=image/resize,w_800,m_lfit) 

- 源端口和目标端口：各占两个字节（1Byte = 8bit ）共 32 位
- 32 位序列号 **sequence number（后文简称 SN）**，传输时都以序列号为标识，是一个相对值，**按顺序编号（记住这一点）**。例如，一段报文的序号字段值是 301 ，而携带的数据共有100字段，显然下一个报文段（如果还有的话）的数据序号应该从401开始
- 32 位确认序列号 **acknowledgment number（后文简称 ack，注意是小写，大写是下面的 Character）**，值为 32 位序列号 + 1，也是一个相对值，确认建立传输。接着上面的例子，确认序列号就会返回 401，告诉发送者期望下一个收到的序号从 401 开始
- 上图红框部分即是标识位，一般我们会用到其中的 SYN、ACK、FIN

  - **SYN**: synchronize 即请求建立同步连接的标识（标识连接的开始）
  - **ACK**: ACKnowledge Character 确认字符，**用于标识 acknowledgement number 是否有效：1 有效 0 无效，注意不要和 acknowledgement number 搞混了**
  - **FIN**: finish 结束连接的标识
  - URG: 当URG=1，表明紧急指针字段有效。告诉系统此报文段中有紧急数据。配合后面的 16 位紧急指针使用（**已废弃**）
  - PSH: 推送，当两个应用进程进行交互式通信时，有时在一端的应用进程希望在键入一个命令后立即就能收到对方的响应，这时候就将PSH=1
  - RST: 重置，当RST=1，表明TCP连接中出现严重差错，必须释放连接，然后再重新建立连接
- 16 位窗口：指的是通知接收方，发送本报文你需要有多大的空间来接受
- 16 位检验和：校验首部和数据这两部分
- 选项：长度可变，定义一些其他的可选的参数

了解以上基础知识后，接下来我们来看看我们知道但却印象不深刻的 TCP 建立连接过程：3 次握手-4次挥手。

*由于浏览器的 devtool 无法观测到 TCP 建立连接到过程，我们使用另外的抓包工具 wireshark 进行抓包*

## 建立连接

首先我们使用 node 在本地的 4000 端口启动一个服务，然后使用浏览器来访问。

简单的服务如下

```javascript
const Express = require('express')

const app = Express()
app.get('/', (req, res) => {
  res.send('hello world')
})

app.listen(4000, () => {
  console.log('listening on port 3000')
})

```

然后需要配置 wireshark 来监听 4000 端口，以抓到浏览器的请求

![image-20220524141558890](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524141558890.png?x-oss-process=image/resize,w_800,m_lfit) 

在配置好之后，刷新一下浏览器即可抓到我们的请求了

![image-20220524141644481](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524141644481.png?x-oss-process=image/resize,w_800,m_lfit)  

#### 三次握手

从上面的图中我们可以看到，连接请求从 54560 端口发出到 4000 端口。这个 54560 端口很明显就是浏览器的端口了，

##### 第一次握手

首先浏览器向服务端口发起连接请求，发送了一个 SYN 标识，**告诉服务器我想要建立连接了**。

![image-20220524142229994](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524142229994.png?x-oss-process=image/resize,w_800,m_lfit) 

同时在 TCP 报文中**携带了 32 位序列号 sequence number**。其相对值为 0 ，绝对值是 864621156（所以其实际值也是这个）

##### 第二次握手

接着就到了第二个 TCP 请求，服务器向浏览器发送了一个 SYN 和 ACK 表示可以建立连接。 

同时返回了建立连接需要的数据：

- 确认序列号 acknowledgement number，相对值为 1，绝对值是 846421157。**刚好是第一次握手的 sequence number + 1**
- 序列号 sequence number 

![image-20220524142952785](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524142952785.png?x-oss-process=image/resize,w_800,m_lfit) 

![image-20220524143245071](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524143245071.png?x-oss-process=image/resize,w_800,m_lfit) 

**也可以看到，在连接信息中 ACK = 1，标识 acknowledgement number 有效** 

![image-20220524153556353](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524153556353.png?x-oss-process=image/resize,w_600,m_lfit) 

##### 第三次握手

第三次握手，浏览器收到服务器的响应后，又发出了最终确认序列号 ack，值为第二次握手返回的  SN 的值 + 1：2268843895。

在连接上也可以看到连接确认字符 ACK 也是 1 ，标识连接有效

![image-20220524144031125](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524144031125.png?x-oss-process=image/resize,w_800,m_lfit) 

同时可以看到 SN 也和服务器同步了

这样就成功建立了同步连接

 ![image-20220524152321274](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524152321274.png?x-oss-process=image/resize,w_600,m_lfit) 

接下来再看看 4 次挥手是怎么做的。

#### 四次挥手

断开链接从 FIN 标志开始

![image-20220524153850174](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524153850174.png?x-oss-process=image/resize,w_800,m_lfit) 

##### 第一次挥手

第一次浏览器请求断开向服务器发送了 FIN 结束标识和 ACK = 1 标识 （**这里的 ACK 标识主要是为了应答上一次连接**，保证连接正常）。

![image-20220524154018469](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524154018469.png?x-oss-process=image/resize,w_600,m_lfit) 

其中 SN 的值为 1236591869。那么推测一下，第二次挥手返回的 ACK 肯定是 1236591869 + 1 = 1236591870

##### 第二次挥手

第二次挥由服务端发送给浏览器确认标识，确认断开链接并返回 ack，确实是我们上面计算的值。同时发送了 SN 。

![image-20220524154225014](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524154225014.png?x-oss-process=image/resize,w_600,m_lfit) 

##### 第三次挥手

第三次挥手依然由服务器给浏览器发送信息，而且 SN 和 ack 是完全相同的。这里可能就有点好奇，为什么不在上一次就直接送 FIN 结束标识而是要有这第三次挥手呢？

**因为如果服务器此时还在传输数据，当接收到浏览器的 FIN 请求时就立刻断开可能导致数据丢失。但是如果不响应浏览器请求浏览器又会继续发送断开请求（即 FIN 报文）。所以为了避免这种情况， TCP 协议要求服务端立刻返回一个 ACK 告诉浏览器我已经收到了断开的请求，浏览器接收到之后进入 FIN_WAIT_1 状态。 **

待数据都传输完成后（如果有的话），再进行这第三次挥手。再次发送 FIN 和 ACK 标志告知浏览器现在可以结束了。

 ![image-20220524155247364](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524155247364.png?x-oss-process=image/resize,w_600,m_lfit) 

上图可以看到第三次挥手 SN 和 ACK 都是一摸一样的，区别是第三次携带了 FIN 标识。

##### 第四次挥手

浏览器收到服务端发来的 FIN 标识和 SN 后，在用其中 SN 校验完成，连接就正式断开了。

![image-20220524155418256](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524155418256.png?x-oss-process=image/resize,w_600,m_lfit) 

可以看到 ack 的值是 985699148 = 985699147 + 1。验证完成，连接正式断开。

#### 总结

TCP 通过 SYN、ACK、FIN 标识来表示连接的开始、确认和结束。同时通过 SN 和 ack 的对应关系来确认连接是否成功。

三次挥手不必多说，**四次挥手实际上是可以合并成三次的，只需要客户端和服务端同时向对方发送 FIN 标识，那么就可以三次挥手也能断开链接。** 

在实际应用中，通常会开启 keep-alive，这样一次 TCP 连接上可以进行多个 http 响应。

**注：keep-alive 控制的是浏览器的行为，服务端依然会发送 FIN 结束标识，只不过浏览器不会响应。连接也就一直不会断开。**

## HTTP 关联

上面我们看到 TCP 通过 SN 和 ack 的值来确认连接，同时还可以在一次 TCP 连接的基础上建立多个 http 连接。那基于 TCP 的多个 http 连接又是如何建立关联的呢？

我们查看抓包到的两个http 请求

![image-20220524155828901](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524155828901.png?x-oss-process=image/resize,w_400,m_lfit) ![image-20220524155855484](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220524155855484.png?x-oss-process=image/resize,w_400,m_lfit) 

上面左边是请求，右边是响应。

可以看到请求片段的长度是 771，SN 是 1236491098。对应的响应头 ack 的值正好是 1236491098 + 771 = 1236491869。

是不是有点奇怪？ack 的值不应该是 SN + 771 + 1 才对吗？这里为什么没有加 1 呢？而相对值确实是 771 + 1 = 772。

**这一点是比较特殊的一点：只有在建立连接和断开链接的阶段（Segment Len 为 0）这时为了对应上请求才会 + 1（TCP 把 SN 按顺序标号不能重复）。在数据传输阶段 ack = SN + Segment Len** 。

总之我们也知道了 **HTTP 请求和响应关系也是通过 SN 和 ack 的关联建立的，这也就为多个 http 请求并行提供了基础，保证不会乱响应** 

*抓包中还有一个  TCP Window Update 的连接与滑动窗口有关，放到以后的细节文章里一起说明*

## 参考文章

理清 HTTP 之下的 TCP 流程：https://mp.weixin.qq.com/s/NhOhl5YlxEUtXjySZFazGQ