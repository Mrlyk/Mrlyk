# package.json

>  package.json 是 npm 包管理工具的配置声明文件，这里做一些注意点的说明

[toc]

## 属性说明

#### *name 项目名

string

注意

- 不能有大写
- 不能有`_`、`.`开头
- 长度小于 214个字符
- 名称最终会成为 url 的一部分，所以也不能有任何非 url 安全字符

#### *version 版本

string：`x.x.x` 

#### description、keywords

描述性属性，都只是方便用户在 npm 网站上搜索

#### main 入口文件

- default: `'index.js'`

如果是一个库包，用户安装后。通过 CommonJS 直接`require`该包，那么引入的就是该入口文件`exports`的

#### type 项目模块规范

- default：`commonJS`

- option：`module`、`commonJS`

当前工程目录下包的引入规范，npm < 7 目前默认是`commonJS`，所以在代码中直接使用`require`可以，使用 ES6 的`import`则会报错。

使用 `module`即可使用`import`语法，但是又不能使用`require`了...

#### dependencies 和 devDependencies 区别

一开始我以为 dependencies 的包就是项目中真正要用的，devDependencies 是打包的时候的工具。很多文章也是这么说的。**其实不是**。

这里存在一个误区：

**依赖中的代码是否会被打包到项目中，取决于项目中是否有用到这个依赖，因为打包工具都会做 tree-shaking，没用到的才会被剔除掉，只要用了肯定会被打包。和安装在 dependencies 还是devDependencies 无关。** 

dev 和 devDependencies 最大的区别是在使用`npm install --production`安装的时候

如果安装依赖时携带了`--production`，那么在 devDependencies 中的依赖都不会被安装。这个特性一般不是给前端的静态应用使用的。因为静态应用都是打包发布到 web 服务器上作为资源，既然要打包那么一般安装在 devDependencies 的打包工具也需要安装。

要清楚 npm 不仅可以作为**前端静态业务应用**的包管理工具，还是 **node 应用**和我们**工具库**的包管理工具。当我们在服务端部署直接运行的 node 应用时就不需要安装像`eslint`之类的工具。同理开发工具库的时候，用户只需要我们的工具库内容，不需要我们的打包工具等，就可以使用`--production`命令了。这样就能减少不必要的依赖安装。

**综上所述** 

- 在 node 应用和工具库开发时，项目运行直接依赖的库最好安装到 dependencies 。这样在部署时使用`npm install --production`命令即可，可以减小包的体积
- 在静态业务应用中，放在哪里其实都可以。但是为了统一标准，最好还是和 node 应用一样，项目运行直接依赖的放到 dependencies ，风格化的工具、打包工具等放到 devDependencies
- 如果安装依赖时，仅使用`npm install`，dependencies、devDependencies 中的依赖都会被安装

#### peerDependencies 指定要扁平化的依赖

{ [key: string]: string }

一般是写工具库的时候用到，在 npm 3 自动扁平化之后，用的就很少了，因为它相当于显示的声明要扁平化的依赖。

```js
{
  name: "myLess"
  peerDependencies: {
    "less": '3.9.x'
  }
}
```

如上就表示要使用我的`myLess`工具，会把 less@3.9.x 一起安装到顶级目录。

#### private 是否私有

- default: `false` 
- option: `true`、`false` 

设置为 true 时无法使用 npm 发布

#### homepage 主页地址

string：`https://xxxxx.xxx.xx`

主页的 url 地址

#### files 包含的文件

Array<file | dir>：`[]` 

发布包时，非直接依赖的需要包含的文件，比如一些静态资源文件夹，被配置后也会一起发布上去。

**注意：.npmignore 指定的优先级更高** 

#### bin 可执行命令

{ [key: string]: string }：可执行命令，如下是 webpack 的

```js
bin: {
  'webpack': 'bin/index.js'
}
```

#### man 指定文档位置

Array\<string>：['./doc/ReadMe1.md']

#### config 添加命令行环境变量

{ [key: string]: string }

```js
{
  config: { "port": "9999" }
}
```

在项目中可以使用`process.env.npm_package_config_port`来引用

#### engines 指定运行环境

{ [key: string]: string }

```js
{
  engines: {
    "node": "> 10.15.0"
  }
}
```

#### publishConfig 发布时生效的一些配置

{ [key: string]: string }

```js
{
  publishConfig: {
    "tag": "1.0.1",
    "registry": "https://registry.npmjs.org/" 
  }
}
```

#### lib 告诉用户工具的函数模块目录

无实质作用