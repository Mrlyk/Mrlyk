# 广告是如何跟踪我们的？所有关于 cookie

> 廖宜康，Happy every day 前端开发工程师！

## 前言

作为前端开发，`cookie`是我们经常需要打交道的东西。我们用它来鉴权，用它来实现行为跟踪，用它给无状态的 http 协议以“状态”。本文就聚焦这个小小的 `cookie`，把 `cookie` 掰开了，揉碎了讲一讲， 它是如何被我们利用的。

本文以 **Chrome 浏览器 96.0.4664.55 版本作为客户端环境**，后续所有代码说明都基于该版本的浏览器。主要阐述以下四点：

- `cookie` 的现有属性
- `cookie` 如何被用来跟踪我们
- `cookie` 前端管理实践
- `cookie` 的未来

话不多说，开冲！

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_570fabbf-f403-42d0-b122-e00875aaa1dc.png" alt="企业微信截图_570fabbf-f403-42d0-b122-e00875aaa1dc?x-oss-process=image/resize,w_200,m_lfit" style="zoom: 33%;" align="left"/>

## 一、cookie 的属性们

在详细的属性说明之前，我们先让新老朋友重新熟悉一下 `cookie`。

`cookie` 官方定义：~~魔法曲奇饼，不是~~

```text
An HTTP cookie (web cookie, browser cookie) is a small piece of data that a server sends to a user's web browser. The browser may store the cookie and send it back to the same server with later requests. Typically, an HTTP cookie is used to tell if two requests come from the same browser—keeping a user logged in, for example. It remembers stateful information for the stateless HTTP protocol.
```

由官方定义可知，`cookie` 是一个存储在用户端设备上的小块数据。这里的定义指的是由 HTTP 请求的响应头`set-cookie`创建的 `cookie`。现在我们也可以使用 `document.cookie`来访问和手动创建第一方 `cookie`。

### 1.1 为什么要 `cookie`

大家都知道，因为有需求所以才有市场，技术也一样，`cookie` 就是这样出现的。`cookie` 的由来是著名的 Netscape 在给客户开发电子商务程序时，客户要求服务端**不必须**存储事务状态。那没办法，服务端不想存，只能客户端出力了。由此 `cookie` 诞生了。
我们常用的 http 协议是无状态的协议，这也是为什么他这么快的原因之一。但很多时候我们需要知道是谁给我们发过来的请求，**我们需要记录用户状态**。

所以 `cookie` 诞生的原因：

- 服务端不想存状态
- 我们需要存状态

可能有人会说，那我这个状态存在`sessionStorage`里行不行，存在`localStorage`里行不行，甚至我自己放个自定义的参数放请求头里来表示状态行不行。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211202170526958.png" alt="image-20211202170526958" style="zoom:50%;" align="left"/>

既然这么说了，当然是行的，浏览器发展到现在，很多存储方式被用来替代 `cookie`，但是 `cookie` 仍然存在其独特性。比如同域之间可共享，服务端可设置......所以具体怎样的方案取决于你的实际场景。

### 1.2 属性详解

打开浏览器的`devtool`面板

![devtool-cookie](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211125135854750.png)

可以看到，上面有 `cookie` 发展至今的所有属性，那这些属性分别代表什么呢？

**`cookie` 属性说明表**

