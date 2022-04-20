# Docker

[toc]

## 一、介绍

### 1、什么是 Docker

Docker 是一个虚拟环境容器，可以将开发环境、代码、配置文件等一起打包到这个容器中，并发布和应用到任意平台中。

### 2、三个主要概念

#### 1）Image（镜像）

类似于虚拟机中的镜像，是一个包含有文件系统的面向 Docker 引擎的只读模版。任何应用程序都需要环境，而镜像就是用来提供运行环境的。镜像的配置根据用户而定，比如用户可以使用 ubuntu 镜像，也可以使用 Apache 镜像。

**layers 分层** 

镜像中还有一个分层的概念。镜像是由一层层只读层叠加而成的，每一层只包含上一层的差异（便于缓存）。所以改变 dockerfile 的时候在中间插入命令，该命令后的 layers 层都会被重新构建，而前面的则使用缓存。
容器与镜像最大的区别就在于可写层上。**如果运行中的容器修改了现有的一个已存在的文件，那该文件将会从镜像的只读层复制到可写层（容器）**，该文件的只读版本仍然存在，只是被可写层中的副本所隐藏了。

#### 2）Container（容器）

类似于一个轻量级的沙盒，可以看作一个极简的 linux 系统环境（包括 root 权限、进程空间、用户空间和网络空间等），以及运行在其中的应用程序。Docker 引擎利用容器来运行、隔离各个应用。容器是镜像创建的实例，可以创建、启动、停止、删除容器。各个容器之间是相互隔离（进程级隔离）的。
*ps：镜像本身是只读的，**容器从镜像启动时，Docker 在镜像的上层创建一个可写层**，镜像本身不变。*

#### 3）Repository（仓库）

类似于代码仓库，但这里是镜像仓库，是 Docker 集中存放镜像的地方。

*ps：注意与注册服务器（Registry）的区别：注册服务器是存放仓库的地方，一般会有多个仓库；而仓库是存放镜像的地方，一般每个仓库存放一类镜像，每个镜像利用tag进行区分，比如Ubuntu仓库存放有多个版本（12.04、14.04等）的Ubuntu镜像。*

官方镜像服务地址：https://hub.docker.com/search?type=image&image_filter=official



### 3、与 VM（虚拟机） 的区别

| 指标     | Docker                                 | VM                               |
| -------- | -------------------------------------- | -------------------------------- |
| 性能     | 共享主机内核，系统级虚拟化，占用资源少 | 需要Hypervisor层支持，资源开销大 |
| 量级     | 轻量                                   | 重度，直接分占主机资源           |
| 安全性   | 只是进程级隔离，存在安全隐患           | 系统级隔离                       |
| 系统底层 | 不支持系统底层操作                     | 支持                             |
| 使用要求 | 可运行在主流的 linux 发行版            | 需要 cpu 虚拟化技术支持          |



## 二、基本操作

> 在命令行中执行的指令，也可以使用桌面端软件。存在一些功能重复的命令，用哪个都可以，有的是老版本的命令

### 1、版本/状态 信息

---

```shell
docker version
# 查看当前 docker 的版本，docker 要启动两个服务，客户端和服务端
```



### 2、运行/停止 命令

---

```shell
docker run -d -p 80:80 docker/getting-started # 生成一个镜像实例，也就是容器，容器名是随机生成的
# -d - 后台运行
# -p 80:80 - 将主机的80端口映射到容器中的80端口
# docker/getting-started - 要使用的镜像
#ps: 可以组合单个字符缩短命令，如上 docker run -dp 80:80 docker/getting-started

docker stop # 停止容器运行 docker stop my_container 
# -t - 可以在一段时间后停止，默认 10s
# ps: 可停止多个容器

docker restart [OPTIONS] CONTAINER [CONTAINER...] # 重启容器实例
# -t 停止容器前的等待时间，默认 10s 
```

**当没有任何前台进程时，docker 会自动停止运行。简单的我们可以写个无限执行的命令来保持容器运行，也可以在启动的时候植入 bash 命令来让容器保持运行状态。** 

#### 2.1 扩展说明

docker run 命令实际上是在指定镜像上 `creates` 了一个可写容器层，然后`start`使用指定的命令。`docker run`相当于 API`/containers/create`then `/containers/(id)/start` 。

所以当要运行一个已经创建好的容器时，直接使用`docker start [name]`即可。

#### 2.2 docker run 的一些重要参数

