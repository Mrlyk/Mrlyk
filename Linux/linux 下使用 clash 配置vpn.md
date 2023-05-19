# linux 下使用 clash 配置 VPN

[toc]

## 操作步骤

### 1、下载符合系统要求的 clash 客户端

下载地址：https://github.com/Dreamacro/clash/releases

下载完成后解压即为可执行的二进制文件，记得赋予执行权限

```shell
chmod +x clash
```

### 2、新增配置文件

一般飞机场会提供配置文件

- 执行 `wget -O config.yaml https://xxxxxxx` 下载 Clash 配置文件
- 执行 `wget -O Country.mmdb https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb` 下载 GeoIP 数据库

### 3、启动

启动可以通过`systemctl`工具来配置后台启动，也可以直接运行二进制文件。

**直接运行**

```shell
./clash -d . # 在当前文件下执行，会读取当前文件夹下的 config.yml 作为配置文件
```

**配置 service **

```service
[Unit]
Description=Clash Daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/clash -d /etc/clash
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

### 4、修改配置

可以通过图形化页面来修改配置，仓库地址：https://github.com/Dreamacro/clash-dashboard

我们需要下载该仓库手动构建文件。之后**重点来了**

**我们需要在 clash 的 config.yaml 中新增两行声明**

```yaml
external-controller: '0.0.0.0:9090'
external-ui: '/usr/local/clash/clash-dashboard/dist'
```

**第一个标明配置启动的端口，第二个标明使用的 UI。缺一不可。**

## 踩坑