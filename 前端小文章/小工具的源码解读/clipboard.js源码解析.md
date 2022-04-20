# clipboard.js 源码解析

业务中经常有点击复制，粘贴的场景

## 功能

- 复制

  ```html
  <!-- Target -->
  <input id="foo" value="https://github.com/zenorocha/clipboard.js.git" />
  <!-- Trigger -->
  <button class="btn" data-clipboard-target="#foo">copy</button>
  ```

- 剪切

  ```html
  <textarea class="hello_textarea" id="bar" rows="5">
  Additionally, you can define a data-clipboard-action attribute to specify if you want to either copy or cut content.</textarea>
   <button class="btn" data-clipboard-action="cut" data-clipboard-target="#bar">
  </button>
  ```

  

- 复制、剪切事件监听

  ```js
  clipboard.on('success', function(e) {
      console.info('Action:', e.action);
      console.info('Text:', e.text);
      console.info('Trigger:', e.trigger);
  
      e.clearSelection();
  });
  
  clipboard.on('error', function(e) {
      console.error('Action:', e.action);
      console.error('Trigger:', e.trigger);
  });
  ```

- 动态设置元素（这个可能在 vue 项目中用的少）

  ```js
  new ClipboardJS('.btn', {
      target: function(trigger) {
          return trigger.nextElementSibling  ||  document.getElementById('name');
      }
  });
  ```


## 如何开始阅读一份源码

1、不要为了读而读，枯燥且没有动力。携带着业务场景或者自己的疑问去读。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210914163059352.png" alt="image-20210914163059352" style="zoom: 33%;" />

2、对方案有一些自己的了解，`document.execCommand`，`navigator.clipboard`,古老的时候就是用 ie 的 activex 控件去访问

带目的性的：

直接查找相关的源码，比如我要看他核心复制是怎么实现的，直接查看相关文件。如果是大型项目的源码一般都是按照功能区分项目结构的。clipboard.js 是个小型的工具插件，通过文件名就能方便的找到功能（文件命名规范的重要性）。

学习性的：

- 从入口文件开始，逐渐深入，全面了解。一般没这个必要，除非我们要学习一个好的公共工具应该如何实现。
- 从用法深入

## 源码解析

从用法深入

**首先实例化了一个 `Clipboard`类，找到该类声明对应的源码**

```js
import Emitter from 'tiny-emitter';

class Clipboard extends Emitter {}
```

发现该类继承自 Emitter，我们的目标是学习 `clipboard.js` 的实现，所以只需要知道这个 `Emitter`干了什么，不需要去关心他的实现。

```text
tiny-emitter 是一个小型（小于1k）事件发射器,和我们经常用的 vue 中的 $emit 作用类似
```

`Emitter`在这里干了什么呢？在`Clipboard`这个类中发现了这样一个实例方法，发现他是在触发点击事件的时候，发射出点击成功和失败的事件。

```js
import Emitter from 'tiny-emitter';

class Clipboard extends Emitter {
  onClick(e) {
   ...
  // this.emit 则是从 Emitter 继承过来的实例方法
    this.emit(selectedText ? 'success' : 'error', {
      ...
  }
}

```

所以我们在上面用的时候才能使用`Clipboard`的实例来监听到这两个方法。这就是 tinny-emitter 这个工具干的事情。

**继续往下看 Clipboard 这个类又做了些什么**

```js
import Emitter from 'tiny-emitter';

class Clipboard extends Emitter {
  constructor(trigger, options) {
    super(); // 实例化 Emitter 类
    this.resolveOptions(options);  // 处理传递进来的选项 
    this.listenClick(trigger); // 监听目标元素的点击事件
  }
  onClick(e) { ... }
}

```

在构造函数中执行了两个方法。

**处理传递进来的选项**

