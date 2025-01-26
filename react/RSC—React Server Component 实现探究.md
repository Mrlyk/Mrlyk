# RSC 实现探究

在本文中，我们将探究怎样实现一个 RSC（React Server Component） 组件。React 官方实现中比本文将要实现的效率高很多，但实现思路是一致的。

[toc]

![image-20240507181231722](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20240507181231722.png?x-oss-process=image/resize,w_600,m_lfit) 

- **React Server**（或大写的 Server）仅指*RSC*服务器环境。仅存在于 RSC 服务器上的组件，在本示例中。称为**服务器组件**。
- **React Client**（或者只是大写的 Client）是指任何使用 React Server 输出的环境。正如您刚刚看到的，[SSR 是一个 React 客户端](https://github.com/reactwg/server-components/discussions/4)，浏览器也是如此。

## 一、如何渲染 jsx

在古早的前端时代， PHP 还是最好的语言的时代，通过 PHP 渲染的一个页面一般是下面这样的:

```php+HTML
<?php
  $author = "Jae Doe";
  $post_content = @file_get_contents("./posts/hello-world.txt");
?>
<html>
  <head>
    <title>My blog</title>
  </head>
  <body>
    <nav>
      <a href="/">Home</a>
      <hr>
    </nav>
    <article>
      <?php echo htmlspecialchars($post_content); ?>
    </article>
    <footer>
      <hr>
      <p><i>(c) <?php echo htmlspecialchars($author); ?>, <?php echo date("Y"); ?></i></p>
    </footer>
  </body>
</html>
```

这样的页面如何在服务端渲染呢？假设我们有一个服务器，有 node 环境，一种最简单的方式如下：

```js
import { createServer } from 'http';
import { readFile } from 'fs/promises';
import escapeHtml from  'escape-html';

createServer(async (req, res) => {
  const author = "Jae Doe";
  const postContent = await readFile("./posts/hello-world.txt", "utf8");
  sendHTML(
    res,
    `<html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <nav>
          <a href="/">Home</a>
          <hr />
        </nav>
        <article>
          ${escapeHtml(postContent)}
        </article>
        <footer>
          <hr>
          <p><i>(c) ${escapeHtml(author)}, ${new Date().getFullYear()}</i></p>
        </footer>
      </body>
    </html>`
  );
}).listen(8080);

function sendHTML(res, html) {
  res.setHeader("Content-Type", "text/html");
  res.end(html);
}
```

这里最需要注意的一点是在解析动态内容，如 `postContent` 的时候，需要注意不能把文章中的特殊字符解析为了 html 字符串。

传统的一种做法是使用一种单独的模版引擎来构造这个 html，规定好动态数据应该如何传入和处理，比如 ejs 模版引擎。

在 jsx 语法中我们使用 `{}` 来标识动态内容。我们知道 jsx 要被渲染成一个 html，中间经过了一步 vNode 的转换，使用 vNode 来描述我们的 jsx 模版，之后再解析这个 vNode 生成 html。

一般一个 jsx 生成的 vNode 对象如下（如何生成下面的 AST 描述可以参考我的《JS 编译技术概览》）：

```js
// Slightly simplified
{
  $$typeof: Symbol.for("react.element"), // Tells React it's a JSX element (e.g. <html>)
  type: 'html',
  props: {
    children: [
      {
        $$typeof: Symbol.for("react.element"),
        type: 'head',
        props: {
          children: {
            $$typeof: Symbol.for("react.element"),
            type: 'title',
            props: { children: 'My blog' }
          }
        }
      },
      {
        $$typeof: Symbol.for("react.element"),
        type: 'body',
        props: {
          children: [
            {
              $$typeof: Symbol.for("react.element"),
              type: 'nav',
              props: {
                children: [{
                  $$typeof: Symbol.for("react.element"),
                  type: 'a',
                  props: { href: '/', children: 'Home' }
                }, {
                  $$typeof: Symbol.for("react.element"),
                  type: 'hr',
                  props: null
                }]
              }
            },
            {
              $$typeof: Symbol.for("react.element"),
              type: 'article',
              props: {
                children: postContent
              }
            },
            {
              $$typeof: Symbol.for("react.element"),
              type: 'footer',
              props: {
                /* ...And so on... */
              }              
            }
          ]
        }
      }
    ]
  }
}
```

而要将这个 vNode 转换成 html ，一个最简单的方法如下：

```js
function renderJSXToHtml(jsx) {
   if (typeof jsx === "string" || typeof jsx === "number") {
    // 字符串直接渲染，当然要处理特殊字符
    return escapeHtml(jsx);
  } else if (jsx == null || typeof jsx === "boolean") {
    // This is an empty node. Don't emit anything in HTML for it.
    return "";
  } else if (Array.isArray(jsx)) {
    // 数组就遍历渲染
    return jsx.map((child) => renderJSXToHTML(child)).join("");
  } else if (typeof jsx === "object") {
    // 渲染 vNode
    if (jsx.$$typeof === Symbol.for("react.element")) {
      // 转换成 html 标签
      let html = "<" + jsx.type;
      // 拼接 props
      for (const propName in jsx.props) {
        if (jsx.props.hasOwnProperty(propName) && propName !== "children") {
          html += " ";
          html += propName;
          html += "=";
          html += escapeHtml(jsx.props[propName]);
        }
      }
      html += ">";
      // 渲染 children
      html += renderJSXToHTML(jsx.props.children);
      html += "</" + jsx.type + ">";
      return html;
    } else throw new Error("Cannot render an object.");
  } else throw new Error("Not implemented.");
}
```

到此我们就获得了一个最简单的 jsx 渲染方法。在 vue 中也设计了类似的渲染器。都是将一个模版（vue 中是 .vue 文件内的 template 标签中，react 直接就是 jsx）转换成 vNode 描述，之后再解析为一个浏览器能识别的 html。

#### 渲染 jsx 中的组件

在上一节中我们渲染了普通的 html，但是组件能直接用上面的方法渲染吗？答案是不能，如果我们直接用上面的方法渲染组件结果是什么样的呢？

```js
<function Component({postContent,author}) {...}>
</function Component({postContent,author}) {...}>
```

我们的 `renderJSXToHtml` 会将 `jsx.type` 作为标签名，而组件的 `jsx.type` 会是一个函数。

所以我们需要在 renderJSXToHtml 中加一个判断

```js
if (jsx.$$typeof === Symbol.for("react.element")) {
  if (typeof jsx.type === "string") { // Is this a tag like <div>?
    // Existing code that handles HTML tags (like <p>).
    let html = "<" + jsx.type;
    // ...
    html += "</" + jsx.type + ">";
    return html;
  } else if (typeof jsx.type === "function") { // Is it a component like <BlogPostPage>?
    // Call the component with its props, and turn its returned JSX into HTML.
    const Component = jsx.type;
    const props = jsx.props;
    const returnedJsx = await Component(props); // 为了支持异步组件，所以需要 await
    return renderJSXToHTML(returnedJsx); 
  } else throw new Error("Not implemented.");
}
```

