---
title: Feign正确的使用方法和性能优化注意事项
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/P60724-115835.jpg'
date: 2016-09-06 19:22:47
categories:
	- 微服务
tags:
	- Feign
	- 微服务
	- Spring Cloud
keywords:
	- Feign
	- 微服务
	- Spring Cloud
description:
---

## Feign正确的使用方法和性能优化注意事项

### 1. feign自定义Configuration和root 容器有效隔离。

- 用@Configuration注解
- 不能在主@ComponentScan (or @SpringBootApplication)范围内，从其包名上分离
- 注意避免包扫描重叠，最好的方法是明确的指定包名

### 2. Spring Cloud Netflix 提供了默认的Bean类型:

* Decoder feignDecoder: ResponseEntityDecoder (which wraps a SpringDecoder)
* Encoder feignEncoder: SpringEncoder
* Logger feignLogger: Slf4jLogger
* Contract feignContract: SpringMvcContract
* Feign.Builder feignBuilder: HystrixFeign.Builder

### 3. Spring Cloud Netflix没有提供默认值，但仍然可以在feign上下文配置中创建：

* Logger.Level
* Retryer
* ErrorDecoder
* Request.Options
* Collection<RequestInterceptor>

### 4. 自定义feign的消息编码解码器：
	
不要在如下代码中getObject方法内new 对象，外部会频繁调用getObject方法。
	
```java
	ObjectFactory<HttpMessageConverters> messageConvertersObjectFactory = new ObjectFactory<HttpMessageConverters>() {
	@Override
	public HttpMessageConverters getObject() throws BeansException {
		return httpMessageConverters;
	}
	};
```

### 5. 注意测试环境和生产环境，注意正确使用feign日志级别。

### 6. apacheHttpclient或者其他client的正确配置：
	
- apacheHttpclient自定义配置放在spring root context，不要在FeignContext，否则不会起作用。
- apacheHttpclient 连接池配置合理地连接和其他参数

### 7. Feign配置

```properties
#Hystrix支持，如果为true，hystrix库必须在classpath中
feign.hystrix.enabled=false
 
#请求和响应GZIP压缩支持
feign.compression.request.enabled=true
feign.compression.response.enabled=true
#支持压缩的mime types
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048

# 日志支持
logging.level.project.user.UserClient: DEBUG
	

```

### 8. Logger.Level支持

必须为每一个Feign Client配置来告诉Feign如何输出日志，可选：
	
* **NONE**, No logging (**DEFAULT**).
* **BASIC**,  Log only the request method and URL and the response status code and execution time.
* **HEADERS**, Log the basic information along with request and response headers.
* **FULL**, Log the headers, body, and metadata for both requests and responses.

### 9. FeignClient.fallback 正确的使用方法

 配置的fallback class也必须在FeignClient Configuration中实例化，否则会报
` java.lang.IllegalStateException: No fallback instance of type class `异常。

例子：

```java
	@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
	public interface HystrixClient {
	    @RequestMapping(method = RequestMethod.GET, value = "/hello")
	    Hello iFailSometimes();
	}
	
	public class HystrixClientFallback implements HystrixClient {
	    @Override
	    public Hello iFailSometimes() {
	        return new Hello("fallback");
	    }
	}
	
	@Configuration
	public class FooConfiguration {
	    @Bean
		@Scope("prototype")
		public Feign.Builder feignBuilder() {
			return Feign.builder();
		}
		
		@Bean
		public HystrixClientFallback fb(){
			return new HystrixClientFallback();
		}
		
	}
	
```

### 10. 使用Feign Client 和@RequestMapping时，注意事项
 
 
当前工程中有和Feign Client中一样的Endpoint时，Feign Client的类上不能用@RequestMapping注解否则，当前工程该endpoint http请求且使用accpet时会报404.
  
 
下面的例子：
 

**有一个 Controller**

```java
@RestController
@RequestMapping("/v1/card")
public class IndexApi {

    @PostMapping("balance")
    @ResponseBody
    public Info index() {
        Info.Builder builder = new Info.Builder();
        builder.withDetail("x", 2);
        builder.withDetail("y", 2);
        return builder.build();
    }
}

```

**有一个Feign Client**

```java
@FeignClient(
        name = "card",
        url = "http://localhost:7913",
        fallback = CardFeignClientFallback.class,
        configuration = FeignClientConfiguration.class
)
@RequestMapping(value = "/v1/card")
public interface CardFeignClient {

    @RequestMapping(value = "/balance", method = RequestMethod.POST, produces = MediaType.APPLICATION_JSON_VALUE)
    Info info();

}
```

if @RequestMapping is used on class, when invoke http /v1/card/balance, like this :

如果 @RequestMapping注解被用在FeignClient类上，当像如下代码请求/v1/card/balance时，注意有Accept header：

```yaml
Content-Type: application/json
Accept: application/json

POST http://localhost:7913/v1/card/balance
```


那么会返回 404。

**如果不包含Accept header时请求，则是OK：**

```
Content-Type:application/json
POST http://localhost:7913/v1/card/balance
```


**或者像下面不在Feign Client上使用@RequestMapping注解,请求也是ok，无论是否包含Accept:**

```java

@FeignClient(
        name = "card",
        url = "http://localhost:7913",
        fallback = CardFeignClientFallback.class,
        configuration = FeignClientConfiguration.class
)

public interface CardFeignClient {

    @RequestMapping(value = "/v1/card/balance", method = RequestMethod.POST, produces = MediaType.APPLICATION_JSON_VALUE)
    Info info();

}

```
 



 