| 属性名    | 属性说明                                                     |
| --------- | ------------------------------------------------------------ |
| name      | `cookie` 的名字                                              |
| domain    | `cookie` 所属域，`domain`表明哪些域名下可以使用该 `cookie`，重要的属性。 |
| path      | `cookie`的使用路径，`path`标识指定了主机下的哪些路径可以接受 `cookie`。 |
| Max-Age   | `cookie`有效期，单位秒。如果为整数，则该`cookie`在`Max-Age`秒后失效。 |
| Expires   | `cookie`的失效时间，如果`cookie`没有设置过期时间，那么 `cookie` 的生命周期只是在当前的会话中，关闭浏览器意味着这次会话的结束，此时 `cookie` 随之失效。现在已经被maxAge属性所取代，需要是一个日期对象。 |
| HttpOnly  | 属性可以阻止通过javascript访问`cookie`。document.`cookie`读取到的内容不包含设置了HttpOnly的`cookie`， 从而一定程度上遏制这类攻击。 |
| secure    | 它是一个布尔值，指定在网络上如何传输`cookie`，默认为false，通过一个普通的 http 连接传输，标记为 true 的`cookie`只会通过被 HTTPS 协议加密过的请求发送给服务端。 |
| SameSite  | 限制第三方url是否可以携带`cookie`。有3个值：Strict/Lax(默认)/None。 |
| SameParty | Chrome 新推出了一个 `First-Party Sets` 策略，它可以允许由同一实体拥有的不同关域名都被视为第一方。之前都是以站点做区分，现在可以以一个 party 做区。`SameParty`就是为了配合该策略。（*目前只有 Chrome 有该属性*） |
| Priority  | 优先级，chrome的提案（firefox不支持），定义了三种优先级，Low/Medium/High，当`cookie`大小超出浏览器限制时，低优先级的`cookie`会被优先清除。（*目前只有 Chrome 有该属性*） |

有人可能觉得啊，就这，就这一个表？是的，就这么多...是不可能的。既然要掰开了，揉碎了讲，当然要对每一个属性做详细的说明。这些点主要是关于平常我们使用 `cookie` 时候要注意的，也就是一些坑。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211202170950847.png" alt="image-20211202170950847" style="zoom:33%;" align="left"/>

不废话了，让我们一个个开始讲。（***倾斜字体***的的属性表示前端可以通过 js 直接操作的属性）

***name***

name 主要注意同名的问题，`cookie` 设置时不同 domain，不同 path 可以使用相同的 name。如果 domain、path 相同，那么后设置的 `cookie` 会覆盖先设置的。

除此之外还要注意一点，通过 js 操作`cookie`时，第一项永远对应的是`name=value`。即`document.cookie="path=/;name=test"`这样设置的结果**不会**是我们看上去的在`/`路径下配置了一个 value 为`test`的`cookie`。而是设置了一个`cookie`的 name 是`path` value 是`/`。

***value***

`cookie` 的值，仅支持字符串，如果使用其他类型，会调用 `toString() `方法进行转换。

***domain***

domain 要注意的点有很多，我们可以用它来实现前端跨域访问，单点登录（二级域名同域），跟踪用户.....主要要注意的有以下几个

- js 设置 `cookie` 配置 `domain`时，只能设置第一方的。如在 example.com 使用`js`设置` cookie`，如`document.cookie = token=123;domain=test.com`是不会生效的。
- js 设置`cookie`配置 `domain`时，如上一个例子，只要是手动配置该属性，无论这个属性前面有没有写明**“."**符号，都会自动加上 `domian=.test.com`，被配置通配域名的形式。
- js 获取`cookie`时，目前通用的访问方式仍然是`document.cookie`这一个 API，所以在获取的时候我们是不知道当前的`cookie`是具体是属于哪一个`domain`的。

***path***

- 设置`cookie`的该属性时，该属性的必须存在于当前`url`的`pathname`中，和`domain`类似，否则无法设置。
- 设置 `path` 时如果设置配置的是相对路径，则会自动匹配设置为完整的`pathname`,如当前`url`为`https://test.com/ab/cd`，设置`document.cookie = token=123;;path=ab`，那么最终`path`就会是`/ab/cd`。
- 大家都知道`cookie`会在发送`http`请求时自动放入请求头。如果`cookie`设置了该属性，则会去匹配请求的路径中是否包含该属性的值，包含则会发送这个`cookie`。这里要注意一下`path`采用的是匹配的模式。如`path`为`/test`，那么`/testtest`也会匹配到。

这里还要注意下，设置`path`时，值要包含在当前`url`。但是我们最常用的`ajax`请求一般不是当前路径，所以会携带不上这个`cookie`。要注意区分，不是当前路径下的所有请求都会携带，而是请求的地址包含这个`path`才会携带。

`domain` 和 `path` 在设置 `cookie`时还会有一个很重要的作用，**影响`cookie`的排序**。将设置`cookie`的方式分为 4 种情况:

