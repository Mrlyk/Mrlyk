# gitlab-CI

> gitlab 自动部署介绍

[toc]

## 一.概念介绍

### 1.1 gitlab-ci && 自动化部署工具的运行机制

以gitlab-ci为例：

(1) 通过在项目根目录下配置**.gitlab-ci.yml**文件，可以控制ci流程的不同阶段，例如install/检查/编译/部署服务器。gitlab平台会扫描.gitlab-ci.yml文件，并据此处理ci流程

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211111142312788.png" alt="image-20211111142312788" style="zoom:50%;" align="left"/>

(2) 每次 `push/merge` 后，gitlab 都会检查项目下有没有 .gitlab-ci.yml 这个配置文件。有的话就会执行里面的脚本。

(3) gitlab-ci 还提供了一个可以由其他平台运行的部署流程的机制，一个叫 `gitlab-runner`的工具。只要在对应的平台上运行这个命令行工具，就可以把当前的机器和对应的 gitlab-ci 流程绑定，即每次跑 ci 都在自己的平台上运行。

### 1.2 pipline & job

pipline 是 gitlab 根据项目的 .gitlab-ci.yml 文件执行的**流程**，**由多个任务节点组成**。而这些 pipline 上的**每一个任务节点**都是一个独立的 **job**。

**每个 job 都属于一个 stage 属性，用来表示这个 job 所处的阶段。**

**一个 pipline 有若干个 stage，每个 stage 上有至少一个 job。**一次构建流程就是一个 pipline 的流通过程。

> stage 表示阶段，当前这个 pipline 执行到了哪个阶段，是 lint 格式化，还是 build 在构建了。所以 job 肯定属于某一个 stage，而 stage 肯定至少做了一件事（包含一个 job）。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211116172345334.png" alt="image-20211116172345334" style="zoom:50%;" align="left"/>

### 1.3 Runner

在**特定机器上**根据项目的 .gitlab-ci.yml 文件，对项目执行 pipline 的**程序**。

Runner 可以分为两种：`Specific Runner` 和 `Shared Runner`。

