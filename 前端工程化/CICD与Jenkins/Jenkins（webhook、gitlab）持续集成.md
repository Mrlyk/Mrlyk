# Jenkins

## 一、介绍

### 1、什么是 Jenkins

Jenkins 是一款开源 CI/CD 软件，用于各种自动化任务，包括构建、测试、部署等等。

### 2、如何使用

Jenkins 支持各种运行方式，可通过系统包、Docker 或一个独立的 Java 程序。

## 二、在 linux 上部署

通过容器的方式在 linux 上部署

#### 拉取 jenkins 镜像

```shell
docker pull jenkins/jenkins:latest
```

*ps: 中文文档的`jenkinsci/blueocean`镜像运行后无法访问，服务正常启动，未排查到具体原因（容器日志无异常，jenkins 自身的日志没找到...问题找到了：Chrome 禁止了部分端口的访问）。同时考虑 blueocean 对官方有一定的修改，学习使用还是用了 原版官方镜像* 

#### 启动容器

```shell
docker run \
 --name jenkins
 -u root \
 -d \
 -p 9898:8080 \
 -p 50000:50000 \ # 可选
 -v jenkins-data:/var/jenkins_home \
 -restart=always \
 jenkins/jenkins
```

运行后查看容器日志可以获得 jenkins 的初始化密码：950af00afa944b539f1021a1943c8318（可以查看 docker 日志获得）

*注意端口不要使用 Chrome 的保留端口* 

#### 访问容器

```shell
docker exec -it jenkinsci-blueocean bash
```

主目录：`/var/jenkins_home` 

#### 运行

容器启动完成后，通过 ip+端口访问 jenkins 页面。第一次访问需要输出上面获得的初始化密码。

1. 进入后，新手安装推荐的插件即可
2. 创建管理员账户：liaoyk_code / 960926。也可以使用 admin 账户继续操作

![image-20220322172659679](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220322172659679.png?x-oss-process=image/resize,w_800,m_lfit) 

3. 配置 jenkins URL，一般使用默认值即可。jenkins 将设置该 URL 为服务的基础 URL

![image-20220322173314406](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220322173314406.png?x-oss-process=image/resize,w_800,m_lfit)  

#### 关闭和重启容器

- 关闭：访问路由`ip+端口/exit`
- 重启：访问路由`ip+端口/restart`

## 三、项目配置

以自动构建 gitlab 仓库的项目的为例。

#### 1、安装 node 环境（可选）

在 系统管理 - 插件管理 中，搜索 NodeJs 并安装，用于后面的构建。

插件下载完成后，进入 系统管理 - 全局工具配置，拉到最下面，配置 node 版本。

*这步如果是使用容器来打包构建其实不需要...*

#### 2、安装 gitlab webhook 触发器

在 系统管理 - 插件管理 中，搜索 gitlab 并安装，用于后面和 gitlab 通信，触发构建。

当然也可以通过 gitlab-ci 的方式，调用 jenkins 的构建 URL 来触发构建。前者更简单，后者适用于更多网络情况。

#### 3、配置仓库权限（可选）

在 系统管理 - 系统配置 中增加 gitlab 仓库权限配置。

在 gitlab 个人设置中，生产 jenkins 用的 Access token。

*这步用了后面的 ssh 密钥来访问 gitlab 的话也不需要*

#### 4、创建第一个项目

1. 新建任务，自定义名称并创建一个流水线项目
2. 可以配置“**参数化构建过程**”，即添加可选参数用于构建，参数添加后可以通过`${PARAMETER_NAME}`的方式访问
3. 编写 pipeline 脚本，pipeline 脚本语法见下方的其他说明。脚本主要用来执行具体的构建任务。
4. 给脚本配置运行环境，脚本默认运行在 jenkins 的沙盒中。没有 node 环境等我们需要的环境配置，所以我们需要给脚本配置运行环境。也即下面说的 agent 代理。

#### 5、创建代理节点（slave 主机）

因为我只有一台主机，为了使用多个环境，方案有虚拟机和  docker 两种。这里只使用节点进行打包构建任务，对系统权限，性能等要求不高，且 docker 更简便。所以使用了 docker 方案。我的镜像基于 node 官方镜像，额外安装了 java 和 ssh。

