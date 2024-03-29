# vs code 远程开发

vscode 提供了三个官方插件，可以直连我们的云端服务器进行开发。和 xshell 之类的工具有异曲同工之妙，还兼具了 vscode 的开发能力。

## 相关插件

#### Remore Development

远程开发工具。通过 ssh 连接到云端服务器之后，可以使用普通目录的访问方式打开文件和上传文件。

#### Remote SSH

通过 ssh 连接到云端服务器需要的插件。上面的开发工具依赖于该插件。连接后，命令面板会变为云端主机的 Shell 面板。

有了这个工具后还可以方便的在多台远程服务见切换，非常方便。

#### Remote Container

通过 ssh 连接到远程（本地）的 Docker 容器中进行操作。同样可以方便的管理目录和文件。

远程还是可以通过 ssh 直接连接，本地可以使用他，远程未连接成功。

## 注意事项

- 连接基于 ssh，默认使用 22 端口。需要云端服务器开放 22 端口

- 服务默认是连接本地网络内的另一台机器的，需要关闭该设置，否则无法连接云端的主机

  ```json
  {
    "remote.SSH.useLocalServer": false
  } 
  ```
- ...