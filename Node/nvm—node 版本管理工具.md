# NVM



## 使用

#### 查看现有 node 版本

```shell
nvm ls # 本地已经安装的

nvm ls-remote # 远程可用的
```

#### 切换版本

切换版本前需要使用 `nvm install`命令先安装该版本

```shell
nvm use v14.15.0 # 切换到 14.15.0 

nvm alias default v14.15.0 # 将默认版本设置为该版本，否则重启 shell 会自动换回默认版本
```

