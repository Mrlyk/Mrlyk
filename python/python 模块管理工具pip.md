# python 模块管理工具

前端常用 npm 作为包管理工具。python 应用最广泛的则是 pip 和 anaconda。

```shell
pip --version # py2 版本
pip3 --version # py3 版本
```

## pip

pip 默认是将模块安装到当前系统的用户目录下，相当于 npm 的全局安装，要注意这一点。

如果我们的 python 没有安装这个工具，可以使用如下命令安装：

```shell
python -m pip install # -m: 将库中的模块当作脚本运行
```

#### pip 常用命令

下面是一些常用命令并与 npm 做对比！

| 命令                    | 功能                                                     | 对比 npm         |
| ----------------------- | -------------------------------------------------------- | ---------------- |
| `pip install`           | 安装包                                                   | `npm install -g` |
| `pip install --upgrade` | 升级包，可以和 npm 一样自升级`pip install --upgrade pip` | `npm update`     |
| `pip uninstall`         | 卸载包                                                   | `npm uninstall`  |
| `pip search`            | 查找包                                                   | `npm search`     |
| `pip list`              | 列出已安装的包                                           | `npm list -g`    |

#### 通过依赖文件安装

在 node 中我们有 package.json 来表明当前项目的依赖。在 python 项目中则一般使用`requirements.txt`。

示例如下：

```txt
# YOLOv5 requirements
# Usage: pip install -r requirements.txt

# Base ----------------------------------------
matplotlib>=3.2.2
numpy>=1.18.5

# Extras --------------------------------------
ipython  # interactive notebook
psutil  # system utilization
thop>=0.1.1  # FLOPs computation
```

有了这样一个依赖文件我们就可以使用`pip install -r requirements`命令安装其中声明的所有模块！

其中`-r, --requirement <file>  Install from the given requirements file`

#### 配置镜像

众所周知的原因国内使用它最好配置一下镜像。

- **临时配置**

  ```shell
  pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package # 使用清华大学的镜像
  ```

- **永久配置**

  和 npm 一样更改配置文件的注册地址，需要 pip 版本高于 10

  ```shell
  pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
  ```

## anaconda