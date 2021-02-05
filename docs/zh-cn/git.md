## Git常用命令

### 🍉**快速开始** — if you’ve done this kind of thing before

[ Set up in Desktop](https://desktop.github.com/)**or**HTTPSSSH



Get started by [creating a new file](https://github.com/Mini-Bar/MemoryPalace/new/main) or [uploading an existing file](https://github.com/Mini-Bar/MemoryPalace/upload). We recommend every repository include a [README](https://github.com/Mini-Bar/MemoryPalace/new/main?readme=1), [LICENSE](https://github.com/Mini-Bar/MemoryPalace/new/main?filename=LICENSE.md), and [.gitignore](https://github.com/Mini-Bar/MemoryPalace/new/main?filename=.gitignore).

> 首先创建一个新文件或上传一个现有的文件。我们建议每个存储库都包含自述文件README、许可证LISCENSE和.gitignore。

### …or create a new repository on the command line



```shell
echo "# MemoryPalace" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/...
git push -u origin main
```

### …or push an existing repository from the command line



```bash
git remote add origin https://github.com/Mini-Bar/MemoryPalace.git
git branch -M main
git push -u origin main
```

### …or import code from another repository

>  You can initialize this repository with code from a Subversion, Mercurial, or TFS project.
>
> 您可以使用来自Subversion、Mercurial或TFS项目的代码初始化此存储库。





#### 备份与还原

![](https://ftp.bmp.ovh/imgs/2021/02/9335d313c93a550e.png)

#### 比较

![](https://ftp.bmp.ovh/imgs/2021/02/ce991602fad5ed74.png)

#### 分支

创建对当前所有的数据产生一个更大“的副本

![](https://ftp.bmp.ovh/imgs/2021/02/74c390f2c83f49b7.png)

### Git优点:

分布式

高效（团队分支

离线工作，服务器压力小

易于合并

![](https://ftp.bmp.ovh/imgs/2021/02/adc39be859cb1ee1.png)

## Git指令

![](https://ftp.bmp.ovh/imgs/2021/02/61dc21e9c830ef36.png)

![](https://ftp.bmp.ovh/imgs/2021/02/39f0cb6a787bfbe5.png)

#### 本地和服务器的搭建

![](https://ftp.bmp.ovh/imgs/2021/02/80c8d48d7b3ce6f5.png)

![](https://ftp.bmp.ovh/imgs/2021/02/99cfbc3b4b7956c6.png)










##  新建代码库和 git clone

在当前目录新建一个git代码库

```shell
git init	
```

新建一个目录，将其初始化为git代码库

```shell
git init [project-name]
```

下载一个项目和他的整个代码历史

```bash
git clone [url]			
```

## 配置 git config

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

显示当前的Git配置

```bash
git config --list
```

编辑Git配置文件

```shell
git config -e [--global]
```

设置提交代码时的用户信息

```shell
git config [--global] user.name "[name]"
git config [--global] user.email "[email address]"
```

## 远程同步   git remote pull push fetch

下载远程仓库的所有变动

```shell
git fetch [remote]
```

显示所有远程仓库

```shell
git remote -v
```

显示某个远程仓库的信息

```shell
git remote show [remote]
```

 增加一个新的远程仓库，并命名

```shell
git remote add [shortname] [url]
```

取回远程仓库的变化，并与本地分支合并

```shell
git pull [remote] [branch]
```

上传本地指定分支到远程仓库

```shell
 git push [remote] [branch]
```

强行推送当前分支到远程仓库，即使有冲突

```shell
git push [remote] --force
```

推送所有分支到远程仓库

```shell
git push [remote] --all
```







## 增加   git add

  添加指定文件到暂存区

```shell
git add [file1] [file2] ..
```
添加指定目录到暂存区，包括子目录
```shell
$ git add [dir]
```
添加当前目录的所有文件到暂存区
```shell
$ git add .
```
添加每个变化前，都会要求确认

对于同一个文件的多处变化，可以实现分次提交
```shell
$ git add -p
```
## 代码提交	git commit
提交暂存区到仓库区
```shell
$ git commit -m [message]
```
提交暂存区的指定文件到仓库区
```shell
$ git commit [file1] [file2] ... -m [message]
```
提交工作区自上次commit之后的变化，直接到仓库区
```shell
$ git commit -a
```
提交时显示所有diff信息
```shell
$ git commit -v
```
使用一次新的commit，替代上一次提交
如果代码没有任何新变化，则用来改写上一次commit的提交信息

```shell
$ git commit --amend -m [message]
```
重做上一次commit，并包括指定文件的新变化
```shell
$ git commit --amend [file1] [file2] ...
```

## 查看提交信息 git reflog/log

 

显示当前分支的最近几次提交
```shell
git reflog
```
显示当前分支的版本历史
```shell
git log
```
显示commit历史，以及每次commit发生变更的文件
```shell
git log --stat
```
搜索提交历史，根据关键词
```shell
git log -S [keyword]
```
显示某个commit之后的所有变动，每个commit占据一行
```shell
git log [tag] HEAD --pretty=format:%s
```
显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
```shell
git log [tag] HEAD --grep feature
```
显示某个文件的版本历史，包括文件改名
```shell
git log --follow [file]
```

```shell
git whatchanged [file]
```
显示指定文件相关的每一次diff
```shell
git log -p [file]
```
显示过去5次提交
```shell
git log -5 --pretty --oneline
```
显示所有提交过的用户，按提交次数排序
```shell
git shortlog -sn
```
显示指定文件是什么人在什么时间修改过
```shell
git blame [file]
```


## 比较 git status/diff

显示有变更的文件
```shell
git status
```
显示暂存区和工作区的差异
```shell
git diff
```
显示暂存区和上一个commit的差异
```shell
git diff --cached [file]
```
显示工作区与当前分支最新commit之间的差异
```shell
git diff HEAD
```
显示两次提交之间的差异
```shell
git diff [first-branch]...[second-branch]
```
显示今天你写了多少行代码
```shell
git diff --shortstat "@{0 day ago}"
```


## 查看 git show

显示某次提交的元数据和内容变化
```shell
git show [commit]
```
显示某次提交发生变化的文件
```shell
git show --name-only [commit]
```
显示某次提交时，某个文件的内容
```shell
git show [commit]:[filename]
```
## 修改 git mv
 改名文件，并且将这个改名放入暂存区
```shell
git mv [file-original] [file-renamed]
```
## 删除 git rm
删除工作区文件，并且将这次删除放入暂存区
```shell
git rm [file1] [file2] ...
```
停止追踪指定文件，但该文件会保留在工作区
```shell
git rm --cached [file]
```


## 撤销恢复 git checkout /reset
 恢复暂存区的指定文件到工作区
```shell
git checkout [file]
```
恢复某个commit的指定文件到暂存区和工作区
```shell
git checkout [commit] [file]
```
恢复暂存区的所有文件到工作区
```shell
git checkout .
```
重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
```shell
git reset [file]
```
 重置暂存区与工作区，与上一次commit保持一致
```shell
git reset --hard
```
重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
```shell
git reset [commit]
```
重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
```shell
git reset --hard [commit]
```
重置当前HEAD为指定commit，但保持暂存区和工作区不变
```shell
git reset --keep [commit]
```
新建一个commit，用来撤销指定commit
后者的所有变化都将被前者抵消，并且应用到当前分支
```shell
git revert [commit]
```
## 暂时移除 git stash
 暂时将未提交的变化移除，稍后再移入
```shell
git stash
```

```shell
git stash pop
```
## 分支 git branch
列出所有本地分支
```shell
git branch
```
列出所有远程分支
```shell
git branch -r
```
列出所有本地分支和远程分支
```shell
git branch -a
```
新建一个分支，但依然停留在当前分支
```shell
git branch [branch-name]
```
新建一个分支，并切换到该分支
```shell
git checkout -b [branch]
```
新建一个分支，指向指定commit
```shell
git branch [branch] [commit]
```
 新建一个分支，与指定的远程分支建立追踪关系
```shell
git branch --track [branch] [remote-branch]
```
切换到指定分支，并更新工作区
```shell
git checkout [branch-name]
```
切换到上一个分支
```shell
git checkout -
```
 建立追踪关系，在现有分支与指定的远程分支之间
```shell
git branch --set-upstream [branch] [remote-branch]
```
合并指定分支到当前分支
```shell
git merge [branch]
```
选择一个commit，合并进当前分支
```shell
git cherry-pick [commit]
```
删除分支
```shell
git branch -d [branch-name]
```
删除远程分支
```shell
git push origin --delete [branch-name]
```

```shell
git branch -dr [remote/branch]
```
## 标签git tag
列出所有tag
```shell
git tag
```
 新建一个tag在当前commit
```shell
git tag [tag]
```
新建一个tag在指定commit
```shell
git tag [tag] [commit]
```
删除本地tag
```shell
git tag -d [tag]
```
 删除远程tag
```shell
git push origin :refs/tags/[tagName]
```
查看tag信息
```shell
git show [tag]
```
提交指定tag
```shell
git push [remote] [tag]
```
提交所有tag
```shell
git push [remote] --tags
```
新建一个分支，指向某个tag
```shell
git checkout -b [branch] [tag]
```
## 其他
生成一个可供发布的压缩包

```shell
git archive
```
