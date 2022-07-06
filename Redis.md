---
title: Redis
categories:
  - Redis
tags:
  - Redis
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp6.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp6.jpg'
abbrlink: 27273
date: 2022-01-30 23:56:38
updated: 2022-4-10 23:37:17
---

>参考视频：【尚硅谷】Redis 6 入门到精通超详细教程  https://www.bilibili.com/video/BV1Rv41177Af

# NoSQL数据库简介

## 技术发展

技术的分类

1. 解决功能性的问题：Java、Jsp、RDBMS、Tomcat、HTML、Linux、JDBC、SVN

2. 解决扩展性的问题：Struts、Spring、SpringMVC、Hibernate、Mybatis

3. 解决性能的问题：NoSQL、Java线程、Hadoop、Nginx、MQ、ElasticSearch

Web1.0的时代，数据访问量很有限，用一夫当关的高性能的单点服务器可以解决大部分问题。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131121351.png)

随着Web2.0的时代的到来，用户访问量大幅度提升，同时产生了大量的用户数据。加上后来的智能移动设备的普及，所有的互联网平台都面临了巨大的性能挑战。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131121400.png)

解决CPU及内存压力：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131121438.png)

解决IO压力：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131121501.png)

## NoSQL数据库

NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指**非关系型的数据库**。 

NoSQL 不依赖业务逻辑方式存储，而以简单的`key-value`模式存储。因此大大的增加了数据库的扩展能力。

- 不遵循SQL标准。

- 不支持[ACID](https://baike.baidu.com/item/acid/10738?fr=aladdin)。

- 远超于SQL的性能。

**适用场景：**

- 对数据高并发的读写

- 海量数据的读写

- 对数据高可扩展性的

**不适用场景：**

- 需要事务支持

- 基于sql的结构化查询存储，处理复杂的关系,需要[即席查询](https://baike.baidu.com/item/%E5%8D%B3%E5%B8%AD%E6%9F%A5%E8%AF%A2/2886987?fr=aladdin)。

>用不着sql的和用了sql也不行的情况，请考虑用NoSql

---

常见NoSQL数据库：

- Memcache：

  - 很**早**出现的NoSql数据库

  - 数据都在内存中，一般**不持久化**

  - 支持简单的key-value模式，**支持类型单一**

  - 一般是作为**缓存数据库**辅助持久化的数据库

- Redis：

  - 几乎覆盖了Memcached的绝大部分功能
  - 数据都在内存中，**支持持久化**，主要用作备份恢复
  - 除了支持简单的key-value模式，还**支持多种数据结构的存储**，比如 list、set、hash、zset等
  - 一般是作为**缓存数据库**辅助持久化的数据库

- MongoDB：

  - 高性能、开源、模式自由(schema  free)的**文档型数据库**
  - 数据都在内存中， 如果内存不足，把不常用的数据保存到硬盘
  - 虽然是key-value模式，但是对value（尤其是**json**）提供了丰富的查询功能
  - 支持二进制数据及大型对象
  - 可以根据数据的特点替代**RDBMS** ，成为独立的数据库。或者配合[RDBMS](https://baike.baidu.com/item/%E5%85%B3%E7%B3%BB%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F/11032386?fromtitle=RDBMS&fromid=1048260&fr=aladdin)，存储特定的数据。

> DB-Engines 数据库排名：https://db-engines.com/en/ranking

## 行式存储数据库（大数据时代）

行式数据库：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131123356.png)

---

列式数据库：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131123418.png)



- HBase：
  - HBase是**Hadoop**项目中的数据库。它用于需要对大量的数据进行随机、实时的读写操作的场景中。HBase的目标就是处理数据量**非常庞大**的表，可以用**普通的计算机**处理超过**10亿行数据**，还可处理有数百万**列**元素的数据表。

- Cassandra：
  - Apache Cassandra是一款免费的开源NoSQL数据库，其设计目的在于管理由大量商用服务器构建起来的庞大集群上的**海量数据集(数据量通常达到PB级别)**。在众多显著特性当中，Cassandra最为卓越的长处是对写入及读取操作进行规模调整，而且其不强调主集群的设计思路能够以相对直观的方式简化各集群的创建与扩展流程。

---

## 图关系型数据库

主要应用：社会关系，公共交通网络，地图及网络拓谱(n*(n-1)/2)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131124227.png)



## 计算机存储单位

**位 bit** (比特)(Binary Digits)：存放一位二进制数，即 0 或 1，最小的存储单位。

**字节 byte**：8个二进制位为一个字节(B)，最常用的单位。

计算机存储单位一般用B，KB，MB，GB，TB，EB，ZB，YB，BB来表示，它们之间的关系是：

1KB (Kilobyte 千字节)=1024B，

1MB (Megabyte 兆字节 简称“兆”)=1024KB，

1GB (Gigabyte 吉字节 又称“千兆”)=1024MB，

1TB (Trillionbyte 万亿字节 太字节)=1024GB，其中1024=2^10 ( 2 的10次方)，

**1PB**（Petabyte 千万亿字节 拍字节）=1024TB，

1EB（Exabyte 百亿亿字节 艾字节）=1024PB，

1ZB (Zettabyte 十万亿亿字节 泽字节)= 1024 EB,

1YB (Jottabyte 一亿亿亿字节 尧字节)= 1024 ZB,

1BB (Brontobyte 一千亿亿亿字节)= 1024 YB.

注：“兆”为百万级数量单位。



# Redis概述安装

## 相关知识

- Redis是一个C语言实现的开源**key-value**存储系统，默认端口是**6379**。

- 和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。

- 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，这些操作都是**原子性**的。

- 在此基础上，Redis支持各种不同方式的**排序**。

- 与memcached一样，为了保证效率，数据都是**缓存在内存**中。区别的是Redis会**周期性**的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。并且在此基础上实现了**master-slave(主从)**同步。

- 默认16个数据库，类似数组下标从0开始，初始默认使用0号库，使用命令 select  <dbid>来切换数据库。统一密码管理，所有库同样密码。

---

**原子性：**有一个失败则都失败

**原子操作**是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。

（1）在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。

（2）在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。

Redis单命令的原子性主要得益于Redis的单线程。

---

Redis是**单线程+多路IO复用技术**

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）

串行  vs  多线程+锁（Memcache） vs  单线程+多路IO复用(Redis)

> Redis与Memcache三点不同: 支持多数据类型，支持持久化，单线程+多路IO复用

## 应用场景

配合关系型数据库做高速缓存：

- 高频次，热门访问的数据，降低数据库IO
- 分布式架构，做session共享

多样的数据结构存储持久化数据：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220131125146.png)

## 安装

| Redis官方网站   | Redis中文官方网站 |
| --------------- | ----------------- |
| http://redis.io | http://redis.cn/  |

- 在官网下载压缩包：redis-6.2.6.tar.gz

- 通过xftp软件将压缩包上传到linux

- 安装C语言的编译环境以及gcc编译器
  - `yum install centos-release-scl scl-utils-build`
  - `yum install -y devtoolset-8-toolchain`
  - `scl enable devtoolset-8 bash`
- 测试gcc版本：`gcc --version`
  - 低版本可能安装不了redis：gcc (GCC) 8.3.1 20190311 (Red Hat 8.3.1-3)
- 解压至opt目录下：`tar -zxvf redis-6.2.6.tar.gz -C /opt/`
- 进入redis目录：`cd /opt/redis-6.2.6/`
- 执行`make`命令进行编译，成功后再执行`make install`进行安装

## 查看默认安装目录

查看默认安装目录：

```
[root@nanzx redis-6.2.6]# cd /usr/local/bin
[root@nanzx bin]# ls
docker-compose  redis-benchmark  redis-check-aof  redis-check-rdb  redis-cli  redis-sentinel  redis-server
```

redis-benchmark：性能测试工具，可以在自己电脑运行，看看自己电脑性能如何

redis-check-aof：修复有问题的AOF文件

redis-check-dump：修复有问题的dump.rdb文件

redis-sentinel：Redis集群使用

redis-server：Redis服务器启动命令

