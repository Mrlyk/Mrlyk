# react Fiber

react 在更新 DOM 时，也是通过 diff 算法来比较节点。不同于 vue 的是它引入了一种 Fiber 架构，能够临时中断更新之后再继续更新。让浏览器有时间处理绘制相关的逻辑。

## Fiber 属性说明 

一个标准的 Fiber 节点具有很多属性，以下是 Fiber 节点上的一些属性及其作用：

- `type`：表示 Fiber 节点的类型，可以是字符串（对应原生 DOM 标签），也可以是函数/类（对应 React 组件）。
- `key`：用于在重复的子元素列表中区分唯一的子元素，有助于 React 的重用机制和性能优化。
- `child`：指向该 Fiber 节点的第一个子节点。
- `sibling`：指向该 Fiber 节点的下一个兄弟节点。
- `return`：指向该 Fiber 节点的父节点。
- `index`：表示 Fiber 在其兄弟节点中的位置。
- `stateNode`：与该 Fiber 节点关联的本地状态（例如，对于 DOM 节点，`stateNode` 就是对应的 DOM 实例）。
- `pendingProps`：将要应用到该 Fiber 节点上的新的 props。
- `memoizedProps`：上一次渲染完成后该 Fiber 节点上的旧 props。
- `updateQueue`：该 Fiber 节点的更新队列，包含了该节点和其子节点的 state 更新和回调。
- `memoizedState`：上一次渲染完成后该 Fiber 节点上的旧 state。
- `effectTag`：表明了该 Fiber 节点在 commit 阶段需要执行的操作，比如插入、更新或删除。
- `nextEffect`：指向链表中下一个将要进行副作用处理的 Fiber。
- `firstEffect` 和 `lastEffect`：分别指向该 Fiber 子树中所有副作用（side effects）的第一个和最后一个 Fiber。这些副作用在 commit 阶段需要被处理。
- `expirationTime` 和 `childExpirationTime`：表示该 Fiber 节点和其子树中任务的优先级。
- `alternate`：指向该 Fiber 节点的替身（double buffering 机制），用于记录前一次渲染的状态。
- `tag`：表示 Fiber 的类型，如 Function/Class Component、Host Component 等。
- `mode`：标识 Fiber 的渲染模式，例如是否启用 Concurrent 模式。
- `flags`：React 17 引入，替代`effectTag`，表示 Fiber 节点在 commit 阶段需要执行的操作。
- `deletions`： 属性是一个数组，该数组用于存储在当前渲染过程中需要删除的子 Fiber 节点。

#### tag 属性——节点类型

tag 是一个数值，不同的数字对应的值如下：

1. `FunctionComponent`：函数组件。
2. `ClassComponent`：类组件。
3. `IndeterminateComponent`：尚未确定类型，通常是因为函数组件在初次渲染前的状态。
4. `HostRoot`：宿主根节点，代表了 React 应用的根节点，也就是挂载的入口点。
5. `HostPortal`：宿主门户节点，用于在不同 DOM 子树中渲染子节点。
6. `HostComponent`：宿主组件，对应普通的 DOM 元素（如 `div`、`span` 等）。
7. `HostText`：文本节点。
8. `Fragment`：React.Fragment，可以让你聚合一组子节点而无需向 DOM 添加额外节点。
9. `Mode`：用于指示 Fiber 节点的渲染模式（如严格模式）。
10. `ContextProvider`：上下文提供者。
11. `ContextConsumer`：上下文消费者。
12. `ForwardRef`：转发 ref 的组件。
13. `Profiler`：性能监测组件，用于收集渲染时间等性能信息。
14. `SuspenseComponent`：Suspense 组件，用于指定加载指示器，并在等待内容加载时显示。
15. `MemoComponent`：使用 React.memo 包装的组件。
16. `SimpleMemoComponent`：简化版的 Memo 组件，通常指向函数组件。
17. `LazyComponent`：懒加载组件，使用 React.lazy 动态加载的组件。
18. `InCompleteClassComponent`：不完整的类组件，在渲染类组件时遇到错误时使用。
19. `DehydratedFragment`：脱水片段，在 SSR 中使用。
20. `SuspenseListComponent`：SuspenseList 组件，用于调整 Suspense 组件的加载顺序。
21. `FundamentalComponent`：基础组件，用于实验性质的新功能。
22. `ScopeComponent`：作用域组件，用于实现新型的事件处理机制。
23. `Block`：块类型组件，用于实验性质的新功能。
24. `OffscreenComponent`：屏幕外组件，用于新的并发特性。

#### flags 属性——操作标识

每个 Fiber 节点都有一个 `flags` （在 React 17 之前是 `effectTag`）属性，它是一个位字段（bitfield），用来表示该 Fiber 在当前渲染周期中需要执行的操作。这个属性的值通过按位运算来设置和检查不同的效果（effects）。