# Git

## git rebase

~~~shell
# 图形查看提交
> git log --graph --pretty=oneline --abbrev-commit
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

