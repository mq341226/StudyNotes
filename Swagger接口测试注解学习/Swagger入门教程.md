# Swagger入门教程

由于Spring Boot能够快速开发、便捷部署等特性，相信有很大一部分Spring Boot的用户会用来构建RESTful API。而我们构建RESTful API的目的通常都是由于多终端的原因，这些终端会共用很多底层业务逻辑，因此我们会抽象出这样一层来同时服务于多个移动端或者Web前端。

这样一来，我们的RESTful API就有可能要面对多个开发人员或多个开发团队：IOS开发、Android开发或是Web开发等。为了减少与其他团队平时开发期间的频繁沟通成本，传统做法我们会创建一份RESTful API文档来记录所有接口细节，然而这样的做法有以下几个问题：

- 由于接口众多，并且细节复杂（需要考虑不同的HTTP请求类型、HTTP头部信息、HTTP请求内容等），高质量地创建这份文档本身就是件非常吃力的事，下游的抱怨声不绝于耳。
- 随着时间推移，不断修改接口实现的时候都必须同步修改接口文档，而文档与代码又处于两个不同的媒介，除非有严格的管理机制，不然很容易导致不一致现象。

为了解决上面这样的问题，本文将介绍RESTful API的重磅好伙伴Swagger2，它可以轻松的整合到Spring Boot中，并与Spring MVC程序配合组织出强大RESTful API文档。它既可以减少我们创建文档的工作量，同时说明内容又整合入实现代码中，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明。另外Swagger2也提供了强大的页面测试功能来调试每个RESTful API。



## 1、Spring Boot中使用Swagger2

### 1.1 创建SpringBoot工程

- 首先，我们需要一个Spring Boot实现的RESTful风格的API工程

![image-20201013114117058](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013114117058.png)



### 1.2 引入maven依赖

- 在该项目的pom.xml文件中引入swagger2的maven依赖，因为使用了consul注册中心进行微服务的注册，所以引入了Feign远程调用微服务的依赖（在此案例中可以省略不写），具体如下所示：

```xml
<!--导入Feign远程调用微服务依赖包-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--导入swagger相关依赖，生成接口文档-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 1.3 创建swagger2的配置类

- 在启动类同级目录下创建Swagger2的启动类Swagger2

![image-20201013133941021](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013133941021.png)

```java
@Configuration
@EnableSwagger2
public class Swagger2 {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.mq.order"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("测试one")
                .termsOfServiceUrl("http://baidu.com/") 
                .contact("mq")
                .version("1.0")
                .build();
    }
}
```

> 其中，使用@Configuration注解表示该类为配置类，让Spring加载该类配置，使用@EnableSwagger2注解来启用Swagger2

再通过`createRestApi`函数创建`Docket`的Bean之后，`apiInfo()`用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。

![image-20201013134545478](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013134545478.png)

`select()`函数返回一个`ApiSelectorBuilder`实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被`@ApiIgnore`指定的请求）。

- **添加文档内容**

在完成了上述配置后，其实已经可以生产文档内容，但是这样的文档主要针对请求本身，而描述主要来源于函数等命名产生，对用户并不友好，我们通常需要自己增加一些说明来丰富文档内容

**我们通过`@ApiOperation`注解来给API增加说明、通过`@ApiImplicitParams`、`@ApiImplicitParam`注解来给参数增加说明。**

```java
/**
 * 订单的控制类
 */
@RestController
@RequestMapping("/order")
public class OrderController {
    /*注入servie*/
    @Autowired
    private IOrderService orderService;

    /*注入RestTemplate*/
    @Autowired
    private RestTemplate restTemplate;

    /*注入discoveryClient*/
    @Autowired
    private DiscoveryClient discoveryClient;

    /*注入ProductFeignClient*/
    @Autowired
    private ProductFeignClient productFeignClient;


    /**
     * 查询所有方法
     */
    @ApiOperation(value = "获取订单列表",notes = "")
    @GetMapping
    public List<Order> findAll() {
        return orderService.findAll();
    }

