# 会话技术，Cookie, Session

## 什么是会话技术

>基本概念，指用户开一个浏览器，**访问一个网站,只要不关闭该浏览器，不管该用户点击多少个超链接，访问多少资源，直到用户关闭浏览器，整个这个过程我们称为一次会话**

## 为什么我们要使用会话技术？
会话跟踪技术可以解决我们很多很多问题。

* 在论坛登录的时候，**很多时候会问你是否自动登录，当你下次登录的时候就不用输入密码了**
* 根据你浏览的商品，猜你喜欢什么商品

## Cookie
会话跟踪技术有Cookie和Session，Cookie技术是先出现的，先讲Cookie技术

## 什么是Cookie

>Cookie 是由W3C组织提出，最早由netscape社区发展的一种机制

* 网页之间的**交互是通过HTTP协议传输数据的**，而Http协议是**无状态的协议**。无状态的协议是什么意思呢？**一旦数据提交完后，浏览器和服务器的连接就会关闭，再次交互的时候需要重新建立新的连接**。
* 服务器无法确认用户的信息，于是乎，W3C就提出了：**给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息。**通行证就是Cookie
  
>Cookie的流程：浏览器访问服务器，如果服务器需要记录该用户的状态，就使用response向浏览器发送一个Cookie，浏览器会把Cookie保存起来。当浏览器再次访问服务器的时候，浏览器会把请求的网址连同Cookie一同交给服务器。

## Session

>Session 是另一种记录浏览器状态的机制。不同的是Cookie保存在浏览器中，Session保存在服务器中。用户使用浏览器访问服务器的时候，服务器把用户的信息以某种的形式记录在服务器，这就是Session

如果说Cookie是检查用户身上的”通行证“来确认用户的身份，那么Session就是通过检查服务器上的”客户明细表“来确认用户的身份的。Session相当于在服务器中建立了一份“客户明细表”。

### 为什么要使用Session技术?
Session比Cookie使用方便，Session可以解决Cookie解决不了的事情【Session可以存储对象，Cookie只能存储字符串。】。

### Session API

* long getCreationTime();【获取Session被创建时间】
* String getId();【获取Session的id】
* long getLastAccessedTime();【返回Session最后活跃的时间】
* ServletContext getServletContext();【获取ServletContext对象】
* void setMaxInactiveInterval(int var1);【设置Session超时时间】
* int getMaxInactiveInterval();【获取Session超时时间】
* Object getAttribute(String var1);【获取Session属性】
* Enumeration getAttributeNames();【获取Session所有的属性名】
* void setAttribute(String var1, Object var2);【设置Session属性】
* void removeAttribute(String var1);【移除Session属性】
* void invalidate();【销毁该Session】
* boolean isNew();【该Session是否为新的】

### Session作为域对象
从上面的API看出，Session有着request和ServletContext类似的方法。其实**Session也是一个域对象**。Session作为一种记录浏览器状态的机制，**只要Session对象没有被销毁，Servlet之间就可以通过Session对象实现通讯**

* 在Servlet4中设置Session属性
  
```java
    //得到Session对象
            
    HttpSession httpSession =request.getSession();

    //设置Session属性
    httpSession.setAttribute("name","看完博客就要点赞！！");
```

* Servlet5中获取到Session存进去的属性
```java
    //获取到从Servlet4的Session存进去的值
            
    HttpSession httpSession =request.getSession();
            
    String value =(String) httpSession.getAttribute("name");
            
    System.out.println(value);
```

* 一般来讲，当我们要存进的是用户级别的数据就用Session，那什么是用户级别呢？只要浏览器不关闭，希望数据还在，就使用Session来保存。