- **Specific Runner** - 我们自己定义的，在自己机器上运行的 runner 程序。通过 gitlab-runner 这个命令行工具，运行 `gitlab-runner register`命令。[gitlab-runner 下载地址](https://docs.gitlab.com/runner/install/)
- **Shared Runner** - gitlab 平台提供的 runner 程序，由 Google 云提供支持。

**Specific Runner 和 Shared Runner 的区别**

| **Specific Runner** | **Shared Runner **   |
| ------------------- | -------------------- |
| 只针对特定项目运行  | 所有项目都可用       |
| 可以自由选择平台    | 默认基于 docker 运行 |
| 免费                | 私人项目超时收费     |

### 1.4 Executor

我们自己的执行 Specific Runner 的平台就是 **Executor**。在自己特定的机器上通过 gitlab-runner 这个命令行软件注册 runner 的时候，命令行就会提示我们输入相应的平台。不同平台的不同特点如下表。

| *Executor*                | SSH  | shell | VirtualBox | Paralles  | Docker | Kubernetes | Custom |
| ------------------------- | ---- | ----- | ---------- | --------- | ------ | ---------- | ------ |
| 每次构建都清空构建环境    | x    | x     | √          | √         | √      | √          | 自定   |
| 重用以前的 clone          | √    | √     | x          | x         | √      | x          | 自定   |
| Runner 文件访问受系统保护 | √    | x     | √          | √         | √      | √          | 自定   |
| Runner 平台迁移           | x    | x     | 部分       | 部分      | √      | √          | √      |
| 无需配置支持并发构建      | x    | x     | √          | √         | √      | √          | 自定   |
| 使用复杂的构建环境        | x    | x     | √          | √         | √      | √          | √      |
| bug 调试和排查            | easy | easy  | difficult  | difficult | middle | middle     | middle |

## 二、gitlab-ci 配置介绍

常用的关键字有如下几个

- stages
- stage
- script
- tags

还有如 cache 缓存配置可以用来优化

### 2.1 stages

stages 定义在 yml 配置文件的最外层，值是一个数组，定义 pipline 的不同流程节点

```yaml
stages: # 分段
	- install
	- build
	- depoly
```

定义如上配置后在 gitlab 的交互界面上就能看到

![image-20211112144409775](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211112144409775.png)

**Job 是 pipeline 的任务阶段，stage、script、tags 这三个关键字都是作为 job 的子属性来使用的**

如下 install-job 是我们 install stages 中的一个 job

```
install-job:
	tags:
		- sss
	stage: install
	script:
		- npm install
```

- stage - 字符串，表示的是当前节点名
- script - 数组，当前流程执行的 shell 脚本
- tags - 数组，**很重要**，是当前 job 的 Excutor 标记。gitlab 的 runner 会通过 tags 去判断能否执行当前这个 job。

### 2.2 tags

如下图的 gitlag-org 就是当前 runner 的 tags，表示**当前这个 runner 只会执行 tag 为 gitlab-org 的 job**，其他的没有 tag 或 tag 不为这个的都不会执行。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211112152258821.png" alt="image-20211112152258821" style="zoom:50%;" align="left"/>



## 三、gitlab-ci 实战

### 3.1 编写一个 gitlab-ci 的 "hello world"

（1）安装 gitlab-runner

```shell
brew install gitlab-runner # 在 mac 上，也可以去官网下安装包
```

（2）初始化 gitlab-runner

```shell
gitlab-runner install # 加载
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner # 也可以手动指定运行 gitlab-ci 的用户。user 是用户名，working-directory 是运行目录，要特别注意这里的权限问题
gitlab-runner start # 运行 停止运行是 stop

gitlab-runner register # 注册

gitlab-runner verify # 如果没配置过 ssh 密钥可能在官网那里是默认不激活的，就需要使用这个命令激活该 runner。如果是已激活的就不需要了
```

注册的时候要在 gitlab 官网的 Settings -> CI/CD -> Runners 中找到 **Specific runners** 的 url 和 token，之后按照提示配置即可。

*ps: runner 的 executor 根据实际机器环境选择，一般 linux 选 shell 就可以*

（3）明确部署步骤

从通用的手动部署的思路来说，安装依赖 install -> 构建产物 build -> 部署到服务 deploy。ci 工具就是将这些步骤自动化。

明确了上述步骤之后就可以着手写一个 gitlab-ci 配置文件了

```yaml
stages: # 基础的三个步骤
  - install
  - build
  - deploy

cache: # 缓存文件
  paths:
    - node_modules
    - dist

install-job:
  tags: # 指定 runner
    - tx_linux
  stage: install # 表明所属的 stage
  script:
    - export NODE
    - npm install

build-job:
  tags:
    - tx_linux
  stage: build
  script:
    - npm run build
  only: # 限定 master 分支
    - master

deploy-job: # 部署
    tags:
      - tx_linux
    stage: deploy
    script:
      - pwd
      - ls -a
      - cp -r ./dist /usr/local/nginx/html # 拷贝静态资源文件到 ng 目录下
      - echo 'deploy success'
    only:
      - master

```

声明完配置之后，当我们将代码推送到 master 分支时，便会自动部署操作

### 3.2 踩坑

1. Specific Runner 虽然是在本机运行的，但是**通过 gitlab-ci 工具来构建我们的项目的时候，运行 shell 的用户是我们在 install 时配置的用户**。这里一定要注意创建的用户的权限和 root 用户是不同的。这会导致我们在执行一些命令的时候产生权限问题。解决方式见附录的用户与项目管理。

2. 不同项目需要注册新的 runner， 可以使用`gitlab-runner register`注册。tags 不同项目可以同名。

## 附：

### 1、gitlab-runner 用户与项目管理

**修改执行用户**

可以通过 `ps -aux | grep gitlab-runner` 命令查看当前的执行用户

（1）先卸载原来 runner 的用户 `gitlab-runner uninstall`

（2）重新设置用户 `gitbal-runner --working-directory /home/gitlab-runner --user test`（user 不设置的话默认是 root 用户）

（3）重启 gitlab-runner，`service gitlab-runner restart`

**删除指定项目**

通过名字删除`gitlab-runner unregister --name <test-runner>`

通过url和token进行删除`gitlab-runner unregister --url <https://xxx> --token <xxx>`

### 2、yml 文件语法

通过缩进控制结构

**YML中的数组写法**

```text
colors
  - red
  - blue
  - yellow
```

相当于JSON中的

```text
{ "colors": ["red","blue","yellow"] }
```

**YML中的对象写法：**

```text
people:
  name: zhangsan
  age: 14
```

相当于JSON中的

```text
{
  "people": {
     "name": "zhangsan"
     "age": 14
  } 
}
```

**YML 片段复用**

```yaml
# 使用 &符号可以定义一个片段的别名
# 使用 <<符号和 * 符号可以将别名对应的YML片段导入
.common-config: &commonConfig
  only: # 表示仅在develop/release分支上执行
    refs:
      - develop
      - release

install-job:
  # 其他配置 ....
  <<: *commonConfig
build-job:
  # 其他配置 ....
  <<: *commonConfig
```

