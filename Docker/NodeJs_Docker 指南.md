# NodeJs_Docker 指南

## 一、为 Node.js 创建 Dockerfile

> 可以使用 dockerfile 来生成指定配置的镜像，dockerfile 可以看作镜像的初始化配置文件

```dockerfile
# 指定解析器指令，用于老的 docker 构建时兼容
# syntax=docker/dockerfile:1

# FROM 指令：指定基于哪个镜像来创建镜像，这里指定官方的 nodejs 镜像 **后面的 run 语句执行相当于是在这个镜像里运行的**
FROM node:14.15.0

#ADD 指令：将 src 源目录中的文件拷贝目标目录中，源目录是本机硬盘地址，目标则是镜像中的，用于读取本地文件
# 和 COPY 的区别是它能直接以网路地址为 src，且它存在一些历史问题会产生困惑
ADD ./google-chrome-stable_current_x86_64 /rpm

# ENV 指令：指定 docker 容器的环境变量
ENV NODE_ENV=production

# 类似于 cd 命令，会从容器的根目录切换到指定的目录（/app）如果文件夹不存在则会创建文件夹
WORKDIR /app

# 在 node 项目中，执行 npm instanll 之前，我们需要将 package.json 和 package-lock.json 放入我们的图像中
# COPY 指令：复制文件，将前面的文件复制到最后一个参数的路径下
COPY ["package.josn", "package-lock.json", "./"]

# RUN 指令：运行命令，在构建镜像（docker build）时运行
RUN npm install --production
# 到此我们就拥有了一个基于 node14.15.0 版本的镜像，并且已经安装了我们的依赖项目。接下来需要将源码放到镜像中。

# 使用 COPY 指令将当前目录中的文件复制到镜像中
COPY . .

# 暴露 80 端口，但是仅仅暴露。启动容器时要使用 -p 或者 -P(和小 p 的区别是随机一个宿主主机端口映射到这个容器 EXPOSE 的端口)或者启动时 --net=host 指定直接使用宿主的端口
EXPOSE 80 

# ENTRYPOINT 容器启动 run 时执行的第一条命令，可以在 run 命令后面追加命令。CMD 则会被 run 命令后追加的命令覆盖掉
ENTRYPOINT ["echo", "Hello world"]

# 有了所有文件之后，我们需要告诉 Docker 容器运行时需要运行什么命令
# CMD 指令：容器运行时（docker run）执行的命令，会被 docker run 的 cli 指令覆盖。同时如果有 ENTRYPOINT 配置，CMD 指令中的参数会被放入 ENTRYPOINT 后作为 ENTRYPOINT 的参数
CMD ["node", "server.js"] # 特别注意 CMD 命令只能有一条（多条最后一条生效），所以我们可以写个脚本来执行多条命令
# CMD node server.js # 这种方式会在容器中将 shell 作为 pid 1 的进程，导致容器丢失完整的进程管理能力，一般不推荐这么写
```

### .dockerignore

提高构建性能，可以使用该文件忽略 node_modules 下的文件

**注意：**

- "[]" 中的命令一定要用**双引号**，单引号会报错
- Run 指令可以执行大部分 shell 命令（除了要求根系统权限的）。**如果存在`apt-get`这种要翻墙的指令可以通过配置docker 自身 ip 的代理** 

**总结：**创建 dockerfile 的过程就像是在实现一个类，我们需要一步步的往下走。先确定镜像继承自谁，再定义一些参数，接着到目录中执行指定的命令

## 二、构建镜像

```shell
# 在当前目录下执行命令，会自动查找 path 目录下的 Dockerfile（注意文件名不要写错）
docker build --tag [name]:[tag] [path]
	-f xxx # 指定配置文件
	--rm # 设置镜像成功后删除中间层，docker 的镜像时分层的（layer）每执行一个命令都会创建一个新的分层
  
# 标记镜像，列表中会出现重复镜像，但并不是复制，只是改变了指向
docker tag [name]:latest [name]:v1.0.0
```

