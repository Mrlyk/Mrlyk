# homebrew

> mac 上的包管理工具

- homebrew 默认将软件安装在`/usr/local/Cellar` 目录
- homebrew 会自动为安装的软件创建软连接到`/usr/local/bin`目录，而这个目录又是默认配置在环境变量中的，所以使用 homebroew 安装的软件无需再配置环境变量！

## 一、homebrew 自身相关

**升级 homebrew**
`brew update`

## 二、包相关

**安装包**
`brew install [app_name]`

**更新包** 不指定 app 时默认更新所有包
`brew upgrade [app_name]` 

**查看包信息**
`brew info [app_name]`

**查看本地已安装的包**

`brew lsit`

## 三、启动 homebrew 安装的服务

启动：`brew service start xxx` 

停止：`brew service stop xxx`

