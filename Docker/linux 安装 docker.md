# linux 安装 docker

官方文档：https://docs.docker.com/engine/install/centos/#install-using-the-repository

[toc]

安装 docker 的方式有好几种，这里用最简单的 yum 的方式来安装

## 配置 yum 仓库

配置了 yum 指定 docker 仓库之后，可以从特定的仓库安装和更新 Docker

#### 安装 yum-utils 工具

```text
Yum-utils 是一个与 yum 包管理器关联的实用程序、插件和示例的集合。
有关更多信息，请参见 http://wiki.linux.duke.edu/YumUtils
```

yum-utils 工具集成了对 yum 仓库进行管理的工具——`yum-config-manager`

```sh
yum install yum-utils
```

**添加 docker 源** 

安装了上面的工具后可以使用该工具来添加 docker 源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## 安装 docker

```sh
yum install docker-ce docker-ce-cli containerd.io
```

如果有多个仓库源的话，会默认安装版本最高的那一个。如果要安装指定版本，如下

#### 查看当前有的版本

```sh
yum list docker-ce --showduplicates | sort -r

# 安装指定版本
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

## 启动 docker

使用 systemctl 系统进程管理工具启动

```sh
systemctl start docker
```

启动过后可以通过`docker run hello-world`来判断是否成功安装

## 更新 docker

```sh
yum -y upgrade docker-ce docker-ce-cli # 直接使用 yum 管理工具升级即可
```