1. 设置`cookie`时没有携带`path`或`domain`
2. 设置`cookie`时携带了`path`
3. 设置`cookie`时携带了`domain`
4. 设置`cookie`时既携带了`path`携带了`domain`

最终的结果是通过`document.cookie`获取`cookie`时结果会按照上面的情况分组排序，并按照 1 - 3 - 2 - 4 的顺序分组排序。什么意思呢？就是情况 1 配置的 `cookie`一定是在情况 3 之前，纵使我是先通过情况 3 设置的`cookie`。而不是我们以为的先设置的一定在前，后设置的一定在后。如下图展示的应该比较明白了。

![image-20220114111712347](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220114111712347.png)

**提升排序会影响到的是我们的匹配删除行为，我们要明确的知道排序才能确定两个同名不同 `path` 的路径哪个才是我们要删除的**。

***Max-Age***

`cookie`的删除都是利用过期时间来实现，将其过期时间`Max-Age`设置为`<= 0`的整数则可以删除该`cookie`。

- `Max-Age`如果设置的是非整数，则过期时间默认为`session`，即到页面关闭即失效。
- `Max-Age`的时间是秒，设置为 10，则表示 10 秒后过期，是符合我们的预期的。注意和下面讲的`expires`的区别。

***Expires***

`Expires`也是表示的过期时间，只不过相对`max-age`其出现的时间更早（对 IE8 以下有更好的兼容性）。现在**更推荐使用**`Max-Age`。使用`Expires`来设置过期时间有时候会出现一些我们困惑的情况。

- `Expires`首先接受的是一个`Date`对象，其次特别要注意的是，`cookie`的过期时间是基于`UTC`时间。而国内的标准时间是 `GMT`时间，比`UTC`时间要快 8 小时。我们通过该属性设置过期时间时，如`expires=${new Date()}`。这里看着是把过期时间设置为了当前，`cookie`会立即失效，但是我们实际是把`cookie`的过期`UTC`时间设置为了当前时间，距离`cookie`真正失效还有 8 小时。有时我们通过`expires`去清除`cookie`时却没有清掉可能就是这个原因。
- 当我们同时设置`Expires`和`Max-Age`时，`Max-age`具有更高的优先级。

**HttpOnly**

该属性主要是用来做基础的 xss 攻击防御，js 在设置`cookie`时是无法配置该属性的。只有服务端通过`set-cookie`响应头返回的字段才能配置该字段。但是这也只是防君子罢了，不用太过依赖该属性。

***Secure***

`secure`和`HttpOnly`一样，都是为了`cookie`安全打出的组合拳。和`HttpOnly`不同的是，它允许通过 js 配置。

**这里插入一点我在实践中发现的问题**。有时候我们需要内网部署一些应用，通过前置机 ng 转发请求的形式来访问云端的服务。而访问前置机一般是直接访问`ip`的形式，这时候肯定不是 https 协议了。但是云端服务的响应头`set-cookie`中如果含有`secure`属性。（1）虽然可以通过 ng 传回来，但是是写不到内网机器里面的。（2）出现这种问题我们的处理方案通常是在 ng 这一层处理掉响应头的`secure`。这里要注意一下**360 浏览器**会检测到这种行为（不清楚具体原理），仍然无法写入`cookie` 。当前版本的 Chrome 则可以成功写入，不排除未了来 Chrome 更新阻止的可能。

***SameSite***

本属性是 chrome 51 新增的一个重要属性。三个值的意义如下

- **Strict**: 仅允许发送同站点请求的的`cookie`。
- **Lax**: 允许部分第三方请求携带`cookie`，即导航到目标网址的 get 请求。包括超链接 ，预加载和 get 表单三种形式发送`cookie` 。chrome 80+ 版本后，默认就是该值。
- **None**: 任意发送`cookie`，设置为 None，必需要同时设置`Secure=true`，也就是说网站必须采用 https。

这里要区分一下跨站（cross-site）和跨域（cross-origin）两者不是一个概念。跨域是指 portal、host、port 之中的任意一个不同都被视为跨域。而跨站则比较宽松，只要二级域名相同就是同站（二级域名指 .com 这种顶级域名的下一级，如 test.com）。

