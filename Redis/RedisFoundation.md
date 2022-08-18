# Redis基础
### Redis是什么
* Redis，英文全称是 Remote Dictionary Server（远程字典服务），是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API，与 MySQL 数据库不同的是，Redis 的数据是存在内存中的。它的读写速度非常快，每秒可以处理超过 10 万次读写操作
* 广泛应用于缓存，另外，Redis 也经常用来做分布式锁;
---
### Redis高性能原理
![高性能](D:\LearningMakeMeHappy\Redis\static\Redis高性能.png)
1. ##### 基于内存实现  
   我们都知道内存读写是比磁盘读写快很多的。Redis是基于内存存储实现的数据库，相对于数据存在磁盘的数据库，就省去磁盘磁盘I/O的消耗。MySQL等磁盘数据库，需要建立索引来加快查询效率，而Redis数据存放在内存，直接操作内存，所以就很快。
2. ##### 高效的数据结构  
   ![](D:\LearningMakeMeHappy\Redis\static\Redis基础数据结构01.png)  
   SDS简单动态字符串  
   ![](D:\LearningMakeMeHappy\Redis\static\SDS简单动态字符串.png)  
   ````
   struct sdshdr { //SDS简单动态字符串
    int len;    //记录buf中已使用的空间
    int free;   // buf中空闲空间长度
    char buf[]; //存储的实际内容
   }
   ````
   1. 字符串长度处理  
   在C语言中，要获取捡田螺的小男孩这个字符串的长度，需要从头开始遍历，复杂度为O（n）; 在Redis中， 已经有一个len字段记录当前字符串的长度啦，直接获取即可，时间复杂度为O(1)。  
   2. 减少内存重新分配的次数  
   * 空间预分配  
   当SDS简单动态字符串修改和空间扩充时，除了分配必需的内存空间，还会额外分配未使用的空间。
   * 惰性空间释放  
   在C语言中，修改一个字符串，需要重新分配内存，修改越频繁，内存分配就越频繁，而分配内存是会消耗性能的。而在Redis中，SDS提供了两种优化策略：空间预分配和惰性空间释放。
3. ##### 合理的编码方式
   Redis支持多种数据基本类型，每种基本类型对应不同的数据结构，每种数据结构对应不一样的编码。为了提高性能，Redis设计者总结出，数据结构最适合的编码搭配。
   Redis是使用对象（redisObject）来表示数据库中的键值，当我们在 Redis 中创建一个键值对时，至少创建两个对象，一个对象是用做键值对的键对象，另一个是键值对的值对象。
   ````
   typedef struct redisObject{
    //类型
   unsigned type:4;
   //编码
   unsigned encoding:4;
   //指向底层数据结构的指针
   void *ptr;
    //...
   }robj;
   ````
   redisObject中，type 对应的是对象类型，包含String对象、List对象、Hash对象、Set对象、zset对象。encoding 对应的是编码。

* String：如果存储数字的话，是用int类型的编码;如果存储非数字，小于等于39字节的字符串，是embstr；大于39个字节，则是raw编码。
* List：如果列表的元素个数小于512个，列表每个元素的值都小于64字节（默认），使用ziplist编码，否则使用linkedlist编码
* Hash：哈希类型元素个数小于512个，所有值小于64字节的话，使用ziplist编码,否则使用hashtable编码。
* Set：如果集合中的元素都是整数且元素个数小于512个，使用intset编码，否则使用hashtable编码。
* Zset：当有序集合的元素个数小于128个，每个元素的值小于64字节时，使用ziplist编码，否则使用skiplist（跳跃表）编码
   

4. ##### 合理的线程模型
* 采用单线程模型，减少了多线程上线文切换、锁竞争带来的消耗
* I/O多路复用，非阻塞方式不在网络IO浪费过多的时间
5. ##### 虚拟内存机制
---
### Redis常用数据结构
Redis 有以下这五种基本类型：
* String（字符串）
* Hash（哈希）
* List（列表）
* Set（集合）
* zset（有序集合）
![Redis基础数据结构](D:\LearningMakeMeHappy\Redis\static\Redis基础数据类型.png)

它还有三种特殊的数据结构类型
* Geospatial
* Hyperloglog
* Bitmap
---
### Redis缓存常见问题
* 缓存穿透
* 缓存击穿
* 缓存雪崩
  ### 
