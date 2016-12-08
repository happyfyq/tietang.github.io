---
title: Hystrix semaphore和thread隔离策略的区别及配置参考
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/hystrix-hystrix-logo.png'
date: 2016-11-18 19:22:47
categories:
	- 技术
	- Hystrix
tags:
	- spring cloud
	- Hystrix
	- semaphore
keywords:
	- 技术
	- Hystrix
	- semaphore
description:
---



# Hystrix semaphore和thread隔离策略的区别及配置参考

## 通用设置说明

Hystrix所有的配置都是`hystrix.command.[HystrixCommandKey]`开头,其中`[HystrixCommandKey]`是可变的，默认是`default`,即`hystrix.command.default`；另外Hystrix内置了默认参数，如果没有配置Hystrix属性，默认参数就会被设置，其优先级：

- hystrix.command.[HystrixCommandKey].`XXX`
- hystrix.command.default.`XXX`
- Hystrix代码内置属性参数值


## Hystrix隔离策略相关的参数

### 策略参数设置

> execution.isolation.strategy= THREAD|SEMAPHORE

### execution.isolation.thread.timeoutInMilliseconds

`hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds `用来设置thread和semaphore两种隔离策略的超时时间，默认值是1000。


- 建议设置这个参数，在Hystrix 1.4.0之前，semaphore-isolated隔离策略是不能超时的，从1.4.0开始semaphore-isolated也支持超时时间了。
- 建议通过CommandKey设置不同微服务的超时时间,对于zuul而言，CommandKey就是service id：`hystrix.command.[CommandKey].execution.isolation.thread.timeoutInMilliseconds`

这个超时时间要根据`CommandKey`所对应的业务和服务器所能承受的负载来设置，要根据`CommandKey`业务的平均响应时间设置，一般是大于平均响应时间的`20%~100%`,最好是根据压力测试结果来评估，这个值设置太大，会导致线程不够用而会导致太多的任务被fallback；设置太小，一些特殊的慢业务失败率提升，甚至会造成这个业务一直无法成功，在重试机制存在的情况下，反而会加重后端服务压力。

### execution.isolation.semaphore.maxConcurrentRequests

这个值并非`TPS`、`QPS`、`RPS`等都是相对值，指的是1秒时间窗口内的事务/查询/请求，`semaphore.maxConcurrentRequests`是一个绝对值，无时间窗口，相当于亚毫秒级的，指任意时间点允许的并发数。当请求达到或超过该设置值后，其其余就会被拒绝。默认值是100。

 

### execution.timeout.enabled

是否开启超时，默认是true，开启。


### execution.isolation.thread.interruptOnTimeout

发生超时是是否中断线程，默认是true。

### execution.isolation.thread.interruptOnCancel

取消时是否中断线程，默认是false。






 
 