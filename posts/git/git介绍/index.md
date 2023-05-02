# Git介绍


本文主要介绍git的基本原理和一些简单的基础操作

<!--more-->

## 什么是Git
Git是一种版本控制系统，方便开发者管理版本内容，同时方便多用户协同工作。不同于svn，它是一种分布式的内容管理系统。同时Git和svn的区别还体现在Git的分支，而svn的分支就是一个单独的目录，也就是说，把主线上的内容拷贝一份就是一个新的分支，单纯对目录的拷贝。对于svn来说，他每一个单独的分支都有，每一次提交之后都会有不同基于全局的版本号，而Git并没有

## Git简单介绍
一般来说，我们使用Git，通常情况下会把本地仓库和远程仓库的内容进行同步
### 本地仓库
一般来说，Git在本地主要由“三棵树”来维护，第一棵树就是工作区，第二棵树暂存区，第三棵树是HEAD
#### “第一棵树”——工作区
这里存放的是实际的工作内容

#### “第二棵树”——暂存区（Index）

#### “第三棵树”——HEAD
他指向你的最后一次提交结果



#### 简要使用情况说明
>* 首先，可以通过`git init`或者`git clone`来创建或者克隆一个仓库到本地，在这个目录下面有一个.git的隐藏文件，在这个文件会保存暂存区的提交。
>* 然后在本地的仓库目录生成文件或者创建新的目录。此时文件仅仅存在于你的工作区中。
>* 通过`git add <file or directory>`把在工作区的文件添加到暂存区中，这事也就意味着，这个目录或者文件开始受到版本管理。
>* 此时实体文件既保存在工作区也保存在暂存区，但是还没有提交到HEAD。我们可以通过`git commit <file or directory>`来把暂存区的文件提交到HEAD中。完成提交

>* 最后采用`git push orign HEAD` 就是把HEAD中的文件推送到远端仓库

### 远端仓库
> TODO:

## 基本操作
### 证书相关
> 1. ssh-keygen -t rsa -C "email@163.com"
> 2. 需要执行 ssh -T git@github.com 来认证

### git config 配置的相关问题
> 1. git config --global user.name "name"
> 2. git config --global user.email "email"
> 3. git config --list 查看所有的config
> 4. git config --global init.defaultBranch dev git 修改生成的默认分支名
> 5. git config --global core.eitor vim 配置git的默认编辑器是vim



### git 分支相关的问题

> 1. git branch 查看分支
> 2. git checkout -b "name" 创建一个新分支并切换到新分支上
> 3. git checkout "name" 切换分支
> 4. git brach -d "name" 删除分支
> 5. git merge 源分支 合并分支

### git 冲突的相关问题（合并代码）

> git stash  
> git push orign master  
> git stash pop

### git 工作区
> 1. git add filename 添加工作区的目录到暂存区
> 2. git checkout filename 撤销工作区的修改（慎用）,必须要在版本管理之下,如果是新增的文件只能使用rm
> 3. git rm 删除本地文件

### git 暂存区
> 1. git rm --cached <file> git 删除添加到暂存区的文件
> 2. git reset filename 将暂存区的文件撤销
> 3. git restore --staged <File> 删除通过git add添加到暂存区的文件，但是保留本地的
> 4. git rm --cached 工作区被删除，暂存区还保留，恢复到本地


### git 本地仓库
> 1. git ci -m "message" 把暂存区的文件提交到本地仓库

### git 远端仓库
> 1. git init 初始化一个远程仓库
> 2. git remote add origin url 给本地仓库添加远程连接
> 3. git remote set-url origin git更换远程连接地址
> 4. git pull origin <branch> 更新该分支代码
> 5. git push origin <branch> 把本地的改动推送到远端仓库

### git 撤销conmmit
> git reset --soft HEAD^ 撤销commit，不改动工作区的代码，保留add  
> git reset --hard HEAD^ 删除工作空间的改动代码，撤销commit 撤销add  
> git commit -amend 修改本次的commit记录  


