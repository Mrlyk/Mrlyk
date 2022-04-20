# ssl证书

## 一、申请免费的证书

在服务器供应商处都提供免费的证书申请入口

腾讯云 ssl 控制台：https://console.cloud.tencent.com/ssl

以我的腾讯云服务为例，在 ssl 控制台提供免费证书申请入口。免费证书仅支持单域名，泛域名证书需要付费。

## 二、创建自签名的 ssl 证书

第一次访问时需要将证书倒入系统，并添加信任。

待续...

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
  ssl_prefer_server_ciphers on; # 有限使用服务端设置的加密套件而不是客户端
  return 301 https://$host$request_uri; # 用于对请求的客户端直接返回响应状态码。在该作用域内return后面的所有nginx配置都是无效的。将http的域名请求转成https
}
```

