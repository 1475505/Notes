# SpringBoot in 项目

## Bean

Spring流行的一个重要原因是：处理了对象与对象之间耦合性。`Bean`是Spring中对对象实例的称呼，可以视Spring为对象管理层。
```java
public class BeanFactory {
    private Map<String, Bean> beanMap = new HashMap<>();
    public Bean getBean(String key){
      return beanMap.get(key) ;
    }
}
```
对于一个项目而言，一些对象是需要 Spring 来管理的，另外一些（例如项目中其它的类和依赖的 Jar 中的类）又不需要。所以我们通过各式各样的**注解**去标识哪些是需要成为 Spring Bean，例如 Component 注解等。
那怎么实例化为 Bean（也就是一个对象实例）呢？很明显，只能通过反射来做了。

有了创建，有了装配，一个 Bean 才能成为自己想要的样子。我们定义一个类为 Bean，如果再显式定义了构造器，那么这个 Bean 在构建时，会自动根据构造器参数定义寻找对应的 Bean，然后反射创建出这个 Bean。

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。

>`@Autowired`与Spring强耦合，优先按类型；`@Resource`是JDK提供的，优先按名称

Bean 具有以下几种作用域：

**singleton**：单例模式，在每个Spring IoC容器中，对应Bean将只有一个实例。（尽量）

**prototype**：原型模式，每次通过容器的getBean方法获取prototype定义的Bean时，都将产生一个新的Bean实例

**request**：对于每次HTTP请求，使用request定义的Bean都将产生一个新实例，即每次HTTP请求将会产生不同的Bean实例。

**session**：对于每次HTTP Session，使用session定义的Bean产生一个新实例。

**globalsession**：每个全局的HTTP Session，使用session定义的Bean都将产生一个新实例。


> AOP: 拦截器以Bean切面获取方法调用的信息，进行功能拓展。


## Bean 的注入

