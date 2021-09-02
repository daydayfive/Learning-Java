[toc]
# Spring Boot 面试题


## 1. 什么是Spring Boot
spirng boot 建立再现有spring框架之上，简化spring配置的框架。


## 2. Spring Boot有哪些优点？
减少开发，测试时间。

通过提供默认值快速开始开发

不需要启动了Tomcat


## 3. 什么是JavaConfig?
Spring JavaConfig 是Spring 社区的产品，它提供了Spring IoC 容器的纯Java 方法，因此它有助于避免使用XML 配置。
使用JavaConfig 的优点在于：
面向对象的配置。由于配置被定义为JavaConfig 中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写他的@Bean方法等。


## 4. 如何重新加载Spring Boot上的更改，而无需重新启动服务器？
这可以使用 DevTools 模式


## 5. 如何在自定义端口上运行spring boot 应用程序？
为了在定义端口上运行spring boot应用程序，你可以在application.properties中指定端口。
application.properties.server.port=8090

## 6.什么是yaml
yaml是一种数据序列化语言。它通常用于配置文件。



## 7. SpringBoot是如何启动Tomcat的
1. 首先，SpringBoot在启动时会先创建一个Spring容器
2. 在创建Spring容器过程中，会利用@ConditionalOnClass技术来判断当前classpath是否存在Tomcat依赖，如果存在则会生成一个启动Tomcat的Bean
3. Spring 容器创建完之后，就会获取启动Tomcat的Bean，并创建Tomcat对象，并绑定端口等，然后启动Tomcat

## 8.SpringBoot中的starter
使用spring+springmvc使用，如果要引入mybatis框架，需要到xml中定义mybatis需要的bean

而springboot中定义starter的jar包，写一个@Configuration配置类，将这些bean定义在里面，然后在starter包的META-INF/spring.factories中写入配置类，springboot会按照约定来加载该配置类

开发人员只需要将相应的starter包依赖进应用，进行响应的属性配置，就可以直接进行代码开发，使用对应的功能了，比如mybatis-spring-boot-starter，spring-boot-starter-redis


