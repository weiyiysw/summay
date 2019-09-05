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

