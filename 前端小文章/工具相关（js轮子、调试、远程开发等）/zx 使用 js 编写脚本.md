# zx

> google 推出的一款允许使用 js 语法编写  shell 脚本的工具，一句话强的吖匹

官方文档：https://github.com/google/zx

```sh
#!/usr/bin/env zx  
# 记得声明上面的执行程序

await $`cat package.json | grep name` # 支持 await 语法

let branch = await $`git branch --show-current` # let 变量声明
await $`dep deploy --branch=${branch}`

await Promise.all([ # promise 语法
  $`sleep 1; echo 1`,
  $`sleep 2; echo 2`,
  $`sleep 3; echo 3`,
])

let name = 'foo bar'
await $`mkdir /tmp/${name}
```

## 安装

```shell
npm i -g zx
```

## 使用

1、拥有权限后，直接执行，自动解析脚本的 shebang 符号（#!）来指定执行程序

```shell
chmod +x ./script.mjs
./script.mjs
```

2、使用工具来执行

```shell
zx ./script.mjs
```

