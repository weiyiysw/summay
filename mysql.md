# MySQL日常常用命令

## 安装mycli

安装Python 2.7或3.4+

[mycli install地址](https://www.mycli.net/install)

[mycli github](https://github.com/dbcli/mycli)

> 以下命令均在mycli下执行

## 常用命令

~~~shell
# host: 服务器地址
# port: 端口 （可省略）
# user passwd: 用户名与密码
# database: 数据库 （可省略）
mycli -h host -P port -u user -p passwd -D database
~~~

## 删除正在执行的SQL

~~~shell
# 列出执行的语句list
show processlist;

# 找到需要终止的语句ID，kill掉即可
kill $ID
~~~

## Mysqldump

~~~shell
# 导出指定数据库, 包含数据
> mysqldump -h$IP -P$Port -u$Username -p$Passwd -databases $Database > $database.sql

# 导出指定数据库，仅结构
> mysqldump --opt -h$IP -P$Port -u$Username -databases $Database > $database.sql

# 导出指定数据库，仅数据
> mysqldump -t -h$IP -P$Port -u$Username -databases $Database > $database.sql
~~~

