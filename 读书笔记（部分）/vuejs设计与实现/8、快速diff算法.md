# 快速diff算法

简单的diff算法像是冒泡排序，双端 diff 算法更优一点像是简单查找，找到就用，只需要遍历一次！

快速diff算法则是在双端算法的基础上进一步优化，**在进入diff算法前先进行一些预处理**！

[toc]

## 前置与后置节点预先处理

快速diff算法可以看作节点预处理+优化版的简单diff算法

我们先说节点预处理。

假设新旧虚拟节点如下：

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
    { type: 'p', children: 'world', key: 1 },
    { type: 'p', children: '2', key: 2 },
    { type: 'p', children: '4', key: 4 },
    { type: 'p', children: '3', key: 3 },
  ],
}
```

我们一眼就可以看到头两个节点的 key 是相同的，而前置与后置节点预处理的方式就像我们正常逻辑思考的方式一样。我们先把头部和尾部一致的处理掉，因为他们不需要移动，只需要更新内容即可。

双端算法会把所有的节点都进行比较，而快速算法的第一步就是减少了要比较多节点数量。

实现代码如下：

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    const oldChildren = n2.children
    const newChildren = n1.children
    
    let j = 0 
    let oldVNode = oldChildren[j]
    let newVNode = newChildren[j]
    // 以 j 为坐标，从头向尾找
    while(oldVNode.key === newVNode.key) {
      patch(oldVNode, newVNode, container)
      j++
      oldVNode = oldChildren[j]
      newVNode = newChildren[j]
    }
    
    // 以两个尾端为坐标从尾向头找
    let oldEnd = oldChildren.length - 1
    let newEnd = newChildren.length - 1
    oldVNode = oldChildren[oldEnd]
    newVNode = newChildren[newEnd]
    
    while(oldVNode.key === newVNode.key) {
      patch(oldVNode, newVNode, container)
      oldVNode = oldChildren[--oldEnd]
      newVNode = newChildren[--newEnd]
    }
    
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

这样就处理完了头尾完全一致的只需要更新内容的情况。

#### 添加元素

接下来我们想象一下如果有新增元素，那么 j 和 newEnd 坐标之间的元素就会是新增的元素。

![image-20230512104546043](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20230512104546043.png?x-oss-process=image/resize,w_600,m_lfit) 

以上面我们的例子为数据，此时 `j = 2`，`newEnd = 2`，`oldEnd = 1`。

**新增的元素插入在哪呢？**我们可以考虑放在 j 元素之后，然后往下排，也可以放在 newEnd 的**下一个**元素之前。因为 insert 默认就是放在某某元素之前，所以方便点我们就选择放在 newEnd 的下一个元素之前。

所以代码中可以这么实现

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
      const anchorIndex = newEnd + 1
      const anchor = anchorIndex < newChildren.length ? newChildren[anchorIndex].el : null
      // 说明有新增
      while(j <= newEnd) {
        patch(null, newChildren[j++], anchor)
      }
    }
    
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

#### 删除元素

删除其实和新增的思路是一样的，如果`newEnd < j`了（新虚拟子节点遍历完了）但是`oldEnd > j`（老虚拟节点还没遍历完）。说明有老的元素不存在了要被删除

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
     //...
    } else if (j > newEnd && j <= oldEnd) {
      while(j <= oldEnd) {
        unmount(oldChildren[j++])
      }
    }
    
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

当然这些都是理想情况，头尾都相同，只是有新增和删除，顺序不变，不涉及到元素的移动复用！

实际情况会复杂的多，肯定涉及到元素顺序变更的情况。如下面这个例子

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'hello', key: 1 },
    { type: 'p', children: '4', key: 4 }, // <-----j
    { type: 'p', children: '4', key: 6 },
    { type: 'p', children: '2', key: 2 }, // <-----oldEnd
    { type: 'p', children: '3', key: 3 }, 
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'world', key: 1 },
    { type: 'p', children: '2', key: 2 }, //<------j 1
    { type: 'p', children: '2', key: 5 },
    { type: 'p', children: '4', key: 4 }, // <------newEnd 3 
    { type: 'p', children: '3', key: 3 },
  ],
}
```

节点预处理只能处理头部的 1 和 尾部的 3。中间的元素则是一定要做移动了，它不满足上面两个条件判断的任何一种情况，所以我们需要新增一个逻辑来处理移动的情况。

## 移动元素

这里其实我们直接把双端算法的逻辑移过来就已经达到了一定的优化作用，但是还不够。我们来看看快速diff算法的第二重优化。

这里我们要做两件事：

1. 找出需要移动的节点并移动他们
2. 找出其中要添加或删除的节点

#### 找出需要移动的节点

如何找到需要移动的节点呢？

**快速diff算法借鉴了简单diff算法的思路**：如果新老元素顺序一致，在递增遍历新虚拟子节点时，通过 key 去老的中查找出来相同的节点他们的索引应该也是递增的，如果没有递增了，说明顺序发生了变化，需要移动。

