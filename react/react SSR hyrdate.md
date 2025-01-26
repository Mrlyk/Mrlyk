# React SSR hyrdate 探究

react SSR 水合是如何实现的呢？本文将做一个简要的探究。

一句话说明：React 在创建 DOM 元素时，判断是否存在可复用的 DOM （SSR 返回的），从而跳过创建 DOM 的步骤。

[toc]

## 源码解析

React 在浏览器上的渲染过程如下，简单理解就是先解析 jsx 创建 vDom ，然后对比 dom 生成操作序列，然后按顺序执行操作。

而水合的简单理解就是在下面主流程的 reconcile 流程中判断能不能复用 dom。

- beginWork 里执行 Reconciler （react 版本的 diff 算法）

  > Reconciler 工作的两个阶段：
  >
  > - **Reconciliation Phase（对比阶段）：** 在这个阶段，React 会对比新旧两棵虚拟 DOM 树，计算出需要实施的更新。这个阶段是可中断的，也就是说React 可以根据任务的优先级中断这个过程，先去执行更高优先级的任务。
  > - **Commit Phase（提交阶段）：** 这个阶段开始时，React 已经有了所有的更新信息，并且会开始一次性地应用所有的更新到实际 DOM 上。这个阶段是同步的，一旦开始就不能中断，直到所有的 DOM 更新完成。

- completeWork 里执行 创建元素、更新属性、拼装

![image-20240507182045865](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20240507182045865.png?x-oss-process=image/resize,w_800,m_lfit) 

#### 水合过程

主流程相同，在 complete 流程中根据是否 hydrated 来跳过上面的 “创建元素” 流程

```js
function enterHydrationState(fiber: Fiber): boolean {
  if (!supportsHydration) {
    return false;
  }

  const parentInstance: Container = fiber.stateNode.containerInfo;
  nextHydratableInstance =
    getFirstHydratableChildWithinContainer(parentInstance);
  hydrationParentFiber = fiber;
  isHydrating = true;
  hydrationErrors = null;
  didSuspendOrErrorDEV = false;
  rootOrSingletonContext = true;
  return true;
}


const SUSPENSE_START_DATA = '$';
const SUSPENSE_END_DATA = '/$';
const SUSPENSE_PENDING_START_DATA = '$?';
const SUSPENSE_FALLBACK_START_DATA = '$!';

function getNextHydratable(node: ?Node) {
  //  跳过不需要水合的节点
  for (; node != null; node = ((node: any): Node).nextSibling) {
    const nodeType = node.nodeType;
    if (nodeType === ELEMENT_NODE || nodeType === TEXT_NODE) {
      break;
    }
    if (nodeType === COMMENT_NODE) {
      const nodeData = (node: any).data;
      if (
        nodeData === SUSPENSE_START_DATA ||
        nodeData === SUSPENSE_FALLBACK_START_DATA ||
        nodeData === SUSPENSE_PENDING_START_DATA
      ) {
        break;
      }
      if (nodeData === SUSPENSE_END_DATA) {
        return null;
      }
    }
  }
  return (node: any);
}
```

