# Linux

[toc]

## 一、常用命令

### 0、man 命令（manual linux 手册）

使用该命令可以查看 linux 系统各个命令的使用手册

```shell
man cp # 查看 cp 命令的帮助信息
man ssh_config # 查看 ssh config 文件的配置字段
	-a # 在所有 man 帮助手册中搜索
	-d # 检查文件信息
	-w # 显示命令所在位置
```

### 1、linux 自带包管理工具 apt（[Advanced Packaging Tool](https://wiki.debian.org/Apt)）

```shell
apt-get update # 更新
apt-get install vim # 安装包
```

### 2、查看磁盘空间状态

```shell
df -h # 查看磁盘占用空间
du -sh myPath # 查看当前 path 下的文件具体大小
	-s 列出总量
	-h 以 G/M 这种单位显示
	--max-depth=<目录层数>
```

### 3、列出当前系统打开的文件（打开的文件无法删除）

```shell
lsof #（list open files）
lsof | grep deleted // 查看已经被删除但进程没结束仍然占用空间的文件
```

### 4、排序

```shell
sort -r # 升序
head -n x # 显示前面的 x 个，默认 10个
tail -n x # 显示后面的 x 个，默认 10个
```

### 5、查找

```shell
find [path] -name 'myName' # 通过 name 查找，可以使用正则
	-mtime +3 # 距离当前时间修改时间超过3天的文件
	-mtime -3 # 距离当前时间修改3天以内的文件
	-mmin -3 # 距离当前时间修改3分钟以内的文件
	-exec # 执行 command 命令 -exec {} “{}”表示 find 查询到的结果，到“;”结束，需要使用 “\”转义分号 
```

### 6、移动或重命名

```shell
mv my1 my2 # 如果 my1、 my2 都是文件，则将 my1 重命名为 my2
					 # 如果 my1 是文件 my2 是目录，则将 my1 移动到 my2 中
					 # 如果都是目录，my2 存在则将 my1 移动到 my2 中，不存在则重命名 my1 为 my2

```

### 7、查看端口占用

查看哪个应用占用了端口

```shell
lsof -i:port # 如果有应用占用端口则返回信息，没有则不返回
```

查看端口占用的情况

```shell
netstat 
	-t # 显示 tcp 协议的
	-l # 显示监控中的服务器的 socket
	-n # 直接使用 ip 地址而不是域名
	-p # 显示正在使用的程序
```

配合 `grep pid`筛选进程 id（可以先通过`ps -ef`找到目标进程）

### 8、查看文件

```shell
head path # 查看头部
	-n 20 # 显示最前面的20行，也可直接写成 20
	-c 10 # 显示最前 10 个字节

tail path # 查看尾部
	-f # 循环读取
	-n 20 # 显示最后20行，也可直接写成 -20
	-c 10 # 显示最后 10 个字节
```

### 9、ssh 远程登录，sshpass 携带明文密码操作

```shell
ssh [OPTIONS] [-p PORT] [USER@]HOSTNAME [COMMAND] # 在 ～/.ssh 文件中可以新增 config 文件来指定哪个 HOST 使用哪个私钥访问，以此来配置多个私钥
 -i # 指定 RSA 或 DSA 认证需要的私钥文件

sshpass -p xxxx ssh root@remote_ip # 携带明文密码进行操作，不需要登录就可以远程操作，比如复制文件到远程（非linux自带，需要安装）
```

特别注意`-i`指定的是私钥，私钥自己保留。公钥发送给被连接的机器或容器，写入目标机器`/root/.ssh`（目录位置不一定）authorized_keys 中（文件不存在可以手动创建）。

之后要赋予文件权限`chmod 600 authorized_keys`

#### ssh 其他功能（连接检测、转发代理）

**连接检测**	

配置好公钥后，可以使用下面的命令判断是否成功连接

```shell
ssh -T xxxx.com 
	-vv # 表示以调试模式连接，会返回连接的链路信息
```

**转发代理**	

ssh 连接时**默认只会读取规范的 `id_rsa`私钥文件**。`ssh-agent`可以用来管理这种行为，让`ssh`知道连接哪个地址的时候使用哪个文件。

详情可以参照：https://www.cnblogs.com/f-ck-need-u/p/10484531.html

```shell
ssh-add [my_私钥] # 向 agent 中添加私钥
```

一般情况下`ssh`默认开启了`ssh-agent`功能。如果没开启，需要在sshd_config中开启`AllowAgentForwarding`选项，在ssh_config中开启`ForwardAgent`选项

