#  ,注册中心Eureka学习

<!--注意：springCloud和springBoot版本要匹配-->

The table below outlines which version of Spring Cloud maps to which version of Spring Boot.

| Release Train                                                | Boot Version                     |
| :----------------------------------------------------------- | :------------------------------- |
| [Hoxton](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Hoxton-Release-Notes) | 2.2.x, 2.3.x (Starting with SR5) |
| [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes) | 2.1.x                            |
| [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes) | 2.0.x                            |
| [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes) | 1.5.x                            |
| [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes) | 1.5.x                            |

Spring Cloud Dalston, Edgware, and Finchley have all reached end of life status and are no longer supported.

```

```



## 1、搭建Eureka服务中心

### 1.1、创建maven工程

### 1.2、引入maven坐标

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### 1.3、配置application.yml文件

```yml
# 配置服务访问端口
server:
  port: 8003

# eureka服务器端控制
eureka:
  instance:
    hostname: localhost
  client:
    # registerWithEureka: 是否将自己注册到Eureka服务中，本身就是所有无需注册
    register-with-eureka: false
    # fetchRegistry : 是否从Eureka中获取注册信息
    fetch-registry: false
    # serviceUrl: 客户端与Eureka服务端进行交互的地址
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 1.4、配置启动类

- 在com.mq.eureka目录下创建启动类EurekaServerApplication

```java
@SpringBootApplication
//EnableEurekaServer : 激活Eureka Server端配置
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }
}
```



## 2、服务注册中心管理后台

打开浏览器，输入网址http://loclahost:8003即可进入EurekaServer内置的管理控制台，显示效果如下：

![ ](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201010114511193.png)



## 3、服务注册到Eureka注册中心

### 3.1 商品服务注册

#### 3.1.1 商品模块引入坐标

- 在商品模块的pom.xml文件中引入eureka client相关坐标，如下所示

```xml
<!--导入eureka client依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
    <version>2.2.0.RELEASE</version>
    <scope>compile</scope>
</dependency>
```



#### 3.1.2 编写相应的application.xml

- 在工程的application.yml中添加Eureka Server的主机地址

```yml
# 添加Eureka Server的主机地址
eureka:
  client:
    # eureka server的访问路径
    service-url:
      defaultZona: http://localhost:8003/eureka/
  instance:
    # 使用ip进行注册
    prefer-ip-address: true
```

#### 3.1.3  修改启动类，增加服务注册注解

```java
/**
 * 启动类
 */
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class ProductApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductApplication.class,args);
    }
}
```

<!--从Spring Cloud Edgware版本开始， @EnableDiscoveryClient 或@EnableEurekaClient 可
省略。只需加上相关依赖，并进行相应配置，即可将微服务注册到服务发现组件上。-->



### <!--注意：-->

- 服务注册到服务中心：引入的maven坐标为：

```xml
<!--导入eureka client依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
    <scope>compile</scope>
</dependency>
```

- 而不是：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-eureka-client</artifactId>
</dependency>
```

### ****这是个细节，找了半天，一定要注意



### 3.2 订单服务注册

#### 3.2.1 订单模块中引入相关坐标

- 在订单模块的pom.xml文件中引入一下maven依赖坐标：

```xml
<!--添加eureka client依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
    <scope>compile</scope>
</dependency>
```



#### 3.2.2 修改订单模块的配置文件

在订单模块的application.yml文件中进行如下配置：

```yml
# 添加Eureka Server的主机地址
eureka:
  client:
    # eureka server的访问路径
    service-url:
      # 8003为eureka的服务端口号
      defaultZone: http://localhost:8003/eureka/
  instance:
    # 使用ip进行注册
    prefer-ip-address: true
```



#### 3.2.3 修改启动类，添加服务注册注解

```java
@SpringBootApplication
@EnableEurekaClient
//@EnableDiscoveryClient
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

    /**
     * 向容器注入RestTemplate
     */
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

> 从Spring Cloud Edgware版本开始， @EnableDiscoveryClient 或@EnableEurekaClient 可省略。只需加上相关依赖，并进行相应配置，即可将微服务注册到服务发现组件上。



### 3.3  用户服务注册

#### 3.3.1 用户模块中引入相关坐标

- 在用户模块的pom.xml文件中导入如下的maven依赖坐标：

```xml
<!--添加eureka client依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
    <scope>compile</scope>
