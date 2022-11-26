前面是项目内容，现在是Spring框架文档，可能涉及英文。

# Spring IOC原理

Spring框架倡导基于POJO（Plain Old Java Object，简单Java对象）的轻量级开发理念。

## IOC

控制反转(Inverse Of Control)：依赖服务器管理依赖自身。
三种依赖注入的方式，构造方法注入（constructor injection）、setter方法注入（setter injection）以及接口注入（interface injection）。

（你简历没写IOP，TBD）

# Spring AOP 原理

## Spring AOP

Spring AOP采用*动态代理机制和字节码生成技术*实现。与最初的AspectJ采用编译器将横切逻辑织入目标对象不同，动态代理机制和字节码生成都是在运行期间为目标对象生成一个代理对象，而将横切逻辑织入到这个代理对象中，系统最终使用的是织入了**横切逻辑的代理对象**，而不是真正的目标对象。

术语：
- aspect:切面，切面有切点和通知组成，即包括横切逻辑的定义也包括连接点的定义
- pointcut:切点，每个类都拥有多个连接点，可以理解是连接点的集合
- joinpoint:连接点，程序执行的某个特定位置，如某个方法调用前后等
- weaving:织入，将增强添加到目标类的具体连接点的过程
- advice:通知，是织入到目标类连接点上的一段代码，就是增强到什么地方？增强什么内容？
- target:目标对象，通知织入的目标类
- Proxy:代理对象，即增强后产生的对象

# Springboot的三种启动方式

1. `@EnableAutoConfiguration` 
    开启自动装配，帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot，并创建对应配置类的Bean，并把该Bean实体交给IoC容器进行管理。
2. `@ComponentScan("com.packagename.controller")`
    根据定义的扫描路径，将符合规则的类加载到spring容器中，比如在类中加入了以下注解 @Controller、@Service、@Mapper 、@Component、@Configuration 等等；
3. `@SpringBootApplication` 
    标注这是一个springboot的应用，被标注的类是一个主程序，SpringApplication.run(App.class, args);传入的类App.class必须是被@SpringBootApplication标注的类。


# 框架启动流程
![](http://img.070077.xyz/20221112132057.png)