默认配置该属性为`Lax`之后，主要影响的应该是我们的 post 表单、iframe、ajax 和 image，大家要注意。

**SameParty**

`SameParty`是 Chrome 为了 `cookie`的安全第三记组合拳。主要用来配合`First-Party`策略。所有开启了  `First-Party Sets` 域名下需要共享的 `Cookie` 都需要增加 `SameParty` 属性。如`Set-Cookie:name=test;Secure;SameSite=Lax;SameParty`。`SameParty`自身是没有值的，但是设置他必须设置了`Secure`且`SameSite`不能是 `strict`

First-Party 策略在后面“Chrome 是如何做的”中会详细说明。

**Priority**

上面说了很多可能有点啰嗦，这个属性就。。。没什么好说的，看上面的表格。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211203180800389.png" alt="image-20211203180800389" style="zoom:50%;" align="left"/>

## 二、cookie 如何被用来跟踪我们

上面一大段介绍看完，大家应该对现在的`cookie`有了全面的了解。那`cookie`又是如何被我们利用的呢？

`cookie`最广泛的用途大家肯定都知道，无非传递传递鉴权信息，做做单点登录，这也是用的最多的用法。
但是`cookie`自诞生以来就是各大广告商窥探用户隐私的利器。例如我今天在某猫搜了个 xx杯，第二天贴吧，微博到处都是这个东西的推销广告。仿佛全世界都知道我看了这个东西（事实上他们确实知道）。那么他们是如何做到的呢？

### 2.1 第三方 cookie

了解普遍的做法之前，我们先区分一下第一方`cookie`和第三方`cookie`。我们一般认为`cookie`的 domain 存在于当前域名或当前域名的父级的`cookie`称为第一方`cookie`，否则为第三方`cookie`。虽然很多情况下“第三方”的`cookie`也是我们主动注入的罢了。

如下图以某宝举例：

![image-20211207140153653](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207140153653.png)

`.taobao.com`下的就是第一方`cookie`，而这个`.mmstat.com` 很明显就是第三方`cookie`了。

### 2.2 广告与隐私

上面说到第三方`cookie`，可以说它是各种网络广告的罪魁祸首，而且是最便捷成本最低的那种。我们继续**以上面的某宝举例**。大家都能发现这个`.mmstat.com`这个域名。那这个网站是干什么的呢。

我首先去百度了下这个域名，域名本身无法访问，但是在百度的结果下面倒是发现一些好玩的词条😅。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207145541629.png" alt="image-20211207145541629" style="zoom:50%;" align="left"/>

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220113142701417.png" alt="image-20220113142701417" style="zoom:50%;" align="left"/>

实际上他并不是什么不健康的网站，他是阿里旗下的一个广告营销平台。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220113143222051.png" alt="image-20220113143222051" style="zoom:50%;" align="left"/>

这里不是打广告...只是让大家知道他是做广告营销的就行了。

那他是如何运作的呢，我们可以根据请求来看。

**标记用户**

首先我们打开某宝，可以看到第一次访问 mmstat.com 这个域名是下面这个请求：

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/resize,w_800,m_lfit.png" alt="mmstat.gif" align="left"/>

加载了一张几乎不可见的 gif 图片，同时通过 set-cookie 将第三方`cookie`写入。

之后又请求了这张 gif 图片。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207165534282.png" alt="image-20211207165534282" align="left">

将第一次获得的信息传入，完成了用户的标记，营销平台成功的知道了你是谁。当你打开其他接入营销平台的网站时，比如视频播放站某酷，你会看到你们存在一个相同的标记 ID。营销平台知道是你小子刚才打开了我家的某宝然后又跑来某酷看电视来了。

**获取用户足迹**

我们知道用户投广告，希望看到的是广告的高转化率，那如何提高转化率呢？最好的方式自然是将商品推荐给需要他的人群。那如何知道用户需不需要这个商品呢？自然是记录用户的某宝搜索记录，浏览记录。将这些数据标记下来，就可以获得商品的目标人群。