</dependency>
```



#### 3.3.2 修改用户模块的配置文件

- 在用户模块的application.yml配置文件中进行如下配置：

```yml
# 添加Eureka Server的主机地址
eureka:
  client:
    # eureka server的访问地址
    service-url:
      # 8003为eureka server的服务端口号
      defaultZone: http://localhost:8003/eureka/
  instance:
    # 使用ip进行注册
    prefer-ip-address: true
```



#### 3.3.3 修改启动类，添加服务注册注解

```java
@SpringBootApplication
@EnableEurekaClient
//@EnableDiscoveryClient
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class,args);
    }
}
```

> 从Spring Cloud Edgware版本开始， @EnableDiscoveryClient 或@EnableEurekaClient 可省略。只需加上相关依赖，并进行相应配置，即可将微服务注册到服务发现组件上。



### 3.4 进行注册

> 启动Erueka Server模块和各个服务模块，访问http://localhost:8003即可访问Eureka Server的后台管理系统，可以看到三个服务模块都已注册进入注册中心。

![image-20201012080852181](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012080852181.png)



### 3.5 Eureka的自我保护机制

#### 3.5.1 自我保护机制的运行过程

- 微服务第一次注册成功之后，每30秒会发送一次心跳将服务的实例信息注册到注册中心。通知Eureka
  Server 该实例仍然存在。如果超过90秒没有发送更新，则服务器将从注册信息中将此服务移除。

- Eureka Server在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%，如果出现低于的情况
  （在单机调试的时候很容易满足，实际在生产环境上通常是由于网络不稳定导致），Eureka Server会
  将当前的实例注册信息保护起来，同时提示这个警告。保护模式主要用于一组客户端和Eureka Server
  之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的
  信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。
- 验证完自我保护机制开启后，并不会马上呈现到web上，而是默认需等待 5 分钟（可以通过
  eureka.server.wait-time-in-ms-when-sync-empty 配置），即 5 分钟后你会看到下面的提示

![image-20201012081601369](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012081601369.png)

- 关闭自我保护

  通过在application.yml中设置eureka.enableSelfPreservation=false 来关闭自我保护功能，如下图所示：

  ```yml
  # 关闭自我保护机制
  server:
    enable-self-preservation: false
  ```



### 3.6 Eureka中的元数据

Eureka的元数据有两种：标准元数据和自定义元数据。

- 标准元数据：主机名、IP地址、端口号、状态页和健康检查等信息，这些信息都会被发布在服务注册表中，用于服务之间的调用。

- 自定义元数据：可以使用eureka.instance.metadata-map配置，符合KEY/VALUE的存储格式。这些元数据可以在远程客户端中访问。

在程序中可以创建DiscoveryClient对象来获取指定微服务的所有元数据信息。如下图所示：

```java
/*注入DiscoveryClient*/
@Autowired
private DiscoveryClient discoveryClient;
```

```java
/**
     * 下订单
     */
@GetMapping("buy/{id}")
public String order(@PathVariable Integer id){
    /*根据微服务名称从注册中心获取相应的元数据信息*/
    List<ServiceInstance> shop_service_product = discoveryClient.getInstances("shop_service_product");
    /*硬编码获取需要调用的服务的URL*/
    //        Product product = restTemplate.getForObject("http://localhost:8000/product/"+id, Product.class);
    /*使用元数据获取主机名以及端口号，进行URL获取*/
    Product product = restTemplate.getForObject("http://" + shop_service_product.get(0).getHost() + ":" + shop_service_product.get(0).getPort() + "/product/" + id, Product.class);
    System.out.println(product);
    return "操作成功!!!";
}
```

> 启动Eureka Server注册中心，调用订单模块的控制层方法，即可成功调用商品模块的指定商品。
>
> 在浏览器中输入http://localhost:8001/order/buy/1，即可在控制台查看商品信息。
>
> Product(id=1, product_name=保温杯, status=1, price=25.60, product_desc=可以保温, caption=米家, inventory=26)



## 4、Eureka服务注册高级

### 4.1 Eureka Server高可用集群

- 在上一个章节，实现了单节点的Eureka Server的服务注册与服务发现功能。Eureka Client会定时连接Eureka Server，获取注册表中的信息并缓存到本地。微服务在消费远程API时总是使用本地缓存中的数据。因此一般来说，即使Eureka Server发生宕机，也不会影响到服务之间的调用。但如果EurekaServer宕机时，某些微服务也出现了不可用的情况，Eureka Server中的缓存若不被刷新，就可能会影响到微服务的调用，甚至影响到整个应用系统的高可用。

- 因此，在生成环境中，通常会部署一个高可用的Eureka Server集群。Eureka Server可以通过运行多个实例并相互注册的方式实现高可用部署，Eureka Server实例会彼此增量地同步信息，从而确保所有节点数据一致。事实上，节点之间相互注册是Eureka Server的默认行为。

![image-20201012082145061](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012082145061.png)

#### 4.1.1 搭建Eureka Server高可用集群

- 创建Eureka ServerA并对a模块的application.yml文件进行如下配置，配置完成进行启动：

```yml
# 搭建高可用的Eureka Server集群
# Eureka1:8003
# Eureka2:8004

