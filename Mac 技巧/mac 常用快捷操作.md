# Mac 常用快捷操作

## 一、快捷键

- 展示/隐藏 隐藏项目：cmd + shift + .

## 二、操作

#### 在 terminal 中快速打开当前文件夹路径

- 将文件拖入 terminal
- 设置快捷键

#### 软件遗留文件清理

- Config: ~/Library/Preferences/XXX
- System: ~/Library/Caches/XXX
- Plugins: ~/Library/Application Support/XXX
- Logs: ~/Library/Logs/XXX

#### 查找文件

- find [path] -name 'regex' // 通过名称正则在当前路径下查找

#### 远程端口连接性测试

nc 命令本来的作用是用于设置路由器，也可以用来检测端口是否联通

```shell
nc [ip] [port]
	-w # 设置超时秒数
	-v # 显示执行过程
	-l # 监听端口
	-z # 使用0输入/输出模式，只在扫描通信端口时使用。
```

#### 环境变量

Mac系统的环境变量，加载顺序为： 

a. /etc/profile 
b. /etc/paths 
c. ~/.bash_profile 
d. ~/.bash_login 
e. ~/.profile 
f. ~/.bashrc 

我的系统中配置了 zsh 作为运行实际运行到 shell，所以也可以在`~/.zshrc`中配置环境变量

#### 本地路由 route

有时候需要回环 ip 地址或者转发请求地址，就可以通过 route 来实现。无论是 MacOS 还是 windows 还是 linux，都默认安装了这个工具。下面以 mac/windows 上的做说明

**添加 route** 

```shell
route add 本机ip mask 255.255.255.255 网关ip
```

这样本机所有的访问都会强行经过网关再回到本机

**删除 route** 

```shell
route delete 本机ip mask 255.255.255.255 网关ip # 可以只输入 ip，删除所有 ip 对应的路由
```

