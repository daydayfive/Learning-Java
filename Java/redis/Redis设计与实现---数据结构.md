# Redis设计与实现

## 数据结构与对象

### 简单动态字符串

* Redis 定义了一个名为简单动态字符串（simple dynamic string, SDS）的抽象类型作为默认字符串表示

* 和C字符串一样都是以'\0'结尾，不算在len属性里

  ```c
  struct sdshdr{
      //记录buf数组中已使用字节的数量
      //等于SDS所保存字符串的长度
      int len;
      
      //记录buf数组中未使用字节的数量
      int free;
      
      //字节数组，用户保存字符串
      char 
          buff[];
  }
  ```

* SDS特点
  * 因为有len属性，获取字符串长度的时间复杂度为O(1),而C字符串中需要O（n）
  * **杜绝缓存区溢出**
    * C字符串在执行字符串拼接时会直接拼接如果空间不够则会产生缓存区溢出
    * SDS在执行字符串拼接时，如sdscat(s,"Tutorial")，空间分配策略会先检查空间是**否满足修改需求**，即检查free属性
    * 如果不满足，会自动扩展SDS的空间，然后才执行实际的修改操作，所以**不会出现缓存区溢出**
  * **减少修改字符串时带来的内存重分配次数**
    * Redis作为数据库，经常被用于速度要求严苛，数据被频繁修改的场合，如果每次修改都需要执行内存分配时，影响性能
    * 1.空间预分配
      * 如果SDS修改后，SDS的长度（len属性）小于1MB.分配和修改后len属性同样大小的未使用空间，即len==free
      * 如果len属性大于1MB，free=1MB
      * 在扩展SDS空间之前，SDS API 会先检查未使用空间是否足够，如果足够的话直接使用未使用空间
    * 2.惰性空间释放
      * 惰性空间释放用于优化SDS的字符串缩短操作
      * 移除后并不会释放多出来的空间，会作为未使用空间保留在SDS中
      * 存在API  直接释放SDS的未使用空间
    * 二进制安全
      * C字符串以'\0'判断字符串是否结束，而SDS使用len属性判断
      * SDS可以在字符串中保留空字符和二进制数据
    * 支持部分C字符串函数

### 链表

* **链表和链表节点的实现**

  * 链表被用于Redis的列表键、发布与订阅、慢查询、监视器

  * 每个**链表节点**使用adlist.h/listNode结构来表示：

    ```c
    typedef struct listNode{
        //前置节点
        struct listNode *prev;
        
        //后置节点
        struct listNode * next;
        
        //节点的值
        void *value;
    }listNode;
    ```

  * 每个链表使用list结构来表示

  ```c
  ypedef struct list{
      //表头节点
      listNode *head;
      
      //表尾节点
      listNode *tail;
      
      //链表所包含的节点数量
      unsigned long len;
      
      //节点值复制函数
      void *(*dup) (void *ptr);
      
      //节点值释放函数
      void (*free) (void *ptr);
      
      //节点值对比函数
      int (*match) (void *ptr,void *key);
  }list;
  ```

* Redis链表特性
  * 双端：获取某个节点的前置节点和后置节点的复杂度都是O(1)
  * 无环：表头节点的prev指针和表尾节点next指针都指向NULL
  * 通过list结构，获取表头和表尾的复杂度都是O(1)
  * 获取节点数量的时间复杂度是O(1)
  * 通过为链表设置不同类型特定函数，链表可以保存不用各种不同类型的值



### 字典

* **字典的实现**
  * redis的字典使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对
  * 哈希表
    * 哈希表由dict.h/dictht结构定义

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993809847-d4fdc291-0ef9-4176-95db-d59829138cf7.png)

```c
typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    
    //哈希表大小
    unsigned long size;
    
    //哈希表大小掩码，用于计算索引值
    //总是等于size-1
    unsigned long sizemask;
    
    //该哈希表已有节点的数量
    unsigned long used;
}dictht
```

* **哈希表节点**

  * 哈希表节点使用dictEntry结构表示，每个dictEntry结构都保存一个键值对

  * 其中，key表示节点的键，union表示key对应的值，可以是指针，uint64_t整数或int64_t整数

  * next是指向另一个哈希表节点的指针，该指针将多个哈希值相同的键值对连接在一起，避免因为哈希值相同导致的冲突

    ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993793919-d14f1820-087a-40bc-b23e-d3f174632c32.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

