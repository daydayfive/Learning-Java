Mybatis 开端

1. 什么是mybatis？

   mybatis是一个基于JAVA的持久层框架

2. 持久化

   数据从瞬时状态变为持久状态

3. 持久层

   完成持久化工作的代码块

4. Mybatis就是帮助程序猿将数据存入数据库中，和从数据库中取数据。

5. 传统的jdbc操作：有很多重复代码块，比如：数据取出时的封装。数据库的建立连接等。通过框架可以减少重复代码，提高开发效率

6. mybatis是一个半自动化的ORM框架。

   O-object

   R-relationship

   M-mapping

7. mybatis的功能，

   ​	mybatis 是支持普通sql查询，存储过程和高级映射的优秀持久层框架。Mybatis消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。

8. 如何使用mybatis

   步骤：

   1>配置maven文件,

   2>mybatis-config.xml 

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   <configuration>
       <!--环境配置，连接的数据库，这里使用的是MySQL-->
       <environments default="mysql">
           <environment id="mysql">
               <!--指定事务管理的类型，这里简单使用Java的JDBC的提交和回滚设置-->
               <transactionManager type="JDBC"></transactionManager>
               <!--dataSource 指连接源配置，POOLED是JDBC连接对象的数据源连接池的实现-->
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"></property>
                   <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybbs"></property>
                   <property name="username" value="root"></property>
                   <property name="password" value="root"></property>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <!--这是告诉Mybatis区哪找持久化类的映射文件，对于在src下的文件直接写文件名，
               如果在某包下，则要写明路径,如：com/mybatistest/config/User.xml-->
           <mapper resource="User.xml"></mapper>
       </mappers>
   </configuration>
   ```

   3>运行测试文件，创建SqlSessionFactory,以及获得SqlSession

   ```java
   public class MyBatisUtil{
   	public static SqlSessionFactory getSqlSessionFactory() throws IOException{
   	      String source ="mubatis.cfg.xml";
   	      InputStream inputStream= Resources.getResourceAsSteam(resource);
   	      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputSteam);
   	      return sqlSessionFactory;
   	}
   	public static SqlSession getSession(){
   		SqlSessionFactory sqlSessionFactory=getSqlSessionFactory();
   		return sqlSessionFactory.openSession();
   	
   	}
   }
   ```

   4>创建实体类 

   ```java
   public class User {
       private int ID;
       private String name;
       private int age;
   
       public User(int id， String name, int age){
           this.ID = id;
           this.name = name;
           this.age = age;
       }
   
       public void setID(int ID) {
           this.ID = ID;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public int getID() {
           return ID;
       }
   
       public String getName() {
           return name;
       }
   
       public int getAge() {
           return age;
       }
   }
   ```

   

5>编写sql语句的映射文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.springtest.User">
    <select id="selectUser" resultType="com.test.springtest.dao.MUser">
        select * from user where id = #{id}
    </select>
</mapper>
```

// namespace, 为这个mapper指定一个唯一的namespace，它习惯上设置为：“包名+sql映射文件名”，这样可以保值名的唯一。

//resultType: 即返回结果集的类型，这里指定为User类型

// selectUser : 在执行的时候，session.select("namespace.selectUser",user);

