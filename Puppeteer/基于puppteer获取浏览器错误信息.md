做法

1、在客户端通过 websocket 暴露给我的 node 服务。

2、node 服务收到请求后，获取信息

用户可以主动上报我们为监控到的异常情况，比如我的更改只是数据获取的方式有误，并没有产生代码或预发错误。但是产生了与预期不相符合的情况。这时候可以告知用户主动上报错误。

优点

1、无需远程

2、排查意料之外的错误

3、获取更详细的信息

4、支持本地化部署，只需配置一个代理即可

```sequence
Web端用户程序->>服务端: 上传browserWSEndpoint
服务端->数据库: 临时存储信息
数据库-->服务端: 存储成功
服务端-->Web端用户程序: 返回注册成功消息
note right of 服务端: 主动发起连接
服务端->数据库: 获取Web端信息
数据库-->服务端: 返回browserWSEndpoint
服务端->>Web端用户程序: 连接到Web端
Web端用户程序->服务端: 实时上传信息
```