```c
/*
 * 哈希表节点
 */
typedef struct dictEntry {
    
    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```



* **字典**

  * 字典被用于redis的数据库和哈希键

  * redis中的字典有dict.h/dict结构表示

    ```c
    typedef struct dict {
    
        // 类型特定函数
        dictType *type;
    
        // 私有数据
        void *privdata;
    
        // 哈希表
        dictht ht[2];
    
        // rehash 索引
        // 当 rehash 不在进行时，值为 -1
        int rehashidx; 
    
        // 目前正在运行的安全迭代器的数量
        int iterators;
    } dict;
    ```

    

* type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的

  * type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值的函数，Redis会为用途不同的字典设置不同的类型特定函数

  * privdata属性则保存了需要传给那些类型特定函数的可选参数

    ```c
    typedef struct dictType {
    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);
    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);
    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);
    // 对比键的函数
    int (*keyCompare)
    (void *privdata, const void *key1, const void *key2);
    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);
    } dictType;
    ```

    

- ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht[0]哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用
- rehashidx记录了rehash目前的进度，如果目前没有在进行rehash，那么它的值为-1
- ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993758179-97f6802f-aa92-4c2d-91d7-e63dccd1e968.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



**rehash**:

* 随着操作的不断执行，哈希表保存的键值对会逐渐地增多或者减少，为了让哈希表的负载因子维持在一个合理的范围之内，当哈希表保存的键值对数量太多或者太少时，程序需要对哈希表的大小进行响应的扩展或者收缩。

* 步骤如下：

  * - 给字典的ht[1]申请存储空间，大小取决于要进行的操作，以及ht[0]当前键值对的数量（ht[0].used）。假设当前ht[0].used=x。如果是**扩展**，则ht[1]的值是**第一个大于等于x\*2的2^n的值**。例如x是30，则ht[1]的大小是64，如果是**收缩**，则ht[1]的值是**第一个大于等于x的2^n的值**。例如x是30，则ht[1]的大小是32
    - 将保存在ht[0]上面的所有键值对，rehash到ht[1]，即对每个键重新采用哈希算法的方式计算哈希值和索引值，再放到相应的ht[1]的表格指定位置
    - 当ht[0]的所有键值对都rehash到ht[1]后，释放ht[0]，并将ht[1]设置为ht[0]，再新建一个空的ht[1]，用于下一次rehash

* rehash条件：

  * 负载因子（load factor)计算：

    * load_factor =ht[0].used / ht[0].size，即负载因子大小等于当前哈希表的键值对数量，除以当前哈希表的大小
    * 当以下任一条件满足，哈希表会自动进行扩展操作：

    - - - 服务器目前没有在执行BGSAVE或者BGREWRITEAOF命令，且**负载因子大于等于1**。
        - 服务器目前正在在执行BGSAVE或者BGREWRITEAOF命令，且负载因子大于等于5。

    - - 收缩：

    - - - 当**负载因子小于0.1**时，redis自动开始哈希表的收缩工作



**渐进式rehash**

* redis对ht[0]扩展或收缩到ht[1]的过程，并不是一次性完成的，而是渐进式、分多次的完成，以避免如果哈希表中存有大量键值对，一次性复制过程中，占用资源较多，会导致redis服务停用的问题
* 渐进式rehash过程如下：
  * 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两张哈希表。
  * 将字典中的rehashidx设置成0，表示正在rehash。rehashidx的值默认是-1，表示没有在rehash。
  * 在rehash进行期间，程序处理正常对字典进行增删改查以外，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，并且rehashidx的值加1。
  * 当某个时间节点，全部的ht[0]都迁移到ht[1]后，rehashidx的值重新设定为-1，表示rehash完成
* 渐进式rehash采用分而治之的工作方式，将哈希表的迁移工作所耗费的时间，平摊到增删改查中，避免集中rehash导致的庞大计算量
* 在rehash期间，对哈希表的查找、修改、删除，会先在ht[0]进行。(没找到的时候，继续到ht[1])里面区查找，诸如此类
* 而增加的操作，会直接增加到ht[1]中，目的是让ht[0]只减不增，加快迁移的速度





