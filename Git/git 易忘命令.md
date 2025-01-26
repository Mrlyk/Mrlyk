# git 易忘命令

> 记录一下 git 偶尔会用到，经常忘记的命令

在回忆下面的易忘命令之前，我们要牢记 git 的分区，工作区、暂存区、本地仓库、远程仓库之间的关系。

![image-20231031161517279](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg_2023/image-20231031161517279.png?x-oss-process=image/resize,w_600,m_lfit) 

[toc]

#### commit 提交

```shell
git commit
	-n // --no-verify 忽略 commit 钩子
	-a // 自动 git add 所有被追踪的文件
```



#### push 推送

```shell
git push origin local_branch:remote_branch  // 将本地当前分支推送到远程不存在的分支
```



#### stash 暂存

```shell
git stash save 'msg' // 暂存并且备注信息 msg
git pop // 取出暂存的第一条记录并删除
git stash apply stash@{1} // stash@{1} 恢复 stash@{1} 的记录，不指定名字时默认第一条
git stash drop stash@{1} // 丢弃某一条 stash
git stash list // 查看 stash 列表
git stash branch myBrach // 从 stash 创建分支
```



#### reset / revert / checkout / restore 回退版本

```shell
git revert {log}  # 回退某一次 commit，会产生回退记录
	-n // --no-commit 回退并且不提交
	
	
git reset {log} # 回退到某个commit 并且放弃这次 commit 之后的所有提交，没有记录慎用
	--hard // 回退并且放弃当前的更改
# 如果想 reset 之后推送到远程可以强制推送，但是注意这种方式要保证分支的安全性，同步团队成员，更推荐使用 revert 命令
git push --force origin <branch-name> # 强制推送
	
	
git checkout HEAD~2 myfile.js # 回退单个文件到某个版本 ~ 波浪线表示该版本往前
git checkout myfile.js # 撤销修改，恢复到最后一次提交状态，相当于 rollback


# 因为 checkout 的本意是用来切换分支的，用 checkout 来回退文件很令人困惑，所以 git 在2.23+ 版本推出了 restore 命令来替代他的作用
git restore -s stash@{0} -- <filename>
```

#### cherry-pick 提取某个版本信息

```shell
git cherry-pick [hash | branchName] // 提取某一次提交到当前分支，可用空格提取多个
	-e // 编辑提交信息
	-n // 只更新到工作区和暂存区，不提交
	-x // 在提交信息末尾追加 cherr-pick 记录
```

#### 修改 commit 信息

1. 刚刚 commit 还未提交

```shell
git commit --amend; # 重新修改 commit 信息并且创建新的提交 amend-修改
```

2. 修改历史的 commit 信息

```shell
git rebase -i [coomit_id] # 变基目标分支，-i 可以修改 commit 信息
```

#### git rebase 变基，合并提交树

合并提交树，操作相当于：

1. 暂存当前所有提交
2. 拉取最新的目标分支提交
3. 将暂存的提交合并到最新的目标分支之后（暂存的提交 id 会被改变）
4. 创建新的提交信息

可以让提交树更清晰

**示例：**

假设有以下三个提交

- fix: bug2
- fix: bug1
- feature: new fn
- feature: init

开发者在新增 new fn 这个特性时，带来了两个 bug。后续又分别用两次 commit 修复了这两个 bug。但实际这都属于一个功能，我们希望 git 提交树更清晰，就可以使用 `git rebase`。

这里我们希望合并 bug2、bug1、new fn 到一个提交上，所以我们需要把现在的“基”变回最开始的 init 提交，并且**通过交互处理后面三个提交**

```sh
git reabse -i [init-hash] # -i 开启交互操作，让我们知道如何处理后面的提交。变基到 init 分支（commit hash）
```

执行该命令后会弹出交互操作页面

![image-20230131094034402](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20230131094034402.png?x-oss-process=image/resize,w_800,m_lfit) 

- **pick**：使用commit。
- **reword**：使用commit，修改commit信息。
- **squash**：使用commit，将commit信息合入上一个commit。
- **fixup**：使用commit，丢弃commit信息。

我们要修改对要合并分之多交互操作，通常将第一个保留，采用 `pick` 操作，后面的则采用 `squash` 操作，将提交压入第一个。

![image-20230131094301687](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20230131094301687.png?x-oss-process=image/resize,w_600,m_lfit) 

之后会出现修改提交信息的界面，可以删除后面几个提交信息，只保留自己想要的即可。

**如果变基后想回退，可以通过`git reflog`命令查看所有变更日志，找到变基前的 commit hash，使用`git reset --hard` 强制回退！**

#### 远程仓库关联的添加与删除

- 删除 `git remote remove origin`
- 添加`git remote add origin [remote_address]`

**如果只是单纯变更远程仓库地址也可以使用下面这个命令** 

```shell
git remote set-url origin https://xxxxx
```

#### 本地分支关联远程分支

`git branch --set-upstream-to=origin/<branch> <local-branch>`

#### git 子模块 git-submodule

通过 Git 子模块，可以在当前 repo 中包含其它 repos、作为当前 repo 的子目录使用，同时能够保持 repos 之间的独立。

```shell
# 在当前 repo 添加一个子模块
git submodule add git@github.com:xxx/xxx.git path/to/xxx
```

可以在 `.gitmodule`文件中看到当前 repo 有哪些 submodule，分别的 `name`, `branch` 等。

```mipsasm
# clone 含有 submodule 的 repo 后：
# 初始化 git submodule 信息
git submodule init
# 更新 submodule，相当于 git pull 吧
git submodule update
```

修改子模块文件后，在当前 repo 执行 `git status` 只会看到有模块的 changes，而不是具体子模块文件

#### git show [id] 查看提交详细信息

```sh
git show ad7b0348fc1 # 查看这次提交的具体文件修改信息
```

#### git blame 查看文件修改人

blame 这个单词的意思是责怪，这个命令的取名真有意思。。。找背锅人是吧！

```shell
git blame [filename] # 查看文件所有内容每一行的修改人
-L # 查看具体行数

git blame -L 20,30 [filename] # 查看文件 20 ～ 30 行的修改人
```

#### git rm 删除文件

git 分为暂存区和工作区，工作区是我们正在进行编码的地方，暂存区则是我们已经修改完的内容（git add）。

使用该命令一般是用来删除暂存区中的文件，比如我们不小心将 .idea 这种本地配置文件提交到了远程，现在要将他们删掉就可以使用该命令。

```shell
git rm --cached [file]
```

如果没有 cache 参数就是直接从工作区中删除，和我们直接手动删没有区别？

删除掉暂存区的文件不会对我们工作区产生影响，所以即使我们有一些更改了 .idea 本地编辑器配置，删除了暂存区中的内容也不会影响他们。

删除后再使用 git commit 和 push 命令同步到远程仓库即可。
