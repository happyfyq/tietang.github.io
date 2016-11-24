---
title: 微服务之Eureka服务发现
thumbnail: 'images/m1.jpg'
date: 2016-09-08 19:22:47
categories:
	- 微服务
tags:
	- spring-cloud
	- 微服务
	- Spring Boot
	- Eureka
	- 服务发现
keywords:
	- 微服务
	- Spring Cloud
	- Spring Boot
	- Eureka
	- 服务发现
description:
---



![](http://upload-images.jianshu.io/upload_images/2519252-3691e262041c9cdd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

当调用API或者发起网络通信的时候，无论如何我们都要知道被调用方的IP和服务端口，大部分情况是通过域名和服务端口，事实上基于DNS的服务发现，因为DNS缓存、无法自治和其他不利因素的存在，有很多局限。传统的DNS方式，都是通过nginx或者其他代理软件来实现，物理机器的ip和port都是固定的，那么nginx中配置的服务ip和port也是固定的，服务列表的更新只能通过手动来做，但如果后端服务很多时，手动更新容易出错，效率也很低，这在后端服务发生故障时，不可用时间就可能会加长。在微服务中，尤其是使用了Docker等虚拟化技术的微服务，其IP和port都是动态分配的，服务实例数也是动态变化的，那么就需要精细而准确的服务发现机制。当微服务app启动后，告诉其他服务自己的ip和端口，这里的其他服务就是Eureka Server和Eureka Client，这样其他服务就知道这个服务有多少实例在线，都在哪些地方，方便去负载均衡和调用。

Eureka属于客户端发现模式，客户端负责决定相应服务实例的网络位置，并且对请求实现负载均衡。客户端从一个服务注册服务中查询所有可用服务实例的库，并缓存到本地。服务调用时，客户端使用负载均衡算法从多个后端服务实例中选择出一个，然后发出请求。Eureka分为Eureka Server和Eureka client， Eureka Server是一个服务注册中心，为服务实例注册管理和查询可用实例提供了REST API，并可以用其定位、负载均衡、故障恢复后端服务的中间层服务。在服务启动后，Eureka Client向服务注册中心注册服务同时会拉去注册中心注册表副本；在服务停止的时候，Eureka Client向服务注册中心注销服务；服务注册后，Eureka Client会定时的发送心跳来刷新服务的最新状态。

客户端发现模式的优点是服务调用、负载均衡不需要和Eureka Server通信，直接使用本地注册表副本，因此Eureka Server不可用时是不会影响正常的服务调用，性能也不会因为网络延迟和服务端延迟受到影响。但其缺点也很明显，但某个服务不可用时，各个Eureka Client不能及时的知道，需要1~3个心跳周期才能感知，但是，由于基于Netflix的服务调用端都会使用Hystrix来容错和降级，当服务调用不可用时Hystrix也能及时感知到，通过熔断机制来降级服务调用，因此弥补了基于客户端服务发现的时效性的缺点。

Eureka Server采用的是对等通信(P2P),无中心化的架构，无master/slave区分，每一个server都是对等的，既是Server又是Client,所以其集群方式可以自由发挥，可以各点互连，也可以接力互连。Eureka Server通过运行多个实例以及彼此之间互相注册来提高可用性，每个节点需要添加一个或多个有效的serviceUrl指向另一个节点。利用Eureka Server这种架构特性， 我在Eureka Server Cluster的部署时采用了三角形通信模型，三角形是一个很好的均衡模型，既是各点互连，又是接力互连，三角形本身就是一个稳定性几何形状，有着稳固、坚定搜索、耐压的特点，家具、建筑、交通等各种行业都有应用。如下图所示，Eureka Cluster的每个实例都和另外2个实例通信交互。
![](<http://7xiovs.com1.z0.glb.clouddn.com/eureka_cluster.png>)


