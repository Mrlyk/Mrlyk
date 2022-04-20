# puppeteer

> [Puppeteer](https://link.zhihu.com/?target=https%3A//github.com/GoogleChrome/puppeteer/) 是 Chrome 团队出品用于操作 headless 无界面浏览器的 API 工具
>
> 无界面(无头)浏览器

[toc]

## 什么是 Headless Chrome

Headless Chrome 是 Chrome 浏览器的无界面形态，可以在不打开浏览器的前提下，使用所有 Chrome 支持的特性运行你的程序。相比于现代浏览器，Headless Chrome 更加方便测试 web 应用，获得网站的截图，做爬虫抓取信息等。相比于出道较早的 PhantomJS，SlimerJS 等，Headless Chrome 则更加贴近浏览器环境。

#### 作用

**除了下面举例的，基本只要手动操作的都可以用 puppeteer 自动完成**

- 生成页面的屏幕截图和 PDF。

- 抓取 SPA（单页应用程序）并生成预渲染内容（即“SSR”（服务器端渲染））。

- 自动化表单提交、UI 测试、键盘输入等。

- 创建最新的自动化测试环境。使用最新的 JavaScript 和浏览器功能，直接在最新版本的 Chrome 中运行测试。

- 捕获站点的[时间线跟踪](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)以帮助诊断性能问题。

- 测试 Chrome 扩展。

  ......
  
## 安装

```shell
npm install puppeteer -S
```

会自动下载一个 chromium 以用来提供 API

## 引入

**注意：puppeteer 是运行在 node 服务端端程序，而不是客户端（浏览器），所以不要想在静态应用中安装使用它，而是在静态应用中访问他。同时通过服务端与客户端通信，获取需要的信息**

具体概念可以查看：https://stackoverflow.com/questions/55031823/how-to-make-puppeteer-work-with-a-reactjs-application-on-the-client-side

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211210141746958.png?x-oss-process=image/resize,w_800,m_lfit" alt="image-20211210141746958" style="zoom: 67%;" />

- **Browser**：对应一个浏览器实例，一个 Browser 可以包含多个 BrowserContext
- **BrowserContext**： 对应浏览器一个上下文会话，就像我们打开一个普通的 Chrome 之后又打开一个隐身模式的浏览器一样，BrowserContext 具有独立的 Session(cookie 和 cache 独立不共享)，一个 BrowserContext 可以包含多个 Page
- **Page**：表示一个 Tab 页面，通过 browserContext.newPage()/browser.newPage() 创建，browser.newPage() 创建页面时会使用默认的 BrowserContext，一个 Page 可以包含多个 Frame。
- **Frame**: 一个框架，每个页面有一个主框架（page.MainFrame()）,也可以多个子框架，主要由 iframe 标签创建产生的
- **ExecutionContext**： 是 javascript 的执行环境，每一个 Frame 都一个默认的 javascript 执行环境
- **ElementHandle**: 对应 DOM 的一个[元素节点](https://www.zhihu.com/search?q=元素节点&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A76237595})，通过该该实例可以实现对元素的点击，填写表单等行为，我们可以通过选择器，xPath 等来获取对应的元素
- **JsHandle**：对应 DOM 中的 javascript 对象，ElementHandle 继承于 JsHandle，由于我们无法直接操作 DOM 中对象，所以封装成 JsHandle 来实现相关功能
- **CDPSession**：可以直接与原生的 CDP 进行通信，通过 session.send 函数直接发消息，通过 session.on 接收消息，可以实现 Puppeteer API 中没有涉及的功能
- **Coverage**：获取 JavaScript 和 CSS 代码覆盖率
- **Tracing**：抓取性能数据进行分析
- **Response**： 页面收到的响应
- **Request**： 页面发出的请求

#### 启动方式

创建 Browser 实例有两种方式

- puppeteer.launch: 每次都启动一个 Chrome 实例
- puppeteer.connect: 连接一个已经存在的 Chrome 实例

```javascript
const puppeteer = require("puppeteer");

//使用 puppeteer.launch 启动 Chrome
(async () => {
  const browser = await puppeteer.launch(); // 可以使用选项 如 launch({headless: false}) 以非无头模式启动（相当于开一个浏览器启动）
  const page = await browser.newPage();
  page.on("console", (msg) => console.log(msg.type(), msg.text())); // 监听 console 输出
  await page.goto("https://www.baidu.com/"); // 页面跳转
  await browser.close(); // 关闭浏览器
})();


//使用 puppeteer.connect 连接一个已经存在的 Chrome 实例
(async () => {
    //通过 9222 端口的 http 接口获取对应的 websocketUrl
    let version = await request({
        uri:  "http://127.0.0.1:9222/json/version",
        json: true
    });
    //直接连接已经存在的 Chrome
    let browser = await puppeteer.connect({
        browserWSEndpoint: version.webSocketDebuggerUrl
    });
    const page = await browser.newPage();
    await page.goto('https://www.baidu.com');
    await page.close();
    await browser.disconnect();
})();
```

两种方式的对比：

- puppeteer.launch 每次都要重新启动一个 Chrome 进程，启动平均耗时 100 到 150 ms，性能欠佳
- puppeteer.connect 可以实现对于同一个 Chrome 实例的共用，减少启动关闭浏览器的时间消耗
- puppeteer.launch 启动时参数可以动态修改
- 通过 puppeteer.connect 我们可以远程连接一个 Chrome 实例，部署在不同的机器上
- puppeteer.connect 多个页面共用一个 chrome 实例，偶尔会出现 Page Crash 现象，需要进行并发控制，并定时重启 Chrome 实

#### 启动选项

```js
{
	headless: false, // 是否无头模式启动，默认 true
	slowMo: 250, // 减缓操作的速度，这里是 250ms
	timeout: 2000, // 等待浏览器实例启动的最长时间 (以毫秒为单位)。默认为 30000(30 秒)。传递 0 以禁用超时
  dumpio: false, // 是否将浏览器进程 stdout 和 stderr 通过管道传输到 process.stdout 和 process.stderr 中。默认为 false。要获取控制台输出需要开启此选项
  devtools: false, // 是否为每个选项卡自动打开 DevTools 面板。如果此选项为 true，则 headless 选项会被设置为 false
  args: args.concat(['--remote-debugging-port=9223']) // 启动浏览器时携带的额外参数
}
```

**禁用下面这些不常用的方法可以加快启动速度**

```js
args: [
  '–disable-gpu',
  '–disable-dev-shm-usage',
  '–disable-setuid-sandbox',
  '–no-first-run',
  '–no-sandbox',
  '–no-zygote',
  '–single-process'
]
```

#### 使用自定义 Chromium

默认情况下，Puppeteer 下载并使用特定版本的 Chromium 以及其 API 保证开箱即用。 如果要将 Puppeteer 与不同版本的 Chrome 或 Chromium 一起使用，在创建`Browser`实例时传入 Chromium 可执行文件的路径即可：

```js
const browser = await puppeteer.launch({executablePath: '/path/to/Chrome'});
```

**还有一种借用其他依赖的方式，待补充（cloud-print-node）**

## puppeteer API

**具体 API 可查看官方文档：https://github.com/puppeteer/puppeteer/blob/v11.0.0/docs/api.md**

**下面列举一些常用的 API**

#### 查找元素

1. `page.$(selector)` - 相当于 `document.querySelector`
2. `page.$$(selector)` - 相当于 `document.querySelectorAll`
2. `page.$x('//img')` - 获取某个 xPath 对应的所有元素

注意：返回值不是 DOM，而是自己封装过的` Promise<ElementHandle>` ，[ElementHandle](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-class-elementhandle) 封装了常用的 click 、boundingBox 等方法。

```js
const puppeteer = require('puppeteer');

puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  await page.goto('https://google.com');
  const inputElement = await page.$('input[type=submit]');
  await inputElement.click();
  // ...
});
```

#### 获取元素

使用上面查找元素的方法不能获取 DOM，要获取的话需要使用下面的方法

1. `page.$eval(selector, pageFunctions[, ages])`
2. `page.$$eval(selector, pageFunctions[, ages])`

`pageFunctions`会在浏览器实例中执行，所以可以用 Window 等 dom 对象；其返回值是整个方法的返回值

```js
const searchValue = await page.$eval('#search', el => el.value);
const html = await page.$eval('.main-container', e => e.outerHTML);
const divsCounts = await page.$$eval('div', divs => divs.length);
```
1. **`page.evaluate(pageFunction [, ...args])` 是上述方法的抽象，可以在浏览器示例中执行任意方法**
2. ElementHandle 实例 可以作为参数传给 page.evaluate

```js
const bodyHandle = await page.$('body');
const html = await page.evaluate(body => body.innerHTML, bodyHandle); // bodyHandle 会被作为参数传入第一个回调函数。也可以不传，执行自己想植入的 js 方法
```

#### 键盘

Puppeteer 通过 `page.keyboard` 对象暴露操作键盘的接口

- [keyboard.down(key[, options])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-keyboarddownkey-options) 
- [keyboard.press(key[, options])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-keyboardpresskey-options) 
- [keyboard.sendCharacter(char)](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-keyboardsendcharacterchar) 
- [keyboard.type(text, options)](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-keyboardtypetext-options) 
- [keyboard.up(key)](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-keyboardupkey) 

```js
await page.keyboard.type('Hello World!', {delay: 100});
await page.keyboard.press('ArrowLeft');

await page.keyboard.down('Shift');
for (let i = 0; i < ' World'.length; i++)
  await page.keyboard.press('ArrowLeft');
await page.keyboard.up('Shift');

await page.keyboard.press('Backspace');
// 结果字符串最终为 'Hello!'
```

方法看名字就知道什么意思，type 和 sendCharacter 作用非常类似，区别是

- sendCharacter 会触发 `keypress` 和 `input` 事件，不会触发 `keydown` 或 `keyup` 事件
- type 会触发`keydown`, `keypress`/`input` 和 `keyup` 事件

#### 鼠标

Puppeteer 通过 `page.mouse` 对象暴露操作键盘的接口

- [mouse.click(x, y, [options\])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-mouseclickx-y-options)
- [mouse.down([options\])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-mousedownoptions)
- [mouse.move(x, y, [options\])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-mousemovex-y-options)
- [mouse.up([options\])](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-mouseupoptions)

```js
// 使用 ‘page.mouse’ 追踪 100x100 的矩形。
await page.mouse.move(0, 0);
await page.mouse.down();
await page.mouse.move(0, 100);
await page.mouse.move(100, 100);
await page.mouse.move(100, 0);
await page.mouse.move(0, 0);
await page.mouse.up();
```

#### tap

手机页面经常使用 tap 事件，用 page.mouse.click() 是不能触发的，需要使用专门的 tap API

- `touchscreen.tap(x, y)` - 触发 touchstart 和 touchend 事件
- `page.tap(selector)` - `touchscreen.tap(x, y)` 的快捷方式，不用自己去定位

#### 页面跳转控制

1. `page.goto(url, options)`
2. `page.goback(options)`
3. `page.goForward(options)`

几个页面跳转的 API 非常简单，options 的 `waitUntil` 参数用来指定满足什么条件认为页面跳转完成，如果值为事件数组，那么所有事件触发后才认为是跳转完成。事件包括：

- `load` - 页面的load事件触发时（默认值）
- `domcontentloaded` - 页面的 DOMContentLoaded 事件触发时
- `networkidle0` - 不再有网络连接时触发（至少500毫秒后）
- `networkidle2` - 只有2个网络连接时触发（至少500毫秒后）

```js
await page.goto('https://www.baidu.com', { waitUntil: ['load'] })
```

#### 重设页面内容

可以复用一个标签页，否则打开多个标签页会产生性能问题

```js
page.setContent(html, {  
  waitUntil: 'networkidle0'
});
```



#### 事件支持

Puppeteer 提供了对一些页面常见事件的监听，用法和 jQuery 很类似

- [page.on('close')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-close)
- [page.on('console')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-console) - 页面调用 `console`某个方法时触发
- [page.on('dialog')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-dialog) - 当js对话框出现的时候触发，比如`alert`, `prompt`, `confirm` 或者 `beforeunload`。Puppeteer可以通过[Dialog](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-class-dialog)'s [accept](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-dialogacceptprompttext) 或者 [dismiss](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-dialogdismiss)来响应弹窗。
- [page.on('domcontentloaded')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-domcontentloaded) 
- [page.on('error')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-error) 
- [page.on('frameattached')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-frameattached) 
- [page.on('framedetached')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-framedetached) 
- [page.on('framenavigated')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-framenavigated) 
- [page.on('load')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-load) 
- [page.on('metrics')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-metrics) 
- [page.on('pageerror')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-pageerror) 
- [page.on('request')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-request) 
- [page.on('requestfailed')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-requestfailed) 
- [page.on('requestfinished')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-requestfinished) 
- [page.on('response') ](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-response) 
- [page.on('workercreated') ](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-workercreated) 
- [page.on('workerdestroyed')](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.0&show=api-event-workerdestroyed) 

#### 终端模拟

Puppeteer 提供了几个有用的方法用来修改设备信息

1. `page.setViewport(viewport)`
2. `page.setUserAgent(userAgent)`

```js
await page.setViewport({
  width: 1920,
  height: 1080
});
await page.setUserAgent('Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36');
```

因为使用太频繁，Puppeteer 通过 `puppeteer/DeviceDescriptors` 提供了全套的设备模拟

```js
const puppeteer = require('puppeteer');
const devices = require('puppeteer/DeviceDescriptors');
const iPhone = devices['iPhone XR'];

puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  await page.emulate(iPhone);
  await page.goto('https://www.google.com');
  // other actions...
  await browser.close();
});
```

所有支持参考：https://github.com/puppeteer/puppeteer/blob/main/src/common/DeviceDescriptors.ts

#### 性能衡量

通过 page.getMetrics() 可以得到一些页面性能数据

```js
{ 
  Timestamp: 382305.912236,
  Documents: 5, // 页面文档数
  Frames: 3,
  JSEventListeners: 129, // 页面事件监听器数
  Nodes: 8810, // 页面 DOM 节点数
  LayoutCount: 38,
  RecalcStyleCount: 56,
  LayoutDuration: 0.596341000346001, // 页面 layout 时间
  RecalcStyleDuration: 0.180430999898817,
  ScriptDuration: 1.24401400075294, // script 时间
  TaskDuration: 2.21657899935963, // 所有浏览器任务时长
  JSHeapUsedSize: 15430816, // 堆内存占用大小
  JSHeapTotalSize: 23449600  // 堆总量
}
```

#### 植入JS代码

`page.exposeFunction(name, puppeteerFunction)` 用于在 window 对象注册一个函数，举个例子：给 window 添加一个 window.readfile 函数

```js
const puppeteer = require('puppeteer');
const fs = require('fs');

puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  page.on('console', msg => console.log(msg.text));
  
  // 注册 window.readfile
  await page.exposeFunction('readfile', async filePath => {
    return new Promise((resolve, reject) => {
      fs.readFile(filePath, 'utf8', (err, text) => {
        if (err)
          reject(err);
        else
          resolve(text);
      });
    });
  });
  
  await page.evaluate(async () => {
    // 在页面中调用注册的方法
    const content = await window.readfile('/etc/hosts');
    console.log(content);
  });
  await browser.close();
});
```

#### 可以在浏览器中执行 JS 的 API

- `page.evaluate(pageFunction[, ...args])` - 在浏览器环境中执行函数
- `page.evaluateHandle(pageFunction[, ...args])` - 在浏览器环境中执行函数，返回 JsHandle 对象
- `page.$$eval(selector, pageFunction[, ...args])` - 把 selector 对应的所有元素传入到函数并在浏览器环境执行
- `page.$eval(selector, pageFunction[, ...args])` - 把 selector 对应的第一个元素传入到函数在浏览器环境执行
- `page.evaluateOnNewDocument(pageFunction[, ...args])` - 创建一个新的 Document 时在浏览器环境中执行，会在页面所有脚本执行之前执行
- `page.exposeFunction(name, puppeteerFunction)` - 在 window 对象上注册一个函数，这个函数在 Node 环境中执行，有机会在浏览器环境中调用 Node.js 相关函数库

#### CDP 协议通信

可用 CDP 协议：https://chromedevtools.github.io/devtools-protocol/

**一般步骤**

1. 和某个 Tab 建立连接
2. 通过 send 发送你想使用的 methods
3. 通过 on 监听你发送 methods 产生的事件, 或者其他 enable 的事件, 并执行对应回调

```js
await cdp.send('Log.enable'); // 发送开启日志方法
cdp.on('Log.entryAdded', async ({ entry }) => { // 监听日志增加事件
  console.log('entry', entry); // 打印增加的日志
});
```

## 实例

#### 图片搜索 & 下载

```js
const path = require('path');
const fs = require('fs');
const http = require('http');
const https = require('https');
const puppeteer = require('puppeteer');
const ora = require('ora');

// const devices = require('puppeteer/DeviceDescriptors');
const iPhoneXR = puppeteer.devices['iPhone XR'];

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.emulate(iPhoneXR);
  
  await page.goto('https://iamge.baidu.com', { waitUntil: ['load'] });
  await page.type('#image-search-input', 'dog');

  await page.tap('#image-search-btn');

  page.on('load', async () => {
    const srcs = await page.$$eval(
      '.sfc-image-content-waterfall img',
      images => images.map(img => img.src)
    );
    
    await browser.close();
    
    let i = 0;
    srcs.forEach(src => {
      const request = src.trim().startsWith('https') ? https : http;
      const dest = path.join(__dirname, `../images/${i++}.jpg`);
      console.log(`正在下载 ${src}`);

      request.get(src, res => {
        res.pipe(fs.createWriteStream(dest));
      });
    });
  });
})();
```

#### 模拟 iPhoneXR 截屏

```js
const path = require('path');
const puppeteer = require('puppeteer');

const iPhoneXR = puppeteer.devices['iPhone XR'];

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.emulate(iPhoneXR);

  await page.goto('https://www.baidu.com', { waitUntil: ['load'] });

  await page.screenshot({
    path: path.join(__dirname, '../image', 'baidu.png'),
    fullPage: true,
  });

  await browser.close();
})();
```

#### 获取 WebSocket 响应

**使用底层 CDP 协议**

Puppeteer 目前没有提供原生的用于处理 WebSocket 的 API 接口，但是我们可以通过更底层的 Chrome DevTool Protocol (CDP) 协议获得

```js
(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    //创建 CDP 会话
    let cdpSession = await page.target().createCDPSession();
    //开启网络调试,监听 Chrome DevTools Protocol 中 Network 相关事件
    await cdpSession.send('Network.enable');
    //监听 webSocketFrameReceived 事件，获取对应的数据
    cdpSession.on('Network.webSocketFrameReceived', frame => {
        let payloadData = frame.response.payloadData;
        if(payloadData.includes('push:query')){
            //解析payloadData，拿到服务端推送的数据
            let res = JSON.parse(payloadData.match(/\{.*\}/)[0]);
            if(res.code !== 200){
                console.log(`调用websocket接口出错:code=${res.code},message=${res.message}`);
            }else{
                console.log('获取到websocket接口数据：', res.result);
            }
        }
    });
    await page.goto('https://netease.youdata.163.com/dash/142161/reportExport?pid=700209493');
    await page.waitForFunction('window.renderdone', {polling: 20});
    await page.close();
    await browser.close();
})();
```