```js
import Emitter from 'tiny-emitter';

class Clipboard extends Emitter {
  constructor(trigger, options) {
    super(); // 实例化 Emitter 类
    this.resolveOptions(options);  // 处理传递进来的选项 
    this.listenClick(trigger); // 监听目标元素的点击事件
  }
  
  resolveOptions(options = {}) { // 默认值 {}
    this.action =
      typeof options.action === 'function'
        ? options.action
        : this.defaultAction;
    this.target =
      typeof options.target === 'function'
        ? options.target
        : this.defaultTarget;
    this.text =
      typeof options.text === 'function' ? options.text : this.defaultText;
    this.container =
      typeof options.container === 'object' ? options.container : document.body;
  }
  
  onClick(e) { ... }
}

```

根据`options`声明了四个实例属性 `action`，`target`，`text`和`container`。

就以我们自己 demo 中的实例化方法为例，不传入任何 options 选项，最后这四个值就如下，他们的默认值我们后面再说，

```js
this.action = this.defaultAction;
this.target = this.defaultTarget;
this.text = this.defaultText;
this.container = document.body;
```

还有一个 `listenClick`方法，看名字就知道是用来监听点击事件的（再次突出命名的重要性，代码的可读性依赖于命名）

```js
import Emitter from 'tiny-emitter';
import listen from 'good-listener';

class Clipboard extends Emitter {
  ...
  resolveOptions(options = {}) { ... }
  listenClick(trigger) {
     this.listener = listen(trigger, 'click', (e) => this.onClick(e)); // listen 方法来自 good-listener 这个工具类，专门用来监听目标元素的点击事件的
   }
  onClick(e) { ... }
}
```

在我们的 demo 中传入了 .btn 这个 trigger，所以最后监听就是 class 中包含 btn 这个类名的元素。

![new Clipboard()](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/new%20Clipboard().png)

至此就了解到了第一部分，复制事件的捕获和分发。

上面还留了几个问题，action、target、text、container 分别是什么。回过头来看我们的 demo，使用 clipboard 的功能时，我们只是在要使用的元素上声明了 `data-clipbord-target`、`data-clipboard-text`，`data-clipboard-action`这几个属性。

那就很明显了，这四个属性就是用来标识我们复制的目标、文本以及动作。还有一个 container ，不用太关心。上面说了我们的 demo 里面没有传 options 参数，所以这四个属性就是四个默认的值，那这个默认值又是怎么取呢？

```js
import Emitter from 'tiny-emitter';
import listen from 'good-listener';

class Clipboard extends Emitter {
  ...
  resolveOptions(options = {}) { ... }
  listenClick(trigger) { ... }
  onClick(e) { ... }
 
  defaultAction(trigger) {
     return getAttributeValue('action', trigger);
  }
  defaultTarget(trigger) {
    const selector = getAttributeValue('target', trigger);
    if (selector) {
      return document.querySelector(selector);
    }
  }
  defaultText(trigger) {
    return getAttributeValue('text', trigger);
  }
}
```

可以看到他们都使用了`getAttributeValue`这个方法，顾名思义获取元素的属性值。

**getAttributeValue**

```js
function getAttributeValue(suffix, element) {
  const attribute = `data-clipboard-${suffix}`;

  if (!element.hasAttribute(attribute)) {
    return;
  }

  return element.getAttribute(attribute);
}
```

这样就拿到了

- target：要复制的目标元素
- action：复制的动作，复制还是剪切
- text：要复制的文本

有了这三要素之后，就可以执行核心功能了，点击然后复制

**listenClick**

上面有提到这个方法监听了 target 元素的点击事件，并且触发了 `this.onClick`事件