```shell
docker run --name test -it centos
# --name - 自定义容器名
# -it - 分配一个伪 TTY 并连接到容器的 stdin，让我们可以在容器中输入命令交互
# -t - 分配一个伪 TTY 通常与 i 连用
# ps: 最后的参数基于的镜像。输入 exit 退出交互 shell

docker run -it --privileged ubuntu bash
# --privileged - 让容器获取系统底层能力，存在风险

docker run -e ENV=DEV --env ENV2=TEST --env-file ./env.list
# -e，--env - 自定义环境变量
# --env-file - 从文件读取环境变量

docker run -itd --network=my-net my_container
# --network - 将 my_container 容器添加到 my-net 网络，一般用于组件容器之间通信的局域网
# --ip - 增加 --ip 指令可以指定 ip 地址
# ps: 将运行中的容器添加到网络可以使用 docker network connect 命令

docker run --rm centos
# --rm - 容器停止后清除容器相关的匿名 volume

docker run --restart always postgres:12
# --restart 重启策略，无论如何退出，容器重启时始终启动 postgres 数据库

dodcker run --link container_id 
# 连接到另一个容器，可以访问另一个容器中的服务

dodcker run -u root container_id 
# 用户，从当前系统的组用户中指定
```

#### 查看启动错误信息

启动失败时，可以通过 log 命令查看错误信息

```sh
docker logs container_id
```

### 3、镜像相关

---

```shell
docker search [xxx] # 查找镜像，如 docker search centos 查找 centos 镜像
	 --filter "is-official=true" # 增加过滤条件，这里表示搜索官方镜像；stars=50 搜索50 star 以上的
docker pull [xxx]:tag # 获取镜像，默认最新版本，如 docker pull centos 将 centos 镜像下载到 docker 仓库中
docker image ls # 列出所有镜像
docker image history my_image # 列出镜像历史
docker image prune # 删除未使用的镜像
docker image rm my_image # 删除镜像 -f 强制删除
docker image push [options] NAME[:TAG] # 将镜像推送到服务器上（注册的 docker 账号的远程服务器）
```

#### 3.1 从当前容器配置上创建新的镜像

> 当我们在当前容器上配置了一些参数、开发环境想用于创建多个容器时，可以基于当前容器来创建一个镜像

```shell
docker commit [my_container] [my_image]# 有点像 git，my_container 是刚才运行过的容器，my_image 是构建的目标镜像
# -a - 作者如 -a "Mike"
# -m - 提交的信息
# -c - 将指定命令用于创建的镜像，如指定环境变量
# -p - 提交期间暂停容器
# ps: 镜像名 ： 后面跟着的是 TAG 标志，对镜像进行标识
```

#### 3.2 从外部配置文件（dockerfile）上创建镜像 

```shell
docker build https://github.com/docker/rootfs.git#container:docker
# 从 github 仓库中创建，也可以自己声明配置文件创建
```



### 4、容器相关

---

````shell
docker ps -a # 列出容器（老的命令）不带 -a 时只列出正在运行的容器
docker container ls -a # 列出容器（新的命令，作用相同）
docker container rename my_container new_name # 重命名容器
docker container rm my_container # 删除容器
docker container run my_container # 运行容器
docker container stop my_container # 停止容器
````

#### 4.1 对运行中的容器执行命令

`dockaer exec`：该命令实际是让我们在容器里能和在宿主机上一样执行命令，给容器分配了一个伪 tty

```shell
docker exec -it my_container bash
# 可以进入运行中的容器的 shell 交互，使用 exit 退出

docker exec -it -e VAR=1 my_container sh
# 在即将创建的 shell 交互中设置一个环境变量
```

#### 4.2 容器与系统的文件交互

```shell
docker cp [container]:path [mypath]
```

将容器内的内容复制到自己的硬盘中来查看，比如 `docker cp 5575e733e050:/app ~/tttt1   ` 

也可以反过来，将系统中的文件复制到容器中

#### 4.3 获取容器的配置信息

```shell
docker inspect -format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my_container_id
# 获取容器的 ip

docker inspect --format='{{.Config.Image}}' my_container_id
# 获取容器的镜像名称
```

#### 4.4 容器的环境变量与初始化启动

因为**容器本质是在镜像上创建了一个可写层**。当我们使用`docker exec`的方式进入容器中，实际并不是进入了容器，而是分配了一个伪 tty（操作终端），所以每次都相当于在一个新的 shell 环境中。导致即使我们在 `profile` 中加入了环境变量之类的，再次进入容器的时候也需要重新`source`一遍才会生效。

要想配置每次容器启动时的环境变量和初始化启动一些进程有两种方式：

