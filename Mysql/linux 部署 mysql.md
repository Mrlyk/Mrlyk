## linux 部署 mysql

#### 1、下载Mysql

8.02社区版本下载地址：https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz

#### 2、解压

#### 3、修改配置文件

目录：`/etc/my.cnf`

```cnf
[mysqld]
bind-address=0.0.0.0 # 允许访问的 ip，允许所有 ip 访问
port=3306 # 端口
log-error=/var/log/mariadb/mariadb.log # 错误日志
```

#### 4、初始化数据库

先确认 numactl 软件包是否安装，没安装使用 yum 安装

`yum -y install numactl`

**初始化命令** 

`mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/mysql/ --datadir=/data/mysql/ --initialize`

初始化会生成初始化密码

<img src="https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20211210193343404.png" alt="image-20211210193343404" style="zoom:50%;" align="left"/>

**设置开机启动**

mysql.service 是 mysql 的服务程序

从服务目录`/mysql/support-files`**找到 mysql.service**，放到 /etc/init.d/ 目录中

**启动 mysql**

`service mysql start` 或者 `systemctl start mysql`

如果报错` ERROR! The server quit without updating PID file (/data/mysql//VM-16-9-centos.pid).`一般是权限问题。

查看报错日志，如果是权限导致目录不能创建则手动创建，并将目录所有者换成 mysql:mysql（初始化时自动生成的用户）
`chown -R mysql:mysql [path]`