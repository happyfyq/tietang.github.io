---
title: 微服务之API网关设计
thumbnail: 'images/m1.jpg'
date: 2016-09-08 19:22:47
categories:
	- 微服务
tags:
	- spring-cloud
	- 微服务
	- Spring Boot
	- API Gateway
	- 网关
keywords:
	- 微服务
	- Spring Cloud
	- Spring Boot
	- API Gateway
	- 网关
description:
---


![P60528-094513-01.jpeg](http://upload-images.jianshu.io/upload_images/2519252-3f8e955deefa1708.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

微服务除了互相之间调用，还需要将API提供给外部应用访问，像浏览器，移动app，第三合作方等等，这就需要前段路由来管理后端微服务提供的服务。提供类似功能的应用有着多样化的名称，比如前置服务器/前置机、路由服务器、(反向)代理服务器，API网关服务器也是其中的一个叫法，只是场景和侧重点不一样。

API网关，顾名思义，就是外部到内部的一道门，其主要功能：

- 服务路由：将前段应用的调用请求路由定位并负载均衡到具体的后端微服务实例，对于前端应用看起来就是1个应用提供的服务，微服务对于前段应用来说就是黑盒，前段应用也不需要关心内部如何分布，由哪个微服务提供。主要有静态路由和动态路由。
	- 静态路由：有时候需要通过域名或者其他固定方式提供和配置路由表
	- 动态路由：通过服务发现服务，动态调整后端微服务的运行实例和路由表，为路由和负载均衡提供动态变化的服务注册信息。
- 安全：统一集中的身份认证，安全控制。比如登录，签名，黑名单等等，还可以挖掘和开发更高级的安全策略。
- 弹性：限流和容错，也是另一个层面的安全防护，防止突发的流量或者高峰流量冲击后端微服务而导致其服务不可用，另一方面可以在高峰期通过容错和降级保证核心服务的运行。
- 监控：实时观察后端微服务的TPS、响应时间，失败数量等准确的信息。
- 日志：记录所有请求的访问日志数据，可以为日志分析和查询提供统一支持。
- 其他，当然还有很多需要统一集中管理的都可以在网关层解决。

在Spring Cloud Netflix中使用Zuul来作为API Gateway组件，并结合Undertow，可以满足大部分性能和网关功能需求了(更高要求可以参考：https://github.com/tietang/ngx-lua-zuul)，Zuul + Undertow一般性能延迟，包括JVM和网络延迟在10%~20%，JVM延迟可以控制到10ms以内。另一方面，Zuul有强大的可定制化，通过ZuulFilter可以定制开发更多的网关功能。如下图所示：在Spring Cloud技术上，Zuul集成了Hystrix，Ribbon，Eureka Client等强大的技术栈，提供了开箱即用的Spring Cloud微服务网关功能。Zuul的强大之处是自由定制，这样对于很多老项目微服务化后，就不能按照Spring Cloud默认的动态路由规则运行，因此在其基础上定制了一些路由规则功能，更好的适应老项目微服务化。

![API gateway](<http://7xiovs.com1.z0.glb.clouddn.com/API-Gateway.png>)

**定制的路由规则的主要功能:**


1. 路由表中包含源路径，微服务名称，目标路径。
2. Endpoint粒度配置支持。
3. 路由支持1对1精确路由。
4. 源路径可以`前缀/**`格式来模糊路由。
5. 目标路径可以使用`前缀/**`格式来装配目标路径。
6. 保留默认动态路由规则：`服务名称/**` --> 是否截去前缀 --> 目标路径。
7. 保留默认动态路由规则是否支持截去前缀的配置参数`stripPrefix`特性。
8. 路由规则可以在不重启服务动态更新，这个功能通过外化配置来支持。
9. 匹配股则采取谁先匹配路由谁，也就是说在路由表中有2个或以上的路由规则可能被匹配到时，匹配最先查询到的规则。

**路由规则格式采用properties格式：**

> 源路径 = 微服务名称, 目标路径

启动时读取配置并解析，放入路由表。请求时通过查询匹配到合适的路由转发。

例如：

```
/api/v1/trade=trade,/v1/trade
/api/customer/**=customer,/api/v1/**
/api/user/**=user

```
在上面的例子中：

- /api/v1/trade会精确的路由到trade微服务的/v1/trade；
- /api/customer/开头的api会路由转发到customer微服务的/api/v1/\*\*，其中后面的\*\*会被前面的\*\*部分替换，比如/api/customer/card->/api/v1/card的转换。
- /api/user/开头的api会路由转发到user微服务的/api/user/**，endppoint不变。


如果在Eureka Server中已经注册了微服务`payment`,那么在zuul启动后会自动添加路由规则，如果stripPrefix=false:

```
/payment/**=payment,/payment/**
```
如果stripPrefix=true:

```
/payment/**=payment,/payment/**
```

API网关通过部署多个实例来保证可用性，前端通过Nginx来负载均衡。
