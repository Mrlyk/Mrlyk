# redis

REmote DIctionary Server(Redis) 是一个由 Salvatore Sanfilippo 写的 key-value 存储系统，是跨平台的非关系型数据库。

官方文档：https://redis.io/docs/getting-started/installation/install-redis-on-mac-os/ （mac）

## 安装

**mac**

使用 mac 自带的包管理器 `brew` 进行安装

```shell
brew install redis
```



## 启动

**mac**

使用 `brew` 安装的使用如下命令启动

```shell
redis-server

brew services start redis # 开机自启
```