jenkins 要使用 agnet 节点用来构建，需要先能连接上 agent 节点的主机（容器）。可以采用密码或者 ssh 的形式，我们采用 ssh 的形式。

1. 在 jenkins主机上（容器中）生成 ssh 密钥 `ssh-keygen`
2. 将生成的公钥拷贝到目标节点容器中，写入 authorized_keys 文件，以建立 ssh 认证
3. 系统管理 - Manage Credentials 添加 ssh 全局凭证，将上面生成的私钥录入
4. 系统管理 - 节点管理 - 新建节点，新增目标容器作为节点（agent），配置好相关参数和端口（ssh 服务端口，容器需要安装和启动 ssh 服务）。**记住这里的远程工作目录，就是后面流水线的实际执行目录**  
5. 启动 docker 容器，**注意配置分卷挂载到上面的远程工作目录，用于 部署 和防止后面重启容器又需要重新下载依赖** 

**注意：**

- 目标容器需要 java 环境以及打包你的程序所需要的环境
- 目标容器需要保持 ssh 服务运行，以建立 ssh 连接（可以使用 dockerfile 配置启动时运行 ssh）
- 配置节点时在高级中可以目标容器的 ssh 端口（由容器启动时映射决定）
- 目标容器的端口需要在容器服务中开放

#### 6、编写流水线

一个简单的流水线如下

```groovy
pipeline {
    agent {
       node {
           label 'node14' // 创建节点时设置的标签
       }
    }
    stages {
        stage('build') {
            steps {
                sh 'npm --version'
            }
        }
    }
}
```

至此一个简单的项目配置就完成了。接下来就是要编写更详细的构建步骤，完成构建。

#### 7、通过流水线构建部署应用

要部署一个应用，我们有这么几步操作

1. 拉取仓库代码，这里就需要我们从构建的目标容器中去 gitlab 上把代码拉下来
2. 打包构建
3. 将打包后的代码发送到要部署的机器（容器中）
4. 在要部署的机器（容器中）完成代码升级

一步一步说明

**拉取仓库代码**	

拉取仓库代码也有两种方式

1. 通过 shell 命令，执行 git clone 直接去远程仓库拉代码。可以借助`sshpass`或者直接启动`ssh-agent`，在用于打包的容器中配置访问代码仓库的密钥来拉取代码
2. 使用流水线语法，其中凭据是我们在构建用的容器中生成的 ssh 密钥

这里既然已经使用了流水线语法，我们就继续使用流水线来完成任务。要注意在这里我们所有的操作都是在打包容器中进行了，所以要拉取代码，需要我们的打包容器由代码仓库的权限。如果使用第一种方式，需要配置打包容器和代码仓库的 ssh 鉴权。

```groovy
pipeline {
    agent {
       node {
           label 'node14'
       }
    }
    stages {
        stage('gitlab') {
            steps {
                echo '开始拉取代码'
                git branch: '${branch}', credentialsId: 'node14v2-gitlab', url: 'git@gitlab.com:mrlyk/hello-vue3.git'
            }
        }
      
    }
}
```

**打包构建** 

```groovy
pipeline {
    agent {
       node {
           label 'node14'
       }
    }
    stages {
        stage('build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                echo '构建成功 '
            }
        }
    }
}
```

**代码部署**

在上面两步中，代码已经打包完成。第3、4两步要**具体情况具体分析**。是使用容器部署？还是物理机器部署？是直接本机部署还是将产物发送到其他地方？

无论怎样，我们的目标只有一个，完成代码的更新。

首先要结合现有的服务运行方式，以前端静态应用为例，现在我的代码在我容器的宿主机的 ng 上运行。那我要做的就是把 ng 指向的静态资源替换掉。所以这里需要做两件事：

1. 将容器中的产物发送给宿主机
2. 替换宿主机 ng 下的静态文件

这两件事情其实很好做，因为用于构建的容器我们在前面使用了分卷来挂载文件，所以宿主机上可以直接拿到打包后的文件。存在的问题是如何通知宿主机去做这个操作。**容器是不能直接访问宿主机的**。

我想到的方案有以下几种：

