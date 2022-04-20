# mongodb 安装配置手册

官方文档：https://www.mongodb.com/docs/manual/reference/configuration-options/

本文主要说明 mongodb 如何安装和配置以及初始化的一些操作。

系统：CentOS 7 / 8 

[toc]

## 一、安装

mongodb 在官网的安装文件分成了好几份，注意我们**只要下载其中的 server** 就可以了，也对其他几个做说明：

- crypto: 字段级的加密工具
- mongos: [路由器](https://mongodb.net.cn/manual/core/sharded-cluster-query-router/)？
- shell: 命令工具（一般直接进入 mongo 服务即可，不太需要这个工具）

如果使用 yum 或者 dnf（yum 的下一代）安装，直接安装 mongo-org 会自动安装好所有工具，参考最后面的文章

## 二、初始化

#### 配置文件

安装好后，我们需要更改配置文件，配置文件默认读取的是`/etc/mongod.conf`文件

主要是配置以下几项：

- `systemLog`: 明确日志的位置
- `storage`: 明确数据库的存放位置
- `net`: 网络设置，启动的 ip 和端口
- `security`: 安全，开启鉴权访问

可以参照以下配置文件

```yaml
systemLog:
	destination: file # 日志的目标格式，如果 file 需要补充 path；还有 syslog 表示直接终端输出
	logAppend: false # 每次登录时日志是新建还是追加，默认 false
	path: /var/log/mongodb/mongodb.log
storage:
	dbPath: /var/lib/mongo # 数据库位置，默认 /data/db | /var/lib/mongo
	journal:
		enabled: true # 持久化存储
		commitIntervalMs: 100 # 持久化存储的写入间隔，越短效果越好，但是对磁盘影响也越大
net:
	port: 27017 # 开方端口，默认 27017，如果是集群的子进程则为 27018
	bindIp: 127.0.0.1 # 支持访问的 ip，支持多个逗号分隔。支持 ipv6、sock 套接字。开启所有都允许 0.0.0.0
processManagement:
	fork: true # 把 mongos 和 mongod 使用子进程在后台运行
	timeZoneInfo: /usr/share/zoneinfo # 时区文件
security:
	authorization: enabled
```

  *其余配置可参考文章开头的官方文档* 

#### 权限

mongodb 默认是没鉴权的，在上面的配置文件中我们开启鉴权后，第一件事就是创建一个管理员账号。

配置之前先对 mongo 对权限系统做个说明

在 mongodb 中，每个数据库都有自己的用户，创建用户的命令`db.createUser()`，创建后该账户就属于**当前所在的数据库**。

```shell
# 创建账户命令
db.createUser({
	user: "user_name",  # 账户名
	pwd: "user_pwd", # 密码
	roles: ["readWrite"] # 账户角色权限
})
```

**创建用户管理员** 

在没有用户之前，我们可以随意创建操作数据库，一旦开启了鉴权并且有了用户之后，就必须登录了（也可以在进入数据库后使用`db.auth(username, pwd)`获取权限）

要创建一个管理员账户，主要是要赋予他`userAdminAnyDatabase`的权限

```shell
db.createUser({
	user: "admin",  # 账户名
	pwd: "admin123", # 密码
	roles: [{role: "userAdminAnyDatabase", db: "admin"}] # 账户角色权限
})

# 用户管理员角色也只有对角色的管理权限，没有其他数据库的权限。我们可以创建一个所有权限都有的角色（不安全，仅测试）
db.createUser({
	user: "root",  # 账户名
	pwd: "rooy123", # 密码
	roles: ["root"] # 根账户角色
})
```

这样就创建好了管理员账户。有了管理员账户之后，就可以创建普通的数据库角色了。普通的数据库角色和管理员主要也是权限之间的区别。

mongodb 提供了很多内置角色：https://mongodb.net.cn/manual/reference/built-in-roles/

这里说明创建一个普通的具有读写权限的角色，**需要上面的角色管理员账号才能创建**

```shell
db.createUser({
	user: "test",
	pwd: "test123",
	roles: [{role: "readWrite", db: "test_db"}]
})
```

创建好了之后，我们可以在 admin 数据库中，使用`show users`查看创建的账户

**其他角色权限说明** 

1. 数据库用户角色：read、readWrite；
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin;
3. 集群管理角色：clusterAdmin、clusterManager、4. clusterMonitor、hostManage；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root
7. 内部角色：__system

**修改密码** 

两种方式

- 切换到 admin 数据库，`db.changeUserPassword("username", "xxx")`

- 使用命令

  ```shell
  db.runCommand({
  	updateUser: "username"，
  	pwd: "xxx"
  })
  ```

**删除账户** 

```shell
use admin
db.dropUser('user001')
```

**通过账户访问** 

创建好账户之后就可以访问，有两种方式

- 启动 shell 时输出命令`mongo -u test -p -authentication db_name`
- 先进入 mongo，再使用`db.auth(username, pwd)`命令。注意：需要先切换到具有权限的数据库才能 auth

**直接进入 mongo 后，如果没有 auth 授权，那么默认是在 test 这个数据库上**  

## 参考文章

linux 安装 mongodb：https://cloud.tencent.com/developer/article/1626642