redis-cli：客户端，操作入口

## 启动及关闭

前台启动，命令行窗口不能关闭，否则服务器停止，`ctrl+C`退出服务：

```shell
[root@nanzx bin]# redis-server
```

后台启动，**推荐**：

```shell
[root@nanzx bin]# cd /opt/redis-6.2.6/

## 备份redis配置文件
[root@nanzx redis-6.2.6]# cp redis.conf redis.conf.bak

## 修改redis.conf(257行)文件将里面的daemonize no改成 yes，让服务在后台启动
## 为了后面远程操作才可以执行
## 将本机访问保护模式(94行)protected-mode yes改成no
## 注释掉(75行)bind 127.0.0.1 -::1，无限制接受任何ip地址的访问
## 进入后可以输入：/daemon 进行查找
[root@nanzx redis-6.2.6]# vim redis.conf

## 指定配置文件启动
[root@nanzx redis-6.2.6]# redis-server redis.conf
## 查看redis进程是否启动成功
[root@nanzx redis-6.2.6]# ps -ef|grep redis
root      39242      1  0 23:27 ?        00:00:00 redis-server 127.0.0.1:6379
root      39336  15470  0 23:28 pts/0    00:00:00 grep --color=auto redis

## 用客户端访问并进行测试验证：
[root@nanzx redis-6.2.6]# redis-cli 
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>

## 可以在客户端内关闭：
[root@nanzx redis-6.2.6]# redis-cli 
127.0.0.1:6379> shutdown
not connected> 

## 也可以这样单实例关闭：
[root@nanzx redis-6.2.6]# redis-cli shutdown
```



# 常用五大数据类型

redis常见数据类型操作命令：http://www.redis.cn/commands.html

**Redis键(key)操作：**

- keys *：查看当前库所有key   (匹配：keys *1)
- exists key：判断某个key是否存在
- type key： 查看key是什么类型，例如string，list等
- object encoding key：查看数据结构，例如list包含了linkedlist和ziplist两种数据结构。
- del key：删除指定的key数据
- unlink key：根据value选择非阻塞删除，仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。
- expire key 10 ：为给定的key设置过期时间为10秒钟
- ttl key：查看还有多少秒过期，-1表示永不过期，-2表示已过期，大于0表示剩余过期时间
- select 1：切换数据库为索引为1的数据库
- dbsize：查看当前数据库的key的数量
- flushdb：清空当前库
- flushall：通杀全部库

> 返回值nil表示空

## 字符串(String)

String是Redis最基本的类型，一个key对应一个value。

String类型是**二进制安全**的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是**512M**。

**内部编码：**raw、int、embstr

**常用命令：**

- set  <key><value>：添加键值对，如果 key 已经保存了一个值，那么这个操作会直接覆盖原来的值。提供选项操作：
  - ex seconds：为键设置秒级过期时间
  - px milliseconds： 为键设置毫秒级过期时间
  - nx：键必须不存在才可以设置成功，用于添加
  - xx：键必须存在才可以设置成功，用于更新

- get  <key>：查询对应键值
- append  <key><value>：将给定的<value> 追加到原值的末尾
- strlen  <key>：获得值的长度
- setnx  <key><value>：只有在 key 不存在时才设置 key 的值
- incr  <key>：将 key 中储存的数字值增1，只能对数字值操作，如果为空，新增值为1
- decr  <key>：将 key 中储存的数字值减1，只能对数字值操作，如果为空，新增值为-1
- incrby / decrby  <key><步长>：将 key 中储存的数字值增减。自定义步长。
- mset  <key1><value1><key2><value2> ..... ：同时设置一个或多个键值对  
- mget  <key1><key2><key3> .....：同时获取一个或多个 value  
- msetnx  <key1><value1><key2><value2> ..... ：同时设置一个或多个键值对，当且仅当所有给定key都不存在时才设置。
- getrange  <key><起始位置><结束位置>：获得值的范围，位置**前包后包，索引从0开始**
- setrange  <key><起始位置><value>：用 <value>  覆写<key>所储存的字符串值，从<起始位置>开始。
- setex  <key><过期时间><value>：设置键值的同时，设置过期时间，单位秒。
- getset <key><value>：以新换旧，设置了新值同时获得旧值。

**数据结构：**

String的数据结构为简单动态字符串(Simple Dynamic String，缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用**预分配冗余空间**的方式来减少内存的频繁分配。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220201160415.png)

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

## 列表(List)

Redis 列表是简单的字符串列表，特点是**单键多值**，按照**插入顺序**排序。

你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个**双向链表**，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203150003.png)

**常用命令：**

- lpush/rpush  <key><value1><value2><value3>.... ：从左边/右边插入一个或多个值。

- lpop/rpop  <key>：从左边/右边**吐**出一个值。**值在键在，值光键亡**。
- rpoplpush  <key1><key2>：从<key1>列表右边**吐**出一个值，插到<key2>列表左边。
- lrange <key><start><stop>：按照索引下标获得元素(从左到右)
  - lrange mylist 0 -1  0左边第一个，-1右边第一个，（0-1表示获取所有）

- lindex <key><index>：按照索引下标获得元素(从左到右)

- llen <key>：获得列表长度 
- linsert <key>  before <value><newvalue>：在<value>的前面插入<newvalue>插入值

- lrem <key><n><value>：从左边删除n个value(从左到右)

- lset<key><index><value>：将列表key下标为index的值替换成value

**数据结构：**

List的数据结构为快速链表quickList。

首先在列表元素较少的情况下会使用一块**连续**的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203154012.png)

Redis将**链表和ziplist结合起来组成了quicklist**。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

## 集合(Set)

Redis set对外提供的功能与list类似都是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的**无序集合**。它底层其实是一个value为null的hash表，所以添加、删除和查找的复杂度都是O(1)。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变。

**常用命令：**

- sadd <key><value1><value2>..... ：将一个或多个元素加入到集合key中，已经存在的元素将被忽略

- smembers <key>：取出该集合的所有值。

- sismember <key><value>：判断集合<key>是否为含有该<value>值，有1，没有0

- scard<key>：返回该集合的元素个数。
- srem <key><value1><value2> .... ：删除集合中的某个元素。
- spop <key>：随机从该集合中**吐**出一个值。
- srandmember <key><n>：随机从该集合中取出n个值。不会从集合中删除 。
- smove <source><destination><value>：把集合中一个值从一个集合移动到另一个集合
- sinter <key1><key2>：返回两个集合的交集元素。
- sunion <key1><key2>：返回两个集合的并集元素。
- sdiff <key1><key2>：返回两个集合的差集元素(key1中的，不包含key2中的)

**数据结构：**

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

## 哈希(Hash)

Redis hash 是一个键值对集合，是一个string类型的field和value的映射表，hash特别适合用于**存储对象**。类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储主要有以下2种存储方式：

- 每次修改用户的某个属性需要，先反序列化改好后再序列化回去。开销较大。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203192359.png)

- 用户ID数据冗余

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203192407.png)

通过 key(用户ID) + field(**属性标签)** 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203192438.png)

**常用命令：**

- hset <key><field><value>：给<key>集合中的 <field>键赋值<value>

- hget <key1><field>：从<key1>集合<field>取出 value 

- hmset <key1><field1><value1><field2><value2>... ：批量设置hash的值

- hexists <key1><field>：查看哈希表 key 中，给定域 field 是否存在。 

- hkeys <key>：列出该hash集合的所有field

- hvals <key>：列出该hash集合的所有value

- hincrby <key><field><increment>：为哈希表 key 中的域 field 的值加上增量

- hsetnx <key><field><value>：将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .

**数据结构：**

Hash类型对应的数据结构有两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

## 有序集合(Zset)

Redis有序集合zset与普通集合set非常相似，是一个**没有重复元素**的字符串集合。

不同之处是有序集合的每个成员都关联了一个评分（score），这个评分（score）**默认**被用来按照从最低分到最高分的方式排序集合中的成员。**集合的成员是唯一的，但是评分可以是重复了 。**

因为元素是有序的，所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表。

**常用命令：**