如果使用`ssh-add`命令报错：**Could not open a connection to your authentication agent** 。是因为在当前的 shell 中`ssh-agent`未执行。可以使用

```shell
ssh-agent bash # 将 ssh-agent 作为当前 bash shell 的子进程执行
eval 'ssh-agent' # 直接作为主进程在后台运行，不推荐，因为即使 shell 退出了他也会在后台一直运行占用资源
```

#### sshpass 免交互 SSH 登录工具

使用这个工具我们可以直接在命令行中提供密码来远程登录主机执行命令。（**需要注意如果直接提供明文密码会在进程中被直接看到，需要确保主机的安全环境**）

需要单独安装

```shell
# 一般下面两个包管理工具有哪个就用哪个安装
yum -y install sshpass 
apt-get install sshpass

# 连接远程主机 ssh 登录并拉代码（也可以执行其他任意命令）
sshpass -p PWD ssh -i xxxxx git@gitlab.com 'git clone https://gitlab.com/mrlyk/hello-vue3'
	-p # 远程主机密码
	-f # 读取文件作为远程主机密码

# 从远程主机拷贝文件到本地
sshpass -p '123456' scp -r root@host_ip:/home/test/ ./tmp/
```

### <font color="red">10、tar 解压</font>/压缩

```sh
tar -xvf [file]
	# -x 从压缩文件中还原文件（解压）
	# -f from 指定文件，从哪里解压或压缩
	# -v 显示指令执行过程
	# -z 通过 gzip 指令处理压缩文件
	# -c create 一个压缩文件（压缩）
	# -C 指定目录 或者 --directory (加在命令最后)
```

### 11、source 和 ./ 的区别

```shell
soruce # 在当前的 shell 运行，会插入环境变量到当前 shell
./ #fork 子进程启动，不会插入环境变量
```

### <font color="red">12、chmod 权限</font> 

**权重：**

1 - 可执行
2 - 只可写
4 - 只可读

**三类用户**

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211103115443172.png" alt="image-20211103115443172" style="zoom:33%;" align="left"/>

```shell
chmod 400 [file] # 给所有者分配可写权限，组和其他用户不分配任何权限
	-R # 递归处理子目录
```

### 13、查看防火墙状况

防火墙1: iptables

```shell
iptables # 这是其中一种也是最常见的一种防火墙
 -L # 列出防火墙规则
 -n # 表示不对 IP 地址进行反查，加上这个参数显示速度将会加快。
 -v # 表示输出详细信息，包含通过该规则的数据包数量、总字节数以及相应的网络接口。
```

防火墙2: firewalld

```shell
systemctl status firewalld # 查看防火墙状态
systemctl start firewalld # 开启防火墙
systemctl stop firewalld # 关闭防火墙
firewall-cmd --query-port=80/tcp # 查询80端口是否开放
firewall-cmd --zone=public --add-port=80/tcp --permanent # 开放80端口
firewall-cmd --remove-port=80/tcp --permanent # 关闭 80 端口
firewall-cmd --reload # 重新载入
firewall-cmd --list-port # 查看已经开放的端口
firewall-cmd --get-zones # 获取可以设置的 zone 区域 drop 表示不接受网络请求 
```

> ps：腾讯云 centos 7 轻量服务器除了要设置开放防火墙（iptables）还需要手动开放 firewalld 防火墙（辣鸡，一点提示没有）

### <font style="color:red;">14、Systemd 指令</font> 

系统中重要的指令集合，用来守护系统进程

- CentOS6：通过 UpStart 处理，分组启动，组之间并行，组内顺序启动。配置文件是`/etc/inittab和/etc/init/*.conf`
- CentOS7：通过 systemmd 处理，由于多种系统都是基于 linux 内核的，比如安卓设备，启动频繁。即使上面的分组依然不能满足需要。

**单元分类**

- service：代表一个后台服务进程，比如 mysqld.service
- socket：并不自己启动守护进程，建立通道，监听 ip 地址和端口
- target：可以视作一批服务的集合，具体看里面的定义

需要定义单元的具体服务文件，才能用来处理具体任务，如设置开机启动等

**常用命令**

