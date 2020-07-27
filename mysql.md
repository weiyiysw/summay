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
# 安装，仅安装mysqldump
> yum -y install holland-mysqldump.noarch

# 导出指定数据库, 包含数据
> mysqldump -h$IP -P$Port -u$Username -p$Passwd --databases $Database > $database.sql

# 导出指定数据库，仅结构
> mysqldump --opt -h$IP -P$Port -u$Username --databases $Database > $database.sql

# 导出指定数据库，仅数据
> mysqldump -t -h$IP -P$Port -u$Username --databases $Database > $database.sql

# 导入数据
> source xxx.sql

# or
> mysql -u $username -P $Port -h $IP -p < xxx.sql
~~~

按客户端 IP 分组，看哪个客户端的链接数最多

~~~shell
select client_ip,count(client_ip) as client_num from (select substring_index(host,':' ,1) as client_ip from information_schema.processlist ) as connect_info group by client_ip order by client_num desc;
~~~

查看正在执行的线程，并按 Time 倒排序，查询执行时间特别长的线程

~~~shell
select * from information_schema.processlist where Command != 'Sleep' order by Time desc;
~~~

找出所有执行时间超过 5 分钟的线程，拼凑出 kill 语句

~~~shell
select concat('kill ', id, ';') from information_schema.processlist where Command != 'Sleep' and Time > 300 order by Time desc;
~~~

查看MySQL最大的连接数

~~~shell
# 查看最大连接数
show variables like '%max_connections%';

# 查看时区
show variables like '%time_zone%';
~~~

## 创建用户

~~~shell
# 创建用户
# username: 请替换为为用户名
# host: 指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符 %
# password: 用户的密码
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
~~~

## 授权

~~~shell
# 授权
# all 所有的权限，可单独设置 select update 等权限
# databasename 数据库名
# databasename.* 代表数据库下所有的表
grant all privileges on databasename.* to username@'%';

flush privileges;
~~~