- zadd  <key><score1><value1><score2><value2>…：将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

- zrange <key><start><stop>  [WITHSCORES] ：返回有序集 key 中，下标在<start><stop>之间的元素[**全包含的区间**]
  - 带WITHSCORES，可以让分数一起和值返回到结果集。

- zrangebyscore <key><min><max> [withscores] [limit offset count]：返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 
- zrevrangebyscore <key><max><min> [withscores] [limit offset count] ：同上，改为从大到小排列。 

- zincrby <key><increment><value>：为元素的score加上增量
- zrem  <key><value>：删除该集合下，指定值的元素 
- zcount <key><min><max>：统计该集合，分数区间内的元素个数 

- zrank <key><value>：返回该值在集合中的排名，从0开始

**数据结构：**

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

**跳跃表（跳表）：**

1、简介

​	有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

2、实例

​	对比有序链表和跳跃表，从链表中查询出51

（1） 有序链表要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203213430.png)

（2） 跳跃表

从第2层开始，1节点比51节点小，向后比较。

21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

在第0层，51节点为要查找的节点，节点被找到，共查找4次。

从此可以看出跳跃表比有序链表效率要高

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220203213440.png)

# 新数据类型

## Bitmaps(位图)

**简介：**

现代计算机用二进制（位） 作为信息的基础单位， 1个字节等于8位， 例如“abc”字符串是由3个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的ASCII码分别是97、 98、 99， 对应的二进制分别是01100001、 01100010和01100011，如下图

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220206175611.png)

合理地使用操作位能够有效地提高内存使用率和开发效率。

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

- Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的**位**进行操作。

- Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做**偏移量**。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220206175745.png)

---

**命令：**

`setbit<key><offset><value>`：设置Bitmaps中某个偏移量的值（0或1），offset偏移量从0开始1

**实例：**每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id。

设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220206180546.png)

unique:users:20220206代表2022-02-06这天的独立访问用户的Bitmaps：

```shell
127.0.0.1:6379> setbit unique:users:20220206 1 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20220206 6 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20220206 11 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20220206 15 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20220206 19 1
(integer) 0
```

> **注意：**
>
> 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。
>
> 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

---

**命令：**

`getbit<key><offset>`：获取Bitmaps中某个偏移量的值，获取键的第offset位的值（从0开始算）

**实例：**

获取id=8的用户是否在2022-02-06这天访问过， 返回0说明没有访问过：

```shell
127.0.0.1:6379> getbit unique:users:20220206 8
(integer) 0
127.0.0.1:6379> getbit unique:users:20220206 1
(integer) 1
127.0.0.1:6379> getbit unique:users:20220206 100
(integer) 0
```

---

**命令：**

`bitcount<key>[start end]` ：统计字符串被设置为1的bit数，可以通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个字节，而 -2 表示倒数第二个字节，start、end 是指bit组的**字节**的下标数，二者皆**包含**。

> 注意：redis的setbit设置或清除的是bit位置，而bitcount计算的是**byte位置**。一个字节等于八位。

**实例：**

计算2022-02-06这天的独立访问用户数量：

```shell
127.0.0.1:6379> bitcount unique:users:20220206
(integer) 5
```

start和end代表起始和结束字节数， 下面操作计算用户id在第1个字节到第3个字节之间的独立访问用户数， 对应的用户id是11， 15， 19。

```shell
127.0.0.1:6379> bitcount unique:users:20220206 1 3
(integer) 3
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220206180546.png)

举例： K1 【01000001 01000000  00000000 00100001】，一个字节等于八位

- `bitcount K1 1 2`： 统计下标第1个字节到第2字节中bit=1的个数，即01000000  00000000 --》1

- `bitcount K1 1 3 `： 统计下标第1个字节到第2字节中bit=1的个数，即01000000  00000000 00100001 --》3

- `bitcount K1 0 -2`： 统计下标第0个字节到下标倒数第2字节中bit=1的个数，即01000001  01000000  00000000 --》3

----

**命令：**

`bitop and(or/not/xor) <destkey> [key…]`：bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。【[菜鸟教程-位运算](https://www.runoob.com/w3cnote/bit-operation.html)】

**实例：**

```shell
2022-02-06 日访问网站的userid=1，2，5，9。
setbit unique:users:20220206 1 1
setbit unique:users:20220206 2 1
setbit unique:users:20220206 5 1
setbit unique:users:20220206 9 1

2022-02-07 日访问网站的userid=0，1，4，9。
setbit unique:users:20220207 0 1
setbit unique:users:20220207 1 1
setbit unique:users:20220207 4 1
setbit unique:users:20220207 9 1

计算出两天都访问过网站的用户数量:
> bitop and unique:users:and:20220206_07 unique:users:20220206 unique:users:20220207
(integer) 2
> bitcount unique:users:and:20220206_07
(integer) 2

计算出任意一天有访问过网站的用户数量（例如月活跃就是类似这种），可以使用or求并集
```

---

**Bitmaps与set对比：**

假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表

| set和Bitmaps存储一天活跃用户对比 |                    |                  |                        |
| -------------------------------- | ------------------ | ---------------- | ---------------------- |
| 数据类型                         | 每个用户id占用空间 | 需要存储的用户量 | 全部内存量             |
| 集合类型                         | 64位               | 50000000         | 64位*50000000 = 400MB  |
| Bitmaps                          | 1位                | 100000000        | 1位*100000000 = 12.5MB |

很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的

| set和Bitmaps存储独立用户空间对比 |        |        |       |
| -------------------------------- | ------ | ------ | ----- |
| 数据类型                         | 一天   | 一个月 | 一年  |
| 集合类型                         | 400MB  | 12GB   | 144GB |
| Bitmaps                          | 12.5MB | 375MB  | 4.5GB |

但Bitmaps并不是万金油， 假如该网站每天的独立访问用户很少， 例如只有10万（大量的僵尸用户） ， 那么两者的对比如下表所示， 很显然， 这时候使用Bitmaps就不太合适了， 因为基本上大部分位都是0。

| set和Bitmaps存储一天活跃用户对比（独立用户比较少） |                    |                  |                        |
| -------------------------------------------------- | ------------------ | ---------------- | ---------------------- |
| 数据类型                                           | 每个userid占用空间 | 需要存储的用户量 | 全部内存量             |
| 集合类型                                           | 64位               | 100000           | 64位*100000 = 800KB    |
| Bitmaps                                            | 1位                | 100000000        | 1位*100000000 = 12.5MB |

## HyperLogLog

**简介：**

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量），可以使用Redis的incr、incrby轻松实现。

但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为**基数问题**。

解决基数问题有很多种方案：

（1）数据存储在MySQL表中，使用distinct count计算不重复个数

（2）使用Redis提供的hash、set、bitmaps等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致**占用空间**越来越大，对于非常大的数据集是不切实际的。

能否能够**降低一定的精度**来平衡存储空间？Redis推出了HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，**计算基数所需的空间**总是固定的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而**不会储存输入元素本身**，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

> 什么是基数?
>
> 比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素个数)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

**命令：**

- pfadd <key>< element> [element ...]： 添加指定元素到 HyperLogLog 中，如果执行命令后估计的近似基数发生变化，则返回1，否则返回0。
- pfcount <key> [key ...]：计算HLL的近似基数，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可。
- pfmerge <destkey><sourcekey> [sourcekey ...]：将一个或多个HLL合并后的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得。

## Geospatial(地理信息定位)

**简介：**

Redis 3.2 中增加了对GEO类型的支持。GEO(Geographic)是**地理信息**的缩写。该类型就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。

**命令：**

- geoadd <key><longitude><latitude><member> [longitude latitude member...]：添加地理位置（经度，纬度，名称）
  - 两极无法直接添加，一般会下载城市数据，直接通过 Java 程序一次性导入。
  - 有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。
  - 当坐标位置超出指定范围时，该命令将会返回一个错误。
  - 已经添加的数据，是无法再次往里面添加的。

- geopos  <key><member> [member...]：获得指定地区的坐标值

- geodist <key><member1><member2>  [m|km|ft|mi ]：获取两个位置之间的直线距离
  - m 表示单位为米[默认值]
  - km 表示单位为千米
  - mi 表示单位为英里
  - ft 表示单位为英尺
- georadius <key><longitude><latitude> radius m|km|ft|mi：以给定的经纬度为中心，找出某一半径内的元素



# Redis的发布和订阅

1、客户端可以订阅频道如下图

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220207173820.png)

2、当给这个频道发布消息后，消息就会发送给订阅的客户端

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220207173829.png)

**发布订阅命令行实现：**

1、 打开一个客户端订阅channel1

```shell
127.0.0.1:6379> 
127.0.0.1:6379> subscribe channel1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel1"
3) (integer) 1
```

2、打开另一个客户端，给channel1发布消息，返回的1是订阅者数量

```shell
127.0.0.1:6379> publish channel1 "hello world!"
(integer) 1
```

3、打开第一个客户端可以看到发送的消息

```
127.0.0.1:6379> subscribe channel1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel1"
3) (integer) 1
1) "message"
2) "channel1"
3) "hello world!"
```

**注意：**发布的消息没有持久化，只能收到订阅后发布的消息

# Java集成Jedis

引入Jedis依赖：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.1.1</version>
</dependency>
```

