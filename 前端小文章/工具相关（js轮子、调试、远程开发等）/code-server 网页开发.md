# code-server 搭建网页版 vscode

[toc]

## 搭建环境

腾讯云服务器

系统：CentOS 7

环境：node、npm、pm2

## 步骤

#### 1、下载安装包

https://github.com/coder/code-server/releases

可以本地下载，使用 scp 命令传到服务器上

#### 2、解压安装包

```sh
tar -xvf code-server-4.1.0-linux-amd64.tar.gz
```

#### 3、创建执行命令

在解压后的项目目录中找到 package.json，添加执行命令

真正的执行文件是 `./bin/code-server`

```js
"scripts": {
    "start": "./bin/code-server --port 7777 --host 0.0.0.0 --auth password"
}
```

使用`--port` 指定端口，`--auth`指定密码访问方式

#### 4、使用 pm2 守护进程

也可以使用其他工具

```shell
pm2 start --name code-server npm -- start
```

#### 5、通过网页访问

访问时需要密码，密码在 `~/.config/code-server/config.yaml`中，可以手动修改

使用方式与本地 vscode 完全相同

## code-server 启动参数

| 参数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| auth                | 自定义身份验证类型，如果不设置则默认只有password。[password, none] |
| cert                | https证书路径。如果没有提供路径，则自动生成。                |
| cert-key            | 非生成证书时证书密钥的路径，如果用自己的https证书认证的话此段必填。 |
| disable-updates     | 禁用更新                                                     |
| disable-telemetry   | 禁用遥测。就是不允许向微软服务器发送错误或数据信息。         |
| help                | 帮助指令。                                                   |
| open                | 启动时在浏览器中打开。不能远程工作。                         |
| bind-addr           | 设置ip地址访问与端口号。[host:port ]                         |
| socket              | socket路径，设置bing-addr的话此指令可以忽略。                |
| version             | 查看当前版本。                                               |
| user-data-dir       | 用户文件路径。                                               |
| extensions-dir      | 扩展文件存储路径。                                           |
| list-extensions     | 列出vscode安装的所有扩展插件。                               |
| force               | 阻止在安装VS代码扩时显示提示 。                              |
| install-extension   | 通过id或者vsix安装指定vscode扩展插件。                       |
| uninstall-extension | 通过id卸载指定vscode扩展插件。                               |
| show-versions       | 显示vscode扩展插件版本。                                     |
| proxy-domain        | 设置代理端口的域名。                                         |
| verbose             | 启用详细日志记录。                                           |

## 其他

微软官方也推出了域名：https://vscode.dev