### 跳跃表

* 跳跃表（skiplist）是一种有序数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。
* 跳跃表支持平均O(log N)，最坏O（N）复杂度的节点查找，还可以通过顺序性操作来批量处理节点。
* 跳跃表类似于平衡树，包括查询的时间复杂度，范围查找，顺序遍历
* Redis 只在两个地方用到了跳跃表：
  * 有序集合键
  * 集群节点中用作 内部数据结构



* **跳跃表的实现**
  * redis的跳跃表由 zskiplistNode  和 zskiplist两个结构定义，其中zskiplistNode结构用于表示跳跃表节点，而zskiplist结构则用于保存跳跃表节点的相关信息。

![image-20201211192502161](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201211192502161.png)

* 图片最左边是zskiplist结构，包含一下属性：

  * header:指向跳跃表的头
  * tail:指向跳跃表的尾
  * length：跳跃表的长度
  * level：记录目前跳跃表内，层数最大的那个节点的层数（不算表头节点）
  * length：记录跳跃表的长度，即跳跃表目前包含节点的数量（表头节点不计算在内）

* **图中其他四个zskiplistNode结构**：

  ```c
  typedef struct zskiplistNode {
      // 成员对象
      robj *obj;
      // 分值
      double score;
      // 后退指针
      struct zskiplistNode *backward;
      // 层
      struct zskiplistLevel {
          // 前进指针
          struct zskiplistNode *forward;
          // 跨度
          unsigned int span;
      } level[];
  } zskiplistNode;
  ```

  * 层（level）：节点中用L1,L2...表示各层，每个层都有两个属性，前进指针（forward）和跨度（span）。每个节点的层高是1到32的随机数（层数越高该节点可以访问的其他节点越多，访问速度就越快）
  * 前进指针（forward）:用于访问表尾方向的节点，便于跳跃表正向遍历节点的时候，寻找下一个节点位置
  * 跨度（span）：记录前进指针所指的节点和当前节点的距离，用于计算排位（rank），访问过程中，将沿途访问的所有层的跨度累积起来，得到的结果就是跳跃表的排位
    * 例子，
    * - - 图中虚线标记了在跳跃表中查找分值为2.0、成员对象对o2的节点时，沿途经过两个跨度为1的节点，因此该目标节点的rank为2
        - ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993674892-cc7334b9-034e-46bf-bc13-e3edd3504214.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)
  * 后退指针（backward）:节点中用BW来表示，其指向当前节点的前一个节点，用于反向遍历时候使用，每次智能后退至前一个节点
  * 分值（score）:各节点中的数字就是分值，跳跃表中，节点按照分值从小到大排列（不同节点可以分值相同）
  * 成员对象（obj）：各个节点中，o1，o2是节点所保存的成员对象。是一个指针，指向一个字符串对象（SDS）



### 整数集合

* 整数集合（intset）是  集合键的底层实现之一，当**一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis就会使用整数集合作为集合键的底层实现。**

* 它可以保存类型为int16_t, int32_t或者int64_t的整数值，并且保证集合中不会出现重复元素

  ```c
  typedef struct intset {
      // 编码方式
      uint32_t encoding;
      // 集合包含的元素数量
      uint32_t length;
      // 保存元素的数组
      int8_t contents[];
  } intset;
  ```

  * encoding决定数组中存储什么类型的整数，如int16_t, int32_t, int_64_t，而不取决于contents数组的类型

  * length属性记录整数集合包含的元素数量，也即是contents数组的长度，length=存储整数类型的长度*元素数量

  * 整数集合的每个元素都是contents数组的一个数组项，**各个项在数组中按值的大小从小到大有序地排列**，**并且数组中不包含任何重复项**



* **升级**

* 将一个新元素添加到整数集合里面，并且新元素的类型比整数集合现有所有元素的类型都要长时，整数集合需要先进行升级，然后才能把生元素添加到整数集合里面。
* 整数集合的升级的步骤：
  * 根据新元素的类型，扩展整数集合底层数组的空间大小，并为新元素分配空间
  * 将底层数组现有的所有元素都转换成与新元素相同的类型，并将类型转换后的元素放置到正确的位上，并且保证有序性，
  * 将新元素添加到底层数组里面
