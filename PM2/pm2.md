# pm2 进程守护

[toc]

## 常用命令

```shell
pm2 start [app] # 启动项目 pm2 start xxx.config.js 可以直接通过配置文件启动
pm2 stop [id] # 停止项目
pm2 lsit # 查看当前运行的项目
pm2 save # 保存当前已启动的服务
pm2 startup # 设置开机启动（需要先保存当前已启动的服务）
pm2 start npm - start # 运行 npm 的 scripts 命令, pm2 start npm -- run start
```

#### 改名

```shell
pm2 start app.js -n newName

pm2 restart id --name newName
```

## 配置

说明一些 pm2 的常用配置

#### 环境变量

```js
module.exports = {
  // ...
  env: {
    NODE_ENV: 'production',
    BASE_URL: '/'
  }
}
```

#### 自动重启

```js
module.exports = {
  // ...
  autorestart: true, // 自动重启
  cron_restart: '0 0 0 * * 0' // 自动重启时间
}
```

- `autorestart`: 自动重启，在**应用崩溃**时会自动重启，**不推荐生产配置**，如果是代码异常引起的，可能引起服务器卡死
- `cron_restart`：按时自动重启，推荐，具体日期配置规则如下。所以上面的配置是在每周日的 0 点 0 分 0 秒重启

```text
* * * * *
┬ ┬ ┬ ┬ ┬
│ │ │ │ │
│ │ │ │ └─── 星期几（0-6），0 表示星期天
│ │ │ └─────── 日（1-31）
│ │ └────────── 小时（0-23）
│ └───────────── 分钟（0-59）
└─────────────── 秒（0-59）
```

## 踩坑

1. **配置中输出的对象的 key 必须是 `apps` 这个名字，定死的。** 

## start 和 deploy

`pm2 start`和`pm2 deploy`都是 pm2 的命令，但它们有不同的作用。

- `pm2 start`：命令用于在 pm2 进程管理器中启动应用程序。你可以使用 pm2 start 命令指定要启动的脚本文件和其他选项，如日志文件路径、应用程序实例数等。pm2 会在后台启动应用程序，并监视它们的运行状况。如果应用程序崩溃或者被终止，pm2 会自动重启它们，以保持应用程序的稳定运行。

- `pm2 deploy`：命令用于在远程服务器上**部署应用程序**。它可以自动从git仓库中拉取代码、安装依赖、构建应用程序、在服务器上启动应用程序等。你可以使用 pm2 deploy 命令来轻松地在多个服务器上部署应用程序，而无需手动执行每个步骤。在使用 pm2 deploy 命令之前，你需要准备好 pm2 的部署配置文件，**指定要部署的 git 仓库、服务器信息、应用程序启动脚本等信息**。pm2 deploy命令会根据配置文件自动完成部署任务，以便你可以快速地将应用程序部署到生产环境中。

下面是一个 `pm2 deploy` 配置的例子：

```js
// example.config.js
module.exports = {
  apps: [{
    name: "my-app",
    script: "server.js",
    env: {
      NODE_ENV: "production"
    }
  }],
  deploy: {
    production: {
      user: "myuser",
      host: "myserver.com",
      ref: "origin/master",
      repo: "git@github.com:myuser/my-app.git",
      path: "/var/www/my-app",
      "post-deploy": "npm install && pm2 reload ecosystem.config.js --env production"
    }
  }
}
```

在上面的示例中，我们定义了一个名为`my-app`的应用程序，在部署配置文件中使用`deploy`属性定义了一个名为`production`的部署环境。在部署环境中，我们指定了服务器的用户名、主机名、代码仓库、部署路径等信息。`post-deploy`属性指定了部署完成后要执行的命令，其中包括安装依赖、使用pm2重新加载配置文件等操作。

## debugger 

有时候会遇到 pm2 启动报错，但是只有短短一行错误提示，不能很好的定位到具体问题。我们可以通过配置`DEBUG` 参数来让 pm2 输出更多信息。

```shell
DEBUG=* pm2 run xxx.js
```

- `=*`：会输出所有启动信息
- `=error`：只会输出错误信息
