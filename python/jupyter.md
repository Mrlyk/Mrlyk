# jupyter

jupyter 是一款交互式的笔记工具，可以实时运行你的代码并将结果保存。同时可以在旁边附上笔记。

vscode 集成了该工具，可以用来编写交互式的笔记。其中对 py 支持的最好，也支持其他语言。

## 配置编写 js 笔记

使用 jupyter 编写 js 笔记需要 node 环境。同时需要一系列前置包。

我们一般使用 `ijavascript `来配置：https://github.com/n-riesco/ijavascript#installation

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install pkg-config node zeromq
sudo easy_install pip
pip install --upgrade pyzmq jupyter
npm install -g ijavascript
ijsinstall
```

## 常用快捷键

- **Enter** : 转入编辑模式
- **Shift-Enter** : 运行本单元，选中下个单元
- **Ctrl-Enter** : 运行本单元
- **Alt-Enter** : 运行本单元，在其下插入新单元
- **Y** : 单元转入代码状态
- **M** :单元转入markdown状态
- **R** : 单元转入raw状态
- **D,D** : 删除选中的单元
- **Shift-M** : 合并选中的单元