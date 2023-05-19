# 双端diff算法

在简单 diff 算法的例子中我说过，有时候移动一次就可以完成的任务在使用简单 diff 算法却需要移动两次，效率不够高。双端diff算法则是来解决这些问题的。

vue2 使用的就是双端 diff 算法！而 vue 3则是借鉴了ivi和inferno框架使用了快速diff算法，我们一个个来看看。

[toc]

## 双端算法的实现

双端算法是两端进行比较的一种算法，我们需要分别定义新、旧子节点的开头和结尾，共四个端点

```js
function patchKeyedChildren(n1, n2, container) {
  const oldChildren = n1.children
  const newChildren = n2.children
  const oldLen = oldChildren.length
  const newLen = newChildren.length

  let oldStartIdx = 0
  let oldEndIdx = oldLen - 1
  let newStartIdx = 0
  let newEndIdx = newLen - 1

  let oldStartVNode = oldChildren[oldStartIdx] // 旧开头
  let oldEndVNode = oldChildren[oldEndIdx] // 旧结尾
  let newStartVNode = newChildren[newStartIdx]
  let newEndVNode = newChildren[newEndIdx]
}
```

![image-20230511145136189](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20230511145136189.png?x-oss-process=image/resize,w_800,m_lfit)  

然后进行4种比较：

1. 老的头部节点和新的头部节点比较
2. 老的尾部节点和新的尾部节点比较
3. 老的头部节点和新的尾部节点比较
4. 老的尾部节点和新的头部节点比较

#### 老头 vs 新头 && 老尾 vs 新尾

这两种比较差不多，如果他们的 key 相同，说明在老的和新的种位置是对应的，不用更改。

只需要**更新内容**即可。

**当然比较完之后他们就不用再参与后面的比较了，因为他们的位置已经确定了不用改，所以我们需要更新四个端点的新位置。**

```js
if (oldStartVNode.key = newStartVNode.key) {
  patch(oldStartVNode, newStartVNode, container)
  // 更新索引位置
  oldStartVNode = oldChildren[++oldStartIdx] // 比较过了就将老头的位置下移一位作为更新后的老头
  newStartVNode = newChildren[++newStartIdx] // 新的也一样
} else if (oldEndVNode.key = newEndVNode.key) {
  patch(oldEndVNode, newEndVNode, container)
  // 更新索引位置
  oldEndVNode = oldChildren[--oldEndIdx] // 老的末尾比较过了上移一位作为更新后的老尾
  newEndVNode = newChildren[--newEndIdx]
}
```

#### 老头 vs 新尾

如果老头和新尾相同，说明在新的子节点中，新的位置已经移动到了尾部，所以我们需要把老头对应的真实的 DOM 也移动到现在的尾部。在更新索引

```js
if (oldStartVNode.key === newEndVNode.key) {
  patch(oldStartVnode, newEndVNode, container) 
  
  const anchor = oldEndVNode.el.nextSibling
  inset(oldStartVNode.el, container, anchor)
  
  oldStartVNode = oldChildren[++oldStartIdx]
  newEndVnode = newChildren[--newEndIdx]
}
```

#### 老尾 vs 新头

如果老尾和新头相同，说明在新的子节点中，新的位置已经移动到了头部，所以我们需要把老尾对应的真实的 DOM 也移动到现在头部元素前。

```js
if (oldEndVNode.key === newStartVNode.key) {
  patch(oldEndVNode, newStartVNode, container) 
  
  const anchor = oldStartVNode.el
  inset(oldEndVNode.el, container, anchor)
  
  oldEndVNode = oldChildren[--oldEndIdx]
  newStartVNode = newChildren[++newStartIdx]
}
```

#### 何时比较完成

在比较过程中，每比较一次我们就更新一次端点位置：下移头部，上移尾部。

当端点重合了说明是最后一次比较，所以我们可以判断头部 > 尾部了比较就完成了，否则就继续循环比较。

```js
while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
  // ...
}
```

#### 非理想情况的补充

上面的实现描述了双端算法的理想情况，就是新、旧子节点总是满足上面的四个条件之一。

