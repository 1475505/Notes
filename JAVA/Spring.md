Spring框架文档，可能涉及英文。


# Bean

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

## 作用域

**singleton（默认）**：单例模式，在每个Spring IoC容器中，对应Bean将只有一个实例。（尽量）

**prototype**：原型模式，每次通过容器的getBean方法获取prototype定义的Bean时，都将产生一个新的Bean实例

**request**：对于每次HTTP请求，使用request定义的Bean都将产生一个新实例，即每次HTTP请求将会产生不同的Bean实例。

**session**：对于每次HTTP Session，使用session定义的Bean产生一个新实例。

**globalsession**：每个全局的HTTP Session，使用session定义的Bean都将产生一个新实例。

> AOP: 拦截器以Bean切面获取方法调用的信息，进行功能拓展。

## 三级缓存

-   Spring 内部有三级**缓存**，部分解决了循环依赖（只能处理单例作用域的bean的循环依赖，不能处理原型作用域的bean的循环依赖）

```java
Map<String,Object> singletonObjects 
//一级缓存，用于保存实例化、注入、初始化完成的bean实例。当一个bean完全创建好后，它会被放入这个缓存。
Map<String,Object> earlySingletonObjects 
//二级缓存，用于保存实例化完成但[未初始化]的bean实例。当检测到循环依赖时，Spring会将创建中的bean放入这个缓存。
Map<String,ObjectFactory<?> singletonFactories 
//三级缓存，用于保存bean[创建工厂]，以便于后面扩展有机会创建代理对象。当创建bean的过程中检测到循环依赖时，Spring会将创建bean的工厂对象放入这个缓存。
```

![](http://img.070077.xyz/202204240052924.png)


> 二级缓存是可以帮助处理循环依赖的，但是它不能单独解决这个问题。二级缓存（earlySingletonObjects）存储的是提前暴露出来的、尚未完全初始化的bean。当Spring检测到一个bean正在被创建，而这个bean又被其他bean依赖时，Spring会将这个尚未完全初始化的bean放入二级缓存，并提前暴露出来以供其他bean引用。这样可以解决一部分的循环依赖问题。例如，**如果两个bean互相依赖，并且都还没有开始创建，那么二级缓存就无法解决这个问题。**

## 生命周期

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

# Spring IOC原理

Spring框架倡导基于POJO（Plain Old Java Object，简单Java对象）的轻量级开发理念。

## IOC

控制反转(Inverse Of Control)：依赖服务器管理依赖自身。

Spring的IOC容器有两种主要的形式：

1.  BeanFactory：这是最简单的容器，提供了基本的DI支持。
2.  ApplicationContext：这是一个更完全的容器，提供了更多的企业级特性，例如事件发布、资源加载等。它是BeanFactory的子接口。

三种依赖注入（DI）的方式，构造方法注入（constructor injection）、setter方法注入（setter injection）以及接口注入（interface injection）。

1.  **构造器注入**：在这种方式中，依赖关系通过类的构造器参数来注入。Spring容器会在创建bean时，通过构造器参数将依赖的bean注入到目标bean中。这种方式的优点是，**它可以确保bean在创建完成后就已经有了所有必需的依赖，而且这些依赖是不可变的。** 但是，如果一个类有很多依赖，那么使用构造器注入可能会使得构造器的参数列表变得很长，这会影响到代码的可读性和可维护性。
2.  **Setter注入**：在这种方式中，依赖关系通过类的setter方法来注入。Spring容器会在创建bean后，通过调用setter方法将依赖的bean注入到目标bean中。这种方式的优点是，**它使得依赖的注入变得更加灵活，因为你可以在任何时候通过调用setter方法来改变依赖。** 但是，这也意味着bean的状态可能会在其生命周期中发生改变。
3. 接口注入的思想是，一个类通过实现一个或多个特定的接口（通常这些接口定义了一些setter方法）来表明它的依赖。然后，容器通过这些接口方法来注入依赖。这种方式的一个缺点是，它引入了额外的接口，这可能会使代码变得更复杂。

一般依赖关系通过注解来注入。你可以在类的字段、构造器或者方法上使用注解（如@Autowired、@Resource等）来声明依赖关系，Spring容器会自动识别这些注解，并注入相应的依赖。这种方式的优点是，它使得代码更加简洁，因为你不需要编写明确的构造器或者setter方法。但是，这也意味着你的代码会与Spring框架的注解产生耦合。


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


# MVC
![](http://img.070077.xyz/202204240144078.png)


# Spring事务

### 事务注解方式 @Transactional

-   标注在类前：标示类中所有方法都进行事务处理
-   标注在接口、实现类的方法前：标示方法进行事务处理

**Spring在底层数据源的基础上，利用ThreadLocal, SavePoint等技术点实现了多种事务传播属性，便于实现各种复杂的业务。** Spring团队的建议是你在具体的类(或类的方法)上使用@Transactional注解，而不要使用在类所要实现的任何接口上(注解是不能继承的)。

当然，以下是以Markdown表格形式展示的`@Transactional`注解的传播行为和隔离级别：

**传播行为（Propagation）**

| @Transactional<br/>(propagation=Propagation.    | Description |
|----------------|-------------|
| REQUIRED       | （必须事务）如果当前存在事务，就加入到当前事务中，如果当前不存在事务，就新建一个事务。这是默认的传播行为。 |
| SUPPORTS       | （随缘事务）如果当前存在事务，就加入到当前事务中，如果当前不存在事务，就以非事务方式执行。 |
| MANDATORY      | （强制已有事务）如果当前存在事务，就加入到当前事务中，如果当前不存在事务，就抛出异常。 |
| REQUIRES_NEW   | （强制新事务）无论当前是否存在事务，都新建一个事务。 |
| NOT_SUPPORTED  | （插队于事务）以非事务方式执行，如果当前存在事务，就将当前事务挂起。 |
| NEVER          | （强制无事务）以非事务方式执行，如果当前存在事务，就抛出异常。 |
| NESTED         | （嵌套事务）如果当前存在事务，就创建一个嵌套事务，如果当前不存在事务，就新建一个事务。 |
****
**隔离级别（Isolation）**

| @Transactional<br/>(isolation=Isolation.         | Description |
|-------------------|-------------|
| DEFAULT           | 使用后端数据库默认的隔离级别。 |
| READ_UNCOMMITTED  | 允许读取未提交的数据，可能会导致脏读、不可重复读和幻读。 |
| READ_COMMITTED    | 只允许读取已提交的数据，可以防止脏读，但可能会导致不可重复读和幻读。 |
| REPEATABLE_READ   | 在一个事务内，多次读取同一数据，结果都是相同的，可以防止脏读和不可重复读，但可能会导致幻读。 |
| SERIALIZABLE      | 最高的隔离级别，完全避免了脏读、不可重复读和幻读，但效率较低。 |

