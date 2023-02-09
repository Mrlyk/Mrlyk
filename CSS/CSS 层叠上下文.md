# CSS 层叠上下文

在处理 css 的层叠（重叠）问题时，往往我们只是简单的使用 `z-index` 属性。但有时也达不到我们想要的效果，明明 A 元素的 z-index 比 B 高，为什么 A 还是被 B 挡住了呢？这就是 CSS 层叠上下文的影响。

本文我们就来探究一下 CSS 的面试常客——层叠上下文！

**基本介绍：**

层叠上下文（stacking context）是一个概念，和 BFC（块级格式化上下文）一样，定义很抽象。简单理解我们可以将他看作一个火影中的封印结界，一块层叠上下文就是一层封印，封印内部与外部隔绝。但是为了封印一些比较厉害的敌人可能设有多重封印。即大结界里套小结界，小结界套小小结界。。。重要的是我们要知道什么情况下会创建层叠上下文？

## 如何创建层叠上下文？

我们知道创建一个 BFC 常见的方式有设置`overflow` 为非`visible`的值，设置为浮动元素、定位元素等。

同样的要创建一个层叠上下文也有如下途径

1. 页面根元素天生具有层叠上下文，称为根层叠上下文（`<html>`标签）；

2. 设置了`z-index`为具体数值（不为`auto`）的**定位**元素；

3. **CSS3 带来的属性产生的层叠上下文：**

   （1）元素为`flex布局元素`（父元素`display:flex|inline-flex`），同时`z-index`值不是`auto`。

   （2）元素的`opacity`值不是`1`。

   （3）元素的`transform`值不是`none`。

   （4）元素`mix-blend-mode`值不是`normal`。

   （5）元素的`filter`值不是`none`。

   （6）元素的`isolation`值是`isolate`。

   （7）元素的`will-change`属性值为上面2～6的任意一个（如`will-change:opacity、will-chang:transform`等）。

   （8）元素的`-webkit-overflow-scrolling`设为`touch`。

符合上面任意条件的即会创建一个层叠上下文。

了解如何什么时候回创建一个层叠上下文后，它又对我们实际看到的层叠顺序有什么影响呢？

## 如何确认层叠顺序？

首先放张**“一般层叠顺序比较图”**：

![image-20221128163514611](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221128163514611.png) 

元素在同一个层叠上下文中，如果没有其他因素影响层叠顺序，那么他的默认层叠顺序就如上所示。

如果有其他因素影响呢？

**确认层叠顺序的两条黄金准则：**

1. 在同一个层叠水平上，当元素具有层叠上下文时，`z-index`属性大的元素在上，小的在下；
2. 在同一个层叠水平上，层叠顺序相同时，在 DOM 流后面的会覆盖前面的；

这里有个前提，那就是“在同一个层叠水平上“，何谓层叠水平呢？

#### 层叠水平

层叠水平（stacking level），决定了同一个层叠上下文中元素在 Z 轴上的显示顺序。

所有元素都有层叠水平，包括层叠上下文元素，也包括普通元素。决定最终元素展示层叠顺序的也是层叠水平而不是`z-index`。

**我们需要区分`z-index`与层叠水平**：

`z-index`可以影响层叠水平，但是只限于定位元素以及`flex`盒子的孩子元素；而层叠水平所有的元素都存在。

`z-index`可以影响层叠水平，但是只限于定位元素以及`flex`盒子的孩子元素；而层叠水平所有的元素都存在。

`z-index`可以影响层叠水平，但是只限于定位元素以及`flex`盒子的孩子元素；而层叠水平所有的元素都存在。

重要的事情说 3 遍一定要牢记！！！

下面举个例子：

```vue
<html>
  <!-- ... -->
  <div class="hone">
    <div class="A box" style="position: relative;">A</div>
    <div class="B box" style="position: relative;">B</div>
  </div>
  <!-- ... -->
</html>

```

如上 html 代码，两个盒子 A、B 在根元素下。我们说了根元素就是一个天然的层叠上下文。

那 A、B 两个盒子有没有各自产生自己的层叠上下文呢？

