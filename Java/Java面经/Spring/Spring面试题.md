[toc]


# Spring 面试题汇总

## 1.Spring 是什么
Spring是一个轻量级的开源应用框架，旨在降低应用程序开发的复杂度。
Spring是非侵入性，允许应用系统自由选择和组装Spirng框架中的各个功能模块，而不要求应用必须对Spring的某个类继承或实现，极大地提高一致性。
使用IOC容器管理对象的生命周期，以及对象间的依赖关系，降低系统耦合性。
基于AOP的面向切面编程，将具有横切性质的业务放到切面中，从而与核心业务逻辑分离并提高组件复用性。

Spring可以很好地与其他框架集成，被称为框架的框架。


## 2.Spring的特性
* IOC
* AOP
* 事务
* 低侵入性

## 3. 什么是Spring IOC DI
IOC 即 控制反转，即 将对象的创建权 反转给了Spring，实现了程序的解耦。

Ioc的底层实现原理是 **工厂设计模式+反射+XML配置文件。**


DI，即依赖注入。 指的是 Spring在管理某个类的时候会将类依赖的属性注入进来，也就是说在创建对象的过程中，向类里面的属性中设置值。


## 4.Spring IOC 容器的初始化过程
Spring IOC容器的初始化简单的可以分为三个过程：
1. Resource 资源定位，这个Resource 指的是 BeanDefinition的资源定位。这个过程就是容器找数据的过程
2. BeanDefinition 载入，这个载入过程就是把用户定义好的bean 表示成ioc 内部的数据结构（BeanDefinition）
3. 向IOC容器注册这些BeanDefinition的过程，这个过程就是将前面的BeanDefition保存到HashMap中的过程。



## 5. BeanFactory 和 FactoryBean的区别？
* BeanFactory 是接口，定义了IOC容器的基本功能，所有的bean都是由BeanFactory 来进行管理的。
使用场景：
  * 从ioc容器中获取bean
  * 检索ioc容器中收纳柜是否包含指定的bean
  * 判断bean是否为单例
  
* FactoryBean 是个接口，实现了该接口的类是一个bean，这个Bean 不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean，它的实现与设计模式中的工厂模式和装饰器模式类似
使用场景
* ProxyFactoryBean

## 6. Spring AOP 的实现原理？
当bean 执行完 init-method 后， spring 会postProcessAfterInitialization 为bean 的方法匹配 通知器，如果匹配成功则为bean 生成代理对象，并返回给 spring 容器。


而代理对象在调用方法时，会启动拦截器链，这里会根据前置通知 以及后置通知先执行哪些 方法。


Spring AOP使用的动态代理，所谓的动态代理就是说AOP 框架不会去修改字节码，而是在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特点的切点做了增强处理，并回调原对象的方法。

Spring AOP 的动态代理主要有两种方式，JDK动态代理和CGLIB动态里。
JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。
如果目标类没有实现接口，那么Spring AOP 会选择使用CGLIB 来动态代理类。 CGLIB，是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。


## 7. Spring 是如何管理事务的
@Transaction 注解

使用aop方式实现的，

缺点：注解的最细力度作用于方法上，如果像代码块也有事务需求，只能变通下，将代码块变为方法。


## 9. spring 自动装配
当bean 在spring 容器 中组合在一起时，它被称为装配或bean装配。 Spring 容器需要知道需要什么bean以及容器应该使用依赖注入来将bean 绑定在一起，同时装配bean。

spring容器能够自动装配bean。也就是通过检查 beanFactory 的内容让spring自动解析bean 的协作者。自动装配的不同模式：


自动装配容易覆盖



## 4. 有多少种方式完成依赖注入
* setter注入
* 构造函数注入
* 基于注解的注入（接口注入）
  （@Autowired）

## 5. 依赖注入发生在什么时刻
* 第一次通过getBean 方法 向IOC索要Bean时，IOC容器触发依赖注入。
* 在Bean定义资源中为元素配置了lazy-init属性，即让容器在解析注册Bean定义时进行预实例化，触发依赖注入


## 6. Bean的作用域
在Spring中，Bean 是组成 应用程序的主体及由Spring IOC容器所管理的对象，被称之为bean。 

singleton：每个spring ioc 容器仅有一个单实例。
prototype：每次请求都会产生一个新的实例
request：每一次http请求都会产生一个新的实例，同时该bean仅在当前Http 请求内有效。
session：每一次http请求都会产生新的bean，同时该bean仅在当前当前的http session 内有效。
gloabal session-类似于session，不过仅在portlet web应用中才有意义。

后面三种仅当用户使用支持web 的applicationcontext 是才使用。

## 7. bean的生命周期

1. spring容器根据配置中的bean定义， 实例化bean。
2. spring使用依赖注入填充所有属性，如bean中所定义的配置
3. 如果bean实现了BeanNameAware接口，则调用setBeanName()
4. 如果bean实现了BeanFactoryAware接口，工厂通过传递自身的实例来调用setBeanFactory()
5. 如果存在与bean关联的BeanPostProcessor，则调用preProcessBeforeInitialization()方法
6. 如果为bean制定了init方法， 调用它
7. 执行preProcessorAfterInitialization()方法
8. bean 实现 DisposableBean 接口，当 spring 容器关闭时，会调用destory()
9. 调用destory()方法