简单diff算法建立一个数组用来存储**新的虚拟子节点对应的真实DOM在老的虚拟子节点中的位置**，比如上面的例子中结果是

```js
[3, -1, 1]
```

什么意思呢？我们要修改的元素的数量是 newEnd - j，建立一个和要修改的元素长度一致的数组 source，默认填充 -1——>`[-1, -1, -1]`，与要修改的元素一一对应！

新的虚拟子节点key为2的在老的子节点中位置是 3。因为 **source 数组是对新虚拟子节点的一一映射**，所以在source数组第一个位置应该是 3。因为key为5的在老的虚拟子节点中找不到，所以第二个位置依然是 -1。同理key为4的在老的虚拟节点中位置是 1，所以第三个位置是 1。

可以观察到新的虚拟子节点对应的真实DOM在老的虚拟子节点中不是递增的，这是因为老的虚拟子节点不符合我们的顺序要求，要进行移动。

*ps：老的虚拟节点的位置代表了现在真实的DOM的位置* 

**最终的目的是操作这个数组对应的真实DOM让他们成为递增的！**

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
     //...
    } else if (j > newEnd && j <= oldEnd) {
     //...
    } else {
      const count = newEnd - j + 1
      const source = new Array(count)
      source.fill(-1) // [-1, -1, -1]
    }
    
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

但是简单diff算法有一个缺点，它通过冒泡的思想遍历新、老虚拟子节点，时间复杂度较高（O(n^2)）。有没有办法优化一下这一点呢？

快速diff算法使用了hash算法的思想，先建立一个**新的虚拟子节点中 key 和索引的对应 map。**（例子中是`{2:1, 4:3, 5:2}` 

然后再去遍历老的虚拟子节点，**从 map 中找到当前老节点的 key 对应的新索引的位置，如果找不到，说明在新虚拟子节点中这个节点被删除了**。如果能找到，在**source数组对应的新子节点位置（比如当前key在新子节点的第3个位置，那么我们也要在 source 的第 3 个位置记录，记住source和新子节点是一一对应的关系）**记录下当前老的虚拟子节点的位置。 

接下来的思路就很简单，和 diff 算法的核心细想一样：我们也建立一个“最大索引值“，**遍历的是老的虚拟子节点也就是真实的DOM在新的虚拟子节点中的位置**，如果两者完全一致，那么位置索引应该也是自增的。如果当前key对应的子节点在新的子节点中位置索引比当前最大索引值小说明不是自增的了，位置发生了变化。

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
     //...
    } else if (j > newEnd && j <= oldEnd) {
     //...
    } else {
     	// ...
      let newStart = j
      let oldStart = j
      
      let moved = false
      let pos = 0
      
      let keyIndex = {}
      for(let i = newStart;i <= newEnd; i++) {
        keyIndex[newChildren[i].key] = i 
      }
      for(let i = oldStart;j <= oldEnd;j++) {
        oldVNode = oldChildren[i]
        const k = keyIndex[oldVNode.key]
        if(typeof k !== 'undefined') { // 在新的虚拟子节点中对应的 key 还能找到索引
          newVNode = newChildren[k]
          patch(oldVNode, newVNode, container)
          // 注意这里的 k - oldStart, soruce 对应的是新子节点中未处理的数据，我们要找的是这些数据在旧的子节点中的位置。
          // k 对应的是当前数据在新的子节点中的位置，i 则是旧的子节点的位置。source和新子节点是一一对应关系，k 是新子节点中当前的位置，所以我们要修改的也是 source 的 k 位置。当然因为是从 j(newStart/oldStart) 开始遍历的，所以要减去多余的
          source[k - newStart] = i // 记录下当前老的虚拟子节点的位置
          
          if (k < pos) { // 如果新的虚拟子节点列表中位置索引小于当前最大索引值了，说明要移动
            moved = true
          } else {
            pos = k
          }
        } else {
          unmount(oldVNode)
        }
      }
    }
    
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

#### 优化—是否需要移动

在真正开始移动前我们还可以做一步优化，因为我们知道，我们最终要操作的最大元素数量是count——新虚拟子列表中未操作的元素数量。

一旦我们在更新旧的虚拟列表子节点的过程中，更新的数量反而比要操作的最大元素数量还多，说明这些多余的元素都是不需要的，直接卸载

举个例子：

```js
const vnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'hello', key: 1 },
    { type: 'p', children: '4', key: 4 }, // <-----j
    { type: 'p', children: '2', key: 2 },
    { type: 'p', children: '6', key: 6 },
    { type: 'p', children: '7', key: 7 }, // <------oldEnd
    { type: 'p', children: '3', key: 3 }, 
  ],
}

const newVnode = {
  type: 'div',
  children: [
    { type: 'p', children: 'world', key: 1 },
    { type: 'p', children: '2', key: 2 }, //<------j 1
    { type: 'p', children: '4', key: 4 }, // <------newEnd 2
    { type: 'p', children: '3', key: 3 },
  ],
}
```

