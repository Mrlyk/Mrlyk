# Nginx

[toc]

## 一、本地服务

本机使用 brew 安装

安装路径：/usr/local/var/www
默认配置地址：/usr/local/etc/nginx/nginx.conf to 8080 
服务文件地址：/usr/local/etc/nginx/servers/

**启动服务**

```
后台： brew services start nginx
前台： nginx
```



## 二、常用命令

```shell
nginx -s stop # 停止
nginx -s reload # 重启
nginx -c xxx.comf # 指定配置文件
```



## 三、配置结构

### 3.1 全局

```nginx
worker_processes: 2; # 进程数，默认1。一般不超过系统 cpu 线程数量
```

### 3.2 events 配置服务器与用户的网络连接

```nginx
evnets {
	accept_mutex on; # 设置网路序列化，防止[惊群现象]，详细见下面的说明，默认 on
	multi_accept on; # 设置一个进程是否允许多个网络连接，默认 off，可以用来优化连接
	worker_connections  1024;    #最大连接数，默认为512
}
```

**events 一些说明**：

- **accept_mutex **的意义：当一个新连接到达时，如果激活了accept_mutex，那么多个Worker将以串行方式来处理，其中有一个Worker会被唤醒，其他的Worker继续保持休眠状态；如果没有激活accept_mutex，那么所有的Worker都会被唤醒，不过只有一个Worker能获取新连接，其它的Worker会重新进入休眠状态，这就是「[惊群问题](http://en.wikipedia.org/wiki/Thundering_herd_problem)」。

### 3.3 http 嵌套多个 server ，配置代理、缓存、日志、请求数等

```nginx
http {
	include		mime.types; # 文件扩展名与文件类型映射表，一般用默认标准的即可
	default_type		application/octet-stream; # 默认文件类型，默认为 text/plain
	#access_log		off; # 取消服务日志
	log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; # 自定义日志格式，变量即 nginx 内含的变量
  access_log log/access.log myFormat; # 以自定义的格式存储日志，可以在http，server，location块设置
	sendfile on; # 允许 sendfile 方式传输文件，默认 off
  sendfile_max_chunk 100k;  # 每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限
  keepalive_timeout 65;  # 连接超时时间，默认为75s，可以在http，server，location块设
  error_page 404 https://www.baidu.com; # 错误页，ng 通过 location（路由）访问时如果返回这里定义的代码（404）则会跳转到百度
  
  upstream myServer { # 负载均衡，需要在 location 配置中将地址指向 http://myServer
    server myServer.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s; # 失败3次后，在30秒内不会再被分配请求
    server unix:/tmp/backend3;
	}
}
```

**http 一些说明**

- **application/octet-stream**：字节流文件类型，意味着未知的应用程序
- **upstream backend**: **配置负载均衡**，nginx按加权**轮转**的方式将请求分发到各服务器。 在上面的例子中，每7个请求会通过以下方式分发： 5个请求分到`backend1.example.com`， 一个请求分到第二个服务器，一个请求分到第三个服务器。 与服务器通信的时候，如果出现错误，请求会被传给下一个服务器，直到所有可用的服务器都被尝试过。 如果所有服务器都返回失败，客户端将会得到最后通信的那个服务器的（失败）响应结果。

### 3.4 server 配置 ng 虚拟主机等相关参数

```nginx
http {
  server {
    keepalive_requests 120; # 单连接请求上限次数，默认100，在高并发的场景下可以提高
    listen 6666; # 监听端口
    server_name 127.0.0.1; # 监听地址
    root /html; # 静态地址的根目录
  }
}
```

### 3.5 location 路由及各种页面的处理情况

```nginx
http {
  server {
    location /api { # 请求匹配的地址
      #root path; # 静态地址的根目录，注意和项目打包产生静态文件的地址配合好，未设置时回去 sever 中查找
      #index index.html; # 默认页面
      proxy_pass http://myServer; # 请求代理，注意匹配规则：不带URI（没有具体路径，示例就是不带 URI 的）的方式只替换主机名，带URI的方式替换整个URL
      deny 101.xxx.xx.xx; # 拒绝的 ip
      allow 102.xxx.xx.xx; # 允许的 ip
      try_files $uri $uri/ /404.html # 未匹配到第一个路由地址时，顺序匹配后面的地址
    }
  }
}
```

**location 的匹配规则**

- `~`：波浪线，执行正则匹配并**区分大小写**，比如`$request ~ ^https://.*`匹配任何`https`开头的地址
- `~*`：波浪线星号，执行正则匹配并**不区分大小写**
- `^~`：前缀正则匹配
- `!~`：表示不匹配
- `=`：普通字符的**精确匹配**
- `/xxx`：普通匹配，匹配到这种后会继续向后匹配，**相当于`^~ /xxx`**匹配以 /xxx 开头的地址

**匹配优先级**：`=` > `^~` > `~`和`~*` location 中的匹配优先级遵循以上原则，**和配置文件中 的声明顺序无关**。

这里的正则匹配也支持变量存储，比如`$1`存储的是第一个子括号匹配到的值

以上匹配规则还可用于条件判断语句中，如`if（$uri ~* test）` 表示只要请求地址中含有 test 就匹配

### 3.6 几个常见配置变量：

- $remote_addr 与 $http_x_forwarded_for 用以记录客户端的ip地址；
- $remote_user ：用来记录客户端用户名称；
- $remote_port：客户端访问端口
- $time_local ： 用来记录访问时间与时区；
- $request ： 用来记录请求的url与http协议；
- **$request_uri**：用来记录用户访问路径的 pathname，比如 https://example.com/a/b?q=1 就是 /a/b?q=1 这一部分
- $uri：用户访问的地址，比如访问 https://example.com/test.html?q=1, $uri 就是 /test.html
- **$arg_PARAM**：获取请求的参数，比如访问 https://example.com/test.html?q=1, $arg_q 就是 1
- $status ： 用来记录请求状态；成功是200；
- $body_bytes_s ent ：记录发送给客户端文件主体内容大小；
- $http_referer ：用来记录从那个页面链接访问过来的；`$http_`前缀后面可以获取请求头中的内容
- $http_user_agent ：记录客户端浏览器的相关信息；
- **$http_HEADER**：HTTP 请求头中的内容，可以拿到自定义的请求头的数据
- $server_addr：到达服务的地址
- $schema：请求的协议，http 还是 https，**可以根据这个判断用来重写请求头，永久重定向到 https 地址 **

### 3.7 常用操作符

**变量声明**

配置中变量使用`$`符号开头

- `set`：配置中使用 set 声明变量 `set $my_var 1`声明变量`$myvar = 1`

**条件语句**

- `if`：和其他语言一样的条件语句

```nginx
set $target_service = '1.1.1.1';

if ($http_HEADER.gray) $target_service = '111.111.111.111' # 访问到灰度服务器

if ( $scheme = "http" ) { // 如果是好 http 协议访问，重定向到 https 
  rewrite ^/(.*)$ https://$host/$1 permanent;
}

http {
  server {
    location /api { # 请求匹配的地址
      proxy_pass $target_service/api
        try_files $uri $uri/ /404.html # 未匹配到第一个路由地址时，顺序匹配后面的地址
    }
  }
}
```

**map 指令**	

场景： 匹配请求 url 的参数，如果参数是 debug 则设置 $foo = 1 ，默认设置 $foo = 0

$args 是内置变量

```nginx
map $args $foo {
  default 0; # 默认值
  debug   1;
}
```

## 四、其他常用配置

**开启gzip压缩** 

```nginx
http{
	gzip on;  #是否开启gzip模块 on表示开启 off表示关闭
	gzip_buffers 4 16k;  #设置压缩所需要的缓冲区大小
	gzip_comp_level 6;  #压缩级别1-9，数字越大压缩的越好，也越占用CPU时间
	gzip_min_length 100k;  #设置允许压缩的最小字节
	gzip_http_version 1.1;  #设置压缩http协议的版本,默认是1.1
	gzip_types text/plain text/css application/json application/x-javascript text/xml 			application/xml application/xml+rss text/javascript;  #设置压缩的文件类型
	gzip_vary on;  #加上http头信息Vary: Accept-Encoding给后端代理服务器识别是否启用 gzip 压缩
}
```

**开启反向代理**

```nginx
server {
        listen       5899; # 监听端口
        server_name  _;
        #charset koi8-r;

        access_log  logs/access.log  main;

        location / {
             proxy_pass https://xxxx.com; # 代理到的地址
             proxy_set_header Host "doctorstation.95169000.com"; # 设置请求头
             proxy_http_version 1.1;
        }
      }
```

## 五、安装 nginx 模块

ng 默认不会安装任何模块，但是当我们想开启 ssl 时，则需要依赖于`nix_http_ssl_module`模块

```sh
nginx -V
# nginx version: nginx/1.21.4
# built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
# configure arguments:   # 后面没东西表示没安装任何模块
```

#### 安装模块

在 nginx 解压后的目录中，执行 configure 并携带参数，以生成携带 ssl 模块的 nginx 可编译文件

```sh
./configure --with-http_ssl_module
```

然后执行编译即可

```sh
make 
make install 
```

默认会直接在已经安装的 nginx 上直接附加上该模块，可以通过`nginx -V`查看模块是否安装成功

## 其他注意事项

#### proxy_pass 的匹配规则

```nginx
server {
  listen       80;
  server_name  localhost;

  location /api1/ {
    proxy_pass http://localhost:8080;
  }
  # http://localhost/api1/xxx -> http://localhost:8080/api1/xxx

  location /api2/ {
    proxy_pass http://localhost:8080/;
  }
  # http://localhost/api2/xxx -> http://localhost:8080/xxx

  location /api3 {
    proxy_pass http://localhost:8080;
  }
  # http://localhost/api3/xxx -> http://localhost:8080/api3/xxx

  location /api4 {
    proxy_pass http://localhost:8080/;
  }
  # http://localhost/api4/xxx -> http://localhost:8080//xxx，请注意这里的双斜线，好好分析一下。

  location /api5/ {
    proxy_pass http://localhost:8080/haha;
  }
  # http://localhost/api5/xxx -> http://localhost:8080/hahaxxx，请注意这里的haha和xxx之间没有斜杠，分析一下原因。
  
  location /api6/ {
    proxy_pass http://localhost:8080/haha/;
  }
  # http://localhost/api6/xxx -> http://localhost:8080/haha/xxx

  location /api7 {
    proxy_pass http://localhost:8080/haha;
  }
  # http://localhost/api7/xxx -> http://localhost:8080/haha/xxx

  location /api8 {
    proxy_pass http://localhost:8080/haha/;
  }
  # http://localhost/api8/xxx -> http://localhost:8080/haha//xxx，请注意这里的双斜杠。
}
```

`proxy_pass`根据是否是 URI 来决定整个替换还是选择加在前面，**只要不是纯净的协议+主机+端口，后面带有路径的一律都是 URI** 

由上面的规律可以看出，要么`location`和`proxy_pass`都带`\`，要么都别带。不然路径代理容易出问题。