```js
export function canHydrateInstance(
  instance: HydratableInstance,
  type: string,
  props: Props,
  inRootOrSingleton: boolean,
): null | Instance {
  while (instance.nodeType === ELEMENT_NODE) {
    const element: Element = (instance: any);
    const anyProps = (props: any);
    if (element.nodeName.toLowerCase() !== type.toLowerCase()) {
      if (!inRootOrSingleton || !enableHostSingletons) {
        // Usually we error for mismatched tags.
        if (
          enableFormActions &&
          element.nodeName === 'INPUT' &&
          (element: any).type === 'hidden'
        ) {
          // If we have extra hidden inputs, we don't mismatch. This allows us to embed
          // extra form data in the original form.
        } else {
          return null;
        }
      }
      // In root or singleton parents we skip past mismatched instances.
    } else if (!inRootOrSingleton || !enableHostSingletons) {
      // Match
      if (
        enableFormActions &&
        type === 'input' &&
        (element: any).type === 'hidden'
      ) {
        if (__DEV__) {
          checkAttributeStringCoercion(anyProps.name, 'name');
        }
        const name = anyProps.name == null ? null : '' + anyProps.name;
        if (
          anyProps.type !== 'hidden' ||
          element.getAttribute('name') !== name
        ) {
          // Skip past hidden inputs unless that's what we're looking for. This allows us
          // embed extra form data in the original form.
        } else {
          return element;
        }
      } else {
        return element;
      }
    } else if (isMarkedHoistable(element)) {
      // We've already claimed this as a hoistable which isn't hydrated this way so we skip past it.
    } else {
      // We have an Element with the right type.

      // We are going to try to exclude it if we can definitely identify it as a hoisted Node or if
      // we can guess that the node is likely hoisted or was inserted by a 3rd party script or browser extension
      // using high entropy attributes for certain types. This technique will fail for strange insertions like
      // extension prepending <div> in the <body> but that already breaks before and that is an edge case.
      switch (type) {
        // case 'title':
        //We assume all titles are matchable. You should only have one in the Document, at least in a hoistable scope
        // and if you are a HostComponent with type title we must either be in an <svg> context or this title must have an `itemProp` prop.
        case 'meta': {
          // The only way to opt out of hoisting meta tags is to give it an itemprop attribute. We assume there will be
          // not 3rd party meta tags that are prepended, accepting the cases where this isn't true because meta tags
          // are usually only functional for SSR so even in a rare case where we did bind to an injected tag the runtime
          // implications are minimal
          if (!element.hasAttribute('itemprop')) {
            // This is a Hoistable
            break;
          }
          return element;
        }
        case 'link': {
          // Links come in many forms and we do expect 3rd parties to inject them into <head> / <body>. We exclude known resources
          // and then use high-entroy attributes like href which are almost always used and almost always unique to filter out unlikely
          // matches.
          const rel = element.getAttribute('rel');
          if (rel === 'stylesheet' && element.hasAttribute('data-precedence')) {
            // This is a stylesheet resource
            break;
          } else if (
            rel !== anyProps.rel ||
            element.getAttribute('href') !==
              (anyProps.href == null ? null : anyProps.href) ||
            element.getAttribute('crossorigin') !==
              (anyProps.crossOrigin == null ? null : anyProps.crossOrigin) ||
            element.getAttribute('title') !==
              (anyProps.title == null ? null : anyProps.title)
          ) {
            // rel + href should usually be enough to uniquely identify a link however crossOrigin can vary for rel preconnect
            // and title could vary for rel alternate
            break;
          }
          return element;
        }
        case 'style': {
          // Styles are hard to match correctly. We can exclude known resources but otherwise we accept the fact that a non-hoisted style tags
          // in <head> or <body> are likely never going to be unmounted given their position in the document and the fact they likely hold global styles
          if (element.hasAttribute('data-precedence')) {
            // This is a style resource
            break;
          }
          return element;
        }
        case 'script': {
          // Scripts are a little tricky, we exclude known resources and then similar to links try to use high-entropy attributes
          // to reject poor matches. One challenge with scripts are inline scripts. We don't attempt to check text content which could
          // in theory lead to a hydration error later if a 3rd party injected an inline script before the React rendered nodes.
          // Falling back to client rendering if this happens should be seemless though so we will try this hueristic and revisit later
          // if we learn it is problematic
          const srcAttr = element.getAttribute('src');
          if (
            srcAttr !== (anyProps.src == null ? null : anyProps.src) ||
            element.getAttribute('type') !==
              (anyProps.type == null ? null : anyProps.type) ||
            element.getAttribute('crossorigin') !==
              (anyProps.crossOrigin == null ? null : anyProps.crossOrigin)
          ) {
            // This script is for a different src/type/crossOrigin. It may be a script resource
            // or it may just be a mistmatch
            if (
              srcAttr &&
              element.hasAttribute('async') &&
              !element.hasAttribute('itemprop')
            ) {
              // This is an async script resource
              break;
            }
          }
          return element;
        }
        default: {
          // We have excluded the most likely cases of mismatch between hoistable tags, 3rd party script inserted tags,
          // and browser extension inserted tags. While it is possible this is not the right match it is a decent hueristic
          // that should work in the vast majority of cases.
          return element;
        }
      }
    }
    const nextInstance = getNextHydratableSibling(element);
    if (nextInstance === null) {
      break;
    }
    instance = nextInstance;
  }
  // This is a suspense boundary or Text node or we got the end.
  // Suspense Boundaries are never expected to be injected by 3rd parties. If we see one it should be matched
  // and this is a hydration error.
  // Text Nodes are also not expected to be injected by 3rd parties. This is less of a guarantee for <body>
  // but it seems reasonable and conservative to reject this as a hydration error as well
  return null;
}
```