我们依然以某宝举例，当我点进一个商品时，发送了这样 5 个请求

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207170929304.png" alt="image-20211207170929304" align="left"/>

最后一个请求没什么好说的，就是传入了一个 tmall 的跳转路径，因为我点击的商品在 tmall 而不是某宝。我们重点看一下前四个请求。

根据 `cookie`的基本作用我们知道，这四个请求都携带了我们第一次打开某宝时所写入的`cookie`信息，也就是标记我们是谁的信息，同时在这四个请求里传入了一些参数来标记我们的行为，我们拉出来一个个分析下。

我们从上面的图中可以看到，第一个和第二个，第三个和第四个请求几乎是一摸一样的。首先看下第一个和第二个请求

![image-20211207172355635](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207172355635.png)

<center>第一个请求参数 1-1</center>

![image-20211207172946596](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207172946596.png)

<center>第二个请求参数 1-2</center>

可以看到两个请求参数几乎是相同的，唯一的不同在于第一个请求的 gokey 中多了一个 `aws=1`的参数，当然不是某宝的开发我们自然是不知道每个参数的作用的，但是我们可以通过参数中包含的信息来推测。这里可以看到`_p_url`这个参数中包含的信息时比较明显的。

从这个参数的值来分析，发现这个参数包含的信息就是**我们是如何找到这个商品的**。参数中明确的写出我们是通过 `s.taobao.com`来搜索 `q=%E8%83%8C%E5%8C%85`到达此页面的，q 的值就是我搜索的商品名“背包”。同时链接上还携带了一些其他参数，比如`sourceId`等。通过这些参数和一开始植入的第三方`cookie`，这里营销平台就知道是"你在某宝首页通过搜索背包进入的该商品页"。那么你就被标记为了“一个需要背包的用户”。搜索作为一种主动行为，占的权重肯定是比较高的，那这个信息就是非常有用的。

第三个请求和第四个请求的情况类似于第一、第二个，只不过包含的信息更详细。第三个和第四个请求的地址相同，参数也几乎相同，唯一的不同也是`aws`这个值。

我们以第三个分析一下：

![image-20211207175805940](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207175805940.png)

可以看到，大量信息被放入 gokey 中。

![image-20211207175854625](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207175854625.png)

转码后可以看到一些被发送的信息，例如

- serach_radio_all
- GongYingLianDIsts
- list_model
- isp4p
- item_click_form

上述信息大家通过定义的名称也大致能猜出其包含的数据信息。通过以上信息营销平台就记录了你的这些行为：

- 你打开了淘宝
- 你在淘宝首页搜索了背包
- 你在第一页点进了一款背包
- 你只是看了看没有购买
- ......

有了这些信息之后，当我们打开接入了营销平台广告的网站，比如某酷。你会发现他也有`mmstat.com`的第三方`cookie`吗，并且存在一个`id`信息和你在逛某宝的时候是相同的。这时候广告平台就知道了：你小子刚才想买个包，现在又跑到某酷看剧来了，等会就给你推送点包的广告，嘿嘿嘿！

而这一切的基础就是通过一个小小的第三方`cookie`来对我们进行标记，完成用户画像，再将这些信息整合把商品推送给我们吗，那赚我们的钱就像“抢劫”一样。本来你只是产生了买个包的想法但是没决定到底买不买，结果人家通过广告投放不间断的告诉你买个包吧，买个包吧，你的钱就这样花出去了。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211207181537855.png?x-oss-process=image/resize,w_200,m_lfit" align="left"/>

**总结**

简单总结一下广告跟踪你基本就是三步走

1. 首次访问网站时通过第三方`cookie`植入将你标记。
2. 浏览网站时通过植入的`cookie`不断的向营销平台发送你的浏览足迹（几乎精确到了每一步操作，一个点击，一个停滞都会被记录）。
3. 在访问其他社交网站时会值入相同的第三方`cookie`将你标记，同时给你推送相关广告。

具体到实现的操作会比这复杂的多，希望大家可以根据上方的分析有所了解。

