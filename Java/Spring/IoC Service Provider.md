# IoC Service Provider

* **IoC Service Provider的职责** 

  * **业务对象的构建管理**

    ​	IOC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作始终需要有人来做。所以，IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现。

  * **业务对象间的依赖绑定**

    

      

* IoC Service Provider 如何管理对象间的依赖瓜西

  * 直接编码方式

    ​		通过为相应的类指定对应的具体实例，可以告知IoC容器，当我们要这种类型的对象实例时，请将容器中注册的、对应的那个具体实例返回给我们。 

  * 配置文件方式

    ​		像普通文本文件、properties文件、XML文件等，都 可以成为管理依赖注入关系的载体。不过，最为常见的，还是通过XML文件来管理对象注册和对象间 依赖关系，比如Spring IoC容器和在PicoContainer基础上扩展的NanoContainer，都是采用XML文件来 管理和保存依赖注入信息

  * 元数据方式

    ​		通过@Inject，我们指明需要IoC Service Provider通过构造方法注入方式，为FXNewsProvider注入 其所依赖的对象。至于余下的依赖相关信息，在Guice中是由相应的Module来提供的

  