```js
import Emitter from 'tiny-emitter';
import listen from 'good-listener';
import ClipboardActionDefault from './actions/default';

class Clipboard extends Emitter {
  ...
  resolveOptions(options = {}) { ... }
  listenClick(trigger) { ... }
  onClick(e) { ... }
  defaultAction(trigger) { ... }
  defaultTarget(trigger) { ... }
  defaultText(trigger) { ... }
  
  onClick(e) {
    const trigger = e.delegateTarget || e.currentTarget; // 获取当前点击的目标元素
    const selectedText = ClipboardActionDefault({ // 处理复制事件
      action: this.action(trigger),
      container: this.container,
      target: this.target(trigger),
      text: this.text(trigger),
    });

    // 触发事件
    this.emit(selectedText ? 'success' : 'error', {...}
}
```

**ClipboardActionDefault ()**

这个方法则是 clipboard.js 最核心的方法，默认的复制方法。上面说了一大堆其实都和复制无关，只是一些前置准备工作。到这里才是真正开始进行复制了。首先来看下他的源码

```js
import ClipboardActionCut from './cut';
import ClipboardActionCopy from './copy';

const ClipboardActionDefault = (options = {}) => {
  /**
  * 以 demo 第一个 input 的为例子
  * action：没传默认用 copy
  * container: document.body
  * target: #foo
  * text: undefined
  */
  const { action = 'copy', container, target, text } = options;

  if (action !== 'copy' && action !== 'cut') {
    throw new Error('Invalid "action" value, use either "copy" or "cut"');
  }

  // 处理边界情况
  if (target !== undefined) {
    // 如果目标节点的常量类型是元素节点（nodeType === 1）则继续
    if (target && typeof target === 'object' && target.nodeType === 1) {
      // 禁用的元素无法复制
      if (action === 'copy' && target.hasAttribute('disabled')) {
        throw new Error( ... );
      }
      // 只读的元素无法剪切和复制
      if (
        action === 'cut' &&
        (target.hasAttribute('readonly') || target.hasAttribute('disabled'))
      ) {
        throw new Error( ...);
      }
    } else {
      throw new Error('Invalid "target" value, use a valid Element');
    }
  }

  // 如果有文本，则直接传入文本。我们的 demo 中没有传入，所以继续往下
  if (text) {
    return ClipboardActionCopy(text, { container });
  }

  // 根据 action 使用剪切还是复制。未传入时，默认使用的是 copy
  if (target) {
    return action === 'cut'
      ? ClipboardActionCut(target)
      : ClipboardActionCopy(target, { container });
  }
};

```



**ClipboardActionCopy**

核心代码 copy 的实现

```js
import select from 'select';
import command from '../common/command';
import createFakeElement from '../common/create-fake-element';

const ClipboardActionCopy = (
  target,
  options = { container: document.body }
) => {
  let selectedText = '';
  // 如果传入的是 text 则临时创造 fake 元素来承载
  if (typeof target === 'string') {
    const fakeElement = createFakeElement(target);
    options.container.appendChild(fakeElement);
    selectedText = select(fakeElement);
    command('copy');
    fakeElement.remove();
  } else {
    // 否则使用 select 选中其中的内容
    selectedText = select(target);
    command('copy');
  }
  return selectedText;
};
```

可以看到 copy 时干了三件事。和我们平常复制手动复制文字的时候其实是一样的

复制东西一般三步:

1. 鼠标点击 **select(target)**
2. 拖动选中 **select(target)**
3. ctrl + c **command('copy')**

程序的作用就是将动作用代码来解释。

这其中的前两步都存在于 select 方法中，这也是核心方法之一，一起来看一下