到这里大家可能就觉得有一种被监控的感觉。上述只是对购物网站的基础分析，像我们日常用的最多的百度，微博等都接入了类似的广告营销平台，再加上广告联盟的存在，可以说我们每个人在网络上的每一步足迹都是被记录在案的。只是这些数据可能分散在不同的厂商，加上国家监管，各大厂商也不会肆无忌惮。不过被监控的感觉始终是不舒服的，这也是为什么 Google 收到了超 100 亿美元的关于隐私问题的罚款。而这一切都源于一个小小的`cookie`。

**注：以上是我根据请求对广告行为与`cookie`相关的一些分析，对某宝的分析仅仅是举例，大家知道广告平台是怎么操作的即可。**

## 三、cookie 前端管理实践

说完了`cookie`的基本属性与对我们的日常影响，我来谈谈在实践中我是如何管理`cookie`的，供大家参考。

### 3.1 前端 cookie 常见问题

在聊如何做的之前我们先说说我在`cookie`上遇到过的一些问题：

- 主域名内嵌子项目，`cookie` 在不同项目间的行为难以统一

- `cookie` 鉴权相关操作在不完全了解的情况下不敢轻易修改

- 不同项目使用公司统一的二级域名情况下，因为 cookie 的 name 相同可能造成的混乱

- 项目中`cookie`如果某个属性要调整或者项目要扩展子系统，可能会有遗漏

- 公司域名更改，`cookie`相关信息也要一并更改，可能遗漏

上述的问题不外乎三点 (1)`cookie`管理混乱；(2)`cookie`后续维护困难；(3)不同项目之间的 `cookie`冲突。

### 3.2 cookie 管理

既然存在这些问题那就想办法解决，我这边的做法是在数据初始化时进行管理 cookie，前端拦截未定义`cookie`的操作。

我想要管理好项目中所有的`cookie`,甚至处理好多项目同二级域名的问题，那就在项目初始化的时候就定义好所有要用的`cookie`。之后所有对`cookie`的操作都是对这些已定义的`cookie`进行处理。

首先我们建立一个对象来声明我们项目中要使用的所有的`cookie`

```js
// cookie-schema.js
{
  cookie1: {
    name: 'cookie1', // cookie 名称
    domain: 'root', // root-根域名 sub-当前域名及子域名 current-当前域名（默认值）
    path: '/', // 匹配路径
    expires: 30 * 24 * 3600 * 1000, // 过期时间 30d
    secure: false // https 传输
  },
  cookie2: {
    name: 'cookie2',
    domain: 'sub',
    expires: 30 * 24 * 3600 * 1000, // 30d
    path: '/cookie-test',
    secure: false
  }
}
```

如上，`cookie-schema.js`中就是我们项目中所有要使用的`cookie`。这个可以是前端直接自己声明，如果要处理不同项目之间冲突的问题，也可以专门建立数据中台。将所有项目可用的`cookie`放在后台配置，在项目初始化时去加载项目的可用`cookie`。

感兴趣的同学可以配合使用我的这个小工具，有详细说明：

**Cookie-Util：**https://github.com/Mrlyk/cookie-util

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211209112028981.png" alt="image-20211209112028981" style="zoom:50%;" align="left"/>

## 四、cookie 的未来

目前由于`cookie`带来对种种隐私问题，各大浏览器厂商也开始限制第三方`cookie`。其中 safari 已经完全禁止了第三方`cookie`，但是对市场影响最大的依然是 Chrome 还没有这么做，所以大家的感受不是很强烈。

### 4.1 Google 是如何做的

Google 由于`cookie`带来的侵犯隐私问题，已经面临超 100 亿美元的罚款。因此 Google 在 2020 年初宣布 Chrome 将在未来的**两三年中完全禁用第三方 cookie**。当然阻力是很大的，广告行业的抵制声音最大，可以理解😄

到现在已经快两年过去了，可以看到 Google 对`cookie`也是动作不断。从 Chrome 51 推出 SameSite，到 89 版本推出 `First-Party Sets` 策略，还有`Trust Tokens`（这个笔者还未使用过）一步步在增加对三方`cookie`的限制，同时对历史问题提出解决方案。

**First-Party Sets**

