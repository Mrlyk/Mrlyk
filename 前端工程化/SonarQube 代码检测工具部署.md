# SonarQuebe 代码检测工具部署

用以扫描分析代码问题的工具。

为什么需要？

- 全组统一的代码风格
- 提前的代码可读性检查
- 详细的代码分析数据

ESLint 存在的问题？

- 各个项目风格不统一
- 统一后历史问题无法处理
- 无法清晰的展示项目问题

为什么选它？

- 可实现项目统一规则配置
- 支持上述的可维护性检测
- 支持 vue、js、ts、react、css、less
- sonarlint 插件支持
- 配置简单且有灵活的可配置规则
- 方便的结合现有 DevOps 平台
- 支持中文

缺点

- 后置检查（可通过 sonarlint 插件解决）
- 不支持自定义 JS/TS 的检查规则...
- 存在一定安全风险，部署相关

[toc]

## 支持的框架和语言

- ECMAScript 3, 5, 2015, 2016, 2017, 2018, 2019, and 2020
- TypeScript 4.4
- React JSX
- Vue.js
- Flow
- CSS, SCSS, Less, also 'style' inside PHP, HTML and VueJS files

## 执行步骤

1. 用户本地可以使用 IDE 的插件进行代码分析
2. 用户上传到源代码版本控制服务器
3. 持续集成，使用Sonar Scanner进行扫描
4. 将扫描结果上传到SonarQube服务器
5. SonarQube server将结果写入db
6. 用户通过web ui查看扫描结果
7. SonarQube导出结果到其他需要的服务

明确了使用步骤之后，我们就需要根据步骤来部署我们的 SonarQube 服务

## 部署

麻烦点就手动从头装，可以参考官方文档或者这篇文章：https://juejin.cn/post/6959732969795747871。从头装就需要下载压缩包，配置环境变量，还要额外安装数据库，比较麻烦。官方直接提供了 docker 镜像，所以这里直接使用镜像来启动服务。

#### 创建分卷

为了防止容器删除时数据、日志、扩展数据的丢失，创建专门的 docker 分卷来存储这些数据

*创建 sonarquebe 分卷* 

```sh
docker volume create --name sonarqube_data
docker volume create --name sonarqube_logs
docker volume create --name sonarqube_extensions
```

*创建数据库需要的分卷*	

```sh
docker volume create --name postgresql # sql 命令
docker volume create --name postgresql_data # 数据
```

#### 拉取需要的镜像

*拉取 sonarquebe 镜像* 

```sh
docker pull sonarqube:community
```

因为 sonarquebe 需要将结果写入数据库，所以我们还需要装个数据库。注意如果是 Orcal 的数据库需要单独下载 JDBC，Mysql 在现在的版本中已经不被支持了。官方推荐使用 PostgreSQL 数据库（阿里开源的关系型数据库）。

*拉取 PostgreSQL 数据库镜像*	

```sh
docker pull postgres:12
```

#### 启动服务

sonarquebe 依赖与数据库，所以要先启动数据库

*启动数据库* 

```sh
docker run -d --name postgres -p 5432:5432 \
> -v postgresql:/var/lib/postgrepsql \
> -v postgrepsql_data:/var/lib/postgrepsql_data \
> -e POSTGRES_USER=sonar \ # 初始用户名
> -e POSTGRES_PASSWORD=sonar_960926 \ # 初始密码
> --restart=always \ # 启动容器时自动重启
> postgres:12
```

如果想查看一下数据库，可以先进入 postgres 容器，再使用创建的账户登录数据库即可

```sh
docker -it postgres bash # 进入 postgres 容器

psql -U sonar -W # 输入密码
```

*启动 sonar 服务* 

```sh
docker run -d \
> -p 9000:9000 \
> -e SONAR_JDBC_URL:jdbc:postgresql://postgres:5432/sonar \
> -e SONAR_JDBC_USERNAME:sonar \
> -e SONAR_JDBC_PASSWORD:sonar_960926 \
> -v sonarqube_data:/opt/sonarqube/data \
> -v sonarqube_extensions:/opt/sonarqube/extensions \
> -v sonarqube_logs:/opt/sonarqube/logs \
> --link postgres \ # 连接到数据库所在的容器
> --restart=always \
> sonarqube:community
```

#### 可选：配置中文插件

插件地址：https://github.com/xuhuisheng/sonar-l10n-zh

两种方式

1. 启动服务后，在 Administration 菜单的 marketplace 中，搜索 Chinese，直接安装
2. github 上下载插件的 jar 包，复制到 sonar 容器的`/opt/sonar/extenesions/plugins`目录。之后手动重启容器

## 代码分析实践

服务部署完成后，访问 ip+9000 端口即可访问服务页面。

第一次登录时默认账号密码：admin/admin，第一次强制修改密码。

*我的测试账号密码：admin/sonar_960926 令牌：588b2445ceb4d4afaf993879de1056fab10d316e* 

sonarqube 可以和各种 DevOps 平台如 gitlab 配合使用也可以直接手动配置项目，下面先说明一下手动配置项目。基于 DevOps 的后面说明。

#### 给项目添加配置文件

在项目的根目录中创建一个名为`sonar-project.properties`