## 三、将镜像实例化为容器运行

通过上述的 Dockerfile 文件，成功的构建一个镜像之后。就可以使用```Docker run```命令来创建和启动容器。

容器运行时会执行 Dockerfile 中 CMD 指定的命令。

### 3.1 注意点

1. 如果 Dockerfile CMD 命令输错了，镜像依然会创建完成，要修改的话需要直接找到镜像的配置文件修改（待确认）
2. 容器启动后要想修改容器的执行参数，也需要找到容器的配置直接修改（未操作过，都是重新创建一个）
3. 容器启动之后是独立运行的，外部是无法访问到容器内部网络的，要想访问到容器内网络需要暴露容器的端口
4. 容器启动默认是基于当前 shell的，shell 退出容器也会停止，可以使用 -d 命令在后台运行

```shell
# 暴露容器的端口使用 -p（--publish）命令 [host port]:[container port]
docker run -p 4000:8000 my_docker # 将容器的8000端口暴露到主机的4000端口上

# 使用 -d（--deteah）在后台运行容器
docker run -dp 4000:8000 my_docker
```

## 四、使用容器进行开发

### 4.1 本地数据库和容器

> 在容器中创建卷（Volumes）来持久化数据传输

```shell
# 创建两个卷，一个用于数据库，一个用于数据库配置
docker volume create mongodb
docker volume create mongodb_config

# 同时需要一个网络来让我们的应用程序和位于卷上的数据库通信
docker network create mongodb
```



### 4.2 启动带有 volume 和 network 的容器

```shell
docker run -it --rm -d \	# --rm - 容器停止时，删除关联的匿名分卷
-v mongodb:/data/db \
-v mongodb_config:/data/configdb \
-p 27017:27017
--network mongodb \
mongo # 使用官方的 mongo 镜像
```

完成上述操作，就可以启动一个容器和服务进行开发了。



### 4.3 使用 dokcer compose 启动容器开发

docker compose 相当于启动的配置文件

**docker-compose.yml 文件的简易结构**

```shell
version: '3.8' # 用来指定 docker compose 的版本，注意要与 docker engine 的版本对应

services:
  webapp: # 构建的 docker 镜像名，会带当前项目名的前缀，比如这里会生成一个镜像 demo-webapp（也可以单独配置 image 属性）
    build: # 构建的参数
      context: . # 构建的开始目录，这里是当前目录
    ports:  # 启动的容器端口映射，相当于 -p 4000:8000 -p 9229:9229
      - 4000:8000
      - 9229:9229 # 9229 是 supervisor 插件默认的调试端口（命令中使用 --inspect 开启调试）
    environment: # 定义环境变量，相当于 -e 
      - SERVER_PORT=4000
    volumes: # 定义 volume，相当于 -v
      - ./:/app # 这里把当前目录绑定到了容器内的 app 目录，所以当前目录和容器中的目录是共享的，改变当前目录内容，容器中的文件也会改变 ./ -> /app
    command: npm run dev # 会替换掉 dockerfile 里的 CMD 命令
```

文件创建好后，使用

```shell
$ docker-compose up 
	-f [file_name] # 可以通过 -f 命令指定配置文件
```

启动 docker 服务，默认会读取当前目录和父目录下的`docker-compose.yml, docker-compose.yaml, compose.yml, compose.yaml` 文件。

可以查看 cli 命令文档获取详细参数：https://docs.docker.com/compose/reference/

如果项目就在当前目录中，那么修改当前目录中的项目文件就可以进行开发了。

## 五、构建测试

暂时略过



## 六、配置 CI/CD

[官方文档](https://docs.docker.com/language/nodejs/configure-ci-cd/)

官方文档是与 github 的 CI/CD 配置结合。

**在 Docker 中运行 Jenkins 配置 CI ，参照 docker 使用 CICD 与 Jenkins。**



## 七、部署到服务器

