# 简单diff算法

在上一节我们完成了一个傻瓜式的渲染器设计，其具有渲染器的基本功能。

但是它最大的问题是在更新子节点时的性能，我们不可能每次都卸载所有旧节点再挂载所有新节点。操作 DOM 和通过 js 算法优化后再更新存在着数量级级别的性能差距。

diff 算法就是为了解决这个问题，在js层面对更新的数据先做优化再去更新。

[toc]

## 减少 DOM 操作

假设有下面这两个 vnode 

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: '1' },
    { type: 'p', children: '2' },
    { type: 'p', children: '3' },
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: '4' },
    { type: 'p', children: '5' },
    { type: 'p', children: '6' },
  ],
}
```

按照我们傻瓜式的渲染器，这里要先通过3次DOM操作卸载掉原来所有的DOM，再通过3次DOM操作补上现有的3个DOM。一共操作了6次。

但是观察发现如果我每次更新只将老的 vnode 中的文本更新一下，那么只需要3次操作就可以达到目的。

**所以我们可以遍历老的vnode上的children，对children子节点分别调用`patch`方法进行更新。**

但是如果新老节点的数量不同，我们就需要注意了，如果新的多了，那么我们就要新增，如果新的少了，那么我们就要将原来的卸载。

所以我们不能直接遍历新的 vnode，而是遍历其中少的那一个，达到尽可能复用的目的即可。

```js
function patchChildren(n1, n2, container) {
  // ...
  if (Array.isArray(n2)) {
    if(Array.isArray(n1)) {
      const oldChildren = n1.children
      const newChildren = n2.children
      const oldLen = oldChildren.length
      const newLen = newChildren.length
      const commonLength = Math.min(oldLen, newLen)
      for(let i = 0;i < commonLength;i++) {
        patch(oldChildren[i], newChildren[i], container)
      }
      // 新的原来没有的加上去
      if (newLen > commonLength) {
        for(let i = commonLength;i < newLen;i++) {
          patch(null, newChildren[i], container)
        }
      }
      // 老的多的卸载掉
      if (oldLen > commonLength) {
        for(let i = commonLength;i< oldLen;i++) {
          unmount(oldChildren[i])
        }
      }
    }
  }
}
```

但是这里很快有一个问题，如果元素不是全都更新了，而只是换了个顺序。

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: '1' },
    { type: 'p', children: '2' },
    { type: 'p', children: '3' },
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: '3' },
    { type: 'p', children: '1' },
    { type: 'p', children: '2' },
  ],
}
```

这里因为 type 每个都不同，所以每个都还是会执行删除和重新path的操作，还是要6次操作！

## 基于key的复用（简单diff算法）

为了解决上面这个问题，vue 使用了独一无二的 key 来对 vnode 进行标记，如果 key 相同说明元素可以复用。

**注意：元素可以复用并不代表其中的内容不更新了，复用指的是父节点层面的，子内容还是有可能更新**

