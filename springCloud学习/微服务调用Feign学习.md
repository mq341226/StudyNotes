# 微服务调用Feign学习

## 1、之前学习使用RestTemplate实现REST API的调用，代码如下：

```JAVA
/**
 * 下订单
 */
@GetMapping("buy/{id}")
public Product order(@PathVariable Integer id){
    Product product = restTemplate.getForObject("http://shop-service-product/product/" + id, Product.class);
    return product;
}
```

> 由代码可知，我们是使用拼接字符串的方式构造URL的，该URL只有一个参数。但是，在现实中，URL中往往含有多个参数。这时候我们如果还用这种方式构造URL，那么就会非常痛苦。那应该如何解决？



## 2、Feign简介

Feign是Netflix开发的声明式，模板化的HTTP客户端，其灵感来自Retrofit,JAXRS-2.0以及WebSocket.Feign可帮助我们更加便捷，优雅的调用HTTP API。

- 在SpringCloud中，使用Feign非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了。
- Feign支持多种注解，例如Feign自带的注解或者JAX-RS注解等。
- SpringCloud对Feign进行了增强，使Feign支持了SpringMVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。



## 3、使用Feign进行服务调用

### 3.1 导入依赖

- 在服务消费者所在的模块的pom.xml文件中导入以下依赖：

```xml
<!--导入Feign远程调用微服务依赖包-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 3.2 在启动类中添加Feign注解，开启对Feign的支持

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

    /**
     * 向容器注入RestTemplate
     */
    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

> 通过@EnableFeignClients开启Spring Cloud对Feign的支持

### 3.3 创建FeignClient接口

- 在服务消费方创建FeignClient接口，此接口是在Feign中调用微服务的核心接口。

```java
/**
 * Feign接口，用于调用微服务
 */
//指定需要调用的微服务名称
@FeignClient(name = "shop-service-product")
public interface ProductFeignClient {
    //调用的请求路径
    @GetMapping("/product/{id}")
    Product findById(@PathVariable("id") Integer id);
}
```

> 定义各参数绑定时，@PathVariable、@RequestParam、@RequestHeader等可以指定参数属性，在Feign中绑定参数必须通过value属性来指明具体的参数名，不然会抛出异常
>
> @FeignClient：注解通过name指定需要调用的微服务的名称，用于创建Ribbon的负载均衡器。所以Ribbon会把shop-service-product 解析为注册中心的服务。

### 3.4 使用FeignClient替换RestTemplate

- 修改服务调用方的控制类，添加ProductFeginClient的自动注入，并在order方法中使用ProductFeginClient 完成微服务调用

```java
/*注入ProductFeignClient*/
@Autowired
private ProductFeignClient productFeignClient;
```

```java
/**
 * 使用FeignClient进行微服务的调用
 */
@GetMapping("buy/{id}")
public Product order(@PathVariable Integer id){
    Product product = productFeignClient.findById(id);
    return product;
}
```

3.5 进行测试

开启注册中心和相关微服务，在浏览器中或者Postman测试工具中输入服务消费方访问地址，效果如下：

![image-20201013092737827](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013092737827.png)

## 4、Feign和Ribbon的联系和区别

- Ribbon是一个基于 HTTP 和 TCP 客户端 的负载均衡的工具。它可以 在客户端 配置RibbonServerList（服务端列表），使用 HttpClient 或 RestTemplate 模拟http请求，步骤相当繁琐。

- Feign 是在 Ribbon的基础上进行了一次改进，是一个使用起来更加方便的 HTTP 客户端。采用接口的方式， 只需要创建一个接口，然后在上面添加注解即可 ，将需要调用的其他服务的方法定义成抽象方法即可， 不需要自己构建http请求。然后就像是调用自身工程的方法调用，而感觉不到是调用远程方法，使得编写客户端变得非常容易。

## 5、负载均衡

- Feign中本身已经集成了Ribbon依赖和自动配置，因此我们不需要额外引入依赖，也不需要再注册RestTemplate 对象。另外，我们可以像consul学习中讲的那样去配置Ribbon，可以通过ribbon.xx 来进行全局配置。也可以通过服务名.ribbon.xx 来对指定服务配置，在服务消费方所在的模块中，修改其application.yml配置文件，进行如下配置：

```yml
# 配置负载均衡策略(需要调用的微服务名称)
# 格式：serviceID.ribbon.NFLoadBalancerRuleClassName
shop-service-product:
  ribbon:
#    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 随机进行访问
    ConnectTimeout: 250 # Ribbon的连接超时时间
    ReadTimeout: 1000 # Ribbon的数据读取超时时间
    OkToRetryOnAllOperations: true # 是否对所有操作都进行重试
    MaxAutoRetriesNextServer: 1 # 切换实例的重试次数
    MaxAutoRetries: 1 # 对当前实例的重试次数