> 连接Redis注意事项：
>
> redis.conf中注释掉`bind 127.0.0.1` ，然后设置`protected-mode no`，开放6379端口或者关闭防火墙`systemctl stop/disable firewalld.service`

测试：

```java
    @Test
    void testJedis() {
        Jedis jedis = new Jedis("192.168.2.110", 6379);

        //Key操作
        System.out.println("Key操作--------------");
        jedis.set("k1", "v1");
        jedis.set("k2", "v2");
        jedis.set("k3", "v3");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys.size());
        for (String key : keys) {
            System.out.println(key);
        }
        System.out.println(jedis.exists("k1"));
        System.out.println(jedis.ttl("k1"));
        System.out.println(jedis.get("k1"));

        //String类型操作
        System.out.println("String类型操作--------------");
        jedis.mset("str1", "v1", "str2", "v2", "str3", "v3");
        System.out.println(jedis.mget("str1", "str2", "str3"));

        //List类型操作
        System.out.println("List类型操作--------------");
        jedis.lpush("mylist", "java", "php", "c++", "python");
        jedis.linsert("mylist", ListPosition.BEFORE, "c++", "c#");
        List<String> list = jedis.lrange("mylist", 0, -1);
        for (String element : list) {
            System.out.println(element);
        }

        //Set类型操作
        System.out.println("Set类型操作--------------");
        jedis.sadd("orders", "order01");
        jedis.sadd("orders", "order02");
        jedis.sadd("orders", "order02");
        jedis.sadd("orders", "order03");
        jedis.sadd("orders", "order04");
        Set<String> smembers = jedis.smembers("orders");
        for (String order : smembers) {
            System.out.println(order);
        }
        System.out.println("删除member");
        jedis.srem("orders", "order01", "order02", "order03");
        smembers = jedis.smembers("orders");
        for (String order : smembers) {
            System.out.println(order);
        }

        //Hash类型操作
        System.out.println("Hash类型操作--------------");
        jedis.hset("hash1", "userName", "lisi");
        System.out.println(jedis.hget("hash1", "userName"));
        Map<String, String> map = new HashMap<>();
        map.put("telephone", "13810169999");
        map.put("address", "nanzx.top");
        map.put("email", "abc@163.com");
        jedis.hmset("hash2", map);
        List<String> result = jedis.hmget("hash2", "telephone", "email");
        for (String element : result) {
            System.out.println(element);
        }

        //Zset类型操作
        System.out.println("Zset类型操作--------------");
        jedis.zadd("zset01", 100d, "z3");
        jedis.zadd("zset01", 90d, "l4");
        jedis.zadd("zset01", 80d, "w5");
        jedis.zadd("zset01", 70d, "z6");
        List<String> zrange = jedis.zrange("zset01", 0, -1);
        for (String e : zrange) {
            System.out.println(e);
        }

        jedis.close();
    }
```



# 事务和锁机制

## 简介

Redis事务的主要作用就是**串联多个命令**防止别的命令插队。

**Redis事务三特性：**

- 单独的隔离操作 
  - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

- 没有隔离级别的概念 
  - 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

- 不保证原子性 
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 

## Multi、Exec、discard命令

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212001323.png)

组队成功，提交成功：

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
```

组队阶段报错，提交失败：

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set e2
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> keys *
(empty array)
```

组队成功，提交有成功有失败情况：

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> incr k1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
```

## 事务的错误处理

组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212113409.png)

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212113425.png)

## 事务冲突问题

一个请求想给金额减8000，一个请求想给金额减5000，一个请求想给金额减1000

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212115249.png)

**悲观锁**(Pessimistic Lock)：

顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212115540.png)

**乐观锁**(Optimistic Lock)：

顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。**Redis就是利用这种check-and-set机制实现事务的**。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220212115548.png)

## Watch命令

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

```shell
127.0.0.1:6379> get k
"0"
127.0.0.1:6379> watch k
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> incrby k 20
QUEUED
127.0.0.1:6379(TX)> exec
(nil)
127.0.0.1:6379> get k
"-20"
```

在执行exec前新建窗口执行以下命令：

```shell
127.0.0.1:6379> decrby k 20
(integer) -20
```

执行exec后返回nil说明事务被打断，incrby的命令没有被执行。

unwatch命令：取消 WATCH 命令对所有 key 的监视。

如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。

---

# Redis持久化

## RDB（Redis DataBase）

在指定的**时间间隔**内将内存中的数据集**快照**写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

---

### 持久化流程

Redis会单独**创建（fork）一个子进程**来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个**临时文件替换上次持久化好的文件**（默认名称是dump.rdb，与`redis-server`同个目录）。 整个过程中主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是**最后一次持久化后的数据可能丢失**。

RDB 触发机制分为使用指令（`SAVE` 和 `BGSAVE`）手动触发和 redis.conf 配置(`save 3600 1 `)自动触发。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220213222047.png)

**扩展：**

- SAVE 和 BGSAVE 两个命令都会调用 rdbSave 函数，但它们调用的方式各有不同：

  - SAVE 直接调用 rdbSave ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

  - BGSAVE 则 fork 出一个子进程，子进程负责调用 rdbSave ，并在保存完成之后向主进程发送信号，通知保存已完成。 Redis 服务器在BGSAVE 执行期间仍然可以继续处理客户端的请求。

  - 可以通过lastsave 命令获取最后一次成功执行快照的时间

- 执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义。
- 内存中的数据对象<==rdbLoad==磁盘中的RDB文件，内存中的数据对象==rdbSave==>磁盘中的RDB文件

---

### Fork进程

- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程

- 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后都会被exec系统调用，出于效率考虑，Linux中引入了“**写时复制技术**”

- 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

---

### 相关配置

RDB相关配置在redis.conf文件中的SNAPSHOTTING模块下：

```shell
################################ SNAPSHOTTING  ################################
# 将数据库保存到磁盘.
# 格式：save <秒> <更新>
# 如果指定的秒数和数据库写操作次数都满足了就将数据库保存。 

# 使用一个空字符串参数或者注释掉“save”这一行配置项可以完全禁用快照，如下例所示：
save ""

# 下面是保存操作的实例：  
# 3600秒（1小时）内至少1个key值改变（则进行数据库保存--持久化）  
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化）  
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）  
save 3600 1
save 300 100
save 60 10000

# 当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes
# 如果后台存储（持久化）操作进程再次工作，Redis会自动允许更新操作。  
stop-writes-on-bgsave-error yes

# 对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。
# 如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。推荐yes
rdbcompression yes

# 在存储快照后，redis使用CRC64算法来进行数据校验。推荐yes
# 但是在存储或者加载rbd文件时会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
# 没有校验的RDB文件会有一个0校验位，来告诉加载代码跳过校验检查。  
rdbchecksum yes

# sanitize-dump-payload no

