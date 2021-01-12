# Spring Boot构建RESTful API与单元测试

* <font color='red'>@Controller</font>:修饰class,用来创建处理http请求的对象
* <font color='red'>@RestController</font>: Spring 4之后加入的注解，原来在@Controller 中返回json 需要 @ResponseBody 来配合，如果直接用@<font color='red'>RestController</font>替代 @Controller 就不要再配置@ResponseBody，默认返回json格式
* <font color='red'>@RequestMapping</font>:配置url映射



RESTful API 具体设计如下：

![img](http://blog.didispace.com/content/images/posts/springbootrestfulapi-1.png)