```

- 启动两个shop_service_product ，重新测试可以发现使用Ribbon的轮询策略进行负载均衡:

![image-20201013093917243](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013093917243.png)

## 6、服务调用Feign高级

### 6.1 Feign的配置

从Spring Cloud Edgware开始，Feign支持使用属性自定义Feign。对于一个指定名称的FeignClient（例如该Feign Client的名称为feignName ），Feign支持如下配置项：

```yml
feign:
  client:
    config:
      feignName: ##定义FeginClient的名称
      connectTimeout: 5000 # 相当于Request.Options
      readTimeout: 5000 # 相当于Request.Options
      # 配置Feign的日志级别，相当于代码配置方式中的Logger
      loggerLevel: full
      # Feign的错误解码器，相当于代码配置方式中的ErrorDecoder
      errorDecoder: com.example.SimpleErrorDecoder
      # 配置重试，相当于代码配置方式中的Retryer
      retryer: com.example.SimpleRetryer
      # 配置拦截器，相当于代码配置方式中的RequestInterceptor
      requestInterceptors:
        - com.example.FooRequestInterceptor
        - com.example.BarRequestInterceptor
      decode404: false
```

- feignName：FeginClient的名称
- connectTimeout ： 建立链接的超时时长
- readTimeout ： 读取超时时长
- loggerLevel: Fegin的日志级别
- errorDecoder ：Feign的错误解码器
- retryer ： 配置重试
- requestInterceptors ： 添加请求拦截器
- decode404 ： 配置熔断不处理404异常

### 6.2 请求压缩

Spring Cloud Feign 支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。通过下面的参数即可开启请求与响应的压缩功能：

```YML
feign:
  compression:
    request:
      enabled: true # 开启请求压缩
    response:
      enabled: true # 开启响应压缩
```

同时，我们也可以对请求的数据类型，以及触发压缩的大小下限进行设置：

```YML
feign:
  compression:
    request:
      enabled: true # 开启请求压缩
      mime-types: text/html,application/xml,application/json # 设置压缩的数据类型
      min-request-size: 2048 # 设置触发压缩的大小下限
```

> 注：上面的数据类型、压缩大小下限均为默认值。

### 6.3 日志级别

在开发或者运行阶段往往希望看到Feign请求过程的日志记录，默认情况下Feign的日志是没有开启的。
要想用属性配置方式来达到日志效果，只需在application.yml 中添加如下内容即可：

```yml
feign:
  client:
    config:
      shop-service-product:
      loggerLevel: FULL
logging:
  level:
    COM.MQ.order.fegin.ProductFeginClient: debug
```

- logging.level.xx : debug : Feign日志只会对日志级别为debug的做出响应

- feign.client.config.shop-service-product.loggerLevel : 配置Feign的日志Feign有四种

  日志级别：

  - NONE【性能最佳，适用于生产】：不记录任何日志（默认值）
  - BASIC【适用于生产环境追踪问题】：仅记录请求方法、URL、响应状态代码以及执行时间
  - HEADERS：记录BASIC级别的基础上，记录请求和响应的header。
  - FULL【比较适用于开发及测试环境定位问题】：记录请求和响应的header、body和元数
    据。



# 7、服务注册与发现总结

## 7.1 组件的使用方式

### 7.1.1 注册中心

#### （1）Eureka

- 搭建注册中心

  - 创建EurekaServer服务模块

  - 在pom.xml文件中引入依赖spring-cloud-starter-netflix-eureka-server
  - 在application.yml文件中配置EurekaServer模块
  - 在启动类中通过@EnableEurekaServer 激活Eureka Server端配置

- 服务注册
  - 服务提供者引入spring-cloud-starter-netflix-eureka-client 依赖
  - 在application.yml文件中通过eureka.client.serviceUrl.defaultZone 配置注册中心地址

#### （2）consul

- 搭建注册中心
  - 下载安装consul
  - 启动consul consul agent -dev

- 服务注册
  - 服务提供者引入spring-cloud-starter-consul-discovery 依赖
  - 在application.yml文件中通过spring.cloud.consul.host 和spring.cloud.consul.port 指定Consul Server的请求地址

### 7.1.2 服务调用

#### （1）Ribbon

- 通过Ribbon结合RestTemplate方式进行服务调用只需要在声明RestTemplate的方法上添加注解@LoadBalanced即可
- 可以在application.yml文件中通过{服务名称}.ribbon.NFLoadBalancerRuleClassName 配置负载均衡策略

#### （2）Feign

- 服务消费者引入spring-cloud-starter-openfeign 依赖
- 通过@FeignClient 声明一个调用远程微服务接口
- 启动类上通过@EnableFeignClients 激活Feign