但是实际上肯定会出现不满足上面四个条件的，比如下面这个例子：

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'hello', key: 1 },
    { type: 'p', children: '2', key: 2 },
    { type: 'p', children: '3', key: 3 },
    { type: 'p', children: '4', key: 4 },
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: '2', key: 2 },
    { type: 'p', children: '4', key: 4 },
    { type: 'p', children: 'world', key: 1 },
    { type: 'p', children: '3', key: 3 },
  ],
}
```

为了处理这种情况我们需要补充处理。

补充处理之前我们需要弄明白两件事：

- **双端算法移动节点的目的是什么？** ——**把真实的 DOM 和新的虚拟子节点中的位置对应起来** 
- **双端算法快在哪里**？——**节点只处理一次，处理过一次的就不再处理**

明白了这件事之后我们就可以下手了，我们不一定非要满足上面的四种理想情况再动手，可以从新的虚拟子节点中的头部端点开始，找找看它原来在老的虚拟子节点中的哪里，找到之后把它移动到顶部来。同时将它置空表示已经处理过了。

之后再更新新的虚拟子节点中的头部位置。

```js
if () {
  //...
} else {
  const idxInOld = oldChildren.findIndex(vnode => vnode.key === newStartVNode.key)
  if (idxInOld > 0) {
    const vnodeToMove = oldChildren[idxInOld]
    patch(vnodeToMove, newStartVNode, container)
    
    // 更新真实的DOM的位置到头部
    const anchor = oldStartVNode.el
    insert(vnodeToMove.el, container, anchor)
    newStartVNode = newChildren[++newStartIdx]
    
    // 置空
    oldChildren[idxInOld] = undefined
  }
}
```

同时我们在循环中判断是否是已经处理过的节点，是的话只需要更新索引值即可

```js
if(oldStartVNode === undefined) {
  oldStartVNode = oldChildren[++oldStartIdx]
} else if (oldEndVNode === undefined) {
  oldEndVNode = oldChildren[--oldEndIdx]
}
```

#### 添加元素

新的元素肯定也符合四种理想情况，所以我们需要在非理想情况中增加处理方式.

怎么处理呢？很简单，新增的元素`idxInOld`肯定找不到，所以我们需要在找不到条件中处理新增的。

因为双端算法的处理策略是处理过的就不再处理了，所以进入这个条件之后，当前的`newStartVNode`就是我们新增的元素，而它在当前新虚拟列表子节点的顶部，所以我们需要将它对应的真实DOM也渲染到当前列表的顶部。**之后记得更新索引！！！**

```js
if (idxInOld > 0) {
  // ...
} else {
  const anchor = newStartVNode.el
  patch(null, newStartNode, container, anchor)
  
  newStartVNode = newChildren[++newStartIdx]
}
```

这样就完美了吗？当然不是，还有一种情况需要考虑：

##### 新增元素未被处理到的情况

在上面的处理方式中，我们默认处理新增的顶部的元素。但是**注意我们的循环条件**

```js
while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {}
```

当**新增的元素位置比较特殊刚好是最后几个才被处理的元素时**，因为新的元素比老的元素多，此时`oldStartIdx`和`oldEndIdx`都已经指向了同一个老的虚拟节点。又一轮循环完成后，**oldStartIdx 已经移动到 oldEndIdx 后面去了（反过来说 oldEndIdx 移动到 oldStartIdx 前面去了也一样）**。带来的结果就是退出循环，但是**新的节点还没有处理完**！！！

**我们需要把新的节点也补充上去**

补充在哪呢？从头开始，虚拟节点的头是在“当前”位置的头部，所以新增的真实的DOM也需要挂载“当前”位置的头部。

如果有多个新增的，遍历补充即可。

```js
if (oldEndIdx < oldStartIdx && newStartIdx <= newEndIdx) {
  for (let i = oldStartIdx; i < oldEndIdx;i++) {
    patch(null, oldChildren[i], container, oldStartIdx.el) // 锚点不用再变了，因为本来就是按新的虚拟子节点的顺序的，从上往下
  }
}
```

#### 删除元素

删除元素的思路和添加未被处理元素的思路是一样的。

如果有元素被删除了，那么遍历完新的虚拟子节点之后，老的必定还没有遍历完。**即`newEndIdx < newStartIdx`之后，`oldStartIdx` <=  `oldEndIDx`** 

```js
if (oldStartIdx <= oldEndIdx && newEndIdx < newStartIdx) {
  for (let i = oldStartIdx;i < oldEndIdx;i++) {
    unmount(oldChildren[i])
  }
}
```

这是我们只需要把所有没遍历过的老虚拟节点都卸载了就好！

## 双端diff算法的优势

双端算法的实现我们已经了解，那双端算法到底快多少呢？

在简单 diff 算法中，我们已经对传统的 diff 算法做了优化（传统 diff 算法会比较每一个节点其时间复杂度达到了O(n^3)）。

简单 diff 算法虽然只进行同级比较，但是有点类似于冒泡比较，每次比较都需要遍历同层的所有老节点去找匹配的，其时间复杂度也达到了O(n^2)。

双端 diff 算法则每个节点只比较一次，之后就不再参与比较，时间复杂度只有O(n)。