# 导出数据库的文件名称  
dbfilename dump.rdb

rdb-del-sync-files no

# rdb文件的保存路径。默认为Redis启动时命令行所在的目录下
dir ./
```

### 优缺点

- 优点：

  - 适合大规模的数据恢复
  - 对数据完整性和一致性要求不高更适合使用
  - 节省磁盘空间
  - 恢复速度快
- 缺点：

  - Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

  - 虽然Redis在fork时使用了**写时拷贝技术**，但是如果数据庞大时还是比较消耗性能，造成服务器在某某毫秒内停止处理客户端。

  - 在备份周期内做一次备份，如果Redis意外down掉且备份未完成时，就会丢失最后一次快照后的所有修改。

## AOF（Append Of File）

以日志的形式来记录每个写操作（**增量保存**），将Redis执行过的所有写指令记录下来(**读操作不记录**)， 只许**追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

### 持久化流程

（1）客户端的请求**写命令**会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新加载AOF文件中的写操作达到数据恢复的目的；

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220214225422.png)

### 相关配置

AOF相关配置在redis.conf文件中的APPEND ONLY MODE模块下：

```shell
############################## APPEND ONLY MODE ###############################
# no表示关闭AOF，默认不开启
appendonly no

# 默认追加命令的文件名称
appendfilename "appendonly.aof"


# always：始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好。最安全。
# everysec：每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。默认配置。
# no：redis不主动进行同步，把同步时机交给操作系统，在需要的时候刷新数据即可。最快。
# appendfsync always
appendfsync everysec
# appendfsync no

#重写配置参考下面
no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

#如果aof load truncated设置为yes，则会加载一个被截断的aof文件，Redis服务器开始发送日志，通知用户该事件。
aof-load-truncated yes

#当重写AOF文件时，Redis能够在AOF文件中使用RDB前导，以更快地重写和恢复。启用此选项后，重写的AOF文件由两个不#同的节组成：[RDB文件][AOF尾部]
#加载时，Redis识别出AOF文件以“Redis”字符串开头，并加载带前缀的RDB文件，然后继续加载AOF尾部。
aof-use-rdb-preamble yes
```

### Rewrite重写压缩

**简介**

AOF采用文件追加方式，文件会越来越大。为避免出现此种情况，新增了**重写机制**，当AOF文件大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集。可以手动执行`bgrewriteaof`命令进行重写。

**重写原理：**

- AOF文件持续增长而过大时，会fork出一个新进程来将文件重写（也是先写临时文件最后再rename）
  - redis4.0版本后的重写是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。对应配置文件中的：`aof-use-rdb-preamble yes`

- 如果配置文件中 `no-appendfsync-on-rewrite yes` ，表示不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）

- 如果配置文件中`no-appendfsync-on-rewrite no`，则会写入aof文件，往磁盘存储，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）

 **重写流程：**

1. `bgrewriteaof`触发重写，判断当前是否有`bgsave`或`bgrewriteaof`在运行，如果有则等待该命令结束后再继续执行。

2. 主进程fork出子进程执行重写操作，保证主进程不会阻塞。

3. 子进程遍历redis内存中数据到临时文件，客户端的写请求**同时写入**aof_buf缓冲区和aof_rewrite_buf重写缓冲区，保证原AOF文件完整以及新AOF文件生成期间的新的数据修改动作不会丢失。

4. 子进程写完新的AOF文件后，向主进程发信号，父进程更新统计信息。主进程把aof_rewrite_buf中的数据写入到新的AOF文件。

5. 使用新的AOF文件覆盖旧的AOF文件，完成AOF重写。即使` bgrewriteaof `执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 Bgrewriteaof 成功之前不会被修改。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220215001207.png)

**重写触发机制：**

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 

- Redis会记录上次重写时的AOF文件大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍**且**文件大于64M时触发。

- `auto-aof-rewrite-percentage`：设置重写的基准值，文件达到100%时开始重写（现在文件是原来重写后文件的2倍时触发）

- `auto-aof-rewrite-min-size`：设置重写的基准值，最小文件64MB。达到这个值开始重写。

- 例如文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

- 系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size，如果Redis的AOF文件当前大小>= base_size +base_size*100% (默认)**且**当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。 

### 优缺点

- 优点：

  - 备份机制更稳健，丢失数据概率更低。
  - 可读的日志文本，通过操作AOF稳健，可以处理失误操作。
- 缺点：

  - 比起RDB占用更多的磁盘空间。
  - 恢复备份速度要慢。
  - 每次读写都同步的话，有一定的性能压力。

## 总结

- Redis默认持久化策略是RDB

- AOF默认不开启

- AOF和RDB同时开启，系统默认加载AOF的数据
- 如果对数据不敏感，可以选单独用RDB
- 不建议单独用 AOF，因为可能会出现Bug
- 如果只是做纯内存缓存，可以都不用

**官网建议：**

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储

- AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以**redis协议**追加保存每次写的操作到文件末尾

- Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大

- 只做缓存：如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式

- 同时开启两种持久化方式：

  - 当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。同时使用两者时服务器重启也只会找AOF文件。
  - 建议不要只使用AOF，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)， 快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。


# Redis主从复制

## 简介

Redis主从复制（master/slaver机制），是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)，数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave以读为主。

默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

**作用：**

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

4. 读写分离：可以用于实现读写分离，主库写、从库读，读写分离不仅可以提高服务器的负载能力，同时可根据需求的变化，改变从库的数量；

5. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220215223044.png)

## 搭建一主多从

（1）创建/myredis文件夹：`mkdir /myredis`

（2）复制redis.conf配置文件到文件夹中：`cp /opt/redis-6.2.6/redis.conf /myredis/`

（3）创建三个配置文件并进行如下配置：

redis6379.conf：

```shell
include /myredis/redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
daemonize yes
appendonly no
```

redis6380.conf：

```shell
include /myredis/redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb
daemonize yes
appendonly no
```

redis6381.conf：

```shell
include /myredis/redis.conf
pidfile /var/run/redis_6381.pid
port 6381
dbfilename dump6381.rdb
daemonize yes
appendonly no
```

（4）启动三个redis服务：

```shell
[root@nanzx myredis]# redis-server redis6379.conf 
[root@nanzx myredis]# redis-server redis6380.conf 
[root@nanzx myredis]# redis-server redis6381.conf 
```

（5）查看系统进程，看看三台服务器是否启动：

```shell
[root@nanzx myredis]# ps -ef|grep redis
root      83477      1  0 22:50 ?        00:00:00 redis-server *:6380
root      83490      1  0 22:52 ?        00:00:00 redis-server *:6379
root      83501      1  0 22:52 ?        00:00:00 redis-server *:6381
root      83507  83426  0 22:52 pts/0    00:00:00 grep --color=auto redis
```

（6）查看三台主机运行情况

```shell
[root@nanzx myredis]# redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:dca7c4a6677d7db6d409eec75cb489af07eeb610
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

（7）配从(节点)**不配主(节点)**，在6380和6381上执行: `slaveof 127.0.0.1 6379`

```shell
[root@nanzx myredis]# redis-cli -p 6381
127.0.0.1:6381> slaveof 127.0.0.1 6379
OK
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:6
master_sync_in_progress:0
slave_read_repl_offset:42
slave_repl_offset:42
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:877005c9ff59e7d6156934a911a3f0a01246002d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:42
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:29
repl_backlog_histlen:14

[root@nanzx myredis]# redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=154,lag=0
slave1:ip=127.0.0.1,port=6381,state=online,offset=154,lag=1
master_failover_state:no-failover
master_replid:877005c9ff59e7d6156934a911a3f0a01246002d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:154
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:154
```

（8）在主机上写，在从机上可以读取数据，在从机上写数据报错：

```shell
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> exit
[root@nanzx myredis]# redis-cli -p 6380
127.0.0.1:6380> get k1
"v1"
127.0.0.1:6380> set k2 v2
(error) READONLY You can't write against a read only replica.
```

（9）主节点挂掉重启就行，一切如初。从节点重启需重设：`slaveof 127.0.0.1 6379`，可以将这个命令配置到文件中，永久生效。也可在redis-server启动命令后加入 `--slaveof <masterip> <masterport>`