* 向整数集合添加新元素的时间复杂度为  O(N)



* 升级的好处
  * 提升灵活性：可以随意将int16，int32，int64的值放入集合而不用管集合规定什么类型的整数
  * 节约内存：需要什么类型的数组就用什么类型的数组，不会将int16类型的值存放在int64类型的数组里





### 压缩列表

* 压缩列表（ziplist）是列表键和哈希键的底层实现之一
* 当一个列表对象/哈希对象/有序结合对象只包含少量元素，并且每个元素要么就是小整数值，要么就是长度比较短的字符串，那么Redis 就会使用压缩列表来做列表键/哈希键/有序集合键的底层实现
* 不是字符串对象/集合对象的底层实现
* 压缩列表是Redis为了节约内存而开发的，是由一系列特殊编码的连续内存块组成的顺序性数据结构





**压缩列表的构成**

![img](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587823574953-08308676-d7a5-4621-8d03-fd34c17e9db8.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![image-20201211202942545](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201211202942545.png)



**压缩列表节点的构成**

* 可以保存一个字节数组或者一个整数值

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993195599-ba42d246-391d-41da-ba0f-f2a5d41cf70e.png)

* previous_entry_length:

以字节为单位，记录了压缩列表中**前一个节点的长度**

有了这个长度，就可以根据当前节点的起始地址来计算出前一个节点的起始地址，实现从表尾向表头遍历

- encoding属性

- 记录了节点的**content属性**所保存**数据的类型**以及**长度**

* content属性
  - 保存节点的值，节点值可以是一个字节数组或者整数，值的类型和长度由节点的encoding属性决定

  - 

**连锁更新**

​			节点的previous_entry-length属性以字节为单位，记录了压缩列表中前一个节点的长度。previous_entry_length属性的长度可以是1字节或者5字节

​			将一个长度大于等于254字节的新节点设置为压缩列表的表头节点时，如果插入前的头节点previous_entry_length的长度为1字节，则需要变成5个字节，如果增加的这4个字节导致长度超过了254字节，就会继续影响后面的节点，即发生连锁更新

​			连锁更新的最坏时间复杂度O(N^2)

​			发生的概率很低，不用在意



### 对象

* 在前面的数个章节里，我们陆续介绍了Redis用到的所有主要数据结构，比如简单动态字符串（SDS)、双端链表、字典、压缩列表、整数集合等等
* Redis并没有直接使用这些数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统，这个系统包含字符串对象，列表对象，哈希对象，集合对象和有序集合对象这五种类型的对象，每种对象都用到了至少一种我们前面所介绍的数据结构。
* Redis在执行命令之前，根据对象的类型来判断要给对象是否可以执行给定的命令。  并且可以针对不同的使用场景，为对象设置多种不同的数据结构实现，从而优化对象在不同场景下的使用效率。
* Redis的对象系统实现了基于引用计数技术的内存回收机制，当程序不再使用某个对象的时候，这个对象所占用的内存就会被自动释放（JVM不使用，无法解决互相引用的问题）
* 通过引用计数技术，实现了对象共享机制，让多个数据库键共享一个对象来节约内存
* Redis 的对象带有访问时间记录 信息，该信息可以用于计算数据库键的空转时长，在服务器启用了maxmemory功能的情况下，空转市场较大的哪些键可能会有限被服务器删除。



#### **对象的类型与编码**

* Redis使用对象来表示数据库中的键和值

