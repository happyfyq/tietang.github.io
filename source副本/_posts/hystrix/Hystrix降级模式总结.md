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

## 失败回退降级模式
失败回退需要实现HystrixCommand.getFallback方法或者HystrixObservableCommand. HystrixObservableCommand()方法。

* 快速失败Fail Fast
	* 如果业务异常，就抛出一个异常
* 静默失败Fail Silent
	* 失败时返回一个空response或者移除业务功能，例如返回null，空字符串，空map，空list等
* Fallback: Static
	* 失败时，返回默认值来替代引起失败的原因
* Fallback: Stubbed
	* 返回替代值，还没理解
* Fallback: Cache via Network
	* 当后端服务失败时，从网络缓存获取返回值
* Primary + Secondary with Fallback
	* 故障转移，当主服务失败时，调用从服务。当从服务也失败时结合其他模式
* Client Doesn’t Perform Network Access
	* 
* Get-Set-Get with Request Cache Invalidation