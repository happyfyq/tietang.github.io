---
title: RestTemplate遇上Hystrix
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/P60828-123503.jpg'
date: 2016-09-02 09:20:00
categories:
	- 技术
	- Hystrix
tags:
	- hystrix
	- Circuit Breaker
keywords:
description:
---

# RestTemplate遇上Hystrix


## RestTemplate集成Hystrix和Robbin

查看RestTemplate源代码，可以看到RestTemplate继承了InterceptingHttpAccessor类，InterceptingHttpAccessor类通过ClientHttpRequestInterceptor接口提供了扩展功能。

实现intercept方法，在该方法中封装HystrixCommand和Ribbon逻辑即可。

下面的代码是集成了HystrixCommand的例子：

```java
@Override
    public ClientHttpResponse intercept(
            final HttpRequest request, final byte[] body,
            final ClientHttpRequestExecution execution) throws IOException {
        final URI originalUri = request.getURI();
        String serviceName = mapCommandKey(originalUri);

        log.info("{} :{} {} ", serviceName, request.getMethod().name(), originalUri.toString());
        return new RestTemplateHystrixCommnad(serviceName, () -> {
            return execution.execute(request, body);
        }, hystrixFallback).execute();

    }
```

下面是集成了HystrixCommand和Ribbon的例子

```java
@Override
    public ClientHttpResponse intercept(
            final HttpRequest request, final byte[] body,
            final ClientHttpRequestExecution execution) throws IOException {
        final URI originalUri = request.getURI();
        String serviceName = mapCommandKey(originalUri);

        log.info("{} :{} {} ", serviceName, request.getMethod().name(), originalUri.toString());
        return new RestTemplateHystrixCommnad(serviceName, () -> {
            return this.loadBalancer.execute(serviceName, instance -> {
                HttpRequest serviceRequest = new HystrixLoadBalancerInterceptor.ServiceRequestWrapper(
                        request,
                        instance);
                return execution.execute(serviceRequest, body);

            });
        }, hystrixFallback).execute();
    }
```

2）RestTemplate支持Hystrix异步特性

Hystrix的执行在线程隔离模型下是支持异步的，因此也扩展一个RestTemplate异步执行。如下代码所示，通过调用`queue()`方法返回一个Future。

```java
Future<String> fs = new CommandHelloWorld("World").queue();
String s = fs.get(); 
```
这样可以修改RestOperations接口方法为异步方法：

从：

```java
<T>  T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException;
```

到：

```java
<T> Future<T> getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException;
```

然后在原生的RestTemplate做一层代理，在代理层集成Hystrix和Ribbon，无论是JDK动态代理还是硬编码实现代理都变得容易，然后就可以这样来调用了：

```java
HystrixAsyncRestOperations asyncRestTemplate =null;
//先依次调用
Future<String> future1 = asyncRestTemplate.getForObject("http://tietang.wang/", String.class);
Future<String> future2 = asyncRestTemplate.getForObject("http://tietang.wang/2016/03/17/hystrix/%E6%80%8E%E6%A0%B7%E4%BD%BF%E7%94%A8Hystrix/", String.class);
//再依次获取调用结果
String html1 = future1.get();
String html2 = future2.get(100, TimeUnit.MILLISECONDS);//异步超时，建议
```
