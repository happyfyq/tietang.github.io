---
title: 微服务之spring-cloud分布式外部化和中心化配置管理
thumbnail: 'images/m1.jpg'
date: 2016-09-08 19:22:47
categories:
	- 微服务
tags:
	- spring-cloud
	- 微服务
	- Spring Boot
	- 分布式配置管理
keywords:
	- 微服务
	- Spring Cloud
	- Spring Boot
	- 分布式配置管理
description:
---

![DPP_0001.jpg](http://upload-images.jianshu.io/upload_images/2519252-16f4dd62aebd0502.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

微服务化之后，应用的数量剧增，零时需要调整配置参数时，无论是运维直接在服务器上修改还是工程中修改配置后重新打包部署，对运维来说工作量是巨大的，而且人为的操作会加大出错的几率，那么外化和中心化配置可以更好的解决分布式环境的配置问题。Spring Cloud提供了2种方式的外化配置：

1. Spring Cloud Config 通过本地文件系统，git/svn仓库来管理配置文件，可以满足基本外化需求，但不能精细的管理配置项。
2. Spring Cloud Zookeeper Config 通过Zookeeper分级命名空间来储存配置项数据，并且支持基础上下文和profile命名空间，另外Zookeeper可以实时监听节点变化和通知机制，应该是首选。

Spring Cloud Zookeeper Config提供的功能：

- 和Spring Boot无缝集成，能完全无缝替代properties或yml文件的配置。
- 支持默认上下文命名空间。
- 支持profile命名空间。
- 支持应用名称命名空间。
- 命名空间上支持配置的继承。
- 支持更改实时通知和endpoint `/refresh`被动刷新，该特性不太好用。

Spring Cloud Zookeeper Config 基本能满足要求了，但其实时通知机制会造成应用暂停体验不好，需要初始化操作的配置（比如数据库连接池，http连接池等）实时更新的意义不大。需要实时更新的是那些在运行时从配置缓存中实时获取的参数，因此在此基础上增加基于Spring Cloud Zookeeper Config和CuratorFramework的实时通知组件，基本设计思路是这样的：

1. 每一个需要动态更新的节点下增加push_status节点，任意值。
2. 监听push_status节点的值变事件（NodeDataChanged）。
3. 当修改push_status节点值时，会通知所有监听该节点的应用端。
4. 应用端收到NodeDataChanged事件，递归获取push_status节点下所有的节点数据，并更新配置缓存。