在上面介绍广告是如何跟踪我们的说明中，我们可以清楚的看到`mmstat.com`和`taobao.com`不是一个同一个域名，但是他们却来自同一家企业。这种三方`cookie`可以看作是我们主动接入的，我们是不希望他们被浏览器干掉的。所以 Google 推出了`First-Party Sets`策略，配合 Chrome 的`Saome-Party`属性，我们可以允许一个第一方`party`内的域名之间的`cookie`互相访问。

当然为了防止该策略被滥用，Google 也做出了如下限制：

- `First-Party Sets` 中的域必须由同一组织拥有和运营。
- 所有域名应该作为一个组被用户识别。
- 所有域名应该共享一个共同的隐私政策。

同时，该策略不允许在不相关的站点之间交换用户信息，所以单点登录还是需要使用其他方案。

要使用该方案可以通过提供定义当前域名与其他域名的关系的清单文件来声明它们是一个`party`的成员（或所有者），文件需要是位于`.well-known/first-party-set`路由下的 JSON 文件

以 Google 官方的例子，假设`a.example`, `b.example`, `c.example`希望形成一个由`a.example`拥有的 `first-party`。那就需要声明如下 JSON 文件：

```json
// https://a.example/.well-known/first-party-set
{
  "owner": "a.example",
  "members": ["b.example", "c.example"],
  ...
}

// https://b.example/.well-known/first-party-set
{
	"owner": "a.example"
}

// https://c.example/.well-known/first-party-set
{
	"owner": "a.example"
}
```

**注意：该方案还在试用阶段，chrome 89 ~ 93 可以试用**

对 Google 的隐私策略有兴趣的同学可以看一下这篇开者文档：https://developer.chrome.com/docs/privacy-sandbox/ 

### 4.2 cookie Store API

>  The Cookie Store API provides an asychronous API for managing cookies, while also exposing cookies to [`service workers`](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API).

官方也新增了对`cookie`操作的新 API，相对于原来使用`document.cookie`繁琐操作，这个 API 更加的方便，但是只能在 HTTPS 协议的下使用，同时兼容性还很差。使用方法类似于我上方的**Cookie-Util**工具，感兴趣的同学可以看一下下面的官方文档。

官方文档：https://developer.mozilla.org/en-US/docs/Web/API/Cookie_Store_API

### 4.3 第三方 cookie 何去何从

透过上面的一系列分析，可以看出官方在未来主要打击的是第三方`cookie`。不久 Chrome 也要完全禁用这种第三方`cookie`。那么没了这些第三方`cookie`对我们的业务有什么影响呢？

- 单点登录，如果之前是依赖三方`cookie`做淡点登录的同学要特别注意寻找新方案了
- 异常追踪工具，通常使用第三方`cookie`来标记用户。被禁止后可能会造成 UV 暴涨的问题
- 用户行为分析工具，失去这个标记后可能会失效
- .......

当然我们回到`cookie`作用的本质，是对用户进行标记。那我们可以不可以用其他方法来标记用户解决一些问题呢？当然是可以的，比如已经有很多人在用的**浏览器指纹**。

从普通用户的角度上说，我们当然希望自己的隐私被保护的更好。但是隐私这堵墙也会协助这些巨无霸公司把自己的壁垒筑的越来越高。比如Google 推出 FloC 模型来分析用户群体的行为，当第三方`cookie`被屏蔽，广告商要想好好的打广告还得找回这些公司去使用他们的方案。

从技术的角度，我们更应该关注的是这些行业规范定制者的规范变更，对任何规范的变动保持警惕，及时处理对自己项目产生的风险。

**全文完**

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211209144001002.png" alt="image-20211209144001002" style="zoom:50%;" align="left"/>

## 参考

HTTP cookies: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/cookies)

Wiki cookie: https://en.wikipedia.org/wiki/HTTP_cookie

当浏览器全面禁用三方 cookie: https://mp.weixin.qq.com/s?__biz=Mzk0MDMwMzQyOA==&mid=2247490361&idx=1&sn=ebc8dcc4d095cc7ba748827dff158f2b&source=41

详解 cookie 新增的 SameParty 属性: https://juejin.cn/post/7002011181221167118