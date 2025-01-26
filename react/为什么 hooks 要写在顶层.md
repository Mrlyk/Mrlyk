# 为什么 hooks 要写在顶层

在开发中我们有时会遇到下面这种报错

```text
Rendered fewer/more hooks than expected. This may be caused by an accidental early return statement.
```

意思是说我们再两次渲染之间，渲染的 hooks 数量改变了。

举个例子，下面这段代码。在第二次渲染时，比起第一次渲染会少调一个 useState 的 hook。这就是 hooks 典型的没有写在顶层而是写在了条件语句当中。

```jsx
let firstRender = true;

function RenderFunctionComponent() {
  let initName;
  
  if(firstRender){
    [initName] = useState("Rudi");
    firstRender = false;
  }
  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```

## 错误的原因

直接原因：react 依赖于 hook 的调用顺序，调用顺序必须保持一致。

可以理解为React 使用了一种**链表结构**来追踪 Hook 的状态。当一个函数组件被调用时，React 会遍历这个链表，并更新当前渲染周期内 Hooks 的状态。每个 Hook 在链表中占据一个固定的位置，如果 Hooks 的调用顺序发生变化，链表的结构也会发生变化，导致 React 无法正确追踪状态。

可以看下面的图，第一次渲染时，因为`[initName] = useState("Rudi");` 这个 hook 会被调用，所以链表的指针从这个 hook 开始统计。最终链表存在三项。

![image-20240311113207976](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20240311113207976.png?x-oss-process=image/resize,h_800,m_lfit)  

**问题出现在第二次渲染时** 

1. 重渲染时链表指针重置为 0 开始；
2. react 从指针 0 开始获取状态，0 存取的状态实际上是`[initName] = useState("Rudi")` 这个 hook 的，所以 firstName 被设置为了对应的 "Rudi"。指针 ++ 变为了 1；
3. react 继续执行`  const [lastName, setLastName] = useState("Yardley")` 这个 hook，**但是指针**现在指向了`const [firstName, setFirstName] = useState(initName)` 的，此时他对应的状态也是 "Rudi"；
4. 最终结果是 firstName 与 lastName 这两个变量全部被设置为“Rudi”。

![image-20240311113503287](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20240311113503287.png?x-oss-process=image/resize,h_600,m_lfit) 

## 为什么这样设计

参考官方说明 [Why Do React Hooks Rely on Call Order?](https://overreacted.io/why-do-hooks-rely-on-call-order/) 

简单来说官方考虑了很多问题，比如链表的设计不会有命名冲突，处理方式更简洁，多层继承问题也得以解决等等。

## 参考文档

Rules of Hooks：https://legacy.reactjs.org/docs/hooks-rules.html#explanation

Why Do React Hooks Rely on Call Order?：https://overreacted.io/why-do-hooks-rely-on-call-order/