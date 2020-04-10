# Git

## Git基础

> 参考资料 [git-pro_cn](https://git-scm.com/book/zh/v2)

### Git配置



### 获取Git仓库

#### 初始化

~~~shell
# 在目录下执行，即可将当前目录用作git仓库
$ git init
~~~

#### 克隆现有仓库

~~~shell
# https模式，克隆现有的仓库
$ git clone https://github.com/weiyiysw/summay.git

# SSH模式，克隆现有仓库
$ git clone git@github.com:weiyiysw/summay.git

# 克隆后，直接rename根目录(ssh模式或https模式均可)
$ git clone git@github.com:weiyiysw/summay.git mysummary
~~~

#### 记录更新

~~~shell
# 检查仓库当前状态
$ git status

# 跟踪新文件，或者将处于已修改的文件变为暂存状态
$ git add file

# 提交，引号中填写需要提交的msg
$ git commit -m "msg"

# 文件状态简览
# A：代表新追踪文件
# M：代表修改。如果终端显示了颜色，红色代表已修改未暂存，绿色代表已修改且暂存
# ??：代表未跟踪文件
$ git status -s

# 查看修改

# 查看未暂存的修改
# 可输入文件名，也可以不输入，输入文件名则只查看该文件
$ git diff [file]

# 查看已暂存的修改
$ git diff --cached [file]

# 移除
$ git rm file

# 移动文件
$ git mv oldfile newfile
~~~

#### 查看提交历史

~~~shell
# 查看提交历史
$ git log

# 想看到每次提交的简略统计信息
$ git log --stat

# git --pretty
$ git log --pretty=oneline
$ git log --pretty=format:"%h - %an, %ar : %s"

# 常用的查看提交的快捷指令
$ git log --graph --pretty=oneline --abbrev-commit
~~~

> [更加详细的log显示查看]([https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2](https://git-scm.com/book/zh/v2/Git-基础-查看提交历史))

#### 撤销操作

~~~shell
# 撤销并重新提交
# 注意：此命令会将暂存区中的文件提交，如果暂存区存在文件的话
$ git commit --amend

# 取消暂存
$ git reset HEAD <file>
# or
$ git restore --staged <file>

# 抛弃更改
$ git checkout -- <file>
# or
$ git restore <file>
~~~

#### 远程仓库

~~~shell
# 查看远程仓库
$ git remove [-v]

# 添加远程仓库
# git remote add <shortname> <url>
$ git remote add acen git@github.com:weiyiysw/summay.git

# 从远程仓库抓取
$ git fetch acen

# 推送到远程仓库
# git push <remote> <branch>
$ git push acen master

# 远程仓库重命名
$ git remote rename oldName newName

# 远程仓库移除
$ git remote remove remoteName
~~~

#### 打标签

~~~shell
# 显示标签
$ git tag [-l or --list]

# 轻量标签
$ git tag <tagName>

# 附注标签：指定 -a，并添加信息
# 与轻量标签的区别：附注标签会包含打标签者的信息、日期、附注信息等。
$ git tag -a <tagName> -m "msg"

# 后期添加标签
# sha-1是提交的校验和
$ git tag -a <tagName> <sha-1>

# 共享标签，推送到远程仓库
$ git push origin <tagname>

# 共享标签，此命令会一次性推送多个标签
$ git push origin --tags

# 删除标签，本地仓库删除
$ git tag -d <tagName>

# 远程删除，解释：将冒号前面的空值推送到远程仓库标签名处，从而删除
$ git push <remote> :refs/tags/<tagname>

# 远程删除，这种方式更加直观
$ git push origin --delete <tagname>
~~~

### 分支

~~~shell
# 创建新分支，但不切换到分支
$ git branch <branchName>

# 创建新分支并切换到分支
$ git checkout -b <branchName>

# 切换到已有分支
$ git checkout <branchName>

# 删除分支
$ git branch -d <branchName>
~~~

### 合并

~~~shell
# 合并分支时，需要切换到当前分支，在执行合并命令，将其他分支代码合并过来。
$ git merge branchName

# 分支上的commit msg如果不需要，就使用以下2条命令，将分支上的代码合并过来，并重新创建提交
$ git merge --squash branchName
$ git commit -m "msg"
~~~





---

分割线

---

## git rebase

~~~shell
# 图形查看提交
$ git log --graph --pretty=oneline --abbrev-commit
*   95678c3 (HEAD -> master) Merge branch 'master' of XXX
|\
| * 98726ca (origin/master) fix build.sh
* | edb7ce8 fix mvn dependency conflict
|/
* f9366d9 create table DDL add comment
* 5cf8d2a fix handlers
* 86bcad6 consumer msg
# 将提交整理成一条直线
> git rebase origin/master
First, rewinding head to replay your work on top of it...
Applying: fix mvn dependency conflict

# 重新查看，变成一条直线啦
> git log --graph --pretty=oneline --abbrev-commit
* da37d56 (HEAD -> master) fix mvn dependency conflict
* 98726ca (origin/master) fix build.sh
* f9366d9 create table DDL add comment
* 5cf8d2a fix handlers
* 86bcad6 consumer trip msg
~~~

## 修改commit msg

~~~shell
# 修改最近一次的commit信息
> git commit --amend
# 然后就会进入vim编辑模式，编辑完保存即可

# 合并多次提交记录，然后按提示操作
> git rebase -i [startpoint] [endpoint]
~~~

## 获取 commit id

~~~shell
# 获取完整的commit-id
> git rev-parse HEAD

# 获取short commit-id
> git rev-parse --short HEAD
~~~

## 本地文件变更不提交

通常有一些文件或配置往往只需要在本地变更即可，不需要提交到Git仓库。

1. `.gitignore`文件方式
2. 如果我们的文件本身就被git管理了，那么就不适合用第一种方式了。

~~~shell
# 本地设置该文件，变更不提交  （assume：假设）
> git update-index --assume-unchange filename

# 取消
> git update-index --no-assume-unchange filename
# 或者
> git update-index --really-refresh
~~~

## Git设置默认分支

~~~shell
# 设置默认分支
> git branch --set-upstream-to=origin/master master
# 设置完成后，直接使用 git pull / git push命令即可拉取/推送默认分支
~~~

