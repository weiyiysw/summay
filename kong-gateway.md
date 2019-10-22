# KONG网关

## 介绍

Kong构建在NGINX上，利用了NGINX的稳定性和有效性。Kong是一个运行在NGINX里的一个Lua应用。Kong通过OpenResty分发的。这使整个基础成为了插件式的架构，Lua脚本（或者叫插件）可以在运行时启动和运行。因为这样，我们倾向认为Kong是一个微服务架构的模范。它的核心是实现数据库抽象，路由和插件管理。插件可以存在于单独的代码库中，并且可以通过几行代码注入到请求生命周期的任何位置。

## 安装

### 1. Docker

#### 带数据库

1. 创建docker网络

   ~~~shell
   $ docker network create kong-net
   ~~~

2. 启动数据库

   ~~~shell
   # 使用 PostgreSQL
   $ docker run -d --name kong-database --network=kong-net \
   -p 5432:5432 -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" \
   -v /opt/postgres/data:/var/lib/postgresql/data postgres:9.6
   
   # 或者 Cassandra
   $ docker run -d --name kong-database --network=kong-net \
   -p 904:9042 cassandra:3
   ~~~

3. 准备数据库

   ~~~shell
   $ docker run --rm \
        --network=kong-net \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=kong-database" \
        kong:latest kong migrations bootstrap
   ~~~
   
4. 启动kong，这里选择使用`host`模式启动的，方便访问`nacos dns plugin`服务

   > host模式只能在Linux下操作

   ~~~shell
   # network模式，使用网卡 kong-net
   $ docker run -d --name kong \
        --network=kong-net \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=kong-database" \
        -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
        -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
        -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
        -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
        -p 8000:8000 \
        -p 8443:8443 \
        -p 8001:8001 \
        -p 8444:8444 \
        kong:latest
        
   # docker host 启动，同时配置 DNS_RESOLVER,
   # 注意把PG_HOST 和 DNS_RESOLVER 的ip 替换
   $ docker run -d --name kong \
        --network=host \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=<your ip>" \
        -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
        -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
        -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
        -e "KONG_DNS_RESOLVER=<your ip>:8849,114.114.114.114,8.8.8.8" \
        kong:latest
   ~~~

5. 使用kong

   ~~~shell
   $ curl -i http://localhost:8001/
   ~~~

6. 部署konga

   ~~~shell
   $ docker run -d -p 1337:1337 --network kong-net --name konga -e "NODE_ENV=production" pantsel/konga:latest
   ~~~

### 2. Kubernetes安装

待补充

## Kong特性

[参考GitHub](https://github.com/Kong/kong)，不在赘述。

## Kong对象

| 名称     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| service  | upstream services的抽象。服务与路由关联，可以有很多的路由。  |
| route    | 定义了匹配客户端请求的规则。每一个路由都关联到一个服务。一个服务可以有多个路由关联。每一个匹配到的请求将会代理到关联的服务上。 |
| consumer | API可能没有用户概念，会出现随意调用的情况。为此Kong提供了一种consumer对象（全局共用），如某API启用了key-auth，没有身份的访问者将无法调用该API。 |
| upstream | 上游对象代表虚拟主机名，可用于对多个服务（目标）上的传入请求进行负载平衡。 |
| plugin   | 插件。                                                       |

## Kong代理

kong的核心是其路由功能。kong监听了`8000`、`8001`、`8443`、`8444`4个端口。4个端口可分为两类，

* `proxy_listen`：代理端口，http为8000，https为8443。
* `admin_listen`：管理端口，http为8001，https为8444。

kong会将接收的请求与你配置的路由进行匹配。如果给定的请求匹配上了一个路由规则，kong就会代理这个请求。路由的属性由子系统决定。

子系统和路由属性：

* `http`：`methods`、`hosts`、`headers`、`paths`、（如果是`https`，还有`snis`属性）
* `TCP`：`sources`、`destinations`、（如果是`tls`，还有`snis`属性）
* `grpc`：`hosts`、`headers`、`paths`、（如果是`https`，还有`snis`属性）

如果尝试配置路由时添加不支持的属性，将会报错。例如：`http`路由`sources`或`destinations`属性。

如果kong接收一个请求不匹配任何路由，那将会报404。

### 路由的匹配规则

* 请求必须包含所有已配置的属性。
* 请求里设置的值，必须匹配上至少一个配置的值。

当使用path前缀作为转发时，最长的path将会被最先验证。