```shell
systemctl # Systemd 的主命令
systemctl reboot # 重启
systemctl list-units # 列出所有运行中的单元
systemctl list-unit-files # 列出所有可用单元
systemctl list-unit-files | grep enable # 查看自启动的软件

systemctl is-enabled mysqld.service # 查看某个单元是否开机启动，这里是查看 mysql
systemctl status mysqld.service # 查看某个单元的状态
systemctl start mysqld.service # 启动某个单元
systemctl stop mysqld.service # 停止某个单元

system restart mysqld.service # 重启某个单元
system reload mysqld.service # 重载某个单元，只重新加载配置，不重启。并不是所有服务都可以
system kill mysqld.service # 终止某个单元，用于 stop 无响应的时候

## 设置开机启动
systemctl enable mysqld.service
systemctel disable mysqld.service # 关闭开机启动
```

**例子：设置 nginx 开机启动**

如果没有 service 文件，可以自己创建，这里以 nginx 为例子

1. 在系统服务目录`/usr/lib/systemd/system/`创建 nginx.service 文件
2. 写入如下内容

```service
[Unit] # 服务说明
Description=nginx # 描述服务，在调用 systemctl 命令时会出现该描述
After=network.target # 声明服务应该在什么之后启动，因为 nginx 肯定需要 network 模块，所以肯定要在他之后
  
[Service]
Type=forking # 服务类型，不提供时默认 simple，具体见下方服务类型
ExecStart=/usr/local/nginx/sbin/nginx # 具体运行命令
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true # 表示是否给服务分配独立的临时空间
  
[Install] # 告诉系统用户范围，如果没有用户也不会被系统装载。除了必填的 WantedBy 之外还可以使用 Alias 起别名
WantedBy=multi-user.target # 这里可以视为多用户的意思，但实际上指的是服务的访问方式，比如我访问远程服务器，一般使用 shell 通过 ssh 直接登录。那我这就是一种经典的访问方式，被包含在其中。
```

3. 重新读取所有服务文件

   `systemctl deamon-reload`

4. 设置开机启动

   `systemctl enable nginx.service`

**Service Type - 服务类型**

- simple - 会被系统认为是主进程
- forking -会开启一个子进程，之后会变成主进程。父进程会在子进程启动完成后关闭。
- oneshot - type 和 ExecStart 都未定义时默认使用。和 simple 很像，区别是会在其他单元启动前完成。
- dbus - 使用该 Type 要求 BusName 选项必填，其他和 simple 相同。
- notify - 和 simple 很像，但是会通过 sd_notify 函数发送一个通知，发送完成后才启动。

一般来说，前台程序用 simple 就行了，后台就 forking

### 15、通过域名查询主机 ip 信息 nslookup

```shell
nslookup www.baidu.com
```

### <font color="red">16、编译与安装</font>

（1）gcc 编译环境安装

`yum -y install make zlib lib-devel gcc-c++ lib tool openssl openssl-devel`

- zlib - 解压库，提供多种解压方式
- lib-devel - 链库，用于编译
- openssl - 密钥，devel 同上

（2）安装

1. `./configure` - configure是一个脚本，一般由 Autoconf 工具生成，它会检验当前的系统环境，看是否满足安装软件所必需的条件：比如当前系统是否支持待安装软件，是否已经安装软件依赖等。configure脚本最后会生成一个Makefile文件。**（可配置）**
2. `make` - 从 Makefile 中读取指令，然后编译
3. `make install` - 从 Makefile 中读取指令，安装到指定位置

### 17、远程文件上传与下载 scp 与 rsync

**scp**

```shell
scp [options] [源文件] [目标地址]
	-i # 使用 ssh 证书登录（默认使用密码）
	-1 # 强制使用 ssh1 协议
	-4 # 强制使用 IPV4 寻址
	-C # 允许压缩
	-p # 保留原文件的修改时间，访问时间和访问权限
	-q # 不显示进度条
	-r # 递归复制整个目录
	-v # 显示详细信息
	-P # 指定数据传输用到的端口号
```

本地复制到远程：`scp local_file root@remote_ip:remote_foleder`

远程复制到本地就是顺序换过来

**rsync** 

与 scp 类似，但是更强大和灵活一些，可以方便的排除某些文件

```shell
rsync [options] [源文件] [目标地址]
	-r # 递归复制整个目录
	-l # 保留软连接
	-p # 保留文件权限
	-t # 保留时间信息
	-a # 相当于开了上面所有选项，还多一些额外的，详细不表
	--progress # 显示进度
	--exclud=xxx # 排除文件（夹），支持通配符。注意不能跟路径，只能是文件或文件夹名
```



### 18、切换用户

```shell
su [options] [user_name]
	-f # 不读启动档
	-c # 执行完指令后推出，如 su -c ls root 切换为 root 用户后执行 ls 命令然后切换回原用户
```