1.  `.bashrc` 中写入环境变量和启动脚本，因为我们才进入容器的方式最终是使用`bash`这个 shell（不同系统可能用不同的 shell ）。所以`.bashrc`命令是一定会在初始化时执行的
2. 编写`dockerfile`，声明环境变量和启动脚本，并制作新的镜像。上面说了容器是在镜像上的可写层，所以我们的操作应该是围绕镜像来操作才能达成我们的目的，也符合 docker 本身的意义

*注意：docker 的持久化，是指数据的持久化（见下面的分卷），容器本身不是也不需要是，也不能是持久的。*

## 三、存储系统

Docker 的容器上存在一些**存储问题**：

- 容器内的所有内容都是存在容器层上。如果容器删除，数据也会随之消失；
- 容器内的可写层与容器的主机强耦合，很难将数据转移；
- 容器中写入数据需要**存储驱动程序**来管理文件系统，存储驱动程序基于 linux 内核提供联合文件系统。性能不如直接写入主机的分卷；

所以需要一些方法来处理这些问题

在 docker 中存在多种方式让容器在主机中存储文件：`Volumes`、`bind mounts`（如果是 linux 还有 tmpfs mount）

**区别：**

**Volumes**：在主机上创建（`/var/lib/docker/volumes/`），由 Docker 自身管理，非 Docker 进程不应该对这个目录进行修改，也是存储数据的最佳方式。

**bind mount**：存储在主机的任意位置，可以由 Docker 和非 Docker 进程修改，非常高效但具有一个安全隐患：可以通过容器来访问主机的文件系统，而且可能会被其他进程影响目录结构。

**tmfs mount**：挂载在内存中，不会写入文件系统。如果只是暂时生成数据，使用这种方法也不错。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210701174144785.png" alt="image-20210701174144785" style="zoom:50%;" />

### 1、最佳方式：Volumes 分卷 

#### 1.1 volumes 的特性

- `volumes` 可以在容器之间共享和重用；
- 对 `volumes`的修改会立马生效；
- 对 `volumes` 的更新，不会影响镜像；
- `volumes` 默认会一直存在，即使容器被删除；
- `volumes`可以保证目录结构，不会被主机影响；
- 可以方便的迁移或者备份 `volumes`
- `volumes`存储在 linux VM 中，具有更高的 I/O 性能；
- `volumes`启动程序允许将`volumes`存储在云服务器上；

一个创建好的 volume 可以同时安装到多个容器中。当没有容器在使用该卷时，该卷仍然可供 Docker 使用，并且不会自动删除。

**注意：**如果创建`volumes`时目录中已经存在内容，那么这些内容会被复制到`volumes`中。同样如果启动一个容器时指定一个不存在的`volumes`，那么这会创建一个空的`volumes`。

#### 1.2 操作方式

```sh
# 创建 volume，可以在容器或服务创建期间创建卷
docker volume create my_vol --name my_vol
# -v（--volume）- 三个字段组成且顺序不可变（见1.3扩充说明）
# --mount - 和 -v 作用相似，只不过将具体参数放在具体字段中（见1.3扩充说明）

docker volume ls # 列出卷
docker volume inspect my_vol # 检查卷
docker volume rm my_vol # 删除卷
docker volume prune # 删除未使用的 volume
```

#### 1.3 扩充说明

`docker volume create -v `和 `--mount`的作用说明

- **-v**：三个字段组成且顺序不可变，第一个卷名称（不写时随机生成匿名 volumes）;第二个是文件或目录在容器中挂载的路径；第三个是可选的，以逗号分隔的选项列表。

