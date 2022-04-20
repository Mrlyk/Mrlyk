# MVC、MVP、MVVM

MVC、MVP、MVVM 是 Web 开发中常用的设计模式，通过分离关注点来改进代码的组织方式，对复杂问题进行解耦、优化。

使用简单的例子来展示这三种设计模式，下面是一个点击按钮 contanier 内容加 1的页面。

```html
<div>
	<span id="container">0</span>
  <button id="btn" onclick="javascript:add()">+</button>
</div>

<script>
  function add (){
    const container = document.getElementById('container');
    const current = parseInt(container.innerText);
    container.innerText = current + 1;
  }
</script>
```

可以看到视图和逻辑混杂在一个面上，如果是简单的问题还好，但是随着业务复杂，代码将极难维护。

## MVC

- M - Model 数据模型，数据相关操作
- V - View 视图层，页面渲染
- C - 控制器

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211222142757458.png" alt="image-20211222142757458" style="zoom:50%;" align="left"/>

MVC 模型通过把视图渲染和数据处理做隔离，通过 Controller 接收 View 操作，传递给 Model 处理数据。数据处理完成后，Model 驱动 View 渲染。

开头的例子用 MVC 改写

*view*

```html
<div>
	<span id="container">0</span>
  <button id="btn">+</button>
</div>
```

*controller*

```js
const button = document.getElementById('btn');
// 接收视图指令
button.addEventListener('click', () => {
  const container = document.getElementById('container');
  
  // 调用模型
	add(container);
}, false);
```

*model*

```js
function add (node) {
  // 业务逻辑处理
  const currentValue = parseInt(node.innerText);
  const newValue = currentValue + 1;
  
  // 更新视图
	node.innerText = newValue + 1;
}
```

1. 视图层最简单，处理页面的渲染
2. 控制器在用户点击按钮的时候把请求转发给模型处理，在 web 开发中一般页面、接口请求的路由也是控制器负责
3. 模型层定义了 +1 操作的实现，并更新视图数据

这种传统的设计模式存在的问题是 View 和 Controller 存在耦合，Controller 需要拿到 View 的相关信息，但 Controller 作为 View 和 Model 沟通的桥梁，这部分耦合可以接受。不可接受的是 View 和 Model 之间也存在耦合。

## MVP

MVP 是 Model View Presenter 的缩写，可以说是 MVC 模式的改良，相对于 MVC 有了各层负责的任务和数据流动方式都有了部分变化。

- Model - 和业务无关的基础数据方法
- View - 视图渲染逻辑
- Presenter - 响应视图指令，处理业务逻辑，同时连接 Model 层获取底层数据，返回指令结果到视图，驱动视图渲染

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211222144510547.png" alt="image-20211222144510547" style="zoom:50%;" align="left"/>

**相对于 MVC 的变化**

1. 彻底分离 View 和 Model 层，Model 只负责最底层的数据处理逻辑
2. Presenter 接管路由和业务逻辑，但要求 View 实现 View Interface，方便和 View 也解耦
3. View 层只负责发起指令和页面渲染，不再主动监听数据变化，也被称为**被动视图**

开头的例子用 MVP 改写

*View*

```html
<div>
	<span id="container">0</span>
  <button id="btn">+</button>
</div>
<script>
  // View Interface
	const globalConfig = {
  	containerId: 'container',
    buttonId: 'btn',
  };
</script>
```

*Presenter*

```js
const button = document.getElementById(globalConfig.containerId); // 根据 View Interface 获取目标视图。不再需要去 View 层查找，解耦 View 和 Presenter
const container = document.getElementById(globalConfig.buttonId);

// 响应视图指令
button.addEventListener('click', () => {
  const currentValue = parseInt(container.innerText);
  // 调用数据模型
	const newValue = add(currentValue);
  // 更新视图
  container.innerText = newValue;
}, false);
```

使用 MVP 模型可以让 Model 更稳定，因为他只处理和业务无关的基础数据。但是会带来 Presenter 臃肿的问题。很多时候我们在用 MVC 其实是在用 MVP 模式。

## MVVM

MV-VM，Model View - ViewModel。可以看作 MVP 模式的变种，View 和 Model 与 MVP 相同，ViewModel 依靠 DataBinding 把 View 和 Model 做了自动关联，大部分情况下 Web 框架替我们实现了这一部分。

- Model - 和业务无关的基础数据方法
- View - 视图渲染逻辑
- ViewModel - 数据与视图的 DataBinding

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211222145714329.png" alt="image-20211222145714329" style="zoom:50%;" align="left"/>

开头的例子用 MVVM 改写

*View*

```html
<div id="test">
  <!-- 数据和视图绑定 -->
	<span>{{counter}}</span>
  <button v-on:click="counterPlus">+</button>
</div>
```

*Model*

```js
function add (num) {
  return num + 1;
}
```

*ViewModel*

我们常用的 Vue 就是 MVVM 模式，这里用 Vue 来展示

```js
new Vue({
  el: '#test',
  data: {
    counter: 0
  },
  methods: {
  	counterPlus: function () {
     	// 只需要修改数据，无需手工修改视图
    	this.counter = add(this.counter);
    }
  }
})
```

## 总结

MVC、MVP、MVVM 三种设计模式都旨在解决数据与视图的分离问题。

- MVC 做了第一步，实现简单，但是三层分离的不彻底，依然存在联系
- MVP 通过 Presenter 彻底解耦了 View 和 Model，同时剥离了业务逻辑和底层数据逻辑，让 Model 更稳定，但是 Presenter 会变得臃肿
- MVVM 通过 DataBinding 实现了视图和数据绑定，一般由框架实现，增加了理解成本。

注意有 Controller 的不一定就是 MVC 模式，也可能是 MVP 模式，叫法而已，看具体是如何使用的。