### 19、查看系统架构

`arch` 返回当前系统架构

### 20、root 查看其他账户密码

`passed [user_name]` 如果用户无密码则会让你设置

### 21、wget 下载文件

```sh
wget [options] [url]
	-O # 控制下载位置和名称。 -O [myPath/fileName]
	--http-user=[USERNAME] --http-password=[PASSWORD] # 传递用户名密码
	--no-check-certificate # 接受自签名的证书
	-r # 递归下载相关文件 -r -l 3 允许三层以内深度的连接（0的话就是不限制）
	-b # 后台下载
	-c # 支持断点续传下载 如果中断了下载，可以添加这个参数来继续上一次的下载
```

默认下载到当前目录

与之类似的还是有 **curl**命令

```shell
curl [options] [url]
	-b # 发送cookie
	-H # 请求头
	-X # 请求方法
#curl -H "Content-Type: application/json" -X POST -d '{"user_id": "123", "coin":100}' # 发送一个携带参数的 post 请求
```

curl 功能更强大，wget 更轻便。curl 可以看作一个不会渲染内容的服务端浏览器，可以做很多事。如果只是下载内容，使用 wget 即可。

### 22、chown 更改文件所有者

chown（英文全拼：**change owner**）命令用于设置文件所有者和文件关联组的命令。需要超级用户 **root** 的权限才能执行此命令。

```shell
chown [options] [user]:[group] [path] # user : 新的文件拥有者的使用者 ID group : 新的文件拥有者的使用者组(group)
	-R # 对所有子目录递归执行
	-f # 忽略错误信息
	-v # 现实详细信息
```



### 23、ln 创建软、硬连接

软连接类似于快捷方式，可以跨文件系统；硬连接相当于创建副本，但是不占用实际空间，只有所有硬连接关联的文件删除，源文件才会被真正删除。

```shell
ln -s [source_file] [soft_link_name]
```

### 24、后台运行软件 & 符

Linux 终端命令的末尾加上一个 & 符号表示将这个任务放到后台去执行，例如以下命令表示将 ssh 服务放到后台运行

```shell
 /usr/sbin/sshd -D &
```

### 25、shell 输入

```shell
cat a.text > b.text # 将文件 a 输入文件 b，覆盖
cat a.text >> b.text # 将文件 a 输入文件 b，追加
```

### 26、eval 执行计算（命令）

和 js 中的 eval 有点像，都可以用来解释和执行其中的命令

```shell
eval `ssh-agent -s` # 启动 ssh-agent
eval $(ssh-agent -s) # 作用同上，区别是 `` 反引号会自动转译 一次 字符中的'\'号，推荐使用这个
```

### 27、查找历史执行过的命令 ctrl+r

使用`ctrl+r`快捷键可以输入关键字，查找历史执行过的命令。找到后再按右箭头就可以获取到了。非常方便

### 28、查看系统运行状态 top

展示 cpu、内存等运行信息，查看进程对系统资源对占用

```shell
top
```

### 29、sed 利用脚本处理文本文件

文档：https://www.runoob.com/linux/linux-comm-sed.html

```shell
 # 利用 sed 将 regular_express.txt 内每一行结尾若为 . 则换成 !
 sed -i 's/\.$/\!/g' regular_express.txt
 	-e # 指定处理脚本
 	-i # 插入
 	-d # 删除

nl /etc/passwd | sed -e '2d'  # 删除第二行（nl 和 cat 类似，但是会输出行号）
```

### 30、nano 文本编辑器

比 vim 更简单一点的文本编辑器

```shell
nano test.txt
	-w # 更正确的侦测单词边界，nano 在编辑本过长时会自动换行。有时会有问题，需要加上该选项
	-B # 储存当前文件的备份
```

编辑操作：

- 复制一整行：Alt+6
- 剪贴一整行：Ctrl+K
- **粘贴：Ctrl+U** 
- 搜索: Ctrl+W

## 二、系统目录结构

- **/boot：**存放的启动Linux 时使用的内核文件，包括连接文件以及镜像文件。

- **/etc：**存放**所有**的系统需要的**配置文件**和**子目录列表，**更改目录下的文件可能会导致系统不能启动。

- **/lib**：存放基本代码库（比如c++库），其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。

- **/sys**： 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中

**指令集合：**

- **/bin：**存放着最常用的程序和指令

- **/sbin：**只有系统管理员能使用的程序和指令。

**外部文件管理：**