#### 存在的问题

上面的水合过程效果是 ok 的，但是需要等到所有资源（js、css）被下载完成后再进行水合。这就会导致水合时机非常滞后。

举个例子的话就像下面这样，页面虽然加载出来了，但是用户点不了。

![2024-05-08 14.15.43](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/2024-05-08%2014.15.43.gif?x-oss-process=image/resize,w_600,m_lfit) 

## React 的解决方案

针对这个场景 React 本身推出了流式 SSR，利用流式 SSR 可以将需要请求数据的模块先 loading 住，后续再完成水合。

![image-20240508142123580](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20240508142123580.png) 

当然我们需要完成对应的改造，不仅前端需要针对流式渲染对组件进行处理，也需要组件针对这类模块进行拆分。

#### 仍然存在的问题

**💡** **React 官方自己提出的问题和思考：**

1. 纯展示数据的 Server 组件能不能不用打包了，客户端只要渲染结果就行，不需要下载它的 JS ？
2. Server / Client 能否无缝继承，Server 组件直接把数据透传给 Client 组件？
3. 能不能让 Server 组件可以动态的渲染 Client 组件，然后按需加载 Client 组件？
4. 刷新 Server 组件的时候，能不能保留 Client 组件的 state、焦点、动画 ？
5. Server 组件能不能渐进增量渲染，不要一整个 HTML Chunk 的形式，可控度太差了。

## 两套方案

#### RSC 协议【倾向】

这一套方案是支付宝前端团队的同学提供的。

自定义 RSC 协议，直接回传 JSX 给 React 到客户端处理。这样就减少了重复的 jsx 解析过程，直接在客户测处理一次即可。

参照文档：https://aliyuque.antfin.com/peiqiao.ppq/docs/rsc-08#Rne1V



#### Resumability:  hydartion 替代方案

21年的时候一个新的前端 qwik 给我们带来了不一样的解决方案

qwik 中提出了一个全新的思路来避免水合带来的额外开销：

1. **将所有必需的信息序列化为 HTML 的一部分。**

   qwik 将需要的状态以及事件序列化保存在 Server 端下发的 HTML 模版中，需要序列化信息需要包括事件处理函数内容, 哪些节点需要哪些类型的事件处理函数, `State`（应用状态）, 和`FRAMEWORK_STATE`（框架状态）。

2. **依赖于事件冒泡来拦截所有事件的全局事件处理程序。**

   qwik 中事件处理程序是在全局处理的，这样就不必在在特定的 DOM 元素上单独注册所有事件。

3. qwki 内部存在一个可以延迟恢复事件处理程序的工厂函数。

   该工厂函数主要用于处理节点事件绑定，也就是用来识别某个事件处理函数中应该存在什么脚本逻辑。

最终下发的 HTML 模版可能长这样：

```html
<div q:host>
  <div q:host>
    <button on:click="./chunk-a.js#button">Trip Biz</button>
  </div>
  <div q:host>
    <button q:obj="1" on:click="./chunk-b.js#count[0]">10</button>
  </div>
</div>
<script id="qwikloader">/* qwik 中设置全局事件监听器的代码 */</script>
<script id="qwik/json">/* 用于反序列化的 JSON 相关信息 */</script>
```

## 参考

- RSC From Scratch. Part 1: Server Components：https://github.com/reactwg/server-components/discussions/5
- Introducing Zero-Bundle-Size React Server Components：https://react.dev/blog/2020/12/21/data-fetching-with-react-server-components
- RSC 技术详解：https://aliyuque.antfin.com/peiqiao.ppq/docs/rsc-08

