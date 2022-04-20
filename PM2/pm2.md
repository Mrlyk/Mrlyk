# pm2 进程守护

## 一、常用命令

```shell
pm2 start [app] # 启动项目
pm2 stop [id] # 停止项目
pm2 lsit # 查看当前运行的项目
pm2 save # 保存当前已启动的服务
pm2 startup # 设置开机启动（需要先保存当前已启动的服务）
pm2 start npm -- start # 运行 npm 的 scripts 命令
```



