## 注册中心Consul学习

### 1、consul简述

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

**Consul 的优势：**

- 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是Paxos, 而 etcd 使用的则是 Raft。
- 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。
- 支持健康检查。 etcd 不提供此功能。
- 支持 http 和 dns 协议接口。 zookeeper 的集成较为复杂, etcd 只支持 http 协议。
- 官方提供 web 管理界面, etcd 无此功能。
- 综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究。

**特性**

- 服务发现
- 健康检查
- Key/Value 存储
- 多数据中心



### 2、consul与Eureka的区别

### **（1）一致性**

**Consul强一致性（CP）**

- 服务注册相比Eureka会稍慢一些。因为Consul的raft协议要求必须过半数的节点都写入成功才认
  为注册成功
- Leader挂掉时，重新选举期间整个consul不可用。保证了强一致性但牺牲了可用性。

**Eureka保证高可用和最终一致性（AP）**

- 服务注册相对要快，因为不需要等注册信息replicate到其他节点，也不保证注册信息是否
  replicate成功

- 当数据出现不一致时，虽然A, B上的注册信息不完全相同，但每个Eureka节点依然能够正常对外提
  供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到。如此保证了可用性但牺
  牲了一致性。

### **（2）开发语言和使用**

- eureka就是个servlet程序，跑在servlet容器中
- Consul则是go编写而成，安装启动即可



### 3、consul的下载与安装

- Consul 不同于 Eureka 需要单独安装，访问https://www.consul.io/downloads.html下载 Consul 的最新版本，我这里是consul1.5x。根据不同的系统类型选择不同的安装包，从下图也可以看出 Consul 支持所有主流系统。

![image-20201012085604087](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012085604087.png)



**在linux虚拟中下载consul服务**

```cmd
#从官网下载最新版consul服务
wget https://releases.hashicorp.com/consul/1.5.3/consul_1.5.3_linux_amd64.zip
##使用unzip命令解压
unzip consul_1.5.3_linux_amd64.zip
##将解压好的consul可执行命令拷贝到/usr/local/bin目录下
cp consul /usr/local/bin
##测试一下
consul
```

**启动consul服务**

```cmd
##以开发者模式快速启动，-client指定客户端可以访问的ip地址
  [root@node01 ~]# consul agent -dev -client=0.0.0.0
     ==> Starting Consul agent...
   Version: 'v1.5.3'
   Node ID: '49ed9aa0-380b-3772-a0b6-b0c6ad561dc5'
   Node name: 'node01'
   Datacenter: 'dc1' (Segment: '<all>')
   Server: true (Bootstrap: false)
   Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
   Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
   Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false,
   Auto-Encrypt-TLS: false
```

- 启动成功之后访问： http://IP:8500 ，可以看到 Consul 的管理界面

![image-20201012090052511](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012090052511.png)

### 4、基于consul的服务注册

#### 4.1 创建Maven工程

![image-20201012111754946](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012111754946.png)

#### 4.2 修改每个微服务的pom.xml文件，导入以下坐标

```xml
<!--导入consul相关依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
<!--actuator用于心跳检查-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> 其中spring-cloud-starter-consul-discovery 是SpringCloud提供的对consul支持的相关依赖。
> spring-boot-starter-actuator 适用于完成心跳检测响应的相关依赖

#### 4.3 修改每个微服务的application.yml文件，进行consul服务发现的相关配置

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/shop?serverTimezone=UTC
    username: root
    password: root
  application:
    name: shop_service_order
  # consul配置
  cloud:
    consul:
      # 配置访问consulServer的访问地址
      host: localhost
      # 配置端口
      port: 8500
      # 服务注册与发现的配置
      discovery:
        # 是否注册
        register: true
        # 实例id
        instance-id: ${spring.application.name}:${server.port}
        # 服务实例名称
        service-name: ${spring.application.name}
        # 服务实例端口
        port: ${server.port}
        # 健康检查路径
        health-check-path: /actuator/health
        # 健康检查时间间隔
        health-check-interval: 15s
        # 开启IP地址注册
        prefer-ip-address: true
        # 实例的请求ip
        ip-address: ${spring.cloud.client.ip-address}
```

其中spring.cloud.consul 中添加consul的相关配置

- host：表示Consul的Server的请求地址

- port：表示Consul的Server的端口

- discovery：服务注册与发现的相关配置

  - instance-id ： 实例的唯一id（推荐必填），spring cloud官网文档的推荐，为了保证生成一
    个唯一的id ，也可以换成${spring.application.name}:${spring.cloud.client.ipAddress}
  - prefer-ip-address：开启ip地址注册
  - ip-address：当前微服务的请求ip

  

#### 4.4 在consulServer控制台进行查看

![image-20201012112906962](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012112906962.png)

> 可以看到三个微服务都已注册进入注册中心



# <!--注意-->

## Rabbion使用的细节：

## 1、服务调用

- 在RestTemplate生成方法之上添加@LoadBalanced注解，如下所示：

```java
/**
 * 向容器注入RestTemplate
 */
@LoadBalanced
@Bean
public RestTemplate getRestTemplate() {
    return new RestTemplate();
}
```

- 使用注册中心中所显示的serviceId，shop-service-product作为url的地址，如下所示，而不是使用在application.yml文件中所定义的application.name

```java
/**
 * 下订单
 */
@GetMapping("buy/{id}")
public String order(@PathVariable Integer id){
    Product product = restTemplate.getForObject("http://shop-service-product/product/" + id, Product.class);
    System.out.println(product);
    return "操作成功!!!";
}
```

![image-20201012142213317](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012142213317.png)

## 2、负载均衡

> Rabbion默认使用轮询的机制调用相同的微服务

### 2.1 负载均衡策略

Ribbon内置了多种负载均衡策略，内部负责复杂均衡的顶级接口为com.netflix.loadbalancer.IRule ，实现方式如下：

![image-20201012145634076](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012145634076.png)

- com.netflix.loadbalancer.RoundRobinRule ：以轮询的方式进行负载均衡。
- com.netflix.loadbalancer.RandomRule ：随机策略
- com.netflix.loadbalancer.RetryRule ：重试策略。
- com.netflix.loadbalancer.WeightedResponseTimeRule ：权重策略。会计算每个服务的权重，越高的被调用的可能性越大。
- com.netflix.loadbalancer.BestAvailableRule ：最佳策略。遍历所有的服务实例，过滤掉故障实例，并返回请求数最小的实例返回。
- com.netflix.loadbalancer.AvailabilityFilteringRule ：可用过滤策略。过滤掉故障和请求数超过阈值的服务实例，再从剩下的实力中轮询调用。

### 2.2 修改负载均衡策略

在服务消费者（shop_service_order）的application.yml配置文件中修改负载均衡策略，进行如下配置：

```yml
# 配置负载均衡策略(需要调用的微服务名称)
# 格式：serviceID.ribbon.NFLoadBalancerRuleClassName
shop-service-product:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

> 策略选择：
> 1、如果每个机器配置一样，则建议不修改策略 (推荐)
> 2、如果部分机器配置强，则可以改为 WeightedResponseTimeRule

### 2.3 rabbion重试测试

在服务消费者（shop_service_order）的application.yml配置文件中修改负载均衡策略，进行如下配置：

```yml
ribbon:
  ConnectTimeout: 250 # Ribbon的连接超时时间
  ReadTimeout: 1000 # Ribbon的数据读取超时时间
  OkToRetryOnAllOperations: true # 是否对所有操作都进行重试
  MaxAutoRetriesNextServer: 1 # 切换实例的重试次数
  MaxAutoRetries: 1 # 对当前实例的重试次数
```