- **--mount**：具体参数放在具体字段中，以逗号分隔。如`--mount 'type=volume,src=<VOLUME-NAME>‘`

  > - `type`类型：其可以是[`bind`](https://docs.docker.com/storage/bind-mounts/)，`volume`，或 [`tmpfs`](https://docs.docker.com/storage/tmpfs/)
  > - `source`：对于命名卷，这是卷的名称。对于匿名卷，此字段被省略。可以指定为`source` 或`src`。
  > - `destination`：在容器中的挂在路径。可以指定为`destination`，`dst`或`target`。
  > - `readonly`：（如果存在）会使绑定挂载以[只读](https://docs.docker.com/storage/volumes/#use-a-read-only-volume)方式[挂载到容器中](https://docs.docker.com/storage/volumes/#use-a-read-only-volume)。
  > - `volume-opt`：可以多次指定的选项采用由选项名称及其值组成的键值对。（不明白作用）

#### 1.4 启动一个有存储的容器

> 卷不存在时会被创建

**--mount**

```shell
docker run -d \
--name devtest \
--mount src=myvol2,target=/app \
[my_image]:latest
```

**--v**

```shell
docker run -d \
--name devtest
-v myvol2:/app
[my_image]:latest
```



## 四、Docker 网络

> Docker 中网络作为容器之间的通信方式，在安装 Docker 时会自动创建三个网络，用户也可以根据业务需要创建 `user-defined`网络

网络相关命令一般以 `docker network`开头，比如要检查网络的信息 `docker network inspect [network_name]`

### 1、none 网络

none 网络，就是无网络，用来做网络隔离，保证安全性。

#### 1.1 创建方式

```shell
# 手动指定
docker run --network=none [my_image]
```

### 2、host 网络

主机网络，和主机享有同样的网络。优点是速率最高，而且可以通过容器直接配置 host 网络。也存在一些缺点，比如会直接占用主机的端口。

#### 2.1 创建方式

```shell
docker run --network=host [my_image]
```



### 3、bridge 网络

创建容器时默认选择的网络。Docker 在安装时会创建一个名为 docker0 的网络。此模式会为每一个容器分配 Network Namespace 和设置 IP，并将容器连接到一个虚拟网桥上。

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20210702183259464.png" alt="image-20210702183259464" style="zoom: 33%;" />

**默认创建的网络连接方式**

```json
"Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "64e2543db9327f6f903521364555295e92599c6c3cd8d9b1d04f374d92e17019",
                    "EndpointID": "00bf0ebf4c9df6143bba01c54f7d3b3669725a5b4fdae3f209158929cfe4160c",
                    "Gateway": "172.17.0.1",// 网关，网关在默认创建的 docker0 上
                    "IPAddress": "172.17.0.2", // 容器分配的虚拟 ip
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
```



### 4、user-defined 网络

除了上述三种自动创建的网路，用户也可以自定义创建 `user-defined` 网络。

Docker 提供三种 `user-defined`网络驱动：

### 4.1 bridge

```shell
docker network create --driver bridge my_bridge_net # 创建
--subnet 172.22.16.0/24 # 指定子网网段
--gateway 172.22.16.1 # 指定网关
--ip 172.22.16.8 # 指定静态ip，否则自动分配
```



#### 4.2 overlay

#### 4.3 macvlan



### 5、断开/删除网络

```shell
docker network disconnect my_bridge_net my_container # 断开网络
docker network rm my_bridge_net # 删除网络，需要与该网络相连的容器都断开了连接
```



## 五、Docker Compose

> compose 用于定义和运行 docker。通过 compose，可以使用 YML 文件配置应用程序需要的所有服务。使用 compose 可以帮我们更清晰的描述 docker 的启动配置，而不是写大段的 cli 命令。（在 mac 下会随着 Docker Desktop 一起安装）

### 1、基本使用

使用 compose 的三个基本步骤

1. 使用 Dokcerfile 定义镜像（可以指定 Dockerfile，默认读取同目录下的）
2. 使用 docker-compose.yml 定义应用程序的服务
3. 执行 docker-compose up 命令来启动运行整个应用程序

#### 1.1 常用的其他可选参数

大部分的配置都能在 docker compose 中直接声明。比如网络配置，volumes 配置......

官方文档：https://docs.docker.com/compose/compose-file/compose-file-v3/

```yaml
services:
	test: # 镜像名
		image: webapp:latest # 单独定义镜像名，否则以项目名作为镜像名
  	depends_on: # 表达服务之间的依赖关系，使用 up 命令时按依赖顺序启动和停止容器
  		- db # 会在 webapp 启动前先启动 db，在 webapp 关闭前先关闭 db
		build: # 构建相关参数
			context:./dir # 构建的执行目录
			dockerfile: ./Dockerfile # 指定构建的 dockerfile 文件
			cache_from: # 缓存镜像
				- node-docker3:latest
			command: npm run dev # 覆盖 dockerfile 中的默认命令，也可以是一个数组
	test2: # 创建第二个镜像
		image: node-docker3 # 指定镜像，会覆盖掉 dockerfile 中的
		
volumes: # 创建分卷
	db_dta: 
```

### 2、操作命令

```shell
docker-compose -f [my_compose.yml] --build up
-f # 指定文件 compose 文件
--build 构建服务，不添加此参数时会使用缓存的容器直接启动；添加时会重新构建镜像和容器
up 创建和启动容器
stop 停止容器，按文件中的顺序停止
```



