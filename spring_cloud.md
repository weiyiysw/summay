# Spring Cloud

## Zipkin

[ZipKin官网](https://zipkin.io/)

Zipkin是一个分布式追踪系统。在微服务架构中，它用于采集时间数据用于解决延迟的问题。其特性包括两方面：收集和查找数据。

追踪者（Tracers）存在应用里，在操作发生的地方，记录时间和元数据。它们通常是工具库，因此它们的使用对用户是透明的。

应用程序里将数据发送到Zipkin的组件被称为Reporter。Reporters通过多种传输方式里的一种将追踪数据发给Zipkin的collectors，collectors会将追踪数据持久化并存储。之后，存储的数据被提供的UI界面的API调用。

![zipkin架构图](images\zipkin_architecture.png)

## Zipkin Components

ZipKin由四个组件组成

* collector
* storage
* search
* web UI

## Transport

三种主要的传输方式：

* HTTP
* Kafka
* Scribe

> rabbitmq属于kafka里？

## Zipkin Collector

一旦追踪数据到达了后台运行的Zipkin Collector进程，它会被验证、存储、并且建立索引。

## Storage

存储是可插拔的。除了Cassandra外，原生支持ElasticSearch和MySQL。

> 开发调试时，也可以支持写内存（in-memory）。

## Web UI

Zipkin团队创建了一个GUI界面，友好的呈现来查看追踪数据（traces）。web ui查看提供基于service、time、annotation的方法，查看traces。