回顾上面产生层叠上下文的方法，他们虽然是定位元素，但是没有明确配置`z-index`，所以`z-index`是默认的`auto`。也即没有产生自己的层叠上下文，**都在根元素层叠上下文中，他们的层叠水平是相同的。**

这时候利用两条准则去判断，很明显的符合准则第 2 条，B 会在 A 之上，结果也是如此，

![image-20221128165214522](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221128165214522.png) 

那如果我现在想 A 在 B 上方呢？

有人立刻就知道了，给 A 加上一个有具体数值的`z-index`让他产生层叠上下文，调整`z-index`的数值刚好可以影响这两个定位元素的层叠水平，达到目的！这样当然可以。那这里就有个问题：

给 A 的`z-index`设置为 0 呢？设置为 0 就产生了层叠上下文呢，这样可以吗？我们需要理解一下层叠上下文！

#### 理解层叠上下文

层叠上下文元素有几个重要**特性**：

1. 层叠上下文的层叠水平要比普通元素高。**元素一旦成为定位元素，`z-index`就会自动生效，默认值为`auto`也可以看作 0 级（但不是 0 级）**；
2. **层叠上下文可以嵌套，内部层叠上下文及其所有子元素均受制于外部的“层叠上下文”**；
3. 每个层叠上下文和兄弟元素独立，也就是说，当进行层叠变化或渲染的时候，只需要考虑后代元素的渲染；
4. 每个层叠上下文是自成体系的，**当元素发生层叠的时候，整个元素被认为是在父层叠上下文（不是父元素）的层叠顺序中**；



了解以上特性之后，根据特性 1 我们知道定位元素可以视为 `z-index: 0`，所以再给 A 设置为 0，他和 B 这个定位元素仍然在同一层叠水平上，B 遵守黄金准则 2 依然在 A 上方。那么怎么办呢？

这里有三个方式让 A 叠在 B 上方：

1. 将`z-index`设置为大于 0 的值，符合黄金准则 1，这是大多数人采用的方式 ；
2. 调整 A 和 B 的 DOM 顺序，符合黄金准则 2；
3. **取消 B 的定位属性，这样 B 就是一个普通的 block，纵使 A 的 z-index 是 0，但它具有层叠上下文，它的层叠水平会比普通元素 B 高，符合特性 1 ；**

```vue
<html>
  <!-- ... -->
  <div class="hone">
    <div class="A box" style="position: relative; z-index: 0;">A</div>
    <div class="B box" style="">B</div>
  </div>
  <!-- ... -->
</html>
```

在项目中，特别是复杂组件中，DOM 层级管理非常麻烦，所以了解特性 1 ，尽量减少`z-index`设置，有助于 DOM 层级管理。（*这里要注意特性 1 中说的具有层叠上下文的元素层叠水平比普通元素要高是指默认情况下。如果`z-index`设置为负值，层叠水平依然会低于普通元素。！* ）

**第 2 个特性如何理解呢？**

可以回看我们上面的“一般层叠顺序比较图”，图中有两个层叠上下文A、B。其中 A  `z-index: 1`，B `z-index: 2`。

现在在 A 中有一个  child-box，将它的`z-index`设置为`9999`，他会在 B 上方还是下方呢？

```vue
<html>
  <!-- ... -->
  <div class="A box" style="position: relative; z-index: 1;">
    A
    <div class="child-box" style="position: relatvie; z-index: 9999"></div>
  </div>
  <div class="B box" style="position: relative; z-index: 2;">
    B
  </div>
  </div>
<!-- ... -->
</html>
```

要回答这个问题，先比较 child-box 的父层叠上下文的层叠水平和 B 的层叠水平。查看上面 html 代码可知，A、B 两个盒子符合创建层叠上下文的途径 2，child-box 的父层叠上下文元素就是 A。

**根据第 2 个特性**，child-box 的层叠顺序无论有多高，都受限于 A 这个父元素的层叠顺序，而 A 的层叠水平比 B 低，所以这里 child-box 无论设置多高的`z-index`都肯定会在 B 下方。

