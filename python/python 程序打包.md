# python 程序打包

python 工程开发完后我们需要打包给用户使用，打包一般使用 pyinstall 工具，下面说一说这个工具的用法。

pyinstall 官方文档：https://pyinstaller.org/en/stable/

## 安装

```shell
pip3 install pyinstaller
```



## 使用

```shell
pyinstaller -参数 主文件.py # 也可以对着 spec 文件打包，参数不是必填的
```

其中参数如下表：

| 参数                          | 参数意义                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| **-F --onefile**              | 1.打包单个文件，产生一个文件用于部署(默认)，如果代码都写在一个.py文件时使用，项目有多个文件时不要使用  `pyinstaller -F xxx.py` 例：`pyinstaller --onefile xxxx.py` |
| **-D --onedir**               | 1.打包多个文件，产生一个目录用于部署(默认)，用于框架编写的代码打包  例：pyinstaller -D xxx.py（项目入口文件） 例：`pyinstaller --onedir xxx.py`（项目入口文件） |
| **--key=keys**                | 1.使用keys进行加密打包  例：`pyinstaller --key=1234 -F xx.py` |
| **-K --tk**                   | 1.在部署时包含 TCL/TK                                        |
| **-a --ascii**                | 1.不包含编码.在支持Unicode的python版本上默认包含所有的编码   |
| **-d --debug**                | 1.产生debug版本的可执行文件                                  |
| **-n name --name=name**       | 1.可选的项目(产生的spec的)名字name 2.第一个脚本的主文件名将作为spec的名字(默认)  `pyinstaller -F -n my_file_name xxx.py `例：`pyinstaller -F --name=my_file_name xxx.py` |
| **-o dir -- out=dir**         | 1.指定spec文件的生成目录dir 2.如果没有指定且当前目录是PyInstaller的根目录,会自动创建一个用于输出(spec和生成的可执行文件)的目录 3.**如果没有指定切当前目录不是PyInstaller的根目录，则会输出到当前的目录下** |
| **-p dir --path=dir**         | 1.**用来添加程序所用到的包的所在位置，设置导入路径**(和使用pythonpath效果相似) 2.可以用路径分割符(Windows使用分号,Linux使用冒号)分割,指定多个目录.也可以使用多个-p参数来设置多个导入路径，让Pyintaller自己去找程序需要的资源 |
| **-w --windowed --noconsole** | 1.表示去掉控制台窗口，使用Windows子系统执行，当程序启动的时候不会打开命令行(只对Windows有效)  例：pyinstaller -c xxx.py 例：`pyinstaller xxx.py --noconsole` |
| **-c --nowindowed --console** | 1.表示打开控制台窗口，使用控制台子系统执行,当程序启动的时候会打开命令行(默认)(只对Windows有效)  例：pyinstaller -c xxx.py 例：`pyinstaller xxx.py --console` |
| **-i --icon=<file.ioc>**      | 1.将file.ico添加为可执行文件的资源，改变程序的图标(只对Windows系统有效)  例：pyinstaller -F -i file.ico xxx.py 例：`pyinstall -F --icon=<file.ioc> xxx.py` |
| **--icon=<file.exe,n>**       | 1.将file.exe的第n个图标添加为可执行文件的资源(只对Windows系统有效) |
| **-v file --version=file**    | 1.将verfile作为可执行文件的版本资源(只对Windows系统有效)     |
| **-s --strip**                | 1.可执行文件和共享库将run through strip.注意Cygwin的strip往往使普通的win32 Dll无法使用 |
| **-X --upx**                  | 1.如果有UPX安装(执行Configure.py时检测),会压缩执行文件(Windows系统中的DLL也会)(参见note) |

#### spec 文件

sepc 文件时打包的描述文件，在打包后会自动生成，也可以使用`pyi-makespec`命令主动生成。其参数和上面的一样。只不过这个命令只会生成 spec 文件。

#### 资源文件路径问题

打包之后最常见的问题就是资源文件的路径问题，资源文件不能正确加载程序会直接闪退。

**我们需要通过 spec 文件来配置资源文件的路径！**

打开 spec 文件会看到下面的描述文字

```spec
a = Analysis(
    ['game-alien-invasion.py'],
    pathex=[],
    binaries=[],
    datas=[('images', 'images')], #  看这里看这里
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)
```

其中 datas 接收一个元祖列表，每一项表明了不同资源文件的位置。

每一项：

- 第一个属性表明要打包的资源文件，相对主文件的路径
- 第二个属性表明打包后的资源文件，相对打包后的主文件的路径，一般情况下两项时相同的

配置好之后直接使用`pyinstaller xxx.spec`即可正确打包！