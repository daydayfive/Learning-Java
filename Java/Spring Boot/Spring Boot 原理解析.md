[TOC]


#Spring Boot原理解析

## 1. Profile 功能
为了方便多环境适配，springboot 简化了profile 功能。


### 1. application-profile 功能
* 默认配置文件 application.yaml；任何时候都会加载
* 指定环境配置文件  application-{env}.yaml
* 激活指定环境
  * 配置文件激活
  * 命令行激活 java -jar xxx.jar --spring.profiles.active=prod  --person.name=haha
    * 修改配置文件的任意值，命令行优先

* 默认配置与环境配置同时生效
* 同名配置项，profile配置优先


 