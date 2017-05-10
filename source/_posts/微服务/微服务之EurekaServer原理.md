---
title: 微服务Eureka Server原理
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/xinglonghu_l.jpg?imageView2/1/w/800/h/600/q/75|watermark/2/text/Qnkg6ZOB5rGk/font/5a6L5L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/30/dy/30'
date: 2017-03-26 19:20:00
categories:
	- 微服务 
tags:
	- 微服务
	- Eureka
	- Spring Cloud
keywords: 微服务,Eureka Server, Spring Cloud
description:
---
 
 

Eureka的相关知识，在之前的《微服务之Eureka服务发现》中已经讲了很多，这里不再重复，本文主要通过Eureka Server源码和配置来阐述Eureka Server的工作原理。

Eureka提供了一系列REST的API，供Eureka Client来调用，实现服务注册，注销，心跳，状态更新等等操作，参考官网[EurekaREst操作](<https://github.com/Netflix/eureka/wiki/Eureka-REST-operations>)。


```
REST API <Jersey>
Response Cache <com.google.common.cache.LoadingCache>
InstanceRegistry <ConcurrentHashMap>

EvictionTimer<java.util.Timer>
CacheUpdateTask<java.util.Timer>

```

## REST API 

基于Jersey实现，主要以appId[appname]和instanceId为操作维度，内容可以是xml或者json。相关的实现可以在`com.netflix.eureka.resources`包中找到。基于appId和instanceId和各种操作组合的实现，可以认为是InstanceRegistry的操作入口。

## InstanceRegistry

Registry是Eureka Server的核心，服务发现就是围绕Registry来实现。以下提到的类都可以在`com.netflix.eureka.registry`包中找到。


整个Registry由4个接口组成：

* LeaseManager: register,cacel,renew,evict等基本操作
* LookupService: 这个抽象是要是client和server端共用，EurekaClient也继承了该接口，主要用来查找服务和服务实例。
* InstanceRegistry: 提供了实例相关的丰富的操作。
* PeerAwareInstanceRegistry: eureka server之间的注册信息复制

如下是类关系图：

![](<http://7xiovs.com1.z0.glb.clouddn.com/eureka_registry.png>)


## 几个重要的定时任务

### 心跳补偿任务EvictionTask

```
[com.netflix.eureka.registry.AbstractInstanceRegistry.EvictionTask]
```

主要是用来做心跳补偿，目的是用来取消或清理过期的注册信息，通常是eureka client在停止前未成功发送cacel请求。例如eureka client停止时网络不通了、eureka client进程奔溃了等等。

这个定时任务是通过`eureka.server.eviction-interval-timer-in-ms`参数来配置处理间隔，默认是60s。

补偿时间=当前时间-该任务最后执行时间-执行间隔
其中`该任务最后执行时间-执行间隔`主要是计算出实际执行时间的细微差别，也为后面的`补偿时间>0`埋下伏笔。

`补偿时间>0`或者`最后更新时间 + leaseDuration[默认是90s，客户端durationInSecs]+补偿时间<当前时间` 时会执行evict清理过期实例。

**补偿时间>0,** 即就是当前任务执行和上次执行的时间间隔大于配置的时间间隔，正常情况补偿时间应该很小。

**最后更新时间+leaseDuration+补偿时间<当前时间**，即就是最后一次成功心跳到当前时间的间隔比eureka client配置的间隔大。

其中**leaseDuration**: 在eureka client中通过`eureka.instance.lease-expiration-duration-in-seconds[leaseExpirationDurationInSeconds]`参数来配置，默认是90s。

下面是源代码：

**计算补偿时间：**

```java
long getCompensationTimeMs() {
            long currNanos = getCurrentTimeNano();
            long lastNanos = lastExecutionNanosRef.getAndSet(currNanos);
            if (lastNanos == 0l) {
                return 0l;
            }

            long elapsedMs = TimeUnit.NANOSECONDS.toMillis(currNanos - lastNanos);
            long compensationTime = elapsedMs - serverConfig.getEvictionIntervalTimerInMs();
            return compensationTime <= 0l ? 0l : compensationTime;
        }
```

**这个是补偿清理逻辑：**

```java
public void evict(long additionalLeaseMs) {
        logger.debug("Running the evict task");

        if (!isLeaseExpirationEnabled()) {
            logger.debug("DS: lease expiration is currently disabled.");
            return;
        }

        // We collect first all expired items, to evict them in random order. For large eviction sets,
        // if we do not that, we might wipe out whole apps before self preservation kicks in. By randomizing it,
        // the impact should be evenly distributed across all applications.
        List<Lease<InstanceInfo>> expiredLeases = new ArrayList<>();
        for (Entry<String, Map<String, Lease<InstanceInfo>>> groupEntry : registry.entrySet()) {
            Map<String, Lease<InstanceInfo>> leaseMap = groupEntry.getValue();
            if (leaseMap != null) {
                for (Entry<String, Lease<InstanceInfo>> leaseEntry : leaseMap.entrySet()) {
                    Lease<InstanceInfo> lease = leaseEntry.getValue();
                    if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
                        expiredLeases.add(lease);
                    }
                }
            }
        }

        // To compensate for GC pauses or drifting local time, we need to use current registry size as a base for
        // triggering self-preservation. Without that we would wipe out full registry.
        int registrySize = (int) getLocalRegistrySize();
        int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
        int evictionLimit = registrySize - registrySizeThreshold;

        int toEvict = Math.min(expiredLeases.size(), evictionLimit);
        if (toEvict > 0) {
            logger.info("Evicting {} items (expired={}, evictionLimit={})", toEvict, expiredLeases.size(), evictionLimit);

            Random random = new Random(System.currentTimeMillis());
            for (int i = 0; i < toEvict; i++) {
                // Pick a random item (Knuth shuffle algorithm)
                int next = i + random.nextInt(expiredLeases.size() - i);
                Collections.swap(expiredLeases, i, next);
                Lease<InstanceInfo> lease = expiredLeases.get(i);

                String appName = lease.getHolder().getAppName();
                String id = lease.getHolder().getId();
                EXPIRED.increment();
                logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
                internalCancel(appName, id, false);
            }
        }
    }
```



从上面的逻辑上看，配置的时间间隔有2个：

- eureka server端：`eviction-interval-timer-in-ms`
- eureka client端：`lease-expiration-duration-in-seconds`

这2个参数同时作用着清理逻辑，配置时就要注意，`eviction-interval-timer-in-ms`要比`lease-expiration-duration-in-seconds`配置的要小，产生的结果就完全不一样。



## ResponseCache
通过readOnlyCache和readWriteCache实现：

- readOnlyCache: java.util.concurrent.ConcurrentMap
- readWriteCache: com.google.common.cache.LoadingCache

readWriteCache会自动从registry中更新。

这个缓存只作用于获取整个注册表和app的实例注册信息。

### 缓存更新CacheUpdateTask

这个任务用来定期更新只读缓存,逻辑上比较简单，定期从读写缓存中取出K/V，比较是否一致，不一致更新。

更新间隔通过参数`eureka.server.response-cache-update-interval-ms`来配置，默认是30s。


### readWriteCache

过期时间通过参数`eureka.server.response-cache-auto-expiration-in-seconds`来配置，默认是180s。

## eureka Server几个重要的配置





eureka.server.enable-self-preservation

是否开启自我保护模式，默认为true。

在开启状态下，Eureka Server会保持

默认情况下，如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。

Eureka通过“自我保护模式”来解决这个问题——当Eureka Server节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

 
```
    #是否开启自我保护模式，默认为true。
    enableSelfPreservation: true
    #默认是85%
    renewal-percent-threshold: 0.85
    #默认是15分钟
    renewal-threshold-update-interval-ms: 15
    #缓存更新时间，默认30s
    response-cache-update-interval-ms: 10
    #缓存过期时间，默认180s
    response-cache-auto-expiration-in-seconds: 30
    # 实例过期清理时间间隔，默认60秒
    eviction-interval-timer-in-ms: 10

```

## eureka client几个重要的配置

eureka.client.registry-fetch-interval-seconds

表示eureka client间隔多久去拉取服务注册信息，默认为30秒，对于api-gateway，如果要迅速获取服务注册状态，可以缩小该值，比如5秒

eureka.instance.lease-expiration-duration-in-seconds

leaseExpirationDurationInSeconds，表示eureka server至上一次收到client的心跳之后，等待下一次心跳的超时时间，在这个时间内若没收到下一次心跳，则将移除该instance。

默认为90秒
如果该值太大，则很可能将流量转发过去的时候，该instance已经不存活了。
如果该值设置太小了，则instance则很可能因为临时的网络抖动而被摘除掉。
该值至少应该大于leaseRenewalIntervalInSeconds
eureka.instance.lease-renewal-interval-in-seconds

leaseRenewalIntervalInSeconds，表示eureka client发送心跳给server端的频率。如果在leaseExpirationDurationInSeconds后，server端没有收到client的心跳，则将摘除该instance。除此之外，如果该instance实现了HealthCheckCallback，并决定让自己unavailable的话，则该instance也不会接收到流量。







 



 