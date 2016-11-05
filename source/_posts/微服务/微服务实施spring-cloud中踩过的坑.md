---
title: 微服务实施spring-cloud中踩过的坑
thumbnail: 'images/m1.jpg'
date: 2016-09-08 19:22:47
categories:
	- 微服务
tags:
	- spring-cloud
	- 微服务
	- Spring Boot
keywords:
	- 微服务
	- Spring Cloud
	- Spring Boot
description:
---

![P61023-131735-01.jpg](http://upload-images.jianshu.io/upload_images/2519252-d2cc7d03dc185c51.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 注册IP问题

早期的Spring Cloud Eureka在注册获取网卡IP时，不能区分外网网卡和内网网卡，如果安装了虚拟机和docker也不能区分虚拟网卡，每次启动注册的IP都有可能不一样，如果要注册为外网网卡IP，那运行带宽就不够，这个bug应该说是比较严重的问题，因此重写了网卡IP获取的逻辑来解决，同时也反馈给了spring cloud团队，再后期的版本中添加了网卡接口排序和通过名称过滤的功能来得到解决。

## HealthCheck的问题

在一些极小概率的情况下，会导致Eureka Server 下线微服务实例，出现“Remote status from Eureka server is down”的问题，即便是重启微服务也无济于事，不过已经有码友在spring cloud 官方github贴出了解决方法的issue。

## Zookeeper版本带来的性能问题

现象是一个团队在实施微服务时，发现部署到服务器上的微服务，在没有任何请求时，仍然CPU占用20~30%；匪夷所思的是，同样的微服务包在本地开发机器、本地物理机、本地虚拟机运行并未出现。通过jvisualvm观察,如下图：

![](<http://7xiovs.com1.z0.glb.clouddn.com/zk_jvisualvm.png>)

可以看到和Zookeeper有关，同时我的另一个Demo微服务在任何环境下却未出现该问题。比较jar中依赖包之后发现，2个包中唯一不一样的是Apache Curator版本，一个是2.8.0, spring cloud默认依赖的版本；没问题的是Apache Curator 2.9.1。**Apache Curator 2.8.0 BUG！！！**在[Apache Curator 2.9.1 Release Notes](<https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314425&version= 12332392>也发现了对此bug的描述。升级到2.9.1问题解决，皆大欢喜。同时上报此bug给Spring Cloud团队，升级Apache Curator版本。

## Feign使用不当带来的性能问题

Feign在Spring容器中使用了独立应用上下文，要注意命名空间上隔离，详情参考：[**Feign正确的使用方法和性能优化注意事项**](<http://tietang.wang/2016/09/06/%E5%BE%AE%E6%9C%8D%E5%8A%A1/Feign%E4%BD%BF%E7%94%A8%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/>) |http://www.jianshu.com/p/191d45210d16来绕过坑。

这几个坑是不能容忍的，踩过其他小坑就不计其数了，但都不伤大雅，可以替代。
