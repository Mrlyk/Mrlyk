# python 模块管理工具

前端常用 npm 作为包管理工具。python 应用最广泛的则是 pip 和 anaconda。

[toc]

## pip

```shell
pip --version # py2 版本
pip3 --version # py3 版本
```

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

Anaconda实际上是一个软件的发行版，附带了Conda、python和150多个科学软件包及其相关的包。

conda 比 pip 更加强大。因为他不仅可以管理 py 包，**还可以创建 py 需要的虚拟环境**（类似 docker）。

#### 安装

国内镜像地址：https://mirrors.tuna.tsinghua.edu.cn/anaconda/?C=M&O=D

官方地址：https://anaconda.org/anaconda/conda

直接下载安装包或者脚本安装命令安装即可。

通过安装包安装的会自动加执行文件添加到环境变了。

默认安装位置：`/opt/miniconda3`（这里我安装的 mini 版本，只有 conda 和 python）

#### 模块管理

`conda`的命令和 `pip` 工具基本一致。下面举几个🌰：

**安装模块**

```shell
conda install nodejs=16 # 安装 16 版本的 nodejs
```

**卸载模块**

```shell
conda remove nodejs # 卸载 nodejs
```

**查找模块**

```shell
conda search nodejs # 查找模块
```

#### 虚拟环境管理

**1. 查看当前虚拟环境列表** 

```shell
conda env list
```

可以查看当前已经创建的虚拟环境列表。

默认会有一个叫 `base` 的基础环境。

**2.创建虚拟环境** 

```python
conda create -n env_name list_of_packages #其中，-n后的参数env_name表示环境名称，接着可以跟着0个或多个包名称。
```

举个🌰：

```shell
conda create -n myenv numpy python=2.7
```

创建一个名叫 `myenv` 的环境，同时配置 numpy （默认最新版本）和 python 为 2.7 版本。

**3.进入虚拟环境**

```python
conda activate myenv
```

**4.离开虚拟环境**

想要离开当前环境时，则只需要执行如下命令即可：

```python
conda deactivate
```

**5.删除虚拟环境**

当某个环境我们不再需要时，可以直接执行如下命令来删除该环境：

```python
conda env remove -n env_name
```

#### 安装package到指定的环境

```text
# 安装package
conda install -n myenv numpy
# 如果不用-n指定环境名称，则被安装在当前活跃环境
# 也可以通过-c指定通过某个channel安装
```

因为 conda 可以方便的创建虚拟环境，所以当我们有这方便的需求时最好就是用他。

#### 注意

安装好 anaconda 之后默认就会进入他的虚拟环境。这时候我们的操作都是在他的虚拟环境中进行的。如果暂时不需要开发，可以使用离开虚拟环境的命令`conda deactivate`来退出！！！

如果要不每次都自动进入 conda 环境，可以修改其配置，使用如下命令：

```shell
conda config -set auto_activate_base false
```