- 写一个简单的 node 服务在宿主机运行，容器构建完成后请求该服务。通过该服务来替换文件
- 新增宿主机作为 jenkins 的一个节点，直接在宿主机使用`cp`命令把文件拷过去
- 使用文件变化的 watch 工具比如`chokidar`，同样也要启动一个 node 服务，观察分卷中的文件变化。每次文件改变就执行替换操作

考虑到安全性和扩展性第一种方案是更好的选择。我们可以**通过访问容器的网关以访问到宿主机的服务（容器以默认的桥接的网络方式启动的话）** 

```js
// 用 express 写了个最简单的服务，贴一下核心的一段
router.post('/', async (req, res) => { // 接收 /deploy 的 post 请求
  const { app } = req.body
  console.log('部署应用：', app)
  const targetPath = path.join(rootPath, app) // 替换的目标路径
  try {
    await fs.access(targetPath)
  }catch(e) {
    console.log('文件夹不存在，创建')
    fs.mkdir(targetPath, { recursive: true })
  }
  // 复制文件夹
  const distPath = path.join(distRootPath, `${app}/dist`)  // 打包后的文件路径，由容器挂载的分卷决定
  execSync(`cp -r "${distPath}" "${targetPath}"`) // 在这里把目标文件夹替换掉即可
  execSync('nginx -s reload') // 重启一下 nginx 
  res.json('部署成功')
})
```

发起请求的话可以直接通过`curl`命令，也可以给 jenkins 安装`Http Request`插件。这里简单点我们直接就用`curl`发起请求

```shell
curl -H "Content-Type: application/json" -X POST -d '{"app":"hello world"}' http://172.17.0.1:7788/deploy # 后面跟着服务的路径，参数是部署的应用名
```

至此一套完整的 jenkin + gitlab 部署流程就完成了

## 其他说明

### 流水线语法

官方文档：https://www.jenkins.io/zh/doc/book/pipeline/syntax/

使用 jenkins 最重要的是书写 pipeline 脚本，其语法和**其他的 CI 工具脚本有一定的相似性**。先说明 jenkins 流水线的几个基本概念。

jenkins 流水线语法分为**声明式**和**脚本化**两种

- pipline：流水线，定义了整个 CD 流水线模型。pipeline 块是声明式语法的关键部分
- node：节点，它是 jenkins 环境的一部分。node 块是脚本化语法的关键部分
- stage：阶段，和 gitlab-ci 工具的 stage 类似，表示构建过程的特定过程。一个 stages 可以有多个 stage
- steps：步骤，步骤包含在一个 stage 内（可以有多个具体操作），执行具体的命令。比如如要执行 shell 脚本，就使用 sh 命令

了解这些概念之后对语法做一个简单说明，具体的还是查看官方文档。

#### 声明式语法 

声明式语法顾名思义，操作都靠语句声明。如果想要执行自定义脚本也可以使用`script`逃生舱

```groovy
pipeline { // 定义一个流水线
  agent any // 定义在什么代理环境上执行，还可以配置 docker。也可以在不同的阶段使用不同的 agent
  tools {
    maven 'apache-maven-3.0.1' // 定义自动安装和配置环境变量的工具，可以在全局工具配置中添加（配置第三方工具时会报语法错误，原因不明...）
  }
  options { // 可以单独为流水线提供配置项，如下配置这条流水线超时 1分钟后就中止运行
    timeout(time: 1, unit: 'MINUTES') 
  }
  stages {
    environment {
      NODE_ENV = 'test' // 定义环境变量 使用 ${env.NODE_ENV} 访问
    }
    parameters { // 定义步骤可访问的参数
      string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
    }
    stage('Build') { // 定义执行阶段，和 gitlab-ci 的 stage 相同
      agent { label: master } // 指定子阶段的代理
      steps { // 定义具体执行方方法
        sh 'echo "Hello World"'
        echo '${params.PERSON}'
      }
      withEnv(["ANOTHER_ENV_VAR=here is some value"]) { // withEnv 配置额外环境变量
        echo "ANOTHER_ENV_VAR = ${env.ANOTHER_ENV_VAR}"
      }
    }
    stage('Deploy') {
      steps {
        // 
      }
      script { // 声明式语法中脚本化语法的逃生舱，要想在声明式语法中使用工具比如下面的配置 sonar 静态代码检测就需要依靠这个逃生舱了
        def scanerHome = tool 'sonral-scannerv1'  
        withSonarQubeEnv('SonarServer') {
          sh "${scanerHome}/bin/sonar-scanner -Dsonar.projectKey=test3 -Dsonar.projectName=hello-vue3"
        }
      }
      when {
        branch "master" // 指定何时会进入这个步骤，这里只有 master 分支才会进入该步骤
      }
    }
    post { // 根据流水线构建情况执行操作
      always { // always 表示总是执行，还有其他的比如 success、failure
        //
      }
    }
  }
}
```