![image-20221128173824487](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221128173824487.png) 

同理，如果 A 的层叠水平（定位元素受`z-index`影响）比 B 高，那么不管 child-box 的层叠水平有多低，就算是`-99999`，child-box 依然会在 B 上方！

**下面是 QA 环节**

可能有人要问，如果 A、B 在同一个层叠水平呢？如下

```vue
<div class="home">
  <!-- A、B 通过 z-index 设置的层叠水平相同 -->
  <div class="A box" style="position: relative; z-index: 0;">
    A
    <!-- child-box 层叠水平受 z-index 影响设置的很高 -->
    <div class="child-box" style="position: relative; z-index: 999; height: 200px;">A-child-box</div>
  </div>
  <div class="B box" style="position: relative; z-index: 0;">
    B
  </div>
</div>
```

- 在同一个层叠水平上，就又回到我们上面说的黄金准则了，B 在 html 中的顺序在 A 后面，所以 B 层叠水平会比 A 高，这样 child-box 无论多高都高不过 B！（受限于特性 2 ）



可能又有人问如果 child-box 设置负的`z-index`，那他会在 A 上方还是下方呢？

- 这里还是要牢记特性 2 ，**内部层叠上下文及其所有子元素均受制于外部的“层叠上下文”**，即结界就是底线，里面的“怪”永远突破不了结界，所以无论 child-box 设置的多低都会在 A 之上。



那又有人问如果 A 不是定位元素，没有创建层叠上下文，child-box 和 A 的层叠顺序如何比较呢（代码如下）？

```vue
<div class="home">
  <!-- 没有创建 层叠上下文 -->
  <div class="A box" style="">
    A
    <div class="child-box" style="position: relative; z-index: -999; height: 200px;">A-child-box</div>
  </div>
  <div class="B box" style="position: relative; z-index: 0;">
    B
  </div>
</div>
```

- 这就要看**特性 4**，child-box 与 A 发生了层叠，child-box 与 A 的父层叠上下文都是根层叠上下文 。父层叠上下文相同那么就是在同一层叠水平上，直接使用黄金准则比较即可。child-box 是定位元素，受`z-index`影响，层叠水平是负值，A 是个普通元素（没有层叠上下文也就没有了结界），可以看作`z-index: auto`。所以 child-box 会在 A 下方。

总之元素比较先比父层叠上下文，同时在心中默念，`z-index`只影响定位元素与 flex 子元素的层叠水平。

#### 注意 CSS3 带来的影响

直接看一个🌰：

```vue
<div class="home">
  <div class="A box" style="transform: scale(1)">
    A
    <div class="child-box" style="position: relative; z-index: -9999; height: 200px">
      A-child-box
    </div>
  </div>
  <div class="B box" style=""></div>
</div>
```

大家觉得 child-box 会在 A 上面还是下面呢？

分析这个问题，先看 child-box 的父层叠上下文在哪个元素上。如果不了解产生层叠上下文**途径 3** 的同学可能就不清楚`transform`不为`none`的情况下会产生层叠上下文。所以这里 child-box 的父层叠上下文是 A。那他和 B 的层叠水平谁高呢？

回忆**特性 1**，层叠上下文的层叠水平要高于普通元素，所以 A 的层叠水平要高于普通元素 B。所以 child-box 受父层叠上下文影响，不管他自身的层叠水平有多低，都比 B 高。

这里要注意一个误区：

有人可能认为，如果我给 A 一个负的`z-index`，那是不是 child-box 就在 B 下方了呢？

心里默念口诀：`z-index` 只影响定位元素与 flex 子元素的层叠水平，**而不是产生了层叠上下文之后，`z-index`就可以影响这个层叠上下文的层叠水平！**

所以纵使给 A 一个负的`z-index`，child-box 也依然会在 B 之上。

那如果想 B 在 child-box 之上要怎么办呢？提升 B 的层叠水平！

**CSS3 带来的属性产生的层叠上下文其层叠水平可以视作 0**，和将元素设置为定位元素但不设置具体的`z-index`数值效果是一样的。所以如果我们将 B 设置为定位元素，根据黄金准则 2，B 的层叠水平会高于 A，child-box 就会在 A 之下了！

