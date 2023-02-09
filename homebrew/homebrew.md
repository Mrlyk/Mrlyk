# homebrew

> mac 上的包管理工具

- homebrew 默认将软件安装在`/usr/local/Cellar` 目录
- homebrew 会自动为安装的软件创建软连接到`/usr/local/bin`目录，而这个目录又是默认配置在环境变量中的，所以使用 homebroew 安装的软件无需再配置环境变量！

## 一、homebrew 自身相关

**升级 homebrew**
`brew update`

#### brew 和 brew cask

- brew 是从下载源码解压然后 ./configure && make install ，同时会包含相关依存库。并自动配置好各种环境变量，而且易于卸载。 
- brew cask 是 已经编译好了的应用包 （.dmg/.pkg），仅仅是下载解压，放在统一的目录中（/opt/homebrew-cask/Caskroom），省掉了自己去下载、解压、拖拽（安装）等步骤，同样，卸载相当容易与干净。这个对一般用户来说会比较方便，包含很多在 AppStore 里没有的常用软件。

他们的命令区别就是`brew` 和`brew cask`

#### 配置源

配置源文件需要替换掉 3 个仓库才可以。

1） 替换 / 还原 brew.git 仓库地址

*替换成中科院的 brew.git 仓库地址:*

> cd "$(brew --repo)"

> git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

\#=======================================================

*还原为官方提供的 brew.git 仓库地址*

> cd "$(brew --repo)"

> git remote set-url origin https://github.com/Homebrew/brew.git

2）替换 / 还原 homebrew-core.git 仓库地址

*替换成中科院的 homebrew-core.git 仓库地址:*

> cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"

> git remote set-url originhttps://mirrors.ustc.edu.cn/homebrew-core.git

\#=======================================================

*还原为官方提供的 homebrew-core.git 仓库地址*

> cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"

> git remote set-url origin https://github.com/Homebrew/homebrew-core.git

3）替换 / 还原 homebrew-cask.git 仓库地址

*替换成中科院的 homebrew-cask.git 仓库地址:*

> cd "$(brew --repo)" /Library/Taps/homebrew/homebrew-cask

> git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

\#=======================================================

*还原为官方提供的 homebrew-cask.git 仓库地址*

> *cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
> *

> *git remote set-url origin https://github.com/Homebrew/homebrew-cask*

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

