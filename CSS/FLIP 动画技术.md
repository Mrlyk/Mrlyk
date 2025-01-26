# FLIP 动画技术

FLIP是一种记忆设备和技术，最早是由@Paul Lewis提出的，FLIP是First、Last、Invert和Play四个单词首字母的缩写。这里简单的陈述一下：

- First，指的是在任何事情发生之前（过渡之前），记录当前元素的位置和尺寸。可以使用getBoundingClientRect()这个API来处理
- Last：执行一段代码，让元素发生相应的变化，并记录元素在最后状态的位置和尺寸
- Invert：计算元素第一个位置（first）和最后一个位置（last）之间的位置变化（如果需要，还可以计算两个状态之间的尺寸大小的变化），然后使用这些数字做一定的计算，让元素进行移动（通过transform来改变元素的位置和尺寸），从而创建它位于第一个位置（初始位置）的一个错觉
- Play：将元素反转（假装在first位置），我们可以把transform设置为none，将其移动到last位置，让元素有动画效果

用代码标识如下：

```js
/****** First ********/
const first = el.getBoundingClientRect();

/****** Last ********/
// 通过给元素添加一个类名，设置元素最后状态的位置和大小 
//（在.totes-at-the-end中添加相应的样式规则）,布局发生了变化
el.classList.add('totes-at-the-end') 
// 记录元素最后状态的位置和尺寸大小 
const last = el.getBoundingClientRect()

/****** Invert ********/
const deltaX = first.left - last.left 
const deltaY = first.top - last.top 
const deltaW = first.width / last.width 
const deltaH = first.height / last.height

/****** Play ********/
el.animate(
  [ 
    { transformOrigin: 'top left', transform: ` translate(${deltaX}px, ${deltaY}px) scale(${deltaW}, ${deltaH}) ` }, 
    { transformOrigin: 'top left', transform: 'none' }
  ], 
  { duration: 300, easing: 'ease-in-out', fill: 'both' } 
);

```

