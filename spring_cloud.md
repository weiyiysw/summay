# Spring Cloud

## Zipkin

[ZipKin官网](https://zipkin.io/)

Zipkin是一个分布式追踪系统。在微服务架构中，它用于采集时间数据用于解决延迟的问题。其特性包括两方面：收集和查找数据。

### Quick Start

#### 运行Zipkin服务端

> 以下三种方式均是运行Zipkin的服务端。没有增加MQ、Storage。

##### 方式一：Docker

运行此docker命令，即可运行起Zipkin服务端

~~~shell
docker run -d -p 9411:9411  openzipkin/zipkin
~~~

##### 方式二：Jar

如果你安装了Java 8或更高的版本。可以直接下载jar包，并运行。

~~~shell
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
~~~

##### 方式三：Running from Source code

也可以选择从GitHub仓库拉去源码，并运行。

~~~shell
# get the latest source
git clone https://github.com/openzipkin/zipkin
cd zipkin
# Build the server and also make its dependencies
./mvnw -DskipTests --also-make -pl zipkin-server clean install
# Run the server
java -jar ./zipkin-server/target/zipkin-server-*exec.jar
~~~

#### 微服务采集Traces：HTTP发送

> 这里介绍与Spring Cloud Sleuth集成使用。

将`Spring Cloud Sleuth`添加到`pom.xml`文件中。

~~~xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train.version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
~~~

添加完之后，在配置文件中配置：

~~~yml
spring:
  zipkin:
    baseUrl: http://ip:port
  sleuth:
    sampler:
      percentage: 1.0 # 配置为1.0,表示所有请求都采集，生产环境不要配置成1.0，仅测试观察使用。
~~~

> 注意，默认percentage是0.1，即只有10%的Span会发送到Zipkin

配置`spring.zipkin.baseUrl`，Span通过http的方式，发送到ZipkinServer。

### 概念

追踪者（Tracers）存在应用里，在操作发生的地方，记录时间和元数据。它们通常是工具库，因此它们的使用对用户是透明的。

应用程序里将数据发送到Zipkin的组件被称为Reporter。Reporters通过多种传输方式里的一种将追踪数据发给Zipkin的collectors，collectors会将追踪数据持久化并存储。之后，存储的数据被提供的UI界面的API调用。

> instrumented client：可检测的客户端
>
> instrumented server：可检测的服务端

![zipkin架构图](images\zipkin_architecture.png)

#### Zipkin Components

ZipKin由四个组件组成

* collector
* storage
* search
* web UI

#### Transport

三种主要的传输方式：

* HTTP
* MQ：Kafka、RabbitMQ、ActiveMQ
* Scribe：暂时不了解。

#### Zipkin Collector

一旦追踪数据到达了后台运行的Zipkin Collector进程，它会被验证、存储、并且建立索引。

#### Storage

存储是可插拔的。除了Cassandra外，原生支持ElasticSearch和MySQL。

> 开发调试时，也可以支持写内存（in-memory）。

#### Web UI

Zipkin团队创建了一个GUI界面，友好的呈现来查看追踪数据（traces）。web ui查看提供基于service、time、annotation的方法，查看traces。

### 进阶

#### 微服务采集Traces：MQ发送

> 阅读此部分内容前，请先阅读微服务采集Traces：HTTP发送

前面讲了如何采用HTTP发送Traces。Zipkin支持使用MQ发送，如Kafka、RabbitMQ。

##### RabbitMQ

在HTTP发送的基础上，添加dependency：

~~~xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
~~~

配置（注意不需要配置`spring.zipkin.baseUrl`）：

~~~yml
spring:
  zipkin:
    rabbitmq:
      queue: zipkin  # 设置队列的名称
  sleuth:
    sampler:
      percentage: 1.0
~~~

##### Kafka

在HTTP发送的基础上，添加dependency：

~~~xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
~~~

配置（注意不需要配置`spring.zipkin.baseUrl`）：

~~~yml
spring:
  kafka:
    bootstrap-servers: domain:9092
  zipkin:
    sender:
      type: kafka
    kafka:
      topic: zipkin  # 设置队列的名称
  sleuth:
    sampler:
      percentage: 1.0
~~~

> 单独搭建Kafka的话，需要在host里配置Kafka的域名。

#### ZipKin服务端消费MQ

[zipkin-server配置](https://github.com/openzipkin/zipkin/blob/master/zipkin-server/README.md)。可打开链接，根据需要，在启动Zipkin服务端的时候，增加相应的配置。

##### RabbitMQ

示例使用（摘自配置）：

~~~shell
$ RABBIT_ADDRESSES=localhost java -jar zipkin.jar
~~~

##### Kafka

示例使用（摘自配置）：

~~~shell
$ KAFKA_BOOTSTRAP_SERVERS=127.0.0.1:9092 java -jar zipkin.jar
~~~

##### docker

这是docker启动的命令。配置了从Kafka消费数据，同时将消费的结果写入MYSQL中。

~~~shell
#!/bin/bash
docker run --name zipkin_mysql_test \
-p 8020:9411 \
-e STORE_TYPE=mysql \
-e MYSQL_HOST=$mysql_host \
-e MYSQL_DB=$db \
-e MYSQL_USER=$user \
-e MYSQL_PASS=$passwd \
-e KAFKA_BOOTSTRAP_SERVERS=kafka:9092 \
-e KAFKA_GROUP_ID=cg_zipkin_docker \
-e KAFKA_TOPIC=zipkin_span_collector \
--net zoo_kafka_kafka-net \
-d openzipkin/zipkin
~~~

> 存储：生产环境不建议使用MySQL，可以考虑替换为Elasticsearch。
>
> --net：是docker的参数，这是因为我使用docker启动了Kafka，ZipkinServer需要能够连接Kafka，因此在启动的时候，将Kafka与ZipkinServer配置在一个网络中。zoo_kafka_kafka-net，是我docker中配置kafka的网络的名称。

具体的配置，参考本小节ZipkinServer配置的链接即可。

## Spring Boot Admin

### 引子：Spring Boot Actuator

Spring Boot Actuator是Spring Boot的一个子项目，可以为应用增加监控和管理的支持。它暴露出很多的HTTP或JMX端点（endpoints），以便你可以与之交互。

这些端点（endpoints）提供了健康检查、指标监控、日志访问、线程dump、堆dump、环境信息等。

Actuator很强大。在你其他的应用中你仅只需要使用简单的REST调用，就可以很简单和方便的消费端点（endpoints）。但是，它在使用上来说，对人不友好。对于人来说，更加方便的是需要有一个漂亮的用户界面，你可以通过浏览器直观的监控和管理你的应用。这就是Spring Boot Admin做的事情，在actuator和一些其他特性上提供一个友好的用户界面以供使用。

### 功能

SBA能够为注册的应用提供这些功能：

* 显示健康状态
* 显示详情
  * JVM & 内存指标
  * IO指标（micrometer.io metrics）
  * 数据源指标
  * 缓存指标
* 显示构建者信息
* 跟踪和下载日志
* 查看JVM系统和环境属性
* 查看Spring Boot配置属性
* 支持Spring Cloud postable /enc-&/refresh-endpoint
* 简单日志等级管理
* JMX-beans交互
* 查看线程dump
* 查看http-traces
* 查看审计事件
* 查看http-endpoints
* 查看计划任务
* 查看和删除激活sessions
* 查看Flyway / Liquibase数据库混合
* 下载heapdump
* 通知状态变化
* 事件日志状态变化

### Client和Server