# 1、指定引用名称
spring:
  application:
    name: show_eureka_serer

# 配置服务访问端口
server:
  port: 8003

# eureka服务器端控制
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://${eureka.instance.hostname}:8004/eureka/
      
  # 关闭自我保护机制
  server:
    enable-self-preservation: false
```

- 接下来创建lEureka ServerB，首先更改模块的application.yml文件进行如下配置，配置完成进行如下操作，并进行启动：

```yml
# 搭建高可用的Eureka Server集群
# Eureka1:8003
# Eureka2:8004

# 1、指定引用名称
spring:
  application:
    name: show_eureka_serer


# 配置服务访问端口
server:
  port: 8004


# eureka服务器端控制
eureka:
  instance:
    hostname: localhost
  client:

    service-url:
      defaultZone: http://${eureka.instance.hostname}:8003/eureka/
      
  # 关闭自我保护机制
  server:
    enable-self-preservation: false
```

![image-20201012082716487](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012082716487.png)

![image-20201012082915675](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012082915675.png)

![image-20201012083008403](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012083008403.png)

Eureka ServerA和Eureka ServerB都启动成功后，在浏览器中输入http://localhost:8003和http://localhost:8004分别打开两个注册中心管理后台，如下图所示：

![image-20201012083522801](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012083522801.png)

![image-20201012083544911](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012083544911.png)

> 可以看到，Erueka ServerA和Erueka ServerB分别存在对方的注册信息，至此Eureka Server高可用集群搭建完毕。

#### 4.1.2 将服务注册到Erueka Server集群中

如果需要将微服务注册到Eureka Server集群只需要修改yml配置文件即可，如下图所示：

```yml
# 添加Eureka Server的主机地址
eureka:
  client:
    # eureka server的访问路径
    service-url:
      # 8003和8004为eureka的服务端口号
      defaultZone: http://localhost:8003/eureka/,http://localhost:8004/eureka/
  instance:
    # 使用ip进行注册
    prefer-ip-address: true
```

> 修改defaultZone配置添加多个Eureka Server的地址



### 4.2 Eureka中常见问题

#### 4.2.1  服务注册缓慢

- 默认情况下，服务注册到Eureka Server的过程较慢。

- SpringCloud官方文档中给出了详细的原因：
  大致含义：服务的注册涉及到心跳，默认心跳间隔为30s。在实例、服务器、客户端都在本地缓存中具
  有相同的元数据之前，服务不可用于客户端发现（所以可能需要3次心跳）。
- 可以通过配置eureka.instance.leaseRenewalIntervalInSeconds (心跳频率)加快客户端连接到其他服务的过
  程。在生产中，最好坚持使用默认值，因为在服务器内部有一些计算，他们对续约做出假设。



#### 4.2.2 服务节点剔除问题

- 默认情况下，由于Eureka Server剔除失效服务间隔时间为90s且存在自我保护的机制。所以不能有效而
  迅速的剔除失效节点，这对开发或测试会造成困扰。解决方案如下：

- Eureka Server：
  配置关闭自我保护，设置剔除无效节点的时间间隔：

```yml
server:
  # 关闭自我保护机制
  enable-self-preservation: false
  # 剔除无效节点时间间隔
  eviction-interval-timer-in-ms: 4000
```

Eureka Client：
配置开启健康检查，并设置续约时间



#### 4.2.3 监控页面显示ip

在Eureka Server的管控台中，显示的服务实例名称默认情况下是微服务定义的名称和端口。为了更好
的对所有服务进行定位，微服务注册到Eureka Server的时候可以手动配置示例ID。配置方式如下

```yml
instance:
  # 使用ip进行注册
  prefer-ip-address: true
  # 发送心跳续约间隔，以便快速注册
  lease-renewal-interval-in-seconds: 5
  # eureka client发送心跳给server端后，续约到期时间（默认90s）
  lease-expiration-duration-in-seconds: 10
  # 配置服务ip地址
  instance-id: ${spring.cloud.client.ip-address}:${server.port}
```

![image-20201012083623171](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201012083623171.png)





