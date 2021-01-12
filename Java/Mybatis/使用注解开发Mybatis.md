# 使用注解开发Mybatis

1.面向接口编程：

​	好处：扩展性好，分层开发中，上层不用管具体的实现，大家都遵循共同的标准，使得开发变得容易。规范性更好

2.注解的实现：

a)编写Dao接口

```java
public interface UserDao(){
    @Select("select * from user")
    public List<User> getList();
    @Insert("insert into user(name,pwd) values (#{name},#{pwd})")
    public int insert(User user);
}
```

b)在核心配置文件中，导入

```
<mappers>
	<mapper class="com.annotation.UserDao"/>
	</mappers>
```

3.使用

```java
public static void main(String[] args) throws IOException{
	SqlSession session = MyBatisUtil.getSession();
	UserDao userDao =session.getMapper(UserDao.class);
	//通过代理生成对象实例
	List<User> list =userDao.getList();
	for(User u :list){
		System.out.println(u);
	}
}
```