```vue
<div class="A box" style="transform: scale(1);'">
  A
  <div class="child-box" style="position: relative; z-index: -9999; height: 200px">
    A-child-box
  </div>
</div>
<div class="B box" style="position: relative;"></div>
```

代码如上，这时候是因为 B 在 DOM 流中的顺序在 A 之后导致其层叠水平更高。当然也可以通过`z-index`来配置其层叠水平。

## 总结

理解层叠上下文，最重要的是记住他的几个重要特性，以及什么时候会产生层叠上下文。

要明确的知道修改`z-index`是希望`z-inex`能影响元素的层叠水平，从影响层叠水平的方式上来看，只有定位元素和 flex 的孩子元素的层叠水平才会受到`z-index`影响，这点要记住。

很多时候我们不知道为什么层叠关系混乱了，很大一部分原因是不清楚怎么产生了层叠上下文，特别是 CSS3 属性带来的影响。

了解之后我们就可以在层叠这一问题上游刃有余了！

最后给一个比较两个元素层叠水平的思路：**要比较元素层叠水平，需要在同一层叠上下文中进行比较。如果两个元素不在同一个层叠上下文，则寻找到他们的”公倍数“，即在同一个层叠上下文中的元素自身或父元素进行比较。**

比较时将层叠上下文当作一个整体！一般思路如下：

> DOM 结构：
>
> A
> ——A-1
>
> B
> ——B-1
> ————B-2
>
> 假设 A、B 没有产生层叠上下文，B-1 产生了层叠上下文。那么 A-1 所在的父层叠上下文就是根层叠上下文。
>
> B-2 要与 A-1 比较层叠水平，就从自身往上找“公倍数”。找到 B-1 有层叠上下文，但是 B-1 和 A-1 的父层叠上下文不在同一层，所以 B-1 继续往上找，找到根层叠上下文，获得了这个“公倍数”。再将 A-1 和 B-1 进行比较。
>
> 如果 A-1 层叠水平比 B-1 高， 那么 B-1 及子元素 B-2 都会在 A-1 之下。

如果不理解可以看下面两个栗子。

#### 两个🌰

🌰1：比较的元素直接在同一层叠上下文中

```vue
<div class="A box">
  A
  <div class="child-box" style="position: relative; z-index: 1; height: 200px">
    A-child-box
  </div>
</div>
<div class="B box">
  <div class="C" style="position: relative; z-index: 2; width: 40px; height: 40px; background-color: aquamarine;">C</div>
</div>
```

其中 child-box 和 C 的父层叠上下文都是根层叠上下文，是同一个，同时他们自身都产生了层叠上下文且 C 比 child-box 高，所以 C 会在 child-box 上面。同时产生层叠上下文的元素的层叠水平比普通元素高，所以 child-box 也会在 A、B 上面。如下：

![image-20221129152853319](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221129152853319.png) 

🌰2：比较的元素不直接在同一个层叠上下文中

```vue
<div class="A box">
  A
  <div class="child-box" style="position: relative; z-index: 1; height: 200px">
    A-child-box
  </div>
</div>
<div class="B box" style="position: relative; z-index: 0;">
  <div class="C" style="position: relative; z-index: 2; width: 40px; height: 40px; background-color: aquamarine;">C</div>
</div>
```

其中 child-box 的父层叠上下文是根层叠上下文，C 的父层叠上下文是 B，两者是父子元素。这时候为了比较层级，我们需要继续往上找，B 的父层叠上下文是根层叠上下文。这时候 B 和 child-box 的父亲层叠上下文相同可以比较了。

示意图如下：

![image-20221129160100106](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221129160100106.png) 

没有产生层叠上下文的 A 我们直接当他是空气。child-box 的层叠水平受`z-index`影响比 B 要高，所以 B 及其子元素 C 都会在 child-box 之下。

![image-20221129153652067](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20221129153652067.png) 

## 参考

- 《CSS 世界》—— 张鑫旭