```js
function select(element) {
    var selectedText;
    if (element.nodeName === 'SELECT') {
      // 如果是下拉框，则直接取 value
        element.focus();
        selectedText = element.value;
    }
    else if (element.nodeName === 'INPUT' || element.nodeName === 'TEXTAREA') {
        var isReadOnly = element.hasAttribute('readonly'); // 判断是否有只读属性，没有后面加上
      // 这段代码我一下没看出来为什么要这么做，就去翻了一下作者的代码记录，看到，代码规范记录的重要性
      // Prevent keyboard from showing on iOS 阻止 ios 上键盘弹出，又学会了个小技巧
        if (!isReadOnly) {
            element.setAttribute('readonly', '');
        }
        element.select();
        element.setSelectionRange(0, element.value.length); // 这里选中了 input 中的值，类似于我们的拖选
      // 再把自己加上的属性去掉
        if (!isReadOnly) {
            element.removeAttribute('readonly');
        }
      // 最后取值
      // 既然都是直接取 value 的值为什么再 input 的时候要选中呢？更好的交互体验吗？
        selectedText = element.value;
    }
    else {
      // 非 select 和 input 元素的情况下，如果可编辑则聚焦？也是为了交互体验吗？
        if (element.hasAttribute('contenteditable')) {
            element.focus();
        }
        var selection = window.getSelection(); // 获得当前的 selection 对象
        var range = document.createRange(); // 创建一个 range 对象
        range.selectNodeContents(element); // 将目标元素包含在 range 中
        selection.removeAllRanges(); // 移除原来的所有选区（拖蓝的区域）
        selection.addRange(range); // 将 range 添加到选区中
        selectedText = selection.toString(); // 获得选区中的纯文本内容
    }
    return selectedText;
}

```

以上是当我们传入的是 target 不是 string 类型的时候，我们看到在源码中如果传入的是 string，也就是要直接复制的文字，使用的是 `createFakeElement`创建了一个临时的元素来承载  string

```js
export default function createFakeElement(value) {
  const isRTL = document.documentElement.getAttribute('dir') === 'rtl'; // 获取文字的方向
  const fakeElement = document.createElement('textarea'); // 创建一个文本域来承载文本，为什么是文本域不是input？我个人觉得是从语义化的角度来考虑的
  fakeElement.style.fontSize = '12pt';
  fakeElement.style.border = '0';
  fakeElement.style.padding = '0';
  fakeElement.style.margin = '0';
  fakeElement.style.position = 'absolute';
  fakeElement.style[isRTL ? 'right' : 'left'] = '-9999px'; // 不展示出来这个fake元素
  let yPosition = window.pageYOffset || document.documentElement.scrollTop;
  fakeElement.style.top = `${yPosition}px`;

  fakeElement.setAttribute('readonly', ''); // 设置为只读
  fakeElement.value = value;

  return fakeElement;
}
```

这样返回了一个自己创建的元素，在拿到其中的文本内容后就使用 `remove`方法删除掉这个元素。

通过上面的步骤，就拿到了选区中的文本。至此就只剩下最后一个操作，将拿到的文本放入系统的剪贴板。在源码中可以看到最后执行的都是 `command()`这个方法，那这个方法是什么呢？

```js
/**
 * Executes a given operation type.
 * @param {String} type
 * @return {Boolean}
 */
export default function command(type) {
  try {
    return document.execCommand(type);
  } catch (err) {
    return false;
  }
}
```

最后一看，这不还是 document.execCommand 这个方法吗。。。

剪切的操作，实际上和复制没什么区别，就是最后执行 `command`命令的时候传入的参数是`cut`，还是交由浏览器提供 api 来完成的。

到此整个 clipboard 的源码就解读完毕。

![new Clipboard()2](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/new%20Clipboard()2.png)

## 总结

1. 代码可读性
2. 核心原理，简单，点失望，但是简单
3. 学习到一些小技巧
4. 印证一些思考，比如编程实际上就是在用程序的语言来解释动作

### 看起来很简单的一个工具，为什么能获得 **3w+ star** ？

1. <img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210914185606567.png" alt="image-20210914185606567" style="zoom:50%;" />
2. 文档清晰，并且有 demo 地址
3. 持续优化维护
4. 足够小 2kb
6. 取了个好名字

## 其他

### ![image-20210909161520553](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210909161520553.png)

### 兼容性

![image-20210907191613815](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210907191613815.png)

