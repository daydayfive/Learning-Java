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

缺点：注解的最细粒度作用于方法上，如果像代码块也有事务需求，只能变通下，将代码块变为方法。

1. Spring事务底层是基于数据库事务和AOP机制的
2. 对于使用了@Transactional注解的Bean，Spring会创建一个代理对象作为Bean
3. 当调用了代理对象的方法时，会先判断该方法上是否加了@Transactional注解
4. 如果加了，利用事务管理器创建一个数据库连接
5. 并且修改数据库连接的autocommit属性为false，禁止此连接的自动提交，这是实现Spring事务非常重要的一步
6. 然后执行当前方法，方法中会执行sql
7. 执行完方法后，没有异常就直接提交事务
8. 出现了异常，这个异常需要回滚就回滚事务，否则仍然提交事务
9. Spring事务的隔离级别对应的是数据库的隔离级别
10. Spring事务的传播机制是Spring事务自己实现的，也是Spring事务中最复杂的
11. Spring事务的传播机制是基于数据库连接来做的，一个数据库连接一个事务，如果传播机制配置为需要新开一个事务，实际上就是先建立一个数据库连接，在此新数据库连接上执行sql

## Spring事务传播机制
多个事务方法相互调用时，事务如何在这些方法间传播
> 方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。

REQUIRED(Spring默认的事务传播类型)：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务。

SUPPORTS:当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行。

MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。

REQUIRES_NEW: 创建一个新事务，如果存在当前事务，则挂起该事务。（各自回滚自己的）

NOT_SUPPORTED:以非事务方式执行，如果当前存在事务，则挂起当前事务

NEVER:不使用事务，则如果当前事务存在，则抛出异常

NESTED:如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务）




## @Transactional失效

Spring事务的原理是AOP，进行了切面增强，那么失效的根本原因是这个AOP不起作用了！常见情况有如下几种

1. 发生自调用，类里面使用this调用本类的方法（this通常省略），此时这个this对象不是代理类，而是UserService本身
   让this变为UserService本身即可

2. 方法不是public的
   @Transactional 只能用于public的方法上，否则事务不会实现，如果要用到非public方法上，可以开启AspectJ代理模式。

3. 数据库不支持事务（MyIsam）
4. 没有被spring管理（该类没有被spring管理 ，如没有@controller，@service）
5. 异常不会回滚，事务不会回滚（或者抛出的异常没有被定义，默认为RuntimeException）
（try catrch）






因为Spring事务是基于代理来实现的，所以某个加了@Transactional的方法只有是被代理对象调用时，那么这个注解才会生效，所以如果是被代理对象来调用这个方法，那么@Transactional这个是不会生效的

同时如果某个方法是private的，那么@Transactional也会失效，因为cglib基于父子类来实现的，子类是不能重载父类的private方法的，所以无法很好的利用代理，也会导致@Transactional也是失效。



## 8. spring 自动装配
在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要协作的对象引用赋予各个对象，使用Autowired 来配置自动装载模式：

spring框架xml配置中共有5种自动装配：
（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean
（2）byName：通过bean 的名称进行自动装配，如果一个bean的property 与另一bean 的名称相同，就进行装配
（3）byType：通过参数的数据类型进行自动装配
（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配
（5） autodetect:自动


基于注解的方式：
使用@Autowired注解来指定自动装配指定的bean
（1） @Autowired  默认按照类型来装配注入的，默认情况下它要求依赖对象必须存在
（2） @Resource   默认按照名称来注入的，只有找不到名称匹配的bean来装配注入。

## 9. 有多少种方式完成依赖注入
* setter注入
* 构造函数注入
* 基于注解的注入（接口注入）
  （@Autowired）

## 10. 依赖注入发生在什么时刻
* 第一次通过getBean 方法 向IOC索要Bean时，IOC容器触发依赖注入。
* 在Bean定义资源中为元素配置了lazy-init属性，即让容器在解析注册Bean定义时进行预实例化，触发依赖注入


## 11. Bean的作用域
在Spring中，Bean 是组成 应用程序的主体及由Spring IOC容器所管理的对象，被称之为bean。 

singleton：每个spring ioc 容器仅有一个单实例。
prototype：每次请求都会产生一个新的实例
request：每一次http请求都会产生一个新的实例，同时该bean仅在当前Http 请求内有效。
session：每一次http请求都会产生新的bean，同时该bean仅在当前当前的http session 内有效。
gloabal session-类似于session，不过仅在portlet web应用中才有意义。

后面三种仅当用户使用支持web 的applicationcontext 是才使用。

## 12. bean的生命周期

1. spring容器根据配置中的bean定义， 实例化bean。
2. spring使用依赖注入填充所有属性，如bean中所定义的配置
3. 如果bean实现了BeanNameAware接口，则调用setBeanName()
4. 如果bean实现了BeanFactoryAware接口，工厂通过传递自身的实例来调用setBeanFactory()
5. 如果存在与bean关联的BeanPostProcessor，则调用preProcessBeforeInitialization()方法
6. 如果为bean制定了init方法， 调用它
7. 执行preProcessorAfterInitialization()方法
8. bean 实现 DisposableBean 接口，当 spring 容器关闭时，会调用destory()
9. 调用destory()方法



## 13. Spring 常用的注解

1. 声明bean的注解
@Component 组件，没有明确的角色
@Service （Service层）
@Repository （Dao层）
@Controller （Controller层）

2. 注入bean的注解
@Autowired （）
@Resource

3. java配置类
@Configuration  声明当前类为配置类
@Bean  注解作用在方法上，声明当前方法的返回值是一个bean，代替xml的方式
@ComponentScan 用于对compoent 扫描
4. 切面（AOP）相关注解
@Aspect  声明一个切面
使用@After,@Before,@Around 定义 advice ，可直接将拦截规则作为参数


5. @Bean 的属性支持
@Scope 定义bean 的作用域

6. @Value 为属性注入值


7. @Enable *注解说明
@EnableAspectJAutoProxy
@EnableAsync
@EnableWebMvc
@EnableTransactionManagement 开启注解


## 14. Spring 中后置处理器的作用
Spring 中的后置处理器分为BeanFactory后置处理器和Bean后置处理器，它们是Spring底层源码架构设计中非常重要的一种机制，同时开发者也可以利用这两种后置处理器来进行扩展。BeanFactory 后置处理器表示针对BeanFactory的处理器，Spring启动过程中，会先创建出BeanFacoty实例，然后利用BeanFactory处理器来加工BeanFactory，比如Spring扫描的BeanFactory后置处理器实现的，而Bean后置处理器也类似，spring在创建一个Bean的过程中，首先会实例化得到一个对象，然后再利用Bean后置处理器来对该实例对象进行加工，比如我们常说的依赖注入就是基于一个Bean后置处理器来实现的，通过该Bean后置处理器来给实例对象中加了@Autowired注解的属性自动赋值，还比如我们常说的AOP，也是利用一个Bean后置处理器来实现的，基于原实例对象，判断是否需要进行AOP，如果需要，那么就基于原实例对象进行动态代理，生成一个代理对象。




## 15.Spring 种的设计模式
工厂模式：--BeanFactory
          -- FactoryBean

适配器模式 -- AdvisorAdapter接口，对Advisor进行了适配
访问者模式 -- PropertyAccessor接口，属性访问器，用于访问和设置某个对象的某个属性

装饰器模式 -- BeanWrapper
代理模式  -- AOP
模板模式  -- JDBCTemplate
观察者模式 -- 事件监听模式
策略模式  -- 
委派模式
责任链模式 --BeanPostProcessor

