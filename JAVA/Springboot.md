

# Springboot的启动

1. `@EnableAutoConfiguration` 
    开启自动装配，帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot，并创建对应配置类的Bean，并把该Bean实体交给IoC容器进行管理。
2. `@ComponentScan("com.packagename.controller")`
    根据定义的扫描路径，将符合规则的类加载到spring容器中，比如在类中加入了以下注解 @Controller、@Service、@Mapper 、@Component、@Configuration 等等；
3. `@SpringBootApplication` 
    标注这是一个springboot的应用，被标注的类是一个主程序，SpringApplication.run(App.class, args);传入的类App.class必须是被@SpringBootApplication标注的类。


## 框架启动流程
![](http://img.070077.xyz/20221112132057.png)



## Bean 的注入

当 Spring Boot 启动时，ComponentScan 的启用意味着会根据指定的路径范围，扫描所有定义的 Bean。
![](http://img.070077.xyz/20221112152226.png)


# 框架设计模式

在Spring Boot中，广泛使用了许多设计模式来帮助实现其核心功能。以下是其中一些被广泛使用的设计模式：

1. **单例模式**：在Spring中，bean默认是单例的。这意味着Spring容器中每个bean的默认实例只有一个，这样可以节省资源并提高性能。
2. **工厂模式**：Spring使用这个模式来创建bean。当你请求一个bean时，Spring会使用bean的定义来创建一个实例。
3. **代理模式**：Spring AOP（面向切面编程）就是基于代理模式的。例如，当你使用Spring的事务管理时，Spring会创建一个代理来围绕你的方法执行，以便可以在方法执行前后添加额外的行为。
4. **模板方法模式**：Spring JDBC和Spring Boot的JdbcTemplate都是使用了模板方法模式。这个模式定义了一个操作中的骨架，而将一些步骤延迟到子类中。例如，JdbcTemplate处理所有常见的JDBC连接问题，而你只需要提供SQL和可能的参数。
5. **观察者模式**：Spring事件处理就是使用的观察者模式。你可以发布事件，然后有多个监听器监听和响应这些事件。
6. **装饰器模式**：Spring中用于多层缓存的CacheManager就使用了装饰器模式。
7. **策略模式**：Spring Resource接口及其实现是策略模式的一种应用，该模式允许在运行时动态地更改应用的行为。