上面的例子中我们要处理的最大元素数量时 newEnd - j + 1 = 2。

在遍历老的虚拟子节点的过程中，当处理完老的 key-4和key-2之后，处理的元素数量已经是 2 个了。我们要处理的最大元素数量也就 2 个，所以剩余的我们可以不用再处理直接卸载掉即可。

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
      //...
    } else if (j > newEnd && j <= oldEnd) {
      //...
    } else {
      // ...
      let newStart = j
      let oldStart = j

      let moved = false
      let pos = 0

      let keyIndex = {}
      for(let i = newStart;i <= newEnd; i++) {
        keyIndex[newChildren[i].key] = i 
      }
      let patched = 0 // 标志已经处理过的元素的数量
      for(let i = oldStart;j <= oldEnd;j++) {
        oldVNode = oldChildren[i]
        if (patched <= count) { // 处理的元素已经达到最大了，剩下的直接卸载掉
          const k = keyIndex[oldVNode.key]
          if(typeof k !== 'undefined') {
            newVNode = newChildren[k]
            patch(oldVNode, newVNode, container)
            patched++
            source[i - oldStart] = i

            if (k < pos) { 
              moved = true
            } else {
              pos = k
            }
          } else {
            unmount(oldVNode)
          }
        } else {
          unmount(oldVNode)
        }
      }
    }

  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

#### 找出最长递增子序列

在确认了我们要处理的数组之后（`[3, -1, 1]`），我们接下来不是按照简单 diff 算法的思路直接去交换位置，还可以在做进一步的优化，思路和节点预先处理一样。

我们找到要处理的数组的最长递增子序列，我们说过，如果数组中位置索引的值是递增的，说明他们的位置关系是正确的。

所以我们要找出其中最长递增子序列，不移动他们。找出不在最长递增子序列中的数据，移动他们对应的真实DOM。当然对于-1值我们要去新增。

所以我们现在就要先找到最长递增子序列，这是一道经典的算法题，属于动态规划类的问题！具体在这里就不说了。

#### 移动（核心）

找到最长递增子序列之后呢？

还是以上面的例子为例，source：`[3, -1, 1]`，最长递增子序列可以认为是`[-1, 1]`,其对应的索引是`[1, 2]`。

我们想一下，如果索引是递增的，说明他们的相对位置是正常的，而这也是我们找最长递增子序列的原因：**所有要改变的元素中，如果元素索引和最长递增子序列中的索引一致，他就不用改变位置。**当然在最长递增子序列中匹配到之后还是就要将游标上移了！

之后倒序遍历所有需要处理的元素：

- 如果要处理的元素的索引在 source 数组中是 -1`source[i] === -1`，说明这个元素是新增的，新增在它的下一个位置之前就好
- 如果要处理的元素的索引和最大递增子序列的索引对应不上（`i !== seq[s]`），说明位置改变了需要移动
- 如果要处理的元素的索引和最大递增子序列的索引对应上，说明这个元素不用动，比如source数组对应的2号元素，也就是老虚拟子节点中的位置 1 不用动，只需要游标`s`上移即可

```js
function patchKeyedChildren(n1, n2, container) {
  if (Array.isArray(n1.children)) {
    //...
    if (j <= newEnd && oldEnd < j) {
      //...
    } else if (j > newEnd && j <= oldEnd) {
      //...
    } else {
      // ...
      const seq = lis(sources) // 最长递增子序列
      let s = seq.length
      let i = count - 1 // 剩余要处理的元素数量
      // 倒序遍历要处理的元素
      for(i;i >= 0;i--) {
        if(seq[i] === -1) {
          // 说明要新增
          const pos = i + newStart // 要新增的元素的位置
          const newVNode = newChildren[pos]
          // 新增到下一个元素前面
          const nextPos = pos + 1
          const anchor = nextPos < newChildren.length ? newChildren[nextPos].el : null
          patch(null, newVNode, container, anchor)
        } else if(moved) {
          f (i !== seq[s]) {
            // 移动
            const pos = i + newStart
            const newVNode = newChildren[pos]
            const nextPos = pos + 1
            // 移动也是移动到下一个元素之前，因为我们在遍历的就是新的虚拟子节点，所以只需要移动节点对应的真实DOM过来就可以了
            // 至于新的虚拟子节点和老的虚拟子节点的内容更新和真实DOM对应在上面我们构建source数组的时候就已经处理过了
            const anchor =
                  nextPos < newChildren.length
            ? newChildren[nextPos].el
            : null
            insert(newVNode.el, container, anchor)
          } else {
            // 相同时说明位置关系一致，不用移动，游标上移即可
            s--
          }
        }
      }
    }
  } else {
    setElementText(n1, '')
    n2.children.for(c => patch(null, c, container))
  }
}
```