#### 脚本化语法  

**在脚本化语法中可以直接使用 groovy 的脚本，而在声明式的语法中不行**（当然声明式语法提供 script 这样的逃生舱）

```groovy
node { // 定义使用的代理，相当于声明式语法的 agent 字段
  stage('Build') { // 执行阶段和声明式的语法一致
    sh 'echo "Hello World"'
    sh '''
    echo "Multiline shell steps works too"
    ls -lah
    '''
    if (env.NODE_ENV === ' test') { // 支持如下脚本语法
      echo '测试环境'
      env.BUILD_ENV = 'test' // 脚本式设置环境变量，注意不能重写已经在 enviroment 中声明的环境变量
    }
    try {
      sh 'exit 1'
    } catch(e) {
      echo '捕获退出异常'
    }
    def browsers = ['chrome', 'firefox']
    for (int i = 0; i < browsers.size(); ++i) {
      echo "Testing the ${browsers[i]} browser"
    }
  }
}
```

#### 重要字段说明

- **agent**：`any` 表示任何代理环境都可以。设置为`none`就需要在每个阶段指定。可以在 **系统管理 - 节点管理中添加更多节点**。因为不同的项目可能需要不同的构建环境，所以我们有时需要配置节点。

```groovy
agent {
	label 'nodejs12' // 指定运行环境的标签
	docker {
		image 'nodejs:12' // 指定 docker 镜像作为代理环境构建（甚至支持 dockerfile 作为 agent）
		args '-v /tmp:/tmp' // 还接收参数，和传递到 docker run 到参数相同
	}
}
// 上面相当于
 agent {
 	node {
 		label 'nodejs12'
 		customWorkspace '/var/lib/custom' // 支持自定义工作目录
 	}
 }
```

我们还可以借助流水线片段生产器，来自动帮我们生成流水线语法：http://110.42.187.35:9898/job/hello%20world/pipeline-syntax/（我的项目）

#### 第三方工具的使用

在 jenkins web 的插件市场，下载好第三方工具之后如何使用的。一般是如下几个步骤

1. 在全局工具设置中，配置好工具所需的参数。以 SonarQube 为例，需要配置插件名称（重要，作为后面调用的标识）使用的版本，是否自动安装。

2. 在 系统管理-系统配置 中配置工具所需要的服务

3. 在流水线中使用，使用工具目前我的方法是使用脚本化的语法（在声明式语法中使用 script）

   ```groovy
   script {
     def path = tool toolName // 获取到工具的安装路径
     withEnv('tool server') { // 一般工具需要结合服务使用，这里就需要配置工具的使用环境
       sh '${path}' // 从安装路径调用工具
     }
   }
   ```

这里要注意这个 `withEnv`是一种内置函数式的声明方法，可能还有其他方法，比如`withSonarQubeEnv`

```groovy
withEnv(["TEST=haha"]) { // 可以重写任意环境变量
  echo "TEST = ${env.TEST}" // TEST = haha
}
```

### 第三方共享库

官方文档：https://www.jenkins.io/zh/doc/book/pipeline/shared-libraries/

流水线支持创建 "共享库" ，可以在外部源代码控制仓库中定义并加载到现有的流水线中。通过这种方式，可以复用重复的逻辑和书写公共逻辑。

### webhook

webhook 是一种通过 web 回调的形式，去通知其他应用数据更新的一种方式。相比传统的 websocket 和 http，具有可处理数据量大，高效的优点。因为它本身可以看作是单向通信的，仅仅发送通知。

详细说明可以参考下面的文章。

参考文章：https://zhuanlan.zhihu.com/p/133449879