* 每次在Redis的数据库中新创建一个键值对时，至少创建两个对象(redisObject结构表示)，一个对象用作键值对的键，另一个对象用作键值对的值

  ```c
  
  typedef struct redisObject {  
      // 类型  
      unsigned type:4;   
      // 编码方式  
      unsigned encoding:4;  
      // 指向对象的值  
      void *ptr;  
      
      // 不使用(对齐位)  
      unsigned notused:2;  
      // LRU 时间（相对于 server.lruclock）  
      unsigned lru:22;  
      // 引用计数  
      int refcount;  
  
  } robj;
  ```

  * **type属性**：
    * type属性记录了对象的类型，包括：
      * 字符串对象，列表对象，哈希对象，集合对象，有序集合对象
    * **键**总是一个**字符串对象**，而**值**可以是**上述对象的其中一种**
    * 当称呼一个数据库键位列表键时，指的是这个数据库所对应的值位列表对象

  * 编码和底层实现

    * 对象的ptr指针指向对象的底层实现数据结构，而这些数据结构的编码方式由对象的encoding属性决定

    * 对象的编码方式如图。

      ![image-20201212135857022](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201212135857022.png)

    * Redis的5种类型对象都至少由两种不同的编码方式

      * ![image-20201212140025921](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201212140025921.png)

    * Redis可以根据不同 的场景位为对象设置不同的编码，从而优化对象在某一场景下的效率



#### 															**字符串对象**

* 字符串对象的编码可以是int，raw，或者embstr

  

* 如果字符串对象保存的是**整数值**，并且可以用long类型表示(C语言中long类型是32bit)

  * encoding记录的编码类型为REDIS+ENCODING_INT,编码方式称为int

  * 命令：redis->SET number 123

    ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993894262-8cc2a638-d1c2-42d8-9bec-c5c1fcc12975.png)



* 如果字符串对象保存的是一个**字符串值，且长度大于32字节**
  * 使用SDS保存这个字符串值
  * encoding记录的编码类型编程REDIS_ENCODING_RAW, 编码方式称为raw
  * 命令：redis>SET story"Long, long ,long ,long age there lived a king..."

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587954103941-da8a7ddb-fb1d-4cd1-958f-249a45604af7.png)

* 如果字符串对象保存的是一个**字符串值，且长度小于32字节**
  * 使用SDS保存这个字符串值，但是和raw编码方式有所不同
  * 使用embstr编码方式保存，是一种专门用于短字符串的优化编码方式，并且存储数据的结构redisObject紧挨着
  * encoding记录的编码类型编程REDIS_ENCODING_EMBSTR，编码方式称为embstr
  * 命令 ： redis>SET msg "hello";
  * ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993940697-2e51bb70-10b2-4641-a270-fa600b6fcd74.png)
  * 优点
    * raw编码调用两次内存分配函数创建redisObject结构和sdshdr结构，而embstr编码则通过调用一次内存分配函数来分配一块连续的空间，如下图所示![image-20201212143530458](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201212143530458.png)



* **其他类型数据的保存方式**
  * 浮点类型：raw或embstr
  * 长度过长的数：raw或embstr



* **编码的转换**
  * 对int编码方式的字符串对象使用APPEND命令后会将其编码方式转换成raw
  * 命令：![image-20201212144933826](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201212144933826.png)





#### 列表对象（list）

* 列表对象的编码可以是ziplist 或者linkedlist

* ziplist

  * 使用压缩列表作为底层实现，每个压缩列表节点（entry）保存了一个列表元素

  * 命令：redis>RPUSH numbers 1"three" 5

    ​	(integer) 3

    ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587993968531-77b105a3-ec48-473a-9939-cdc663285f42.png)

  * linkedlist

    * 使用双端链表作为底层实现，每个双端链表节点（node）都保存一个**字符串对象**（图中省略了字符串对象的部分内容，字符串对象是唯一一个被其他四种对象嵌套的对象）
    * ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587994009697-64c6a7c9-e1c7-4f5a-a5d6-24e9a0a7c530.png)

**编码的转换**

* 当列表对象可以同时满足以下两个条件时，列表对象使用ziplist编码：
  * 列表对象保存的所有字符串元素的长度都小于64字节
  * 列表对象保存的元素数量小于512个
* 不满足上述条件的转换为linkedlist编码



#### 												哈希对象

* 哈希对象的编码可以时ziplist或者hashtable
* ziplist
  * 使用压缩列表作为底层实现，每当有新的键值对要加入压缩列表尾
  * ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587994068009-06ebfc2d-aa3a-4cd7-9015-e0a97f4112d3.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

* hashtable
  * 使用字典作为底层实现，字典的每一个键/值都是一个字符串对象，对象中保存了键值对的键/值
  * ![image-20201212155233204](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201212155233204.png)

