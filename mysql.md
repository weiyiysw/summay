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

# 增加
set global max_connections = 3000;

# 修改 my.cnf 文件
> vi /etc/my.cnf
max_connections = 3000


# 查看时区
show variables like '%time_zone%';

# 查看数据库的字符集
> use db
> SELECT @@character_set_database, @@collation_database;
~~~

## 创建

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

# 查看某个用户的权限
show grants for dba@localhost;  
~~~

## 吊销

~~~shell
# 语法和 授权的语法差不多，to 转变为 from
revoke all privileges on databasename.* from username@'%';

flush privileges;
~~~

## mycli

~~~shell
# mycli 导出，xxx替换为表名
> \t csv; \o ~/xxx.csv; select * from xxx;
~~~

## MySQL小知识点

> MySQL版本 5.7

### 时区

MySQL的时区问题，大家都可能遇到过。大部分同学可能在网上搜索后，然后安装搜索得出的结果设置一下，就解决了，并没有深入的去了解为何。

MySQL关于时间的字段有`date`、`time`、`timestamp`、`datetime`、`year`。

| 字段      | Zero值              | 范围                                                         | 存储字节 |      |
| --------- | ------------------- | ------------------------------------------------------------ | -------- | ---- |
| date      | '0000-00-00'        | `'1000-01-01'` to `'9999-12-31'`                             | 4        |      |
| time      | '00:00:00'          | `'-838:59:59.000000'` to `'838:59:59.000000'`                | 3        |      |
| timestamp | '00:00:00 00:00:00' | `'1970-01-01 00:00:01.000000'` UTC to `'2038-01-19 03:14:07.999999'` UTC | 4        |      |
| datetime  | '00:00:00 00:00:00' | `'1000-01-01 00:00:00.000000'` to `'9999-12-31 23:59:59.999999'` | 8        |      |
| year      | 0000                | Values display as `1901` to `2155`, or `0000`                | 1        |      |

> `timestamp`以及`datetime`范围包含了小数部分，这个我们在定义时，指定`timestamp(6)`或`datetime(6)`指定小数部分即可。小数部分的范围为[0,6]。

重点要说明的是`timestamp`和`datetime`字段的区别。从表格中范围字段我们可以看出，`timestamp`是utc时间。也就是说：

* timestamp存储的是UTC时间，传入的时间会经过时区换算
* datetime存储的不是，传入什么时间，即存储

从这两个特点，我们可以了解到，如果项目需要支持多时区，考虑`timestamp`。

> 除这2个字段外，其他的字段存储的也是非UTC。

了解完了MySQL内的时间字段的基本内容，回到我们在开发中可能会遇到的时区问题上。

引起时区问题，那么说明我们在数据库的设计上，使用了`timestamp`。`timestamp`在存储时，将时间转换成了UTC时间存储。UTC时间比北京时间是差8小时。

> 可能有的同学会遇到相差13小时的，那是因为CST时区，是含混不清的，出现这个问题的原因是JDBC与MySQL对 “CST” 时区协商不一致。因为CST时区是一个很混乱的时区，有四种含义：
>
> - 美国中部时间 Central Standard Time (USA) UTC-05:00或UTC-06:00
> - 澳大利亚中部时间 Central Standard Time (Australia) UTC+09:30
> - 中国标准时 China Standard Time UTC+08:00
> - 古巴标准时 Cuba Standard Time UTC-04:00
>
> MySQL中，如果time_zone为默认的SYSTEM值，则时区会继承为系统时区CST，MySQL内部将其认为是UTC+08:00。而jdbc会将CST认为是美国中部时间，这就导致会相差13小时，如果处在冬令时还会相差14个小时。

~~~shell
# 查看MySQL当前时区、时间
mysql> show global variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+

# 查看当前时间
mysql> select now();
~~~

