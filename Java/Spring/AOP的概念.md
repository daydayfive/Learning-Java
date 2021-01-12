#  AOP的概念

**AOP**:Aspect-Oriented Programming,面向切面编程

* 功能：让**关注点**代码与业务代码分离

* 面向切面编程就是指：对很多功能都有的重复的代码抽取，再在运行的时候往业务方法上动态植入“切面类代码”

  关注点：重复代码就叫做关注点

![image-20201127194424006](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201127194424006.png)

​			怪不得叫面向切面编程，



JAVA平台上的AOP实现机制：

* 动态代理

  ​	JDK1.3之后，引入了动态代理（Dynamic Proxy）机制，，可以在运行期间，为相应的接口（Interface） 动态生成对应的代理对象。

* 动态字节码增强

* JAVA代码生成

* 自定义类加载器

* AOL扩展



AOP框架概念：

* **JoinPoint**

  ​	在系统运行之前，AOP的功能模块都需要织入到OOP的功能模块中。所以，要进行这种织入过程， 我们需要知道在系统的哪些执行点上进行织入操作，这些将要在其之上进行织入操作的系统执行点就 称之为Joinpoint。 

* **PointCut**

  ​	将横切逻辑织入当前系统的过程中，需要参照Pointcut规定的Joinpoint 信息，才可以

知道应该往系统的哪些Joinpoint上织入横切逻辑

   * Pointcut的表述方式

     	* 直接指定Joinpoint 所在方法名称。
          	* 正则表达式
     	* 使用特定的Pointcut表述语言。

* Pointcut运算

  ​		通常，Pointcut与Pointcut之间还可以进行逻辑运算。这样，我们就可以从简单的Pointcut开始，然后通过逻辑运算，得到最终需要的可能较为复杂的Pointcut。

![image-20201127201818509](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201127201818509.png)

​	

 * **Advice**

   ​	  Advice是单一横切关注点逻辑的载体，它代表将会织入到Joinpoint的横切逻辑。如果将Aspect比 作OOP中的Class，那么Advice就相当于Class中的Method。 按

* **Aspect** 

  ​		Aspect是对系统中的横切关注点逻辑进行模块化封装的AOP概念实体。通常情况下，Aspect可以包含多个Pointcut以及相关Advice定义。



AOP各个概念所在的场景

![image-20201127202059451](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201127202059451.png)