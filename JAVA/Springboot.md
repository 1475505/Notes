

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