（10）断开与主节点复制关系：`slaveof no one`，从节点断开复制后不会删除原有数据，只是无法再获取主节点上的数据变化。

（11）切主操作：`slaveof <masterip> <masterport>`，从节点断开与旧主节点复制关系，与新主节点建立复制关系，**删除从节点当前所有数据**，对新主节点进行复制操作。

## 拓扑

**一主一从：**

- 主要用于主节点出现宕机时从节点提供故障转移支持
- 当应用写命令并发量高且需要持久化时，可以只在从节点上开启AOF，这样可以保证数据安全性以及避免持久化对主节点的性能干扰。
- 主节点要避免自动重启操作，因为没有开启持久化自动重启后数据集为空，从节点继续复制也被清空数据。安全做法是从节点执行`slaveof no one`，再重启主节点。

**一主多从：**

- 主要用于读占较大的场景，利用多个从节点实现读写分离来分担主节点压力，主写从读
- 多个从节点会导致主节点写命令的多次发送从而过度消耗网络带宽，同时加重主节点的负载影响服务稳定性

**树状主从结构：**

- 从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制
- 通过引入复制中间层可以有效降低主节点负载和需要传送给从节点的数据量

##  复制原理

主从复制的具体步骤如下：

（1）slave服务器连接到master服务器，便开始进行数据同步，发送psync命令（Redis2.8之前是sync命令）

（2）master服务器收到psync命令之后，开始执行bgsave命令生成RDB快照文件并使用缓存区记录此后执行的所有写命令（**全量复制**）
（3）master服务器bgsave执行完之后，就会向所有slava服务器发送快照文件，并在发送期间继续在缓冲区内记录被执行的写命令

（4）slave服务器收到RDB快照文件后，会将接收到的数据写入磁盘，然后清空所有旧数据，再从本地磁盘载入收到的快照到内存中，同时基于旧的数据版本对外提供服务。

（5）master服务器发送完RDB快照文件之后，便开始向slave服务器发送缓冲区中的写命令（**增量复制**）

（6）slave服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

（7）如果slave node开启了AOF，那么会立即执行BGREWRITEAOF，重写AOF

# 哨兵模式（Sentinel）

## 简介

主从复制的缺陷：

1. 一旦主节点出现故障，需要人工手动将从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点。（Redis高可用问题）

2. 主节点的写能力和存储能力受到单机的限制。（Redis分布式问题）

**哨兵模式**是一个分布式架构，能自动完成故障发现和故障转移，解决主从复制的高可用问题，其中包含**多个**哨兵节点（Sentinel节点，不存储数据，只支持部分命令）和Redis数据节点（主节点和从节点）。

**哨兵节点具体有以下几个功能：**

- 监控：哨兵节点会定期检测Redis数据节点和其他哨兵节点是否可达
- 通知：哨兵节点会将故障转移的结果通知给应用方
- 主节点故障转移：实现从节点晋升为主节点并维护后续正确的主从关系
- 配置提供者：在哨兵模式中，客户端在初始化的时候连接的是哨兵节点集合，从中获取主节点信息
  - 哨兵节点集合是多个哨兵节点组成，对故障的判断是集合里的节点共同完成，防止误判，同时个别节点不可用，依旧有健壮性。

## 实现原理

实现原理主要包括：三个定时任务、主观下线和客观下线、Sentinel领导者选举、故障转移。

**三个定时任务：**

- 每隔10秒，每个哨兵节点会向主节点和从节点发送info命令获取最新的拓扑结构。这个定时任务的作用具体可以表现在三个方面：
  - 通过向主节点执行info命令，获取从节点的信息，这也是为什么哨兵节点不需要显式配置监控从节点。
  - 当有新的从节点加入时都可以立刻感知出来。
  - 节点不可达或者故障转移后，可以通过info命令实时更新节点拓扑信息。
- 每隔2秒，每个哨兵节点会向Redis数据节点的`_sentinel_:he11o`频道上发送该哨兵节点对于主节点的判断以
  及当前哨兵节点的信息，同时每个哨兵节点也会订阅该频道，来了解其他哨兵节点以及它们对主节点的判断，所以这个定时任务可以完成以下两个工作：
  - 发现新的哨兵节点：通过订阅主节点的`_sentinel_:hello`了解其他的哨兵节点信息，如果是新加入的哨兵节点，将该哨兵节点信息保存起来，并与该哨兵节点创建连接。
  - 哨兵节点之间交换主节点的状态，作为后面客观下线以及领导者选举的依据。
- 每隔1秒，每个哨兵节点会向主节点、从节点、其余哨兵节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达。通过上面的定时任务，哨兵节点对主节点、从节点、其余哨兵节点都建立起连接，实现了对每个节点的监控，这个定时任务是节点失败判定的重要依据。

**主观下线：**

第三个定时任务，每个哨兵节点会每隔1秒对主节点、从节点、其他哨兵节点发送ping命令做心跳检测，当这些节点超过`down-after-milliseconds`没有进行有效回复，哨兵节点就会对该节点做失败判定，这个行为叫做主观下线。从字面意思也可以很容易看出主观下线是当前哨兵节点的一家之言，存在误判的可能。

**客观下线：**

当哨兵主观下线的节点是主节点时，该哨兵节点会通过`sentinelis-master-down-by-addr`命令向其他哨兵节点询问对主节点的判断，当超过< quorum >个数，哨兵节点认为主节点确实有问题，这时该哨兵节点会做出客观下线的决定，这样客观下线的含义是比较明显了，也就是大部分哨兵节点都对主节点的下线做了同意的判定，那么这个判定就是客观的。

>< quorum >代表判断主节点失败至少需要多少个哨兵节点同意

**Sentinel领导者选举：**

故障转移的工作只需要一个哨兵节点来完成即可，所以哨兵节点之间会做一个领导者选举的工作，选出一个哨兵节点作为领导者进行故障转移的工作。Redis使用了Rat算法实现领导者选举，因为Raft算法相对比较抽象和复杂，以及篇幅所限，所以这里给出一个Redis Sentinel进行领导者选举的大致思路：

1. 每个在线的哨兵节点都有资格成为领导者，当它确认主节点客观下线时候，会向其他哨兵节点发送`sentinel is-master-down-by-addr`命令，要求将自己设置为领导者。
2. 收到命令的哨兵节点，如果没有同意过其他哨兵节点的`sentinelis-master-down-by-addr`命令，将同意该请求，否则拒绝。
3. 如果该哨兵节点发现自己的票数已经大于等于`max(quorum，num(sentinels)/2+1)`，那么它将成为领导者。

**故障转移：**

- 在从节点列表中选出一个节点作为新的主节点，选择方法如下：
  - 过滤：“不健康”（主观下线、断线）、5秒内没有回复过哨兵节点ping响应、与主节点失联超过down-
    after-milliseconds*10秒。
  - 选择slave-priority（从节点优先级，值越小优先级越高），最高的从节点列表，如果存在则返回，不存在则继续。
  - 选择复制偏移量最大的从节点（复制的最完整），如果存在则返回，不存在则继续。
  - 选择runid最小的从节点。
- 哨兵领导者节点会对第一步选出来的从节点执行slaveof no one命令让其成为主节点。
- 哨兵领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点，复制规则和paralle1-syncs参数有关。
- 哨兵节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点。

## 部署哨兵节点

在一主多从的模式下部署一个哨兵节点模拟：

（1）自定义的/myredis目录下新建sentinel.conf文件(名字绝不能错)，填写内容：`sentinel monitor mymaster 127.0.0.1 6379 1`

其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。

（2）启动哨兵节点：`redis-sentinel  /myredis/sentinel.conf `。端口号是26379