- **/dev ：**Device(设备)的缩写, 存放的是Linux的外部设备。**注意：**在Linux中访问设备和访问文件的方式是相同的。

- **/media**：类windows的**其他设备，**例如U盘、光驱等等，识别后linux会把设备放到这个目录下。

- **/mnt**：临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。

**临时文件：**

- **/run**：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

- **/lost+found**：一般情况下为空的，系统非法关机后，这里就存放一些文件。

- **/tmp**：这个目录是用来存放一些临时文件的。

**账户：**

- **/root**：系统管理员的用户主目录。

- **/home**：用户的主目录，以用户的账号命名的。相当于 windows 的我的文档 

- **/usr**：用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。

- **/usr/bin：**系统用户使用的应用程序与指令。(bin 是 binary 的简称，二进制可执行文件)

- **/usr/sbin：**超级用户使用的比较高级的管理程序和系统守护程序。

- **/usr/src：**内核源代码默认的放置目录。

**运行过程中要用：**

- **/var**：存放经常修改的数据，比如程序运行的日志文件（/var/log 目录下）。

- **/proc**：管理**内存空间！**虚拟的目录，是系统内存的映射，我们可以直接访问这个目录来，获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件来做修改。

**扩展用的：**

- **/opt**：默认是空的，我们安装额外软件可以放在这个里面。

- **/srv**：存放服务启动后需要提取的数据**（不用服务器就是空）**

#### /dev/null

在类Unix系统中，`/dev/null` - 空设备，是一个特殊的设备文件，丢弃一切写入其中的数据（但报告写入操作成功），读取它则会立即得到一个EOF（end of file）。
当我们执行命令但不想输出结果给用户的时候，可以把结果写入该处。

#### 0、1、2 linux 预留文件描述符

- 0 —— stdin（标准输入）
- 1 —— stdout （标准输出）
- 2 —— stderr （标准错误）

比如我使用如下命令`ls`，输出了两个文件`a.txt、b.txt`，那么这个输出的文本可以用 `1`来表示

```shell
ls 
# a.txt b.txt
```

既然可以用 1 来表示，那就可以这么做

```shell
ls 1> b.txt
```

使用 1 指代标准输出的结果，然后使用`>`（标准重定向指令）将内容重定向到`b.txt`中。

0 和 2 也同理，**使用 2 还可以判断程序是否启动了**

## 三、环境变量配置

环境变成存放目录 `/etc/profile` 全局环境变量

推荐使用脚本来为程序控制环境变量，在`/etc/profile.d`文件中创建 sh 脚本

环境变量之间用冒号 : 分隔，和 windows 上用分号 ; 一样

```tex
PATH=$NODE:$PATH # 这样就可以把 NODE 这个变量路径加入系统的 PATH
```

最终实际是把目标程序的父目录放到了全局，输入实际的程序名即可全局运行

## 四、yum 包管理工具

### yum常用命令

1. 列出所有可更新的软件清单命令：**yum check-update**

2. 更新所有软件命令：**yum update**

3. 仅安装指定的软件命令：**yum install <package_name>**

4. 仅更新指定的软件命令：**yum update <package_name>**

5. 列出所有可安裝的软件清单命令：**yum list**

6. 删除软件包命令：**yum remove <package_name>**

7. 查找软件包命令：**yum search <keyword>**

8. 清除缓存命令:

  - **yum clean packages**: 清除缓存目录下的软件包
  - **yum clean headers**: 清除缓存目录下的 headers
  - **yum clean oldheaders**: 清除缓存目录下旧的 headers
  - **yum clean, yum clean all (= yum clean packages; yum clean oldheaders)** :清除缓存目录下的软件包及旧的 headers

### 配置仓库源

```sh
yum repolist all # 显示所有资源库
yum repolist enabled # 显示所有已启动的资源库
yum repolist disabled # 显示所有被禁用的资源库
```

#### 添加资源库

想要添加源除了下面直接安装并更新的方式，也可以使用 `yum-utils`工具，他集成了许多 yum 相关工具

```sh
yum install yum-utils # 安装 yum-utils

yum-config-manager --add-repo repository_url # 安装后可以使用其中集成的 yum-config-manager 来添加
yum-config-manager --disable rep_name # 禁用某个源
yum-config-manager --enable rep_name # 启用某个源
```

删除仓库的话需要去配置文件中`/etc/yum.repos.d/`手动删除配置项

#### 配置 EPEL源

```
sudo yum install -y epel-release
sudo yum -y update
```
