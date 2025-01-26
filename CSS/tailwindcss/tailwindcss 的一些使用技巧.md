# Tailwind CSS tricks

本文将介绍一些 tailwind css 常用的技巧，让我们日常开发体验更友好。

[toc]

## 动态类名

在 web 程序中，动态更改样式是很常见的，有时候我们需要动态更改类名。

但是记住 tailwind css 需要识别它的特殊类名才能编译成对应的样式。如果像下面这样写是不行滴：

```jsx
const Color = () => {
  const colors = ['red', 'green', 'yellow'];
  const [color, setColor] = React.useState(colors[0]);
  const changeColor = () => {
    setColor('green');
  };
  return (
    <div className={`w-40 h-40 border bg-${color}-500`}></div>
  )
}
```

因为 `bg-${color}-500` 要在 runtime 时才能拿到，tailwind 无法对这个类名做分析。

要做到这一点，我们需要提供完整的类名，像下面这样：

```jsx
const Color = () => {
  const colors = ['bg-red-500', 'bg-green-500', 'bg-yellow-500'];
  const [color, setColor] = React.useState(colors[0]);
  const changeColor = () => {
    setColor('green');
  };
  return (
    <div className={`w-40 h-40 border ${color}`}></div>
  )
}
```

## 在 CSS 中使用 tailwind

- `@apply` 

使用 `@apply` 可以注入 tailwind css

- `theme`

tailwind 提供了几个自定义函数来在 CSS 中访问 tailwind 的值，`theme` 函数就是一个，最终会在构建的时候将 `theme` 函数替换成静态值再生成最后的 CSS。

```css
.__some-external-class {
  /* Using @apply we can use any utility class name from Tailwind */
  @apply text-blue-300 bg-gray-300 py-2 px-6 rounded-lg uppercase;

  /* or using the theme() function */

  color: theme('colors.blue.300');
  background-color: theme('colors.gray.300');
  padding: theme('padding.2') theme('padding.6');
  border-radius: theme('borderRadius.lg');
}
```

#### 动态值

使用`[]`可以直接使用任意值

```jsx
<div class="w-[100vw] bg-[rebbecapurple]"></div>

// 甚至可以在其中访问 theme 函数
<div class="grid grid-cols-[fit-content(theme(spacing.32))]"></div>
 
// 访问自定义的变量，也可以访问 css root 上通过 var 关键字定义的变量
<div class="--my-color: rebbecapurple">
  <h1 class="bg-[--my-color]">Hello</h1>
</div>
```

## 分组和兄弟类

在 tailwind 中使用伪类很方便

```jsx
<button class="bg-purple-500 border border-blue-500 text-white hover:bg-purple-800 hover:border-blue-200 hover:text-gray-200">Click me!</button>
```

在类名前加上需要的伪类状态即可`hover:`、`focus:`、`checked:` 等。

#### 基于父类状态的样式

有时候我们需要基于父类来动态改变样式，比如父类被触摸了，子类字体要变大。这时候就存在一种联动关系。

使用`group`和`group-{modifier}`键字可以帮我们实现这点：

```jsx
<div class="relative rounded-xl overflow-auto p-8">
  
 {/* 在 父类上打上 group 标记 */}
  <a href="#" class="group block rounded-lg hover:bg-sky-500 hover:ring-sky-500">
    <div class="flex items-center space-x-3">
      
      {/* 在 子类上使用 grounp-hover 关键字来触发 */}
     <h3 class="text-sm text-slate-900 font-semibold group-hover:text-white">New project</h3>
    </div>
    <p class="text-sm text-slate-500 group-hover:text-white">Create a new project from a variety of starting templates.</p>
 </a>
</div>
```

这样在父类被 hover 的时候，子类也会对应的更改样式。

#### 基于兄弟样式的状态

上面说的是子类根据父类来变，那如果要根据兄弟类的状态来变呢？tailwind 提供了`peer`关键字。

下面是一个例子：

```jsx
<div class="flex flex-col items-center gap-20 p-10 bg-pink-400">
 <p class="peer cursor-pointer">I am sibling 1</p>
 <p class="peer-hover:text-white">I am sibling 2</p>
</div>
```

在兄弟组件被触摸的时候，自身字体会变为白色。

![2023-07-27 18.09.08](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/2023-07-27%2018.09.08.gif?x-oss-process=image/resize,w_600,m_lfit) 

#### 给分组和兄弟命名

有时候需要面对更复杂的情况，我们需要给 `group`和`peer`命名。

命名的方式很简单，`group/{name}` 这样即可，在使用时也很简单`group-{modifier}/{name}:`。

下面是一个例子：

```jsx
<div class="group/main w-[30vw] bg-purple-300">
 <div class="group/hello peer/second h-20 w-full flex flex-col items-center justify-around">
  <p class="group-hover/hello:text-white">Hello Wolrd!</p>
  <p>All this code belogs to us</p>
 </div>
 <div class="peer-hover/second:bg-red-400 w-[200px] h-[200px] bg-blue-300">
 </div>
 <div class="group-hover/main:bg-green-200 peer-hover/main:bg-red-400 w-[200px] h-[200px] bg-orange-300">
 </div>
```

## 创建自定义工具类

通过 tailwind 的 config 配置 plugins，我们可以创建自定义的工具类。

```js
// tailwind.config.js

const plugin = require('tailwindcss/plugin');

module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: { // ... our previous config },
  plugins: [
    // 使用 theme 函数获取现有的颜色
    plugin(({ theme, addUtilities }) => {
      const neonUtilities = {};
      const colors = theme('colors');
      
      for (const color in colors) {
        // Check if color is an object as some colors in
        // Tailwind are objects and some are strings
        if (typeof colors[color] === 'object') {
          // we opt in to use 2 colors 
          const color1 = colors[color]['500'];
          const color2 = colors[color]['700'];
          
          // 添加自定义类名到工具
          neonUtilities[`.neon-${color}`] = {
            boxShadow: `0 0 5px ${color1}, 0 0 20px ${color2}`,
          };
        }
      }
      // 将工具类添加到 tailwind
      addUtilities(neonUtilities);
    }),
  ],
};
```

然后我们就可以使用它了，注意是直接使用我们添加到工具上的类名：

```jsx
<div class="m-20 w-20 h-10 rounded-lg neon-blue"></div>
```

## 合并类名

在开发中，有时候需要通过 props 传递，或者动态的设置类名，当类名有多个的时候传统的我们使用第三方的工具比如`classnames` 进行合并。

现在也可以使用`tailwind-merge`这个包，更好的兼容 tailwind 的语法。

```jsx
import { twMerge } from 'tailwind-merge'

<button className={
  twMerge('px-2 py-1 bg-red-500 hover:bg-red-800', className)}
>
  Click me!
</button>
```

## 表单控件的颜色

在修改类似 checkbox 的选择框，radio 的单选框这些控件的选框颜色的时候，传统的修改方式比较麻烦。

tailwind 提供了`accent` 关键字， `accent-{color}` 来帮我们轻松做到这一点：

```jsx
<label>
  <input type="checkbox" checked> Browser default
</label>
<label>
  <input type="checkbox" class="accent-pink-500" checked> Customized
</label>
```

![image-20230728104725652](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20230728104725652.png?x-oss-process=image/resize,w_400,m_lfit) 