    /**
     * 根据id进行查询
     */
    @ApiOperation(value = "查询订单",notes = "根据id进行查询")
    @ApiImplicitParam(name = "id",value = "用户id",required = true,dataType = "Integer")
    @GetMapping("/{id}")
    public Order findById(@PathVariable Integer id){
        return orderService.findById(id);
    }

    /**
     * 保存
     */
    @ApiOperation(value="保存订单",notes = "向数据库中插入一条订单信息")
    @ApiImplicitParam(name = "order",value = "新创建的订单实体类",required = true,dataType = "Order")
    @PostMapping
    public String save(@RequestBody Order order){
        orderService.save(order);
        return "保存成功";
    }

    /**
     * 更新
     */
    @ApiOperation(value="更新订单",notes = "对现有订单信息进行更新处理")
    @ApiImplicitParam(name = "order",value = "前端发送过来的order对象",required = true,dataType = "Order")
    @PutMapping
    public String update(@RequestBody Order order) {
        orderService.update(order);
        return "更新成功";
    }

    /**
     * 删除
     */
    @ApiOperation(value="删除订单",notes = "根据id删除订单")
    @ApiImplicitParam(name = "id",value = "数据库中的订单id")
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Integer id) {
        orderService.delete(id);
        return "删除成功";
    }

    /**
     * 下订单
     */
