#  Redis 单机数据库

一. Redis 服务器的数据库

Redis 服务器的数据库默认创建16个，且数据库与数据库之间的数据是隔离的。



## **1.1Redis数据库的原理**

​		Reids服务器用redisServer结构体来表示，其中redisDb是一个数组，保存所有的数据库，dbnum代表数据库的数量（默认16）

```c
struct redisServer{  

    //redisDb数组,表示服务器中所有的数据库
    redisDb *db;  

    //服务器中数据库的数量
    int dbnum;  

}; 
```

​		Redis是C/S结构，Redis客户端通过redisClient 结构体表示：

```c
typedef struct redisClient{  

    //客户端当前所选数据库
    redisDb *db;  

}redisClient;
```

​		Redis客户端连接Redis服务端时的示例图：

![image-20201221140758283](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201221140758283.png)

​		Redis中每个数据库对redisDb结构体来表示：

```c
typedef struct redisDb { 
    int id;         // 数据库ID标识
    dict *dict;     // 键空间，存放着所有的键值对              
    dict *expires;  // 过期哈希表，保存着键的过期时间                          
    dict *watched_keys; // 被watch命令监控的key和相应client    
    long long avg_ttl;  // 数据库内所有键的平均TTL（生存时间）     
} redisDb;
```

从代码上可以发现最重要的**dict *dict**，它用来存放着所有的键值对。对于dict数据结构（哈希表）在上一章已说明，一般将存储所有键值对的dict称为**键空间**。



键空间示意图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib0LSf7wiaom7XfZ9RhpUWsW2AC1gtvq3ibvicwU8XFTib4Y3aicN96AUbrNBlyAB1MHnb9fvDVsYerzoicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



```
Redis的数据库就是使用字典(哈希表)来作为底层实现的，对数据库的增删改查都是构建在字典(哈希表)的操作之上的。
```

例如：

```redis
redis > GET message

"hello world"

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib0LSf7wiaom7XfZ9RhpUWsW2tiagicnJMog5YjszvjrXiafwuQCiaU6CNBEc4fuSl1MzbreesTtosrT5ew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1.2 键的过期时间

​		Redis是基于内存，内存是比较昂贵的，容量肯定比不上硬盘的。

​		由于内存十分**有限**，因此需要干掉不常用的数据，保留常用的数据。这就需要设置键的**过期时间**。

* 设置键的**生存**时间可以通过 **EXPIRE** 或者**PEXPIRE** 命令。
* 设置键的**过期**时间可以通过**EXPIREAT** 或者**PEXPIREAT** 命令。

其实**`EXPIRE`、`PEXPIRE`、`EXPIREAT`**这三个命令都是通过**`PEXPIREAT`**命令来实现的。

我们在redisDb结构体中还发现了`dict *expires;`属性，存放所有键过期的时间。



既然有设置过期时间的命令，肯定也有移除过期时间的命令，查看剩余生存时间的命令：

* PERSIST(移除过期时间)
* TTL(Time To Live)返回剩余生存时间，以秒为单位
* PTTL以毫秒为单位返回键的剩余生存时间



### 1.2.1 过期策略

​	删除策略可分为三种：

* 定时删除（对内存友好，对CPU不友好）

  * 到时间点上就把所有过期的键删除了。

* 惰性删除（对CPU极度友好，对内存极度不友好)

  * 每次从键空间取键的时候，判断一下该键是否过期了，如果过期了就删除。

* 定期删除

  * 每隔一段时间去删除过期键，限制删除的执行时长和频率。

  

  

  Redis采用的是**惰性删除+定期删除**两种策略，所以说，在Redis里边如果**过期键到了过期的时间了，未必被立马删除**！

  

  

  ### 1.2.2内存淘汰机制

  ​		如果定期删除漏掉了很多过期key，也没及时去查，大量过期key堆积在内存里，导致redis内存块耗尽了。

  ​		通过设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

  

  ​	Redis的内存淘汰机制有以下几种：

  ​	![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib0LSf7wiaom7XfZ9RhpUWsW2TMibgqktURXxzDeSVITWJiaKQDlocMf2ibHS3vK3AyOgtibIzRW4LvrXhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一般场景：

```
使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用allkeys-lru淘汰策略，将最近最少使用的数据淘汰
```



