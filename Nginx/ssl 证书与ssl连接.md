# ssl证书

[toc]

## 一、申请免费的证书

在服务器供应商处都提供免费的证书申请入口

腾讯云 ssl 控制台：https://console.cloud.tencent.com/ssl

以我的腾讯云服务为例，在 ssl 控制台提供免费证书申请入口。免费证书仅支持单域名，泛域名证书需要付费。

## 二、创建自签名的 ssl 证书

第一次访问时需要将证书倒入系统，并添加信任。

一般我们使用`openssl`工具生成自签名的证书，**所以我们首先需要安装`openssl`。**一般使用系统自带的包管理工具就可以安装，比如`yum`、`apt-get`、`brew` 

```text
OpenSSL 是一个开源的软件库包，可以用来进行安全通信，避免窃听，同时确认另一端连接者的身份²。OpenSSL 包括主要的密码算法、密钥和证书管理功能及 SSL/TLS 协议，并提供了一个多用途的命令行工具³。

OpenSSL 的常见用法有：

- 生成和管理公钥、私钥和相关参数
- 使用公钥和私钥进行加密、解密、签名、验证
- 管理证书（X.509 格式）认证请求和证书吊销列表
- 计算摘要（hash），支持各种摘要算法
- 生成用户密码

举例说明：

- 生成 RSA 密钥对：`openssl genrsa -out key.pem 2048`
- 从私钥中提取公钥：`openssl rsa -in key.pem -pubout -out pubkey.pem`
- 公钥加密文件：`openssl rsautl -encrypt -in input.file -inkey pubkey.pem -pubin -out output.file`
- 私钥解密文件：`openssl rsautl -decrypt -in input.file -inkey key.pem -out output.file`
- 生成证书请求：`openssl req -new -key key.pem -out req.csr`
- 生成自签名证书：`openssl x509 -req -in req.csr -signkey key.pem -out cert.pem`
- 计算文件的 MD5 摘要：`openssl dgst -md5 input.file`
- 生成用户密码：`openssl passwd -crypt -salt xx password`
```

#### 具体步骤

1. 生成私钥用于解密

`openssl genrsa -out my-key.pem 2048`

2. 生成一个新的 csr 文件

`openssl req -new -key my-key.pem -out my-csr.pem`

需要自己输入国家、省份和组织等政府信息，用于申请数字证书。不是所有的都是必填。比如 challenge pwd 是为了证书更安全才要配置的，可以不设置。

*ps：CSR（Certificate Signing Request）文件是用于向数字证书认证机构（CA）申请数字证书的请求文件。*

3. 基于 csr 文件生产一个自签名证书

`openssl x509 -req -days 365 -in my-csr.pem -signkey my-key.pem -out my-cert.pem`

- `x509`: 该命令用于操作 X.509 格式的数字证书和证书请求文件。
- `-req`: 指定要处理的文件是证书请求文件，而不是数字证书文件。
- `-days 365`: 指定证书有效期为 365 天。默认是 30 天到期。
- `-in my-csr.pem`: 指定要签名的证书请求文件路径。
- `-signkey my-key.pem`: 指定要用于签名的私钥文件路径。
- `-out my-cert.pem`: 指定生成的数字证书文件路径。

4. 验证证书

`openssl x509 -in my-cert.pem -text -noout`

这将显示证书的详细信息，包括颁发机构、有效期、公共密钥等。

## 三、证书文件

- xxx.crt：证书文件，https 请求时服务端返回该证书（公钥），后面的 pem 文件是该证书的其他格式。不同的应用服务器比如 ng、apache 使用的证书格式不同。客户端使用该公钥对自己生成的随机密钥加密
- xxx.key：私钥文件，服务器使用该文件对浏览加密的信息进行解密
- xxx.csr：Certificate Signing Request，申请证书的请求文件（给颁证机构）
- xxx.pem：Privacy Enhanced Mail，pem 格式的证书（文本文件格式，nginx 使用的格式）

#### 对称加密与非对称加密

- 对称加密：就像我们常用的密码的形式，加密和解密都使用同一个密码，两边的信息是对称的，你没有上次相同的密码就不能登录。优点是速度快，缺点是安全性相对没有非对称加密好。常用的对称加密算法有`AES、RC4`
- 非对称加密：需要 公钥和私钥，像我们常用`ssh-keygen`默认使用的就是 RSA 这种非对称加密形式。公钥可以被随意公开，私钥只能自己保留。加密方使用公钥对信息进行加密，我们只有使用私钥才能解开，保证了安全性。

#### ssl 加密

ssl 加密是一种包含对称和非对称两种的加密方式，生成会话密钥之前的操作基于非对称加密。生成会话密钥之后的信息交换属于对称加密。

时序图如下

```sequence
客户端->服务端:发送请求：随机数1、协商的加密算法
服务端-->>客户端:协商好的加密算法、随机数2
服务端-->>客户端:crt 证书（不同应用服务器，证书格式可能不同）
客户端->客户端:crt 证书校验，浏览器内置根证书目录
客户端->客户端:1、随机数1、2以及预主密钥组装会话密钥；2、通过证书（公钥）加密会话密钥
客户端->服务端:发送会话密钥
服务端->服务端:通过证书私钥key解密，随机数1、2组装获得会话密钥
客户端->服务端:发送会话密钥加密过的信息
服务端->客户端:发送会话密钥加密过的信息
Note right of 客户端:如果两边都能正常接收会话密钥加密过的信息，那么 ssl 连接成功建立

```

## Nginx 配置 ssl 证书





```nginx
server {
  listen		443 ssl;
	server_name kcode.tech; # 证书绑定的域名
  ssl_certificate /usr/local/nginx/ssl/xxx.pem; # 证书文件，pem 格式
  ssl_certificate_key /usr/local/nginx/ssl/xxx.key; # 证书私钥
  ssl_session_timeout 5m; # 证书可重用时间（不再次验证）5分钟
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!3DES; # 证书加密套件
  ssl_protocols TLSv1.1 TLSv1.2; # 支持的 ssl 协议
  ssl_prefer_server_ciphers on; # 优先使用服务端设置的加密套件而不是客户端
  return 301 https://$host$request_uri; # 用于对请求的客户端直接返回响应状态码。在该作用域内return后面的所有nginx配置都是无效的。将http的域名请求转成https
}
```