* 编码转换
  * ziplist编码：每一个键值对的字符串长度都小于64字节且键值对数量小于512个
  * 否则使用hashtable



#### 															集合对象

* 集合对象的编码可以时intset或者hashtable
* intset
  * 使用整数集合作为底层实现，集合对象包含的所有元素都保存在整数集合里
  * 命令：redis>SADD number 1 3 5
  * ![img](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587990673367-d968141a-a188-4717-bf63-33ef75774a41.png)
* hashtable
  * 使用字典作为底层实现，字典每一个键都是一个字符串对象，每一个字符串对象都包含一个集合元素，而字典的值全部被设置为NULL
  * 命令：redis>SADD fruits "apple" "banana" "cherry"
  * ![img](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587991521620-1ee023c5-b14d-4509-b800-bc01cb15da55.png)

* 编码转换
  * intset:保存的所有元素都是整数值且元素数量不超过512个
  * 否则使用hashtable编码



#### 											有序集合对象（zset）

* 有序集合的编码可以是ziplist或者skiplist
* ziplist
  * 使用压缩列表作为底层实现，每一个集合元素都是用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员（member），第二个元素保存元素的分值（score）
  * 命令：reids>ZADD price 8.5 apple 5.0 banana 6.0 cherry 
  * ![img](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587994176465-2c8be9c5-f82c-44ab-b9a3-b78783302d9c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

* skiplist
  * 使用zset结构作为底层实现，一个zset结构同时包含一个字典和一个跳跃表
  * zset结构中的zsl跳跃表的每个跳跃表节点的object属性保存的是有序集合的元素（字符串对象），跳跃表节点的score属性保存元素的分值
  * zset结构中的dict字典建立了成员到分值的映射，可以以O(1)的时间复杂度查询成员的分值

```c
typedef struct zset {
    zskiplist *zsl;
    dict *dict;
} zset;
```



* **为什么有序集合需要同时使用跳跃表和字典**
  * 跳跃表和字典中引用的对象都是同一个，所以不会浪费额外的内存
  * 为了同时兼顾O（1）复杂度查询成员分值和范围操作（有序性）

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1266758/1587996177318-7309f304-7ef1-4a7c-8740-1e44b4cf01b0.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_THVrYQ%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

* 实际上 上述 重复展示了各个元素的成员和分值，实际上，共享，不会造成任何数据重复





* 编码转换
  * ziplist：保存的元素数量小于128个且每一个元素成员都小于64字节
  * 否则使用skiplist





#### 												内存回收

* redisObject中refcount属性记录被引用次数

* 因为 C 语言并不具备自动的内存回收功能， 所以 Redis 在自己的对象系统中构建了一个引用计数技术实现的内存回收机制

* 对象的引用计数信息会随着对象的使用状态而不断变化：

  - - 在创建一个新对象时， 引用计数的值会被初始化为1
    - 当对象被一个新程序使用时， 它的引用计数值会被增一
    - 当对象不再被一个程序使用时， 它的引用计数值会被减一
    - 当对象的引用计数值变为 0 时， 对象所占用的内存会被释放



#### 													对象共享

* 除了用于实现引用计数内存回收机制之外，对象的引用计数属性还带有对象共享的作用

* 在 Redis 中， 让多个键共享同一个值对象需要执行以下两个步骤：

  * 将数据库键的值指针指向一个现有的值对象；
  * 将被共享的值对象的引用计数增一

  

  -  Redis 会在初始化服务器时， 创建一万个字符串对象， 这些对象包含了从 0 到 9999 的所有整数值， 当服务器需要用到值为 0到 9999 的字符串对象时， 服务器就会使用这些共享对象， 而不是新创建对象

  

  * 数据结构中 嵌套了字符串对象 的  对象 ，都可以使用共享对象

  

  

  

  #### 													对象的空转时长

  * redisObject中lru属性（Least Recent Used）记录对象最后一次被命令程序访问的时间
  * 命令：OBJECT IDLETIME msg 输出对象的lru属性，且不会更新这个属性
  * 如果服务器打开了maxmemory选项，并且回收内存算法为volatile-lru或alkeys-lru，那么占用内存超过maxmemorty时，空转时间较长的那部分键会优先被服务器释放，从而回收内存
  * 

  