设置MySQL的时区，影响的是时区敏感的时间值的显示和存储。包括MySQL内置的一些函数(如`now()`、`curtime()`)显示的值，以及存储在`timestamp`类型中的值。不影响`date`、`time`和`datetime`列中的值，因为这些数据类型在存取时未进行时区转换。

`timestamp`类型存入数据库的实际是UTC的时间，查询显示时会根据具体的时区来显示不同的时间。

此时，如果我们的应用程序连接数据库时读取`timestamp`字段时，会根据我们连接JDBC串中设置的时区触发转换。

> JDBC串设置的时区和MySQL设置的时区是不影响的。

由此，如何解决时区问题呢。那就很简单设置正确的时区就可以了。

1. 应用程序连接MySQL时，设置JDBC字符串指定时区`serverTimezone=Asia/Shanghai`，这样保证应用读取的时区是正确的。
2. 通过MySQL客户端查询时，为保证和应用程序的时区一致，应该要把MySQL的时区设置为`+8:00`。`set time_zone='+8:00';`。

> MySQL启动的时候，可以改配置将其修改为+8:00
>
> 如果system的时区时+8的，也可以考虑直接设置一下。因为当timezone=system的时候，查询timestamp字段会调用系统的时区做时区转换，有全局锁_libclock_lock的保护，可能导致线程并发环境下系统性能受限。而改为'+8:00'则不会触发系统时区转换，使用MySQL自身转换，大大提高了性能

如果你使用的是`datetime`字段，那么需要保证自己的应用程序所处的时区是北京时间即可。

### 整型字段的length含义

先来看下整型字段的表格。

| 类型      | 存储字节 | 有符号范围                | 无符号范围      |
| --------- | -------- | ------------------------- | --------------- |
| tinyint   | 1        | [-128, 127]               | [0, 255]        |
| smallint  | 2        | [-32768, 32767]           | [0, 65535]      |
| mediumint | 3        | [-8388608, 8388607]       | [0, 16777215]   |
| int       | 4        | [-2147483648, 2147483647] | [0, 4294967295] |
| bigint    | 8        | [-2^63^, 2^63^ -1 ]       | [0, 2^64^ -1]   |

通常，我们在设计表的时候会有如下的写法：

~~~sql
create table xxx (
	...
    num1 int(2),
    num2 int(4)
    ...
)
~~~

很多时候，可能随手就写出上面的`num1`与`num2`的定义，可能会有不少的同学认为括号中的数字定义了该字段存储在数据库中的字节数，实则不然。接下来，就好好的说一下这个字段的含义。

我们打开MySQL的官方文档找到[Numeric Data Type Syntax](https://dev.mysql.com/doc/refman/5.7/en/numeric-type-syntax.html)页面，可以看到又这样一句话

> For integer data types, M indicates the maximum display width. The maximum display width is 255. Display width is unrelated to the range of values a type can store,as described in [Section 11.1.6, “Numeric Type Attributes”](https://dev.mysql.com/doc/refman/5.7/en/numeric-type-attributes.html).

这句话翻译过来是说“对于整型数据类型，M是指最大的显示宽度。最大的显示宽度是255。显示宽度是与可存储的值类型范围是无关的。”

这意味着`int(2)`与`int(4)`可存储值的范围是一样的。但是，他们的区别又是什么呢？

这就需要说到`ZEROFILL`这个可扩展的属性了。

> 如果开启了`ZEROFILL`属性，那么MySQL会给这个字段自动`UNSIGNED`属性。

~~~sql
create table xxx (
	...
    num1 int(2) ZEROFILL,
    num2 int(4) ZEROFILL
    ...
)
~~~

设置`ZEROFILL`后，当我们插入数据时，如果数据位数不够，会自动填补0。

~~~sql
insert into xx (num1, num2) value (3, 13);
insert into xx (num1, num2) value (333, 13333);
~~~

查询后会显示成，不足时，会自动填充0，如果宽度（位数）超过了，则无影响，正常显示。

~~~
num1: 03
num2: 0013

num1: 333
num2: 13333
~~~

> mycli 不支持PAD 0的展示