当 Spring Boot 启动时，ComponentScan 的启用意味着会根据指定的路径范围，扫描所有定义的 Bean。
![](http://img.070077.xyz/20221112152226.png)

-   Spring 内部有三级**缓存**，部分解决了循环依赖

```java
Map<String,Object> singletonObjects 
//一级缓存，用于保存实例化、注入、初始化完成的bean实例
Map<String,Object> earlySingletonObjects 
//二级缓存，用于保存实例化完成但[未初始化]的bean实例
Map<String,ObjectFactory<?> singletonFactories 
//三级缓存，用于保存bean[创建工厂]，以便于后面扩展有机会创建代理对象
```

![](http://img.070077.xyz/202204240052924.png)

## Bean 的生命周期

- 创建：实例化Bean对象，设置Bean属性.
  - Aware（注入beanID，Beanfactory和AppCtx可以在bean中获取到ioc容器）如果通过Aware接口声明了依赖关系，则会注入基础层面的依赖
  - postProcessBeforeInitialization（对实例化的bean添加一些自定义处理逻辑）
  - afterPropertiesSet（属性被设置之后自定义的事情）
  - Bean init方法
  - postProcessAfterInitialization初始化后方法
![](http://img.070077.xyz/202204240143059.png)

- 销毁
 - 若实现了DisposableBean接口，则会调用destroy()方法
 - 若配置了destroy-method属性，则会调用其配置的销毁方法

## MVC
![](http://img.070077.xyz/202204240144078.png)

# Spring项目介绍

## 常用注解和目录

-  Config层：放配置类
-  @ComponentScan：配置要交给Spring管理的类路径。如果启动类不在类路径下，需要加这个注解。
-  Controller层用于定义接口，是请求的入口
-  @RestController注解用于声明返回文本数据，一般返回JSON数据。@Controller注解用于声明返回界面。@RestController = @Controller + @ResponseBody
- 用@Value来读自定义配置项
```java
@GetMapping("/hello") 
== @RequestMapping(value="/hello",method=RequestMethod.GET)

@SpringBootApplication == @EnableAutoConfiguration + @ComponentScan
```
- Validation校验注解：`@NotNull(message = “该实体不能为空”)`

> 通过引入了`devtools`，实现热加载。

## 接口设计

后端会有很多的接口，为了让前端能够统一处理逻辑（登录校验、权限校验），需要统一后端的返回值。

>  [REST 成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)
>  - **级别 0** 
>  - 级别 0 的 API 的客户端通过向其唯一的 URL 端点发送 HTTP POST 请求来**调用** 该服务。每个请求被指定要执行的操作、操作的目标（如业务对象）以及参数。
>  - **级别 1** 
>  - 级别 1 的 API 支持**资源** 概念。要对资源执行修改，客户端会创建一个 POST 请求，指定要执行的操作和参数。
>  - **级别 2**
>  - 级别 2 的 API 使用 HTTP 动词（**谓词**）执行操作：使用 GET 检索、使用 POST 创建和使用 PUT 进行更新。请求查询参数和请求体（如果有）指定操作的参数。这使服务能够利用到 Web 的基础特性，如缓存 GET 请求。
>  - **级别 3**
>  - 级别 3 的 API 基于非常规命名原则设计，HATEOAS（Hypermedia as the engine of application state，超媒体即应用状态引擎）。基本思想是 GET 请求返回的资源的表述，包含用于执行该资源允许的操作的链接。例如，发送 GET 请求检索订单，返回的订单响应中*包含取消订单链接*，客户端可以用该链接来取消订单，由此不再需要将 URL 硬编码在客户端代码中。由于资源的表示包含可允许操作的链接，所以客户端不必猜测可以对当前状态的资源执行什么操作。

新建resp包，新建用于统一处理返回前端的实体响应类CommonResp。

-   使用过滤器记录接口耗时
-   使用拦截器记录接口耗时：配置Interceptor和web全局配置类
-   使用AOP记录接口耗时：配置切面类`aspect`

![](http://img.070077.xyz/202204240135212.png)


## 解决跨域

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer { //加配置类CoreConfig
    /**
     * 允许所有跨域请求
     * @param registry 注册器
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowedHeaders(CorsConfiguration.ALL)
                .allowedMethods(CorsConfiguration.ALL)
                // 允许前端带上凭证
                .allowCredentials(true)
                // 1小时内不需要再预检（发OPTIONS请求）
                .maxAge(3600);
    }

}
```

## 解决前后端交互Long类型精度丢失

```java
@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        objectMapper.registerModule(simpleModule);
        return objectMapper;
    }
}
```

## JWT

传统的`Session` 认证需要服务器内存维护所有用户的登录信息，开销大。

Json web token是有意义的、加密的、包含业务信息的。特性：无状态。由：`Header` `Payload` `Signature` 三个部分组成，分别描述：元数据（数据算法）、Json字段（如：用户名、登录时间）、数据签名（哈希校验）。

> Spring登录机制的实现

- 引入`jjwt`依赖。
- 拦截器。

## 30天趋势图的SQL语句？

```sql
select * from `statistic` 
where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(create_time);
//这个是查询30天前的数据
```

---


# 中间件：MQ

## Why MQ？

以秒杀系统为例：

- 异步处理：秒杀系统中，有一些操作（短信通知、统计数据更新）的优先级较低，便可通过消息队列延后执行，更快地返回结果，自然实现了步骤之间的并发，提升系统总体的性能。
- 流量控制：使用消息队列隔离网关和后端服务，达到“削峰填谷”的作用，而不是直接过载拒绝请求返回错误。后端服务从请求消息队列中获取请求，完成后续秒杀处理过程。
> 通过`token`桶队列，用池化思路可减小复杂度。
- 服务解耦：引入消息队列后，订单服务在订单变化时发送一条消息到消息队列的一个主题 Order 中，所有下游系统都订阅主题 Order，这样每个下游系统都可以获得一份实时完整的订单数据，而不是多次请求获取新的。

也要认识到，消息队列也有它自身的局限性，包括：系统响应延迟问题；增加了系统的复杂度；可能产生数据不一致的问题。

## How MQ?

RabbitMQ 是一个相当轻量级的消息队列，非常容易部署和使用，自称是世界上使用最广泛的开源消息队列。

RocketMQ 有着不错的性能，响应时延很有优势，稳定性和可靠性好。

## 消息模型

- 队列模型：基础的消息模型，但是只有一个出口，不便“解耦
>RabbitMQ：其设置`exchange`分发层，也是其特色，提供配置支持
![](http://img.070077.xyz/202204240009914.png)


- 发布 - 订阅模型：一份消息数据能被消费多次。
![](http://img.070077.xyz/202204240007785.png)
RocketMQ：每个主题包含多个队列，通过多个队列来实现多实例并行生产和消费。
![](http://img.070077.xyz/202204240011684.png)


使用 RocketMQ 事务消息功能实现分布式事务的流程如下图：
![](http://img.070077.xyz/202204240013585.png)

## MQ的可靠性

利用消息队列的有序性可验证是否有消息丢失。在 Producer 端，我们给每个发出的消息附加一个连续递增的序号，然后在 Consumer 端来检查这个序号的连续性。可以利用框架的拦截器机制，在 Producer 发送消息之前的拦截器中将序号注入到消息中。

- 在生产阶段，可设置中间件消息发送的响应，若错误并重发消息。
- 在存储阶段，可以通过配置刷盘和复制相关的参数，让消息写入到多个副本的磁盘上，来确保消息不会因为某个 Broker（存储中间件） 宕机或者磁盘损坏而丢失。
- 在消费阶段，需要在处理完全部消费业务逻辑之后，再发送消费确认。

> 既然上面提到了“重传”，若消息重复呢？

- 一般解决重复消息的办法是，在消费端，让我们消费消息的操作具备幂等性。其任意多次执行所产生的影响均与一次执行的影响相同。利用数据库的唯一约束（INSERT IF NOT EXIST）、设置前置条件（版本号）、记录并检查操作等实现。

# MyBatis

MyBatis 是一款优秀的持久层框架，支持自定义 SQL、存储过程以及高级映射，免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis Generator是官方的代码生成器，可以为所有版本的MyBatis生成持久层代码。首先它会扫描一张数据库表（或者更多）然后生成一套访问表的框架。

## Springboot 集成

- 添加依赖
- 在`MyBatis`接口中，添加`@Mapper` 注解。
- 在`application.yml`中配置数据源。

 > #{}和${}的区别是什么？

-  `${}`是 properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如${driver}会被静态替换为`com.mysql.jdbc.Driver`。
- `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为? 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的? 号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`
