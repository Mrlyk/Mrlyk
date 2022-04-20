# npm 私服搭建

> 通过主流的 verdaccio 来搭建

[toc]

## Vardaccio

> 基于 nodejs 开发的轻量级私有 npm 代理服务，需要 node 环境。

### 1、安装

```shell
npm i verdaccio -g
```

### 2、简单启动

```shell
verdaccio -h # 查看基础操作

verdaccio # 直接本地启动，默认端口4873
```

### 3、配置

启动后可以看到默认的配置文件存放地址，默认在 `用户/.config/verdaccio/.config.yaml`

**重要参数**

- **storage** - 发布到私服的包的存放地址，基于配置文件的路径
- **uplinks** - 链接其他人的仓库，比如官方仓库，淘宝镜像等，这里只是做一个定义，真正的使用在 packages 管理的参数中如下

```yaml
uplinks:
	npmjs:
		url: https://registry.npmjs.org/
  taobao:
		url: https://registry.npm.taobao.org/
```

- **packages** - 最重要的参数，通过配置该参数可以设定包权限

如果我像下面这样配置，那么只要发布的包是 `@liaoyk` 前缀的就是私有包，不会通过 proxy 代理到外部去找。否则就会匹配到 `**`。proxy 配置项表示如果在私有仓库中没找到包则会代理到 npm 公有仓库中拉取并缓存到 storage 文件夹。

```yaml
packages:
  '@liaoyk/*':
    access: $all # $all - 所有人 $authenticated - 注册用户 $anonymous - 未注册用户（匿名用户）
    publish: $authenticated
    unpublish: $authenticated
  '**':
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs #对应 uplinks 中的名字
```

- **listen** - 配置启动端口，`listen: 0.0.0.0:3000` 在 3000 端口启动

### 4、发布

 首次发布，使用 adduser 登录

`npm adduser --registry http://localhost:4873/`

`npm publish --registry http://localhost:4873/`

*ps：npm login 是 npm adduser 的别名* 

### 5、删除已发布的包

不同的私服服务有不同的管理策略，官方的 npm.org 策略是只允许删除 24 小时内发布的包，之后的就不再允许删除。

`npm unpublish --force [pkg_name]`

无法删除的包可以像使用者发送作废信息

`npm deprecate [pkg_name]@[version] <message>`

其中 version 可以指定特定版本，如 @1.0 。也可以指定范围如 @'<2.0'

## 在服务端启动

```shell
pm2 start verdaccio # 使用 pm2 在服务端启动
```



## 权限控制

在使用 `npm aduser --registry xxxx` 之后，会生成一个 token，npm 会将这个 token 存储在全局的 .npmrc 配置文件中

```
# .npmrc
//localhost:4873/:_authToken="/HxkLris3YKET9x0K3MIyg=="
```

在 config.yaml 中默认存在 auth 配置项（3.x版本以上）,默认使用 htpasswd 管理

```yaml
auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    # max_users: 1000
```

可以配合 packages 控制允许发布的人员类型，**再将 auth 配置项中的 max_users 设置为 -1**，禁止其他人员注册，达成权限控制。

## 私服的搭建工具

- [Verdaccio](https://verdaccio.org) 主流工具，node 编写，体验类似 npm 官网
- [sinopia](https://github.com/rlidwka/sinopia) 和 verdaccio 类似，但是没人维护了，verdaccio 是基于这个继续编写的，精简
- [Nenus](https://www.sonatype.com/nexus/repository-oss) 强大的工具，可以同时处理 maven、npm 等私有仓库，只搭建 npm 的话有点浪费