```
# 项目独一无二的 key，不同的项目需要使用不同的标识。可以项目模块中新增
sonar.projectKey=my:project 

# --- optional properties ---

# 项目名
#sonar.projectName=My project
# 项目版本，可以不提供
#sonar.projectVersion=1.0
 
# 源码的相对配置文件的路径
#sonar.sources=.

# sonar 服务地址
# sonar.host.url=https://xxxxx 

# 配置忽略的文件夹，.gitignore 中的文件会被自动忽略
#sonar.exclusions=./dist,./node_modules 
 
# 源码的编码方式，默认使用系统的编码方式
#sonar.sourceEncoding=UTF-8
```

#### 本地安装 SonarScanner

官方文档：https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/

要使用 sonarqube 扫描本地代码需要先本地安装 SonarScanner。下载解压后即可使用，可以配置全局环境变量指向 `install_directory/bin/sonar-scanner`方便后面使用。

#### 使用

配置完成后，直接在项目根目录中使用`sonar-scaner`命令即可，可以结合`git hooks`在提交前自动运行

![image-20220318180755072](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220318180755072.png) 

完成后即可在服务网站上查看扫描结果

![image-20220318180843049](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220318180843049.png?x-oss-process=image/resize,w_1000,m_lfit)  

#### 新代码和所有代码

第一次扫描时会扫描所有包含的代码，第二次则只会扫描新增的代码。如何判断是新增代码？



#### 注意点

- 扫描前 .gitignore 中加入`.scannerwork`文件夹，这是什么文件夹？

## 配置规则

#### 从基础扩展

1. 在“质量配置”中找到要配置规范的语言，比如 js
2. 从内置的 Sonar way 上复制一份规则，并且设为默认，来作为我们的规则。也可以对项目单独制定规则
3. 在“代码规则”中对规则进行详细配置：激活、禁用规则

#### 创建自定义规则

很遗憾，sonarqube 目前不支持 JS/TS 的自定义规则，只能借助第三方工具如 ESLint 来处理。sonarqube 支持接收 ESLint 的分析结果并展示，只需要在配置文件中配置 ESLint 分析结果的报告路径即可。

```
sonar.eslint.reportPaths='./eslint-report.json'
```

其他语言可参照官方文档：https://docs.sonarqube.org/latest/analysis/external-issues/

## 扫描指定代码

使用 fs 写入 include？配合 lint 获取新代码

## 结合 CI 工具使用

7.6 版本之后，免费版不再支持和 gitlab CI 工具直接配合使用。T _ T

需要结合 Jenkins 来执行扫描任务。

要使用 jenkins 来执行扫描任务，需要下面几个步骤：

1. 在 jenkins 上安装`sonnar`插件
2. 在 sonar web 上生成账户可访问的 token，并添加到 jenkins 的凭据中
3. 在 jenkins 的 系统管理-系统配置 中，配置 sonar 服务器地址
4. 添加流水线脚本如下，主要是检测那里

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
        git branch: '${branch}', credentialsId: 'xxx', url: 'git@gitlab.com:xxx/hello.git'
      }
    }
    stage('sonar analysis'){
      steps {
        script {
          def scanerHome = tool 'sonral-scannerv1' // sonar 工具名
          withSonarQubeEnv('SonarServer') { // sonar 服务配置
            sh "${scanerHome}/bin/sonar-scanner -Dsonar.projectKey=test3 -Dsonar.projectName=hello-vue3"
          }
        }
      }
    }
  }
}
```

配置文件也可以写死，也可以放在项目中。最好是单独抽出来，放在 jenkins 的共享库中。

## 结合 IDE 插件使用

Vscode 和 Webstorm 都提供了 SonarLint 插件，通过插件可以配置远程连接的服务器。以远程服务的规则来直接在代码提交前完成校验，作用类似于 ESLint。

以 Vscode 为例，安装好插件后，在用户配置中加入如下配置，即可完成 SonarLint 的服务端配置

```json
{
  "sonarlint.connectedMode.connections.sonarqube": [
    { "serverUrl": "https://sonarqube.mycompany.com", "token": "<generated from SonarQube account/security page>" }
  ]
}
```

对不同的项目，需要在不同的工作区配置项目的唯一 Key。位置是项目根目录的`.vscode/settings.json`文件中

```json
{
  "sonarlint.connectedMode.project": {
    "projectKey": "the-project-key"
  }
}
```

## 工作原理

和 webpack 有异曲同工之处，通过插件式架构。将任务分发给其他工具，最后汇总结果，处理数据，生成统一数据报表。

## 启发

由于 SonarQube 对 JS 不支持自定义的规则，所以有点鸡肋的味道。这里我们的目的是同一整个团队的代码风格，配置统一的代码检测方式，比如 ESLint，StyleLint 等等。而不是让开发者在本地各自配置规则，造成团队不同的项目风格差异明显。

所以我们也可以开发一款类似的工具，提供插件式的扩展，接入上述的开发工具。他们都可以生产代码检测报告，根据报告我们可以产生可视化页面，达到同样的效果。

#### 需求

1. 项目配置统一规则管理
2. 可以接入如 gitlab 这样的平台
3. 可以建立多套规则应用到不同项目