//    @GetMapping("buy/{id}")
//    public Product order(@PathVariable Integer id){
//        Product product = restTemplate.getForObject("http://shop-service-product/product/" + id, Product.class);
//        return product;
//    }

    /**
     * 使用FeignClient进行微服务的调用
     */
    @ApiOperation(value = "使用FeignClient进行微服务的调用",notes = "调用产品微服务的CURD功能")
    @ApiImplicitParam(name="id",value = "调用产品的id编号")
    @GetMapping("buy/{id}")
    public Product order(@PathVariable Integer id){
        Product product = productFeignClient.findById(id);
        return product;
    }
}
```

**完成上述代码添加上，启动Spring Boot程序，访问：[http://localhost:服务端口号/swagger-ui.html](http://localhost:服务端口号/swagger-ui.html)。就能看到RESTful API的页面。**

![image-20201013135200798](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013135200798.png)

**我们可以再点开具体的API请求，以POST类型的/order请求为例，可找到上述代码中我们配置的Notes信息以及参数order的描述信息，如下图所示。**

![image-20201013135304893](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013135304893.png)

- **API文档访问与调试**

**Swagger除了查看接口功能外，还提供了调试测试功能，我们可以看到Example Value（黑色区域：它指明了order的数据结构），此时Value中就有了order对象的模板，点击右上方的`“Try it out！”`按钮，会显示如下页面：**

![image-20201013135850412](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013135850412.png)

**我们只需要稍适修改，点击Excute，即可完成了一次请求调用！**

![image-20201013135948256](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013135948256.png)

![image-20201013140115448](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013140115448.png)

![image-20201013140148877](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013140148877.png)

**此时，你也可以通过GET请求来验证之前的POST请求是否正确。**

![image-20201013140416981](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201013140416981.png)

验证成功！！！

> #### 相比为这些接口编写文档的工作，我们增加的配置内容是非常少而且精简的，对于原有代码的侵入也在忍受范围之内。因此，在构建RESTful API的同时，加入swagger来对API文档进行管理，是个不错的选择。



## 2、Swagger常用注解使用详解

### 2.1 版本说明

1. 这里使用的版本：springfox-swagger2（2.9）springfox-swagger-ui （2.9） 
2. 这里是说明常用注解的含义和基本用法（也就是说已经对swagger进行集成完成） 
   没有集成的请参见 
   [SpringBoot集成springfox-swagger2构建restful API](http://blog.csdn.net/u014231523/article/details/54562695) 
   [SpringMVC集成springfox-swagger2构建restful API](http://blog.csdn.net/u014231523/article/details/54411026) 
   [官网WIKI](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X#quick-annotation-overview) 

### 2.2 常用注解

- @Api()用于类：表示标识这个类是swagger的资源 
- @ApiOperation()用于方法：表示一个http请求的操作 
- @ApiParam()用于方法，参数，字段说明：表示对参数的添加元数据（说明或是否必填等） 
- @ApiModel()用于类：表示对类进行说明，用于参数用实体类接收 
- @ApiModelProperty()用于方法，字段：表示对model属性的说明或者数据操作更改 
- @ApiIgnore()用于类，方法，方法参数：表示这个方法或者类被忽略 
- @ApiImplicitParam() 用于方法：表示单独的请求参数 
- @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam

### 2.3 常用注解的举例说明

具体使用举例说明： 

1. @Api() ：用于类；表示标识这个类是swagger的资源 

   参数说明：tags–表示说明 
   				   value–也是说明，可以使用tags替代 
   但是tags如果有多个值，会生成多个list

   ```java
   @Api(value = "order的控制类",tags = {"订单操作控制类"})
   @RestController
   @RequestMapping("/order")
   public class OrderController {}
   ```

2. @ApiOperation() 用于方法；表示一个http请求的操作 

   参数说明：value用于方法描述 
   				   notes用于提示内容 
   				   tags可以重新分组（视情况而用） 

   ```java
   /**
    * 根据id进行查询
    */
   @ApiOperation(value = "查询订单",notes = "根据id进行查询")
   @ApiImplicitParam(name = "id",value = "用户id",required = true,dataType = "String")
   @GetMapping("/{id}")
   public Order findById(@PathVariable Integer id){
       return orderService.findById(id);
   }
   ```

3. @ApiParam() 用于方法，参数，字段说明；表示对参数的添加元数据（说明或是否必填等） 

   参数说明：name–参数名 
   				   value–参数说明 
                      required–是否必填

4. @ApiModel()用于类 ；表示对类进行说明，用于参数用实体类接收 （就是用于实体类）

   参数说明：value–表示对象名 
   					description–描述 
   					都可省略 

5. @ApiModelProperty()用于方法，字段； 表示对model属性的说明或者数据操作更改(就是实体类中的成员变量属性的说明)

   参数说明：value–字段说明 
   					name–重写属性名字 
   					dataType–重写属性类型 
   					required–是否必填 
   					example–举例说明 
   					hidden–隐藏

6. @ApiIgnore()用于类或者方法上，可以不被swagger显示在页面上 
   比较简单, 这里不做举例

7. @ApiImplicitParam() 用于方法 ：表示单独的请求参数 

   ```java
   @ApiImplicitParam(name = "id",value = "用户id",required = true,dataType = "String")
   ```

8. @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam

    参数说明：name–参数ming 
   					value–参数说明 
   					dataType–数据类型 
   					paramType–参数类型 
   					example–举例说明

   

## 3、基于Spring框架的Swagger流程应用

- ##### 这里要讲的是如何结合现有的工具和功能，设计一个流程，去保证一个项目从开始开发到后面持续迭代的时候，以最小代价去维护代码、接口文档以及Swagger描述文件。

### 3.1 项目开始阶段

- 一般来说，接口文档都是由服务端来编写的。**在项目开发阶段的时候，服务端开发可以视情况来决定是直接编写服务端调用层代码，还是写Swagger描述文件。**建议是**如果项目启动阶段，就已经搭好了后台框架，那可以直接编写服务端被调用层的代码（即controller及其入参出参对象），然后通过Springfox-swagger生成swagger的json描述文件**。而**如果项目启动阶段并没有相关后台框架，而前端对接口文档追得紧，那就建议先编写swagger描述文件，通过该描述文件生成接口文档。**后续后台框架搭好了，也可以生成相关的服务端代码。

### 3.2 项目迭代阶段

- 到这个阶段，事情就简单很多了。后续后台人员，无需关注Swagger描述文件和接口文档，有需求变更导致接口变化，直接写代码就好了。把调用层的代码做个修改，然后生成新的描述文件和接口文档后，给到前端即可。真正做到了一劳永逸。

### 3.3流程

- 总结一下就是通过下面这两种流程中的一种，可以做到代码和接口文档的一致性，服务端开发再也不用花费精力去维护接口文档。

##### 流程一

![img](https:////upload-images.jianshu.io/upload_images/813533-1879ce94558a8553.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

##### 流程二

![img](https:////upload-images.jianshu.io/upload_images/813533-ffc262914e89f87d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