以下面这两个要渲染的 vnode 为例

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'hello', key: 1 },
    { type: 'p', children: '2', key: 2 },
    { type: 'p', children: '3', key: 3 },
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: '3', key: 3 },
    { type: 'p', children: 'world', key: 1 },
    { type: 'p', children: '2', key: 2 },
  ],
}
```

```js
function patchChildren(n1, n2, container) {
  // ...
  if (Array.isArray(n2)) {
    if(Array.isArray(n1)) {
      const oldChildren = n1.children
      const newChildren = n2.children
      const oldLen = oldChildren.length
      const newLen = newChildren.length
      for(let i = 0;i < newLen; i++) {
        const newVNode = newChildren[i]
        for(let j = 0;j < oldLen;j++) {
          const oldVNode = oldChildren[j]
          if (newVNode.key === oldVNode.key) { // 如果 key 相同，就复用并更新
            patch(oldVNode, newVNode, container)
            break
          }
        }
      }
    }
  }
}
```

这样就完成了老的虚拟节点的更新，但是顺序还是错误的，因为我们这里只是把新的虚拟节点内容更新上去了。顺序还是老的，接下来就需要**移动虚拟节点以保证真实DOM的顺序符合新的虚拟节点的顺序。**

#### 如何判断是否需要移动

假设newVNode和oldVNode顺序没有变更，是一一对应的，那么遍历oldVNode的时候，其`j`肯定也是对应递增的。

**一旦我们发现`j`没有递增反而比前一个还小了，说明在newVNode中这里就有位置的移动了。** 

那么我们就需要把**真实的DOM也和newVNode的顺序对应起来！！！也移动到上一个的后面去** 

为了与前一个进行比较，我们会使用一个变量`lastIndex`将前一个索引值记录下来，他被称为——**遇到的最大索引值！**

```js
function patchChildren(n1, n2, container) {
  // ...
  if (Array.isArray(n2)) {
    if(Array.isArray(n1)) {
      const oldChildren = n1.children
      const newChildren = n2.children
      const oldLen = oldChildren.length
      const newLen = newChildren.length
      const lastIndex = 0
      for(let i = 0;i < newLen; i++) {
        const newVNode = newChildren[i]
        for(let j = 0;j < oldLen;j++) {
          const oldVNode = oldChildren[j]
          if (newVNode.key === oldVNode.key) { // 如果 key 相同，就复用并更新
            patch(oldVNode, newVNode, container)
            if (j < lastIndex) {
              // 不递增了，说明顺序有变：当前节点在老的虚拟node中应该是在前面的，但是在在newVNode中变到上一个节点的后面去了（假设只有两个节点，分治的思想）
              // 所以真实的 DOM 也要变一下，当前节点对应的真实 DOM 也需要移动到上一个节点的后面去（n2.el = n1.el）
              const preVNode = newChildren[j - 1] // 拿到上一个虚拟节点
              if (prevNode) {
                const anchor = preVNode.el.nextSibling // 上一个节点后面的节点作为锚点，这样就能把当前节点插入到上一个节点的后面
                insert(newVNode.el, container, anchor)
              }
            } else {
              lastIndex = j
            }
            break
          }
        }
      }
    }
  }
}
```

这样就完成了位置的移动。

**切记：我们要做的是把真实的 DOM 的位置移动到符合 newVNode，而不是要纠正newVNode的位置！！！** 

当然还有删除和新增需要考虑。如果细心一点也能发现，这里移动了两次，但实际上我们只要移动一次就可以完成任务了，把第一个移动到最后面，2、3不动。这就涉及到优化的 diff 算法——双端 diff 算法！

#### 新增vnode

在上面的代码中，如果新增，在旧的节点中找不到对应的 key 就不会加上去。所以我们需要判断

1. 是不是新增
2. 新增在哪

对于是不是新增，我们可以**新增一个标志位，如果找到了就说明不是新增，如果没找到就是新增。**

对于新增在哪和上面的逻辑是一样的，**因为我们是先遍历的新子节点，所以当前这个新增的真实DOM就放在上一个新子节点对应的真实 DOM 后面就好。** 

```js
function patchChildren(n1, n2, container) {
  // ...
  if (Array.isArray(n2)) {
    if(Array.isArray(n1)) {
      //... 
      const lastIndex = 0
      for(let i = 0;i < newLen; i++) {
        const newVNode = newChildren[i]
        let find = false // 新增标志位
        for(let j = 0;j < oldLen;j++) {
          const oldVNode = oldChildren[j]
          if (newVNode.key === oldVNode.key) { // 如果 key 相同，就复用并更新
            find = true // 找到了就不是新增
            patch(oldVNode, newVNode, container)
            if (j < lastIndex) {
              const preVNode = newChildren[j - 1] // 拿到上一个虚拟节点
              if (prevNode) {
                const anchor = preVNode.el.nextSibling 
                insert(newVNode.el, container, anchor)
              }
            } else {
              lastIndex = j
            }
            break
          }
        }
        if (!find) {
          // 没找到要新增
          const preVNode = newChildren[i - 1]
          let anchor = null
          if (preVNode) {
            anchor = preVNode.el.nextSibling
          } else {
            anchor = container.firstChild // 否则锚点就是第一个元素
          }
          patch(null, newVNode, anchor) // 新元素还需要通过patch创建，需要改造一下 patch 接收 anchor 参数
        }
      }
    }
  }
}
```

#### 删除

删除的话这里使用的是暴力解法，思路很简单：**遍历老的子节点，如果在新的中找不到了就说明要删掉**

```js
function patchChildren(n1, n2, container) {
  // ...
  if (Array.isArray(n2)) {
    if(Array.isArray(n1)) {
      //... 
      const oldChildren = n1.children
      const newChildren = n2.children
      const oldLen = oldChildren.length
      const newLen = newChildren.length
      const lastIndex = 0
      // ...
      for(let i = 0;i < oldLen; i++) {
        const oldVNode = oldChildren[i]
        const has = newChildren.find(vnode => vnode.key === oldVNode.key)
        if (!has) {
          unmount(oldVNode)
        }
      }
    }
  }
}
```

