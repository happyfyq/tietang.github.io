---
title: Hystrix简介
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/hystrix-hystrix-logo-tagline-640.png'
date: 2016-03-09 09:22:47
categories:
	- 技术
	- Hystrix
tags:
	- hystrix
keywords: hystrix
description: Hystrix简介
---

 
![](http://7xiovs.com1.z0.glb.clouddn.com/hystrix-hystrix-logo-tagline-640.png)

1. <a href="#what">What Is Hystrix?</a>
1. <a href="#purpose">What Is Hystrix For?</a>
1. <a href="#problem">What Problem Does Hystrix Solve?</a>
1. <a href="#principles">What Design Principles Underlie Hystrix?</a>
1. <a href="#how">How Does Hystrix Accomplish Its Goals?</a>

<a name="what" />
## Hystrix是什么?

在分布式环境中，不可避免的有许多服务依赖，而且有时候一些服务会失败。Hystrix library通过添加延迟容忍和容错逻辑来控制分布式服务之间的相互影响。Hystrix通过服务之间访问的隔离点阻止连锁故障，并提供了失败回退机制（fallback），来改进系统服务的弹性。

####  Hystrix的历史

Hystrix是在2011年由Netflix API 团队的弹性工程演变而来。在2012年，Hystrix日益完善和成熟，在Netflix的许多团队也开始采用。现在，在Netflix，每天有成千上万的线程隔离和数百亿的信号隔离被调用执行。这已经在可用性和弹性上产生了很大的改进。

 
下面的链接提供了围绕Hystrix和挑战，试图解决：

* [&ldquo;Making Netflix API More Resilient&rdquo;](http://techblog.netflix.com/2011/12/making-netflix-api-more-resilient.html)
* [&ldquo;Fault Tolerance in a High Volume, Distributed System&rdquo;](http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html)
* [&ldquo;Performance and Fault Tolerance for the Netflix API&rdquo;](https://speakerdeck.com/benjchristensen/performance-and-fault-tolerance-for-the-netflix-api-august-2012)
* [&ldquo;Application Resilience in a Service-oriented Architecture&rdquo;](http://programming.oreilly.com/2013/06/application-resilience-in-a-service-oriented-architecture.html)
* [&ldquo;Application Resilience Engineering & Operations at Netflix&rdquo;] (https://speakerdeck.com/benjchristensen/application-resilience-engineering-and-operations-at-netflix)

<a name="purpose" />
## Hystrix能做什么?
  
Hystrix被设计为：

- 保护通过第三方客户端库服务依赖访问（通常通过网络），控制其延迟和故障
- 在复杂的分布式系统中阻止连锁故障反应
- 快速失败和快速恢复
- Fallback降级和在可能的情况下优雅地降级
- 启用近实时监测，报警和操作控制


<a name="problem" />
## Hystrix解决了什么问题?

复杂分布式架构的应用程序有许多依赖，其中每一个在某些时候都会不可避免的发生失败。如果这个主应用没有从那些外部失败隔离，那么就会有被拖垮的风险。


例如，1个应用依赖30个服务，每个服务有99.99%可用，那么预期：

> 99.99<sup>30</sup>  =  99.7% uptime  
> 10亿请求中的0.3%  = 3,000,000 次失败  
> 即使所有依赖的服务都能达到 99.99% 的可用率，每月大约有2+小时不可用 
> 随着服务依赖数量的变多，服务不稳定的概率会成指数性提高.
 
**现实通常会更残酷。**

**如果你没有针对整个系统做快速恢复**，即使所有依赖只有 0.01% 的不可用率，累积起来每个月给系统带来的不可用时间也有数小时之多。

***

 

当一切都ok的请求流看起来是这样的：

![](http://7xiovs.com1.z0.glb.clouddn.com/hystrix-soa-1-1280.png)

当许多后端系统中的一个成为潜在故障时，可能会阻塞所有用户请求：

![](http://7xiovs.com1.z0.glb.clouddn.com/hystrix-soa-2-1280.png)


 
一个高并发后端依赖成为潜在危险时，它会在几秒钟中导致所有服务器上的所有的资源耗尽。
在网络中或者客户端库中运行的每一个app都有可能会导致成为潜在的故障源。比失败更坏的是，这个问题app也可能会导致服务之间的延迟增加，从而备份队列，线程，和其他系统资源，导致更多系统的级联故障。

![](http://7xiovs.com1.z0.glb.clouddn.com/hystrix-soa-3-1280.png)

 

当通过第三方客户端网络访问时，这些问题会加剧，这个第三方客户端就是一个黑盒，其实现细节是不清楚的，它能随时改变行为，每个客户端库的网络和资源配置是不同的，通常难以监控和修改。
如果这些网络请求通过第三方客户端发出，问题会变得更加严重，因为这些第三方客户端对于应用来说相当于『黑盒』——无法了解其实现细节，随时可能发生变更，网络/资源配置随客户端的不同而不同，同时又难以监控和修改。同时，应用依赖链中的服务可能非常耗时，或者这些可能导致问题的网络请求根本没有被我们的应用显式地调用！


网络连接可能失败或者降级。服务或者服务器可能失效或者变慢。新依赖的库或者部署的服务可能改变行为或性能，亦或是依赖的客户端库本身有bug。
所有以上这些所描述的失败和延迟都需要被隔离和管理，才不至于因为单个服务失败而导致整个应用活系统垮掉。



<a name="principles" />
## What Design Principles Underlie Hystrix?

Hystrix works by:

* 防止任何单一依赖耗尽容器（如Tomcat）内的所有用户线程
* 隔离和减低负载，对无法及时处理时快速失败，而不是排队
* 提供失败回退（fallback）降级，无论何时都尽可能保护使用者免受破坏。
* 采用隔离技术（如隔离壁，泳道，和熔断器模式）来限制任何一个依赖性的影响。
* 通过最近实时metrics、监控和警告来优化以满足近实时性的要求
* 在 Hystrix许多方面都需要只是配置属性的动态修改并能低延迟传播，提供优化以满足快速恢复的要求
* 能保护应用不受依赖服务的整个执行过程中失败的影响，而不仅仅是网络请求



<a name="how" />
## How Does Hystrix Accomplish Its Goals?

Hystrix does this by:

 
- 使用`HystrixCommand`或者`HystrixObservableCommand`包装所有的外部系统（或者依赖服务）调用，每个`HystrixCommand`或者`HystrixObservableCommand`在隔离的线程中/信号下执行（参考这个例子[command pattern](http://en.wikipedia.org/wiki/Command_pattern)）

- 超时机制，调用时间比定义的超时阀值大时超时。除了默认值，通过设置&ldquo;properties&rdquo;给每个依赖服务定义超时阀值,超时时间一般设为比99.5%平均时间略高即可.当调用超时时，直接返回或执行fallback逻辑。

- 为每个依赖维护一个小的线程池（或信号），如果线程池已满调用将被立即拒绝来代替排队方式。
- 测量成功，失败（抛出异常），超时和线程拒绝。
- 如果错误比率达到设定的阀值，通过手动或者自动方式平稳的停止一个时间周期调用所有请求。
- 当请求失败、拒绝，超时或者发生短路时，执行失败回退（fallback）逻辑。
- 近实时监控度量，动态配置修改
 
 

当使用 Hystrix 包装了你的所有依赖服务的请求后，下面图中所示的系统拓扑将会变成如下形式，每个依赖服务都被隔离开来，Hystrix 会严格控制其在延迟发生时对资源的占用，并在任何失效发生时，执行失败回退逻辑。

 

![](http://7xiovs.com1.z0.glb.clouddn.com/hystrix-soa-4-isolation-1280.png)

 