```shell
[root@nanzx myredis]# redis-sentinel /myredis/sentinel.conf 
118302:X 20 Feb 2022 13:21:26.631 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
118302:X 20 Feb 2022 13:21:26.631 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=118302, just started
118302:X 20 Feb 2022 13:21:26.631 # Configuration loaded
118302:X 20 Feb 2022 13:21:26.633 * Increased maximum number of open files to 10032 (it was originally set to 1024).
118302:X 20 Feb 2022 13:21:26.633 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 118302
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

118302:X 20 Feb 2022 13:21:26.636 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
118302:X 20 Feb 2022 13:21:26.642 # Sentinel ID is c63f36aa0c006c36e4443a5b7af0937a9cd83f3a
118302:X 20 Feb 2022 13:21:26.642 # +monitor master mymaster 127.0.0.1 6379 quorum 1
118302:X 20 Feb 2022 13:21:26.644 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 13:21:26.647 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
```

（3）将主节点关闭，过了二十多秒哨兵节点会重新选举主节点

```shell
118302:X 20 Feb 2022 14:28:31.612 # +sdown master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.612 # +odown master mymaster 127.0.0.1 6379 #quorum 1/1
118302:X 20 Feb 2022 14:28:31.612 # +new-epoch 1
118302:X 20 Feb 2022 14:28:31.612 # +try-failover master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.616 # +vote-for-leader c63f36aa0c006c36e4443a5b7af0937a9cd83f3a 1
118302:X 20 Feb 2022 14:28:31.616 # +elected-leader master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.616 # +failover-state-select-slave master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.684 # +selected-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.684 * +failover-state-send-slaveof-noone slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:31.756 * +failover-state-wait-promotion slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:32.308 # +promoted-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:32.308 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:32.366 * +slave-reconf-sent slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:33.318 * +slave-reconf-inprog slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:33.318 * +slave-reconf-done slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:33.402 # +failover-end master mymaster 127.0.0.1 6379
118302:X 20 Feb 2022 14:28:33.402 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6381
118302:X 20 Feb 2022 14:28:33.402 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6381
118302:X 20 Feb 2022 14:28:33.402 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
118302:X 20 Feb 2022 14:29:03.432 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
```

（4）将关闭的“主节点”重启，它将变为从节点

```shell
118302:X 20 Feb 2022 14:42:58.822 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
```

# Redis集群（Cluster）

