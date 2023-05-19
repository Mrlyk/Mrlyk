# package-lock.json

> pacakge-lock.json 在 npm 5 被添加进来，这里对 package-lock.json 文件做一个说明

[toc]

package-lock.json 文件是在 npm 通过任何方法对 node_modules 或者 package.json 文件进行修改时自动生成的。它详细记录了我们安装时的依赖树，**让我们能在后面的安装中和这一次安装的文件完全相同，而忽略`npm install`自动根据模糊匹配规则带来的版本升级导致的问题。**（因为不是所有开发者都遵守开源版本规范）这也是这个文件的作用。

package.json 只能锁定根据模糊匹配规则匹配的版本，不能精确的锁定到某一个版本。

所以如果我们如果去拉别人的项目可能存在这种问题：某个依赖虽然在 package.json 中版本是最新的，但是在 package-lock.json 中依然是老的。导致我们最终安装的版本还是老的。这种时候如果我们要升级依赖，只能通过`@`指定准确的版本才行。

这里还存在一个依赖的子依赖更新的问题：**即依赖的子依赖版本在我的某个环境中不支持，需要更新**。但是当前依赖又没有升级它的子依赖，这时候我们可以手动安装这个子依赖的最新版本。由于 npm 自动扁平化安装的特性，它会删除重复的依赖（将老的删除掉了），保留顶层的，所以就做到了升级。同时还需要保留 package-lock.json 文件。因为 package-lock.json 中才记录了详细的子依赖版本。**注意：首先要确认你要升级的子依赖确实已经通过扁平化到了顶级，而不是在自己子目录到 node_modules 中**

## V6 版本（当前主流）

#### 部分属性说明

- `version`：准确的版本号信息。不像 package.json 中那样的模糊版本控制。如果是从 github 上直接下的，那就是对应的具体地址

- `integrity`：[子资源完整性校验](https://w3c.github.io/webappsec-subresource-integrity/)。参照了 W3C 的标准方案，由两部分组成`hash函数方法 - dgest摘要(base64(hash(content)))`。其中 hash 函数方法由 SHA1 和 SHA512 两种。[MDN 校验说明](https://developer.mozilla.org/zh-CN/docs/Web/Security/Subresource_Integrity)。 

  **资源完整性校验步骤：** 

  1. 下载包内容
  2. 使用 `hash 函数方法 - dgest摘要`方式生成校验值
  3. 和下载下来的包中的值做比较

- `resolved`：依赖下载的地址，一般是官方发布的地址，也可以通过`.npmrc`配置到镜像地址。文件里显示的是真正下载的地址

## V7 版本



## 其他说明

#### npm-shrinkwrap.json

内容和 package-lock.json 完全相同，但是发布包的时候 package-lock.json 不会被发不上去，npm-shrikwrap.json 会被发上去。同时 npm-shrinkwrap.json 的优先级比 package-lock.json 更高。

