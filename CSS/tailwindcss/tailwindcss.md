# tailwindcss

原子性的 CSS 编写方式







##  为什么不使用内联样式？

对这种方式的一个普遍反应是, “这不就是内联样式吗？” 在某些方面是 — 您是将样式直接应用于元素，而不是为元素分配一个类，然后在这个类中设置样式。

但是使用功能类比内联样式具有一些重要的优点：

- **基于约束的设计**. 使用内联样式, 每个值都是一个魔术数字。 使用功能类, 您是从预定义的[设计系统](https://www.tailwindcss.cn/docs/theme)中选择样式，这使得构建统一的UI变得更加容易。
- **响应式的设计**. 在内联样式中您不能使用媒体查询, 但您可以使用 Tailwind 的[响应式功能类](https://www.tailwindcss.cn/docs/responsive-design)非常容易的构建完全响应式的界面。
- **Hover, focus, 以及其它状态**. 内联样式无法设置 hover 或者 focus 这样的状态, 但 Tailwind 的[状态变体](https://www.tailwindcss.cn/docs/hover-focus-and-other-states)使用功能类可以非常容易的为这些状态设置样式。