> 详解可参考：[Redis集群详解](https://blog.csdn.net/miss1181248983/article/details/90056960/)

## 简介

sentinel模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，主从模式或sentinel模式就不能满足需求了，这个时候需要对**存储的数据进行分片**，将数据存储到多个Redis实例中。cluster模式的出现就是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器。

cluster可以说是sentinel模式和主从模式的结合体，通过cluster可以实现主从和master重选功能，所以如果配置两个副本三个分片的话，就需要六个Redis实例。因为Redis的数据是根据一定规则分配到cluster的不同机器的，当数据量过大时，可以新增机器进行扩容。

**特点：**

* 多个redis节点网络互联，数据共享

* 所有的节点都是一主一从（也可以是一主多从），其中从节点不提供服务，仅作为备用

* 不支持同时处理多个key（如MSET/MGET），因为redis需要把key均匀分布在各个节点上，并发量很高的情况下同时创建key-value会降低性能并导致不可预测的行为
  
* 支持在线增加、删除节点

* 客户端可以连接任何一个主节点进行读写

## 部署集群

（1）配置6个实例：6379,6380,6381,6389,6390,6391，配置基本信息如下，可复制完在vim模式下匹配批量修改：`:%s/源字符串/目标字符串`->`:%s/6379/6380`

```shell
include /myredis/redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
daemonize yes
appendonly no
dir "/myredis/redis_cluster"
logfile "/myredis/redis_cluster/redis_err_6379.log"
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```

- `cluster-enabled yes`：打开集群模式

- `cluster-config-file nodes-6379.conf`：设定节点配置文件名

- `cluster-node-timeout 15000`：设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。

（2）启动6个redis服务：

```shell
[root@nanzx myredis]# mkdir redis_cluster
[root@nanzx myredis]# redis-server /myredis/redis6379.conf 
[root@nanzx myredis]# redis-server /myredis/redis6380.conf 
[root@nanzx myredis]# redis-server /myredis/redis6381.conf 
[root@nanzx myredis]# redis-server /myredis/redis6389.conf 
[root@nanzx myredis]# redis-server /myredis/redis6390.conf 
[root@nanzx myredis]# redis-server /myredis/redis6391.conf 
[root@nanzx myredis]# ls
redis6379.conf  redis6380.conf  redis6381.conf  redis6389.conf  redis6390.conf  redis6391.conf  redis_cluster  redis.conf  sentinel.conf
[root@nanzx myredis]# cd redis_cluster/
[root@nanzx redis_cluster]# ls
nodes-6379.conf  nodes-6380.conf  nodes-6381.conf  nodes-6389.conf  nodes-6390.conf  nodes-6391.conf  redis_err_6379.log  redis_err_6380.log  redis_err_6381.log  redis_err_6389.log  redis_err_6390.log  redis_err_6391.log
```

（3）执行以下命令将六个节点合成一个集群，组合之前确保所有redis实例启动成功且nodes-xxxx.conf文件都生成正常：

```shell
[root@nanzx ~]# redis-cli --cluster create --cluster-replicas 1 192.168.2.110:6379 192.168.2.110:6380 192.168.2.110:6381 192.168.2.110:6389 192.168.2.110:6390 192.168.2.110:6391
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.2.110:6390 to 192.168.2.110:6379
Adding replica 192.168.2.110:6391 to 192.168.2.110:6380
Adding replica 192.168.2.110:6389 to 192.168.2.110:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 3caafd0c5797c8be7c059ae88cfd0726d59cd070 192.168.2.110:6379
   slots:[0-5460] (5461 slots) master
M: 7231523d7d0156f682d4e5ce0bfc6ead757f9db4 192.168.2.110:6380
   slots:[5461-10922] (5462 slots) master
M: e28bf52c7fe25282a5b33940d5078b930db181f1 192.168.2.110:6381
   slots:[10923-16383] (5461 slots) master
S: 57a8fa22b33d070062ac5580f60d7219d8d2d385 192.168.2.110:6389
   replicates 7231523d7d0156f682d4e5ce0bfc6ead757f9db4
S: ed395ce3a4346dc12d24c0a273f1a32b27620e8b 192.168.2.110:6390
   replicates e28bf52c7fe25282a5b33940d5078b930db181f1
S: 112c48af5d9d38a870d18f8ba3dd2eb635a260e1 192.168.2.110:6391
   replicates 3caafd0c5797c8be7c059ae88cfd0726d59cd070
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 192.168.2.110:6379)
M: 3caafd0c5797c8be7c059ae88cfd0726d59cd070 192.168.2.110:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 57a8fa22b33d070062ac5580f60d7219d8d2d385 192.168.2.110:6389
   slots: (0 slots) slave
   replicates 7231523d7d0156f682d4e5ce0bfc6ead757f9db4
S: 112c48af5d9d38a870d18f8ba3dd2eb635a260e1 192.168.2.110:6391
   slots: (0 slots) slave
   replicates 3caafd0c5797c8be7c059ae88cfd0726d59cd070
M: e28bf52c7fe25282a5b33940d5078b930db181f1 192.168.2.110:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: ed395ce3a4346dc12d24c0a273f1a32b27620e8b 192.168.2.110:6390
   slots: (0 slots) slave
   replicates e28bf52c7fe25282a5b33940d5078b930db181f1
M: 7231523d7d0156f682d4e5ce0bfc6ead757f9db4 192.168.2.110:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

- 此处不要用127.0.0.1， 请用真实IP地址

- --replicas 1 表示采用最简单的方式配置集群，一台主机，一台从机，正好三组。

（4）普通方式登录存储数据时，会出现MOVED重定向失败。所以应该以集群方式登录。加上-c参数表明采用集群策略连接，设置数据时会自动切换到相应的主节点进行写操作

```shell
[root@nanzx ~]# redis-cli -p 6379
127.0.0.1:6379> set k1 v1
(error) MOVED 12706 192.168.2.110:6381
127.0.0.1:6379> exit
[root@nanzx ~]# redis-cli -c -p 6379
127.0.0.1:6379> set k1 v1
-> Redirected to slot [12706] located at 192.168.2.110:6381
OK
```

（5）通过` cluster nodes `命令查看集群信息

```shell
92.168.2.110:6381> cluster nodes
ed395ce3a4346dc12d24c0a273f1a32b27620e8b 192.168.2.110:6390@16390 slave e28bf52c7fe25282a5b33940d5078b930db181f1 0 1645371670925 3 connected
e28bf52c7fe25282a5b33940d5078b930db181f1 192.168.2.110:6381@16381 myself,master - 0 1645371669000 3 connected 10923-16383
112c48af5d9d38a870d18f8ba3dd2eb635a260e1 192.168.2.110:6391@16391 slave 3caafd0c5797c8be7c059ae88cfd0726d59cd070 0 1645371671935 1 connected
7231523d7d0156f682d4e5ce0bfc6ead757f9db4 192.168.2.110:6380@16380 master - 0 1645371670000 2 connected 5461-10922
57a8fa22b33d070062ac5580f60d7219d8d2d385 192.168.2.110:6389@16389 slave 7231523d7d0156f682d4e5ce0bfc6ead757f9db4 0 1645371671000 2 connected
3caafd0c5797c8be7c059ae88cfd0726d59cd070 192.168.2.110:6379@16379 master - 0 1645371671000 1 connected 0-5460
```

## slots

- 一个 Redis 集群包含 **16384** 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个。

- 集群使用公式 CRC16(key) % 16384 来计算键key属于哪个槽， 其中CRC16(key)语句用于计算键key的CRC16校验和 。

- 集群中的每个节点负责处理一部分插槽。 举个例子， 如果一个集群可以有多个主节点， 其中：

  - 节点 A 负责处理 0 号至 5460 号插槽。

  - 节点 B 负责处理 5461 号至 10922 号插槽。

  - 节点 C 负责处理 10923 号至 16383 号插槽。

- 在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口。

- redis-cli客户端提供了 –c 参数实现自动重定向。如 redis-cli  -c –p 6379 登入后，再录入、查询键值对可以自动重定向。

- 不在一个slot下的键值，是不能使用mget,mset等多键操作。可以通过{}来定义组的概念，从而使key中{}内相同内容的键值对放到一个slot中去。

```shell
192.168.2.110:6381> mset k1{cust} v1 k2{cust} v2 k3{cust} v3
-> Redirected to slot [4847] located at 192.168.2.110:6379
OK
192.168.2.110:6379> cluster keyslot cust #查询key所在插槽
(integer) 4847
192.168.2.110:6379> cluster countkeysinslot 4847 #查询插槽中key的数量，只能查询该节点负责的插槽
(integer) 3
192.168.2.110:6379> cluster getkeysinslot 4847 10 #返回10个4847插槽的key
1) "k1{cust}"
2) "k2{cust}"
3) "k3{cust}"
192.168.2.110:6379> 
```

## 故障恢复

- 如果主节点下线？从节点能否自动升为主节点？

  - `cluster-node-timeout 15000`，15秒超时，集群自动进行主从切换

- 主节点恢复后，主从关系会如何？

  - 主节点回来变成从节点

- 如果某一段插槽的主从节点都宕掉，redis服务是否还能继续？

  - 如果redis.conf中的参数`cluster-require-full-coverage` 为yes ，那么整个集群都挂掉；
  - 而`cluster-require-full-coverage `为no ，那么该插槽数据全都不能使用，也无法存储。




# 缓存穿透

缓存穿透指一个一定不存在的数据，由于**缓存未命中**这条数据，就会去查询数据库，**数据库也没有**这条数据，所以返回结果是 `null`。如果并发量大且缓存都没有命中，每次查询都请求数据库时，缓存就失去了保护后端持久层的意义，这会给持久层数据库造成很大的压力。

**解决方案：**

- 缓存空对象：是指在持久层没有命中的情况下，对key进行set （key,null），缓存空对象会有两个问题：
  - value为null 不代表不**占用内存空间**，空值做了缓存，意味着缓存层中存了更多的键，需要更多的内存空间，比较有效的方法是针对这类数据设置一个较短的过期时间，让其自动剔除。
  - 缓存层和持久层的数据会有一段时间窗口的不一致，可能会对业务有一定影响。例如过期时间设置为5分钟，如果此时存储层添加了这个数据，那此段时间就会出现缓存层和持久层数据的不一致，此时可以利用消息系统或者其他方式清除掉缓存层中的空对象。

- 布隆过滤器拦截

  - 在访问缓存层和持久层之前，将存在的key用布隆过滤器提前保存起来，做第一层拦截，当收到一个对key请求时先用布隆过滤器验证是key否存在，如果存在再进入缓存层、持久层。可以使用bitmap做布隆过滤器。这种方法适用于数据命中不高、数据相对固定、实时性低的应用场景，代码维护较为复杂，但是缓存空间占用少。

  - 布隆过滤器实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的**误识别率**和删除困难。

---

# 缓存击穿

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是**热点数据**，由于**缓存过期**，会同时访问数据库来查询最新数据，**数据库有数据**并且回写缓存，会导使数据库瞬间压力过大。

**解决方案：**

- 分布式互斥锁，只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完，重新从缓存获取数据即可。
  - 这种方案思路比较简单，但是存在一定的隐患，如果在查询数据库 + 和 重建缓存（key失效后进行了大量的计算）时间过长，也可能会存在死锁和线程池阻塞的风险，高并发情景下吞吐量会大大降低！但是这种方法能够较好地降低后端存储负载，并在一致性上做得比较好。

- 热点Key永不过期，从功能层面来看，为每个value设置一个逻辑过期时间，当发现超过逻辑过期时间后，会使用单独的线程去更新缓存。
  - 这种方案由于没有设置真正的过期时间，实际上已经不存在热点key产生的一系列危害，但是会存在数据不一致的情况，同时代码复杂度会增大。

---

# 缓存雪崩

 由于缓存层承载着大量请求，有效地保护了持久层，但是如果缓存层由于某些原因不可用（**宕机**）或者大量缓存由于超时时间相同在**同一时间段**失效（大批key失效/热点数据失效），大量请求直接到达存储层，存储层压力过大导致系统雪崩。

**解决方案：**

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。

- 可以把缓存层设计成高可用的，即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务。利用sentinel或cluster实现。

- 采用多级缓存，本地进程作为一级缓存，redis作为二级缓存，不同级别的缓存设置的超时时间不同，即使某级缓存过期了，也有其他级别缓存兜底。

- 数据加热，在正式部署之前先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。



# 常见面试题

**过期的数据的删除策略了解么？**

如果假设你设置了一批 key 只能存活 1 分钟，那么 1 分钟后，Redis 是怎么对这批 key 进行删除的呢？

常用的过期数据的删除策略就两个：

1. **惰性删除** ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。
2. **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就 Out of memory 了。

怎么解决这个问题呢？答案就是：**Redis 内存淘汰机制。**

---

**Redis 内存淘汰机制了解么？**

Redis一开始有6种数据淘汰策略，4.0版本后增加2种（7和8），总共8种

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！
7. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
8. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中移除最不经常使用的 key

---

**如何保证缓存和数据库数据的一致性？**

下面单独对 **Cache Aside Pattern（旁路缓存模式）** 来聊聊。

Cache Aside Pattern 中遇到写请求是这样的：更新 DB，然后直接删除 cache 。

如果更新数据库成功，而删除缓存这一步失败的情况的话，简单说两个解决方案：

1. **缓存失效时间变短（不推荐，治标不治本）** ：我们让缓存数据的过期时间变短，这样的话缓存就会从数据库中加载数据。另外，这种解决办法对于先操作缓存后操作数据库的场景不适用。
2. **增加cache更新重试机制（常用）**： 如果 cache 服务当前不可用导致缓存删除失败的话，我们就隔一段时间进行重试，重试次数可以自己定。如果多次重试还是失败的话，我们可以把当前更新失败的 key 存入队列中，等缓存服务可用之后，再将 缓存中对应的 key 删除即可。

