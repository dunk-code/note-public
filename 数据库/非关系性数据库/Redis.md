 菜鸟教程](https://www.runoob.com/redis/redis-conf.html)

查看Redis进程

```bash
ps -ef|gerp redis
```

​	<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509210923.png" alt="image-20210509210922714" style="zoom:80%;" />

# Redis背景

[Redis的故事](https://blog.csdn.net/qq_35190492/article/details/107682394?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162018744016780274148823%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162018744016780274148823&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-1-107682394.pc_search_result_before_js&utm_term=redis%E7%AB%AF%E5%8F%A3%E5%8F%B7%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF6379)

## NoSQL

> 什么是NoSQL

NoSQL = Not Only SQL（不仅仅是SQL）

泛指非关系型数据库，常用的都是关系型数据库。就像我们常用的MySQL，sqlServer一样，这些数据库一般用来存储重要信息，应对普通的业务是没有问题的，但是，随着互联网的高速发展，传统的关系性数据库在应对超大规模超大流量以及高并发的时候力不从心。

> 存储结构

关系型数据库对应的是结构化数据，数据表都是预先定义了结构（列的定义），结构描述了数据的形式和内容，这一点对数据建模至关重要，虽然预定义结构带来了可靠性和稳定性，但是修改这些数据比较困难。

NoSQL数据库基于动态结构，使用于非结构化数据，因为NoSQL数据库是动态结构，可以很容易适应数据结构类型和结构的变化

> NoSQL的特点

- 方便扩展（数据之间没有关系）
- 大数据量高性能（Redis一秒写8万次，读取11万次）
- 数据类型多样型（不需要实现设计数据库，随取随用）

> 传RDBMS和NoSQL

RDBMS

结构化组织、SQL、数据和关系都存在单独的表中、操作数据，数据定义语言、严格的一致性、基础的事务

NoSQL

不仅仅是数据、没有固定的查询语言、键值对存储、列存储、文档存储、图形化存储、最终一致性、CAP定理和BASE理论、高性能、高可用、高可扩展3

## NoSQL的四大分类

### KV键值对

- 新浪：Redis
- 美团：Redis + Tair
- 阿里、百度：Redis + memecache

### 文档型数据库

- MongoDB
  - MongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档
  - MonoDB是一个介于关系型数据库和非关系性数据库中中间的产品（MongoDB是非关系型数据库中功能最丰富的，最像关系型数据库）
- ConthDB

### 列存储

- HBase
- 分布式文件系统

### 图形化数据库

- 他不是存图形的，放的是关系，比如：朋友圈社交网络、广告推荐！
- Neo4J、InfoGrid

|       **分类**        |                  **Examples举例**                  |                       **典型应用场景**                       |                  **数据模型**                  |                           **优点**                           |                           **缺点**                           |
| :-------------------: | :------------------------------------------------: | :----------------------------------------------------------: | :--------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| **键值（key-value）** | Tokyo Cabinet/Tyrant, Redis, Voldemort, Oracle BDB | 内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等等 | Key 指向 Value 的键值对，通常用hashtable来实现 |                          查找速度快                          |        数据无结构化，通常只被当作字符串或者二进制数据        |
|   **列存储数据库**    |               Cassandra, HBase, Riak               |                       分布式的文件系统                       |       以列簇式存储，将同一列数据存在一起       |         查找速度快，可扩展性强，更容易进行分布式扩展         |                         功能相对局限                         |
|   **文档型数据库**    |                  CouchDB, MongoDb                  | Web应用（与Key-Value类似，Value是结构化的，不同的是数据库能够了解Value的内容） |    Key-Value对应的键值对，Value为结构化数据    | 数据结构要求不严格，表结构可变，不需要像关系型数据库一样需要预先定义表结构 |            查询性能不高，而且缺乏统一的查询语法。            |
| **图形(Graph)数据库** |          Neo4J, InfoGrid, Infinite Graph           |           社交网络，推荐系统等。专注于构建关系图谱           |                     图结构                     |     利用图结构相关算法。比如最短路径寻址，N度关系查找等      | 很多时候需要对整个图做计算才能得出需要的信息，而且这种结构不太好做分布式的集群方案。 |

# Redis入门

## Redis概述

>Remote Dictionary Server(Redis) 

Redis是一个开源的使用C语言编写、遵循BSD协议、支持网络、可基于内存、分布式、可选持久性的键值对（Key-Value）存储数据库，并提供多种语言的API。

Redis 通常被称为结构化数据库，因为值（value）可以是字符串(String)、哈希(Hash)、列表(list)、集合(sets)和有序集合(sorted sets)等类型。

与传统数据库不同的是Redis的数据是**存储在内存中**的，所以读写的速度非常快，因此Redis被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。另外，Redis 也经常用来做分布式锁。除此之外，Redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。

> Redis可以做什么

- Redis支持数据的持久化（rdb、aof），可以将内存中的数据保存在磁盘中，重启的时候再次加载使用
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。
- 效率高，可以用于高速缓存
- 发布订阅系统
- 地图信息分析
- 计时器、计数器（浏览量）

### Redis的优缺点

> 优点

- 读写性能优异，Redis能读的速度是110000次/s，写的速度是81000次/s
- 支持数据持久化，支持AOF和RDB两种持久化方式
- 支持事务，Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性执行
- 数据结构丰富，除了支持string类型的value外还支持hash、set、zset、list等数据结构
- 支持主从复制，主机会自动同步到从机，可以进行读写分离

> 缺点

- 数据库容量受到物理内存的限制，不能做海量数据的高性能读写，因此Redis适合的场景主要是局限在数据量较小的高性能操作和运算上
- Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
- Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

### 为什么要用缓存

> 主要从“高性能”和“高并发”这两点来看待这个问题。

#### 高性能

假如用户第一次访问数据库中的某些数据。这个过程会比较慢，因为是从硬盘上读取的。将该用户访问的数据存在数缓存中，这样下一次再访问这些数据的时候就可以直接从缓存中获取了。操作缓存就是直接操作内存，所以速度相当快。如果数据库中的对应数据改变的之后，同步改变缓存中相应的数据即可！

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210603214613.png" alt="image-20210603214611265" style="zoom:80%;" />

#### 高并发

直接操作缓存能够承受的请求是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210603214633.png" alt="image-20210603214630033" style="zoom:80%;" />

### 为什么使用Redis而不使用map/guava做缓存

缓存分为本地缓存和分布缓存。以Java为例，自带的map或者guava实现的是本地缓存，最主要的特点是轻量以及快速，生命周期随着JVM的销毁而结束，并且在多实例的情况下，每个实例都需要各自保存一份缓存，缓存不具有一致性

使用 redis 或 memcached 之类的称为分布式缓存，在多实例的情况下，各实例共用一份缓存数据，**缓存具有一致性**。缺点是需要保持 redis 或 memcached**服务的高可用**，整个程序架构上较为复杂。

### Redis为什么快

关系型数据库跟Redis本质上的区别

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210603214700.png" alt="image-20210603214657283" style="zoom:80%;" />

- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速，数据存在内存中，类似以HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
- 数据结构简单，对数据操作也简单，Redis中的数据结构是专门设计的
- 采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换消耗CPU，不用考虑锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗，不会浪费多核CPU，因为可以通过单机开多个Redis实例

> Redis在处理客户端的请求时，包括获取（socket读）、解析、执行、内容返回（socket写）等都是由一个顺序串行的主线程处理的，这就是所谓的“单线程”。但如果严格来讲，Redis4.0之后并不是单线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用链接释放、大key的删除等等
>
> 使用Redis时，几乎不存在CPU成为瓶颈的情况， Redis主要受限于内存和网络。6.0版本带来了多线程特性，因为读写网络的read/write系统调用占用了了Redis执行期间大部分CPU时间，瓶颈主要在于网络的 IO 消耗。多线程任务可以分摊Redis同步IO读写负荷。
>
> Redis的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行。

- 使用非阻塞I/O多路复用模型，多路指的是多个socket连接，复用指的是复用一个线程，Redis使用epoll作为I/O多路复用技术的实现，在加上Redis自身的事件处理模式将epoll的read、write、close等都转换成事件，不在网络I/O浪费过多的时间
  - 阻塞式IO（处理一个socket就要占用一个线程）让出CPU，进到等待队列，等socket就绪后再次获取时间片继续执行
  - 非阻塞式IO，不让出CPU，频繁检查socket就绪状态，忙等待，难把握轮询间隔，空耗CPU
  - IO多路复用（一次系统调用，监听多个socket），操作系统提供支持，把需要等待的socket加入到监听集合。

> epoll是Linux内核为处理大批量文件描述符而做了改进的poll，是Linux下多路复用I/O接口select/poll的增强版本，他能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率
>
> 当多个请求发送到服务端的时候，实际上会有一个文件事件处理器同时监听多个套接字，并且根据套接字目前执行的任务来关联不同的事件处理器。
>
> 事件处理器只需要将他们做绑定即可，IO多路复用程序时会将所有产生的套接字都存入一个有序且同步的队列中，最后Redis会逐一对这个队列中的元素进行处理
>
> epoll没有最大并发连接的限制，只管你“活跃”的连接 ，而跟连接总数无关。Epoll使用了“共享内存 ”，省去内存拷贝。

- 使用底层模型不同，他们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求

### Redis的使用场景

> 计数器

可以对String进行自增自减运算，从而实现计数器的动能，Redis这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量

> 缓存

将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率Redis安装

> 会话缓存

可以使用 Redis 来统一存储多台应用服务器的会话信息。当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

> 全页缓存（FPC）

除基本的会话token之外，Redis还提供很简便的FPC平台。以Magento为例，Magento提供一个插件来使用Redis作为全页缓存后端。此外，对WordPress的用户来说，Pantheon有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

> 查找表

例如 DNS 记录就很适合使用 Redis 进行存储。查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

> 消息队列(发布/订阅功能)

List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。不过最好使用 Kafka、RabbitMQ 等消息中间件。

> 分布式锁实现

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

## Redis安装

### windows安装

[下载地址](https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100)

双击运行服务 `redis-server.exe`启动Redis服务器

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210504221353.png" alt="image-20210504221341105" style="zoom:50%;" />

运行成功

再次运行redis客户端 启动 `redis-cli.exe`

```sql
127.0.0.1：6379> ping  ----> 测试是否连接成功
PONG 
127.0.0.1：6379> set name changan  -----> 设置 key value
OK
127.0.0.1：6379> get name  ------> 用 key 去寻找 value
"changan"
```

### Linux安装

下载、解压、编译Redis

```bash
$ wget http://download.redis.io/releases/redis-6.0.6.tar.gz
$ tar xzf redis-6.0.6.tar.gz
$ cd redis-6.0.6
make
make install
```

> make报错

目前Redis官网下载的版本为 6.0版本  make安装

会报错是因为gcc版本过低，yum安装的gcc是4.8.5的。因此需要升级gcc，升级过程如下：

```shell
[root@hadoop01 redis-6.0.5]# gcc -v                             # 查看gcc版本

[root@hadoop01 redis-6.0.5]# yum -y install centos-release-scl  # 升级到9.1版本

[root@hadoop01 redis-6.0.5]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

[root@hadoop01 redis-6.0.5]# scl enable devtoolset-9 bash

以上为临时启用，如果要长期使用gcc 9.1的话：
[root@hadoop01 redis-6.0.5]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

安装c++环境

```bash
yum install gcc-c++
```

进入程序安装目录,拷贝配置文件

```shell
cd /usr/local/bin

cp /opt/redis-6.0.6/redis.conf config/
```

修改配置文件，让Redis后台启动

```shell
daemonize yes
```

启动Redis服务（通过指定文件启动）

```shell
redis-server config/redis.conf

#连接测试
redis-cli -h 127.0.0.1 -p 6379
```

查看Redis服务进程

```shell
ps -ef |grep redis
```

![image-20210505112315813](https://gitee.com/zhang-songyao/blog-images/raw/master/20210505112538.png)

关闭Redis服务

```shell
shutdown 

exit
```

![image-20210505112536354](https://gitee.com/zhang-songyao/blog-images/raw/master/20210505112537.png)

# Redis基础

## 基本知识

### 数据库

> Redis有16个数据库，默认使用第0个

![image-20210505115503450](https://gitee.com/zhang-songyao/blog-images/raw/master/20210505115504.png)

使用select num切换数据库

```bash
select 3
```

> 清空当前数据库的内容

```bash
flushdb
```

> 清空所有数据库内容

```bash
flushall
```

> 查看所有所有的key

```bash
keys *
```

### 配置信息

> 获取配置信息

```bash
CONFIG GET CONFIG_SETTING_NAME
```

获取所有配置信息

```bash
CONFIG GET *
```

编辑配置信息

```bash
CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
#实例
CONFIG SET loglevel "notice"
```

#### INCLUDES

这里包括一个或多个其他配置文件。这很有用，如果你有一个标准的模板，去所有的Redis服务器，但也需要自定义一些服务器设置。包括文件可以包括其他文件，所以明智地使用这个。注意选项“include”不会被命令“CONFIG REWRITE”重写。

```bash
include /path/to/local.conf
include /path/to/other.conf
```

#### NWTWORK

`bing 0.0.0.0`对所有人开放，可以指定单个或者多个ip

`protected-mode yes`  开启受保护模式

- 关闭protected-mode模式，此时外部网络可以直接访问
- 开启protected-mode保护模式，需配置bind ip或者设置访问密码

`port 6379`  默认端口号

#### GENERAL

`daemonize yes` 以守护进程的方式运行，默认是 no 我们需要自己设置为yes

`pidfile /www/server/redis/redis.pid` 如果是守护进程方式运行，我们需要指定一个pid文件

`loglevel notice` 设置日志文件的级别

- debug(大量信息，对开发/测试有用)
- verbose(很多很少有用的信息，但不像调试级别那样混乱)
- notice(有点冗长，可能是在生产中需要的内容)
- warning(只记录非常重要/关键的消息)

`logfile "/www/server/redis/redis.log"` 日志的文件位置

`databases` 默认的数据库数量 16 个

`always-show-logo yes` 是否显示log 默认为开启

#### SNAOSHOTTING

> 持久化数据 因为 Redis是内存数据库 如果断电等因素 会失去数据 所以我们需要在一定时间里 持久化数据

在 900s 内 有 1个key进行了操作 那么将会持久化一下
save 900 1
在 300s 内 有 10个key进行了操作 那么将会持久化一下
save 300 10
在 60s 内 有 1w个key进行了操作 那么将会持久化一下
save 60 10000

`stop-writes-on-bgsave-error yes` 持久化 出错了是否继续工作 默认继续

`rdbcompression yes` 是否压缩 rdb文件,默认压缩 压缩会消耗cpu资源

`rdbchecksum yes` 保存 rdb 文件时进行错误的校验

`rdb` 文件保存的目录

#### SECURITY

`requirepass xxxxx` 设置redis 登录密码

##### CLIENTS

`maxclients 10000` 默认有 1w 个用户可以同时连接redis 服务器

`maxmemory <bytes>` redis 配置最大的内存容量

`maxmemory-policy noeviction` 内存到达上限的处理策略 6种

**1、volatile-lru：**只对设置了过期时间的key进行LRU（默认值）

**2、allkeys-lru ：** 删除lru算法的key

**3、volatile-random：**随机删除即将过期key

**4、allkeys-random：**随机删除

**5、volatile-ttl ：** 删除即将过期的

**6、noeviction ：** 永不过期，返回错误

#### APPEND ONLY MODE

`appendonly no` 默认不开启 默认使用rdb持久化方式

- appendfsync always：每修改一个key都会执行 sync，消耗性能
- appendfsync everysec：每一秒执行一次 sync，可能会丢失这1s的数据
- appendfsync no：不执行 sync，这个时候操作系统会自己同步数据速度是最快的

`appendfilename "appendonly.aof"` 持久化文件的名字

### 参数说明

| 序号 |                            配置项                            | 说明                                                         |
| :--: | :----------------------------------------------------------: | :----------------------------------------------------------- |
|  1   |                        `daemonize no`                        | Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ） |
|  2   |                 `pidfile /var/run/redis.pid`                 | 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定 |
|  3   |                         `port 6379`                          | 指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字 |
|  4   |                       `bind 127.0.0.1`                       | 绑定的主机地址                                               |
|  5   |                        `timeout 300`                         | 当客户端闲置多长秒后关闭连接，如果指定为 0 ，表示关闭该功能  |
|  6   |                      `loglevel notice`                       | 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice |
|  7   |                       `logfile stdout`                       | 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null |
|  8   |                        `databases 16`                        | 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id |
|  9   | `save <seconds> <changes>`Redis 默认配置文件中提供了三个条件：**save 900 1****save 300 10****save 60 10000**分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。 | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
|  10  |                     `rdbcompression yes`                     | 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
|  11  |                    `dbfilename dump.rdb`                     | 指定本地数据库文件名，默认值为 dump.rdb                      |
|  12  |                           `dir ./`                           | 指定本地数据库存放目录                                       |
|  13  |              `slaveof <masterip> <masterport>`               | 设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步 |
|  14  |                `masterauth <master-password>`                | 当 master 服务设置了密码保护时，slav 服务连接 master 的密码  |
|  15  |                    `requirepass foobared`                    | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH (password) 命令提供密码，默认关闭 |
|  16  |                      ` maxclients 128`                       | 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
|  17  |                     `maxmemory <bytes>`                      | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
|  18  |                       `appendonly no`                        | 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no |
|  19  |               `appendfilename appendonly.aof`                | 指定更新日志文件名，默认为 appendonly.aof                    |
|  20  |                    `appendfsync everysec`                    | 指定更新日志条件，共有 3 个可选值：**no**：表示等操作系统进行数据缓存同步到磁盘（快）**always**：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全）**everysec**：表示每秒同步一次（折中，默认值） |
|  21  |                       `vm-enabled no`                        | 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制） |
|  22  |                `vm-swap-file /tmp/redis.swap`                | 虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享 |
|  23  |                      `vm-max-memory 0`                       | 将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0 |
|  24  |                      `vm-page-size 32`                       | Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
|  25  |                     `vm-pages 134217728`                     | 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
|  26  |                      `vm-max-threads 4`                      | 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4 |
|  27  |                     `glueoutputbuf yes`                      | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
|  28  |    `hash-max-zipmap-entries 64 hash-max-zipmap-value 512`    | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
|  29  |                    `activerehashing yes`                     | 指定是否激活重置哈希，默认为开启（后面在介绍 Redis 的哈希算法时具体介绍） |
|  30  |                `include /path/to/local.conf`                 | 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |

### [官方文档](https://redis.io/)

Redis是一个开源（BSD许可）的内存数据结构存储，用作**数据库**、**缓存**和**消息中间件MQ**。Redis提供诸如字符串、哈希、列表、集合、有序集合（sort sets）、位图（bitmaps）、超日志（hyperloglogs）、地理空间（geospatial）索引半径查询等数据结构。Redis具有内置的复制（replication）、Lua脚本（Lua scripting）、LRU驱动事件（LRU eviction）、事务（ transactions）和不同级别的磁盘持久性（ persistence），并通过Redis哨兵（Sentinel）和自动分区（Cluster）提供高可用性

## Redis数据类型

### SDS

> Redis需要的不仅仅是一个字符串字面值，而是一个可以被修改的字符串值

Redis没有直接使用C语言中的字符串，而是自己构建了一种名为简单动态字符串（Simple dynamic string，SDS）的抽象类型，并将SDS作为默认的Redis的默认字段

#### 定义

```c
struct sdshdr {
    //记录buf数组已经使用字节的数量
    //等于SDS保存字符串的长度
    int len;
    
    //记录buf数组未使用的字节数量
    int free;
    
    //字节数组，用于保存字符串
    char buf[];
}
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508170801.png" alt="image-20210508170800516" style="zoom:67%;" />

- free属性值为0，表示SDS没有分配任何未使用空间
- len属性的值为5，表示这个SDS保存了一个五字节长的字符串
- buf属性是一个char类型的数组，保存着前五个字节

#### C字符串和SDS的区别

##### 常数负责读获取字符串长度

因为C字符串不记录自身的长度信息，所以为了获取一个C字符串的长度，程序必须遍历整个C字符串，整个操作的复杂度为O(N)，而SDS不同于C字符串，保存着自身的长度，所以Redis将获取字符串长度的复杂度从O(N)降到了O(1)

##### 杜绝缓存区溢出

> 字符串拼接函数

```c
char *stract(char *dest, char *src);
```

因为C字符串不记录自身的长度，所以stract假定用户在执行函数时，已经为dest分配了足够多的内存，可以容纳src中的所有内容，而一旦这个假定不成立时就会产生缓存区溢出

假设存在字符串如下

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508172447.png" alt="image-20210508172446901" style="zoom: 67%;"/>	

程序员由于粗心在拼接字符串前未给s1分配足够的空间，而执行stract(S1, "Cluster");

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508172334.png" alt="image-20210508172333153" style="zoom:67%;" />

相比于C字符串，SDS的空间分配策略完全杜绝了放生缓冲区溢出的可能性：当SDSAPI需要对SDS进行修改的时候，API会先检查SDS的空间是否满足修改所需的要求，如果不满足，API会自定将SDS的空间扩展至执行修改所需要的大小。然后在执行修改，所以SDS既不需要手动修改SDS的空间大小，也不会出现缓冲区溢出的问题。

##### 减少修改字符串时带来的内存重分配的次数

因为C字符串笔记录自身的长度，所以对于一个包含N个字符串的C字符串来说，这个C字符串底层实现总是一个N + 1个字符长的数组（额外一个字符空间保存空字符）。因为C字符串的长度和底层数组的长度之间存在这种关联性，所以每次增长或者缩短一个字符串都对应一次内存重分配操作。

为了避免C字符串的这种缺陷，SDS通过未使用的空间解除了字符串长度和底层数组长度之间的关联：在SDS中，buf数组的长度不一定就是字符数量+1，数组里面可以包含未使用的字节，而这些字节的数量就有SDS的free属性记录

通过未使用的空间，SDS实现了空间预分配和惰性空间释放两种优化策略

###### 空间预分配

- 对于空间小于1M来说,分配空间为原有总长度+同样长度+1byte(B,字节).
- 对于大于1M来说,分配空间为原有总长度+1MB+1byte

> 通过空间预分配策略，SDS将连续增长N次的字符串所需要的内存重分配次数从必定N次降低为最多N次

###### 惰性空间释放

对于需要缩短字符串的情景,即需要释放空间,SDS将需要移除的字符串移除,但是多余出来的的空间不释放,而是保留下来,记录在free里,这样扩展就有多余空间来进行,当然有真正释放空间的方法.

#### 二进制安全

C字符串中的字符必须符合某种编码（ASCII），并且除了字符串末尾外，字符串里面不能包含空字符串，否则最先被程序读入的空字符串将被误认为字符串结尾，因此不能保存二进制文件，通多SDS可以避免这样的问题。

#### 兼容部分C字符串函数

由于SDS一样遵循了C字符串以空字符结尾的惯例：这些API总会将SDS保存的数据的末尾设置为空字符，这是为了让那些保存文本数据的SDS可以重用一部分<String.h>库定义的函数

#### 总结

C字符串和SDS的区别

|             C字符串              |                SDS                 |
| :------------------------------: | :--------------------------------: |
|     获取字符串长度复杂度O(N)     |      获取字符串长度复杂度O(1)      |
| API不安全，可能会造成缓冲区溢出  |    API安全，不会造成缓冲区溢出     |
|    修改字符串长度N必然执行N次    |     修改字符串长度N最多执行N次     |
|         只能保存文本数据         |      可以保存文本或二进制数据      |
| 可以使用所有<string.h>库中的函数 | 可以使用一部分<string.h>库中的函数 |

### 链表

> 列表键的底层实现之一

#### 定义

```c
typedef struct listNode {
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    void *value;
}

typedef struct list {
    //表头节点
    listNode *head;
    //表尾节点
    listNode *tail;
    //链表所包含的节点数量
    unsigned long len;
    //节点值赋值函数
    void *(*dup)(void *ptr);
    //节点释放函数
    void *(*free)(void *ptr);
    //节点值对比函数
    void *(*match)(void *ptr, void *key);
}
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508180119.png" alt="image-20210508180118777" style="zoom:80%;" />

#### 特性

- 双端：链表具有`前置节点`和`后置节点`的引用，获取这两个节点时间复杂度都为O(1)。
- 无环：`表头节点的 prev 指针`和`表尾节点的 next 指针`都`指向 NULL`,对链表的访问都是以 NULL 结束。
- 带表头指针和表尾指针
- 带链表长度计数器：通过 `len 属性 获取链表长度`的`时间复杂度为 O(1)`。
- 多态（保存各种不同类型的值）：链表节点使用 void* 指针来保存节点值，可以保存`各种不同类型的值`。

### 字典

> Redis的数据库底层就是使用字典作为底层实现的

字典，又被称为符号表、关联数组或映射，是一种用于保存键值对的抽象数据结构。字典中的每个键都是独一无二的，程序可以在字典中根据键查找与之关联的值，或者通过键来更新值，又或者根据键来删除整个键值对

#### 实现

```c
//哈希表节点
typedef struct dictEntry{
     //键
     void *key;
     //值
     union{
          void *val;
          uint64_tu64;
          int64_ts64;
     }v;
 
     //指向下一个哈希表节点，形成链表
     struct dictEntry *next;
}dictEntry;

//哈希表
typedef struct dictht{
     //哈希表数组
     dictEntry **table;
     //哈希表大小
     unsigned long size;
     //哈希表大小掩码，用于计算索引值
     //总是等于 size-1
     unsigned long sizemask;
     //该哈希表已有节点的数量
     unsigned long used;
 
}dictht;

//字典
typedef struct dict{
    //类型特定函数
    dictType *type;
    //私有数据
    void *privdata;
    //哈希表
    dictht ht[2];
    //rehash索引
    //当rehash不在进行时，为-1；
    int rehashidx;
}dict;
```

type属性是一个执行dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同类型的特定函数，

privadata属性保存了需要传给那些特定类型特定函数的可选参数

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508183859.png" alt="Redis字典结构"  />

#### 哈希算法

1、使用字典设置的哈希函数，计算键 key 的哈希值

hash = dict->type->hashFunction(key);

2、使用哈希表的sizemask属性和第一步得到的哈希值，计算索引值

index = hash & dict->ht[x].sizemask;

>根据不同的状态，h[x]可以是0或者1
>
>Redis使用的是MurmurHash2算法来计算键的hash值

#### 解决键冲突（哈希碰撞）

方法是链地址法。通过字典里面的 *next 指针指向下一个具有相同索引值的哈希表节点。

#### [rehash](https://blog.csdn.net/csdnlijingran/article/details/89094683)

#### 扩容与收缩

- 服务器目前**没有执行**bgsave或bgrewriteaof命令，并且哈希表的**负载因子>=1**
- 服务器目前**正在执行**bgsave或bgrewriteaof命令，并且哈希表的**负载因子>=5**

> 负载因子的计算

**load_factor=ht[0].used/ht[0].size**

> 负载因子不同的原因

当服务器目前**正在执行**bgsave或bgrewriteaof命令的过程中，Redis需要创建当前服务器的子进程，而大多数操作系统（OS）都采用写时复制（copy-on-write）技术来优化子进程的使用效率，所以在子进程执行期间，服务器会提高执行扩展操作所需要的负载因子，从而尽可能的避免在子进程存在期间进行哈希标的扩展操作，这可以避免不必要的内存写入操作，**最大限度的节约了内存**

当负载因子的值**小于0.1时**，程序就会**对哈希表进行收缩操作**

#### 渐进式rehash

整个rehash过程并**不是一步完成的**，**而是分多次、渐进式的完成**。如果哈希表中，保存着数量巨大的键值对时，若一次进行rehash，很有可能会导致服务器宕机。

步骤：

- 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表
- 维持索引计数器变量 rehashidx，并将它的值设置为0，表示rehash开始
- 每次对字典执行增删改查时，将**ht[0]的rehashidx索引上** 的所有键值对 rehash到ht[1]，将rehashidx值+1。
- 当ht[0]的所有键值对都被rehash到ht[1]中，程序将rehashidx的值设置为-1，**表示rehash操作完成**

> 渐进式rehash的**好处在于**它采取分为而治的方式，**将rehash键值对的计算** `均摊到每个字典增删改查操作`，**避免了集中式rehash的庞大计算量**。

#### rehash执行期间的哈希表操作

渐进式进行期间，字典的删除、查找、更新等操作会在两个表上进行，例如，要在字典中查找一个键的话，程序会先在ht[0]中查找，如果没有找到，就会继续在ht[1]中查找，诸如此类。

对于添加操作来说，新添加的键值对一律保存在ht[1]中，保证了ht[0]包含的键值对数量，只会减少不会增加。最终变为空表。

### 跳跃表

> 有序集合键底层实现之一

> 跳跃表（skiplist）是一种有序数据结构，它通过在每个节点中维持多个指向其它节点的指针，从而达到快速访问节点的目的。
>
> 跳跃表支持平均O(logN)、最坏O(N)复杂度的节点查找，还可以通过顺序性操作来批量处理节点
>
> 大部分情况，跳跃表的效率可以和平衡树相媲美，而且跳跃表的实现相对来说简单

#### 定义

```c
//跳跃表节点
typedef struct zskiplistNode {
     //层
     struct zskiplistLevel{
           //前进指针
           struct zskiplistNode *forward;
           //跨度
           unsigned int span;
     }level[];
 
     //后退指针
     struct zskiplistNode *backward;
     //分值
     double score;
     //成员对象
     robj *obj;
 
} zskiplistNode;

//跳跃表
typedef struct zskiplist{
     //表头节点和表尾节点
     structz skiplistNode *header, *tail;
     //表中节点的数量
     unsigned long length;
     //表中层数最大的节点的层数
     int level;
 
}zskiplist;
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508183926.png" alt="image-20210508183925881" style="zoom:80%;" />

head：指向跳跃表的表头节点。

tail：指向跳跃表的表尾节点。

level：记录当前跳跃表中，层数最高的节点的层数（表头节点的层数不计算）。

length：记录跳跃表的长度，即包含节点的数量。

level：每一层都有前进指针和跨度，从头到尾遍历时，访问会沿着层的前进指针进行。

BW：后退指针，指向前一个节点，从尾到头遍历时使用。

score：分值，跳跃表中的分值按从小到大排列。

obj：成员对象，各个节点保存有各个成员对象。

#### 性质

- 由很多层结构组成；
- 每一层都是一个有序的链表，排列顺序为由高层到底层，都至少包含两个链表节点，分别是前面的head节点和后面的nil节点；
- 最底层的链表包含了所有的元素；
- 如果一个元素出现在某一层的链表中，那么在该层之下的链表也全都会出现（上一层的元素是当前层的元素的子集）；
- 链表中的每个节点都包含两个指针，一个指向同一层的下一个链表节点，另一个指向下一层的同一个链表节点；

#### 操作

- 搜索：从最高层的链表节点开始，如果比当前节点要大和比当前层的下一个节点要小，那么则往下找，也就是和当前层的下一层的节点的下一个节点进行比较，以此类推，一直找到最底层的最后一个节点，如果找到则返回，反之则返回空。
- 插入：首先确定插入的层数，有一种方法是假设抛一枚硬币，如果是正面就累加，直到遇见反面为止，最后记录正面的次数作为插入的层数。当确定插入的层数k后，则需要将新元素插入到从底层到k层。
- 删除：在各个层中找到包含指定值的节点，然后将节点从链表中删除即可，如果删除以后，只剩下头尾两个节点，则删除这一层。
  回到顶部

### 整数集合

> 集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis就会使用整数集合作为集合键的底层实现

#### 实现

```c
typedef struct intset{
     //编码方式
     uint32_t encoding;
     //集合包含的元素数量
     uint32_t length;
     //保存元素的数组
     int8_t contents[];
 
}intset;
```

contents数组是整数集合底层的实现：整数集合的每个元素都是contents数组的一个数组项，各个项在数组中按值的大小从小到大有序排列，并且数组不包含重复项。

length 属性记录了 contents 数组的大小。需要注意的是虽然 contents 数组声明为 int8_t 类型，但是实际上contents 数组并不保存任何 int8_t 类型的值，其真正类型有 encoding 来决定。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509114504.png" alt="image-20210509114502384" style="zoom:80%;" />

#### 升级

> 每当将一个新元素添加到整数集合时，并且新元素的类型比整数集合现有的所有元素类型都要长时，整数集合需要先进性升级（upgrade），然后才能将新元素添加到整数集合中

升级的步骤

- 根据新元素的类型，扩展整数集合底层数组空间大小，并且为新元素分配空间
- 将底层数组现有的所有元素都转换成与新元素相同的类型，并将类型转化后的元素防在正确的位上，而且放置元素的过程中，需要继续维持底层数组的有序性质不变
- 将新元素添加到底层数组中

举例

假设现在有一个INTSET_ENC_INT16编码的整数集合，需要插入int32_t类型的整数65535，如下为升级过程

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509120407.png" alt="image-20210509120405804" style="zoom:80%;" />

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509120426.png" alt="image-20210509120423905" style="zoom:80%;" />

#### 升级的好处

> 一是提升了整数集合的灵活性，另一个是尽可能地节约内存

- 因为存在升级的方法来适应新元素，所以我们可以随意的将int16、int32、int64类型的整数添加到集合中，不用担心类型错误，非常灵活
- 只有在存储长度更长的元素时才进行升级操作，节约了呢内存。

#### 降级

整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态。

### 压缩列表

> 压缩列表是列表键和哈希建的底层实现之一，当一个列表键质包含少量的列表项，并且每个列表项要么就是最小整数，要么就是长度比较短的字符串，那么Redis就会使用压缩列表来做列表键的底层实现。

#### 实现

压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210509145245.png)

> **压缩列表的原理：** 压缩列表并不是对数据利用某种算法进行压缩，而是将数据按照一定规则编码在一块连续的内存区域，目的是节省内存。

压缩列表节点的构成

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509145657.png" alt="image-20210509145655911" style="zoom:80%;" />

- previous_entry_length：记录压缩列表 前一个字节 的长度。previous_entry_length的长度可能是1个字节或者是5个字节，如果上一个节点的长度小于254，则该节点只需要一个字节就可以表示前一个节点的长度了，如果前一个节点的长度大于等于254，则previous length的第一个字节为254，后面用四个字节表示当前节点前一个节点的长度。利用此原理，即**当前节点位置 减去 上一个节点的长度即得到上一个节点的起始位置**，压缩列表 可以 从尾部向头部 遍历。这么做很有效地 减少了 内存的浪费。
- encoding：节点的encoding保存的是节点的content的内容类型以及长度，encoding类型有两种，一种字节数组，一种是整数，encoding区域长度为1字节、2字节或者5字节长。
- content：content区域用于保存 节点的内容，节点内容类型和长度，由encoding决定。

#### 连锁更新

考虑一个问题，在一个压缩列表中，有多个连续的、长度介于250字节到253字节之间的节点e1-eN，因为e1-eN的所有节点的长度都小于254字节，所以记录这些节点只需要1个字节长的previous_entry_length属性，即e1-eN的所有节点的previous_entry_length属性为1个字节，这时如果我们将一个长度大于254字节的新节点new设置为压缩列表的头节点，那么new将成为e1的前置节点。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509150649.png" alt="image-20210509150648486" style="zoom: 80%;" />

e1的previous_entry_length为1个字节无法保存new的长度，因此e1节点的previous_entry_length属性从原来的1字节增长到5字节，此时e1字节本身的长度超过254，一次会引发一系列的更新操作

> 因为连锁更新在最坏的情况下需要对压缩列表执行N次空间重分配操作，而每次空间重分配的最坏复杂度为O()N连，所以连锁更新的最坏复杂度为O(n^2),但是连续更新真正造成性能问题的几率是很低的

- 首先压缩列表中要恰好有多个连续的，长度介于250字节至253字节之间的节点，连续更新才有可能被触发，实际情况下不多见
- 其次只要连续更新的节点数量不多，就不会对性能造成任何影响

> 因为以上原因ziplistpush等命令的平均复杂度为O(N)

### 对象

> Redis并没有直接使用这些数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统，这个系统包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象。每种对象都用到了至少一种之前的数据结构

#### RedisObject

```c
typedef struct redisObject {
    //类型
    unsigned type:4;
    
    //编码
    unsigned encoding:4;
    
    //指向底层实现数据结构的指针
    void *ptr;
} robj;
```

- type：记录了对象类型
- encoding：记录了对象所使用的编码，也即是说这个对象使用了什么数据结构作为对象的底层实现
- ptr：ptr指针执行对象的底层实现数据结构，而这些数据结构由对象的encoding属性决定

|     类型     |           编码            |                      对象                      |
| :----------: | :-----------------------: | :--------------------------------------------: |
| REDIS_STRING |    REDIS_ENCODING_INT     |           使用整数值实现的字符串对象           |
| REDIS_STRING |   REDIS_ENCODING_EMBSTR   | 使用embstr编码的简单动态字符串实现的字符串对象 |
| REDIS_STRING |    REDIS_ENCODING_RAW     |       使用简单动态字符串实现的字符串对象       |
|  REDIS_LIST  |  REDIS_ENCODING_ZIPLIST   |           使用压缩列表实现的列表对象           |
|  REDIS_LIST  | REDIS_ENCODING_LINKEDLIST |           使用双端链表实现的列表对象           |
|  REDIS_HASH  |  REDIS_ENCODING_ZIPLIST   |             使用压缩列表实现的哈希             |
|  REDIS_HASH  |     REDIS_ENCODING_HT     |               使用字典实现的哈希               |
|  REDIS_SET   |   REDIS_ENCODING_INTSET   |           使用整数集合实现的集合对象           |
|  REDIS_SET   |     REDIS_ENCODING_HT     |             使用字典实现的集合对象             |
|  REDIS_ZSET  |  REDIS_ENCODING_ZIPLIST   |         使用压缩列表实现的有序集合对象         |
|  REDIS_ZSET  |  REDIS_ENCODING_SKIPLIST  |       使用跳跃表和字典实现的有序集合对象       |

> 通过encoding属性来设置对象所使用的编码，而不是为特定类型的对象关联一种固定的编码，极大地提高了Redis的灵活性和效率



> Redis支持五种数据类型：String（字符串）、hash（哈希）、list（列表）、set（集合）及zset（sorted set：有序集合）

#### String（字符串）

- String是Redis最基本的数据类型，可以理解成与Memcached一摸一样的类型，一个key对应一个value。
- string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。
- string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

>string 类型的值最大能存储 512MB

#### List（列表）

Redis列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表头部（左边）或者尾部（右边）

>列表最多可存储 2^32 - 1 元素 (4294967295, 每个列表可存储40多亿)。

#### Set（集合）

Redis 的 Set 是 string 类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

#### Hash（哈希）

Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

实例中我们使用Redis HMSET、HGET命令。HMSET设置了两个field=>value对，HGET获取对应field对应的value

> 每个 hash 可以存储 2^32 -1 键值对（40多亿）。

#### zset（有序集合）

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

有序集合的编码可以是ziplist或者skiplist

##### 压缩列表

压缩列表内的集合元素按照分值大小从小到大进行排序，分值较小的元素被防止在靠近表头的位置，而分值比较大的元素被放置在靠近表尾的位置

##### skiplist

```c
typedef struct zset {
    ziplist *zsl;
    dict *dict;
} zset;
```

理论上，有序集合可以单独使用字典或者跳跃表的其中一种数据结构来实现，但是无论单独使用字典还是跳跃表，在性能上对比同时使用字典和跳跃表的性能都会有所降低，举个例子

如果我们只使用字典来实现有序集合，那么虽然以O(1)复杂度查找成员分值这一特性会被保留，但是因为字典以无序的方式来保存集合元素，所以每次执行范围操作（ZRANK、ZRANGE）等命令时，程序都需要对字典保存的所有元素进行排序，完成排序至少需要O(NlogN)时间复杂度，以及额外的O(N)的空间复杂度（因为要创建一个数组来保存排序后的元素）。

另一方面，如果我们只使用跳跃表来实现有序集合，那么跳跃表执行范围操作的所有优点都会被保留，但因为没有字典，所以根据成员查分值这一操作的复杂度将从O(1)上升到O(logN)。

>总结：为了让有序集合的查找和范围型操作尽可能快的执行，Redis选择同时使用字典和跳跃表两种数据结构来实现有序集合

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510175549.png" alt="image-20210510161321232" style="zoom: 80%;" />

### 多态命令的实现

Redis除了会根据值对象的类型来判断键是够够执行指定命令外，还会根据值对象的编码方式，选择正确的命令实现代码来执行命令

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510175550.png" alt="image-20210510170252239" style="zoom:80%;" />

### 内存回收

因为C语言不具备自动内存回收功能，所有Redis在自己对象系统中构建了一个引用计数（reference counting）计数实现的内存回收机制，通过这一机制，程序可以通过对象的引用计数信息，在适当的时候自动释放对象并进行内存回收

```c
typedef struct redisObject {
    //...
    
    //引用计数
    int refcount;
    
    //...
} robj;
```

- 当创建一个对象，引用计数的值被初始化为1
- 当对象被一个新的程序使用时，它的引用数值加1
- 当不在被一个程序使用时，它的引用数值减1
- 当引用计数为0时，释放对象内存

### 对象共享

Redis会在初始化服务器时，创建一万个字符串对象，这些对象**包含0-9999的所有整数值**，当服务器需要用到值0-9999的字符串对象时，服务器就会使用这些共享对象

> 共享字符串可以通过设置REDIS_SHARED_INTEGERS常量修改

```bash
127.0.0.1:6379> set A 100
OK
127.0.0.1:6379> OBJECT REFCOUNT A
(integer) 2
127.0.0.1:6379> SET B 100
O
127.0.0.1:6379> OBJECT REFCOUNT B
(integer) 3
```



<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510175551.png" alt="image-20210510163727805" style="zoom:80%;" />

### 对象的空转时长

redisObjet的最后一个属性，lru属性，该属性记录了对象最后一次命中程序的访问时间

```c
typedef struct redisObject {
    //...
    
    unsigned lru:22;
    
    //...
} robj;
```

通过OBJECT IDLETIME命令查看

```bash
127.0.0.1:6379> OBJECT IDLETIME A
(integer) 160
127.0.0.1:6379> OBJECT IDLETIME A	# 该命令不会修改lru值
(integer) 172
127.0.0.1:6379> get A
"100"
127.0.0.1:6379> OBJECT IDLETIME A
(integer) 2 
```

> 键空转的作用

除了可以被OBJECT IDLETIME命令打印出来外，键的空转时长还有另外一项作用，如果服务器打开了maxmemory选项，并且服务器用于回收内存算法为volatile-lru或者allkeys-lru，那么服务器占用的内存数超过了maxmemory选项所设置的上限值时，空转时长较高的那部分键会优先被服务器释放，从而回收

## 三种特殊的数据类型

### geospatial地理空间

#### 概述

Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作

#### 命令

##### geoadd

> 添加地理位置的坐标，可以将一个或者多个经度、纬度、位置名称添加到指定的key中

语法格式

```bash
GEOADD key longitude latitiude member  [longitude latitude member ...]
```

```bash
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai 106.50 29.53 chongqing 
(integer) 2
127.0.0.1:6379> geoadd china:city 114.05 22.52 shenzhen 120.16 30.24 hangzhou 108.96 34.26 xian
```

##### geopos

> geopos用于从给定的key里返回所有指定名称（member）的位置（经度和纬度），不存的分会nil

语法格式

```bash
GEOPOS key member [member ...]
```

```bash
127.0.0.1:6379> GEOPOS china:city chongqing
1) 1) "106.49999767541885376"
   2) "29.52999957900659211"

```

##### geodist

> geodist用于返回两个给定位置之间的距离

语法格式

```bash
GEODIST key member1 member 2 [m|km|ft|mi]
```

```bash
127.0.0.1:6379> GEODIST china:city chongqing xian km
"575.0470"
127.0.0.1:6379> GEODIST china:city chongqing shanghai km
"1447.6737"
```

##### georadius\georadiusbymember

> georadius以给定的经纬度为中心，返回键包含的位置元素当中，与中心距离不超过给定最大距离的所有位置元素
>
> georadiusbymember 和 GEORADIUS 命令一样， 都可以找出位于指定范围内的元素， 但是 georadiusbymember 的中心点是由给定的位置元素决定的， 而不是使用经度和纬度来决定中心点。

语法格式

```bash
GEORADIUS key longitude latitude radius  m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]

GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```

参数说明：

- m ：米，默认单位。
- km ：千米。
- mi ：英里。
- ft ：英尺。
- WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
- WITHCOORD: 将位置元素的经度和维度也一并返回。
- WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。
- COUNT 限定返回的记录数。
- ASC: 查找结果根据距离从近到远排序。
- DESC: 查找结果根据从远到近排序。

```bash
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km 
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km withdist # 带距离的查询
1) 1) "chongqing"
   2) "341.9374"
2) 1) "xian"
   2) "483.8340"
3) 1) "shenzhen"
   2) "924.6408"
4) 1) "hangzhou"
   2) "977.5143"
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km withcoord # 带经纬度的查询
1) 1) "chongqing"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) 1) "108.96000176668167114"
      2) "34.25999964418929977"
3) 1) "shenzhen"
   2) 1) "114.04999762773513794"
      2) "22.5200000879503861"
4) 1) "hangzhou"
   2) 1) "120.1600000262260437"
      2) "30.2400003229490224"
127.0.0.1:6379> GEORADIUSBYMEMBER china:city xian 1000 km
1) "xian"
2) "chongqing"
3) "beijing"
```

##### geohash

>geohash 来保存地理位置的坐标。geohash 用于获取一个或多个位置元素的 geohash 值。

语法格式

```bash
GEOHASH key member [memver ...]
```

```bash
127.0.0.1:6379> GEOHASH china:city xian chongqing beijing shanghai
1) "wqj6zky6bn0"
2) "wm5xzrybty0"
3) "wx4fbxxfke0"
4) "wtw3sj5zbj0"
```

值越相近，距离越近

> geo底层的实现原理其实就是ZSET ，可以通过zset的命令来操作geo。例如：ZRANGE 、ZREM

### Hyperloglog

#### 概述

Redis HyperLogLog 时用来基数统计的算法，HyperLogLog的优点是，在输入元素数量或者体积非常非常大的时候，计算基数所需要的空间总是固定的，并且很小。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是HyperLogLog只会根据输入的元素来计算基数，而不会存储输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

> 什么是基数

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

#### 命令

##### PFADD

添加指定元素到hyperloglog中

```bash
127.0.0.1:6379> PFADD list1 a b c d e f g
(integer) 1
(3.17s)
127.0.0.1:6379> PFADD list2 a c j k l m
(integer) 1
```

##### PFCOUNT

返回集合中的基数估算值

```bash
127.0.0.1:6379> PFCOUNT list1
(integer) 7
127.0.0.1:6379> PFCOUNT list2
(integer) 6
```

##### PFMEGRE

将多个hyperloglog合并为一个hyperloglog

```bash
127.0.0.1:6379> PFMERGE list3 list1 list2
OK
127.0.0.1:6379> PFCOUNT list3
(integer) 11
```

### Bitmaps

#### 概述

> 位存储

统计用户信息，活跃，不活跃。

Bitmap位图，数据结构，都是操作二进制位来进行记录，就只有0 和 1两个状态

#### 命令

##### setbit：存值

##### getbit：取值

##### bitcount：统计数量

```bash
127.0.0.1：6379> setbit sign 20210415 0 
(integer) 0
127.0.0.1：6379> setbit sign 20210416 1
(integer) 0
127.0.0.1：6379> setbit sign 20210414 0
(integer) 0
127.0.0.1：6379> setbit sign 20210413 1
(integer) 0
127.0.0.1：6379> setbit sign 20210412 1
(integer) 0
127.0.0.1：6379> setbit sign 20210417 1
(integer) 0
127.0.0.1：6379> getbit sign 20210416
(integer) 1
127.0.0.1：6379> getbit sign 20210415
(integer) 0
127.0.0.1：6379> bitcount sign
(integer) 4
```

## Redis命令

### 键（key）

Redis键命令用于管理Redis的键

> 语法

```bash
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [DEL key](https://www.runoob.com/redis/keys-del.html) 该命令用于在 key 存在时删除 key。 |
| 2    | [DUMP key](https://www.runoob.com/redis/keys-dump.html) 序列化给定 key ，并返回被序列化的值。 |
| 3    | [EXISTS key](https://www.runoob.com/redis/keys-exists.html) 检查给定 key 是否存在。 |
| 4    | [EXPIRE key](https://www.runoob.com/redis/keys-expire.html) seconds 为给定 key 设置过期时间，以秒计。 |
| 5    | [EXPIREAT key timestamp](https://www.runoob.com/redis/keys-expireat.html) EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| 6    | [PEXPIRE key milliseconds](https://www.runoob.com/redis/keys-pexpire.html) 设置 key 的过期时间以毫秒计。 |
| 7    | [PEXPIREAT key milliseconds-timestamp](https://www.runoob.com/redis/keys-pexpireat.html) 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 |
| 8    | [KEYS pattern](https://www.runoob.com/redis/keys-keys.html) 查找所有符合给定模式( pattern)的 key 。 |
| 9    | [MOVE key db](https://www.runoob.com/redis/keys-move.html) 将当前数据库的 key 移动到给定的数据库 db 当中。 |
| 10   | [PERSIST key](https://www.runoob.com/redis/keys-persist.html) 移除 key 的过期时间，key 将持久保持。 |
| 11   | [PTTL key](https://www.runoob.com/redis/keys-pttl.html) 以毫秒为单位返回 key 的剩余的过期时间。 |
| 12   | [TTL key](https://www.runoob.com/redis/keys-ttl.html) 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| 13   | [RANDOMKEY](https://www.runoob.com/redis/keys-randomkey.html) 从当前数据库中随机返回一个 key 。 |
| 14   | [RENAME key newkey](https://www.runoob.com/redis/keys-rename.html) 修改 key 的名称 |
| 15   | [RENAMENX key newkey](https://www.runoob.com/redis/keys-renamenx.html) 仅当 newkey 不存在时，将 key 改名为 newkey 。 |
| 16   | [SCAN cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/keys-scan.html) 迭代数据库中的数据库键。 |
| 17   | [TYPE key](https://www.runoob.com/redis/keys-type.html) 返回 key 所储存的值的类型。 |

实例

```bash
127.0.0.1:6379[3]> EXPIRE k1 100
(integer) 1
127.0.0.1:6379[3]> PERSIST k1 #解除key的过期时间，将key持久保存
(integer) 1
127.0.0.1:6379[3]> ttl k1 #以秒为单位，返回给定key的剩余生存时间
(integer) -1 #永久存在
127.0.0.1:6379[3]> get k1
"zhangsan"
127.0.0.1:6379[3]> EXPIRE k1 100
(integer) 1
127.0.0.1:6379[3]> PERSIST k1
(integer) 1
127.0.0.1:6379[3]> RANDOMKEY # 随机获取当前数据库中的一个key
"k1"
127.0.0.1:6379[3]> RENAME k1 k2 #修改k1的名称为k2
OK
127.0.0.1:6379[3]> RANDOMKEY
"k2"
127.0.0.1:6379[3]> RENAMENX k1 k2 #仅当k2不存在时，将 key 改名为 newkey 。
```

### 字符串（String）

Redis 字符串数据类型的相关命令用于管理 redis 字符串值，基本语法如下：

> 语法

```bash
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [SET key value](https://www.runoob.com/redis/strings-set.html) 设置指定 key 的值 |
| 2    | [GET key](https://www.runoob.com/redis/strings-get.html) 获取指定 key 的值。 |
| 3    | [GETRANGE key start end](https://www.runoob.com/redis/strings-getrange.html) 返回 key 中字符串值的子字符 |
| 4    | [GETSET key value](https://www.runoob.com/redis/strings-getset.html) 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。 |
| 5    | [GETBIT key offset](https://www.runoob.com/redis/strings-getbit.html) 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。 |
| 6    | [MGET key1 [key2..\]](https://www.runoob.com/redis/strings-mget.html) 获取所有(一个或多个)给定 key 的值。 |
| 7    | [SETBIT key offset value](https://www.runoob.com/redis/strings-setbit.html) 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。 |
| 8    | [SETEX key seconds value](https://www.runoob.com/redis/strings-setex.html) 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。 |
| 9    | [SETNX key value](https://www.runoob.com/redis/strings-setnx.html) 只有在 key 不存在时设置 key 的值。 |
| 10   | [SETRANGE key offset value](https://www.runoob.com/redis/strings-setrange.html) 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| 11   | [STRLEN key](https://www.runoob.com/redis/strings-strlen.html) 返回 key 所储存的字符串值的长度。 |
| 12   | [MSET key value [key value ...\]](https://www.runoob.com/redis/strings-mset.html) 同时设置一个或多个 key-value 对。 |
| 13   | [MSETNX key value [key value ...\]](https://www.runoob.com/redis/strings-msetnx.html) 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。 |
| 14   | [PSETEX key milliseconds value](https://www.runoob.com/redis/strings-psetex.html) 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。 |
| 15   | [INCR key](https://www.runoob.com/redis/strings-incr.html) 将 key 中储存的数字值增一。 |
| 16   | [INCRBY key increment](https://www.runoob.com/redis/strings-incrby.html) 将 key 所储存的值加上给定的增量值（increment） 。 |
| 17   | [INCRBYFLOAT key increment](https://www.runoob.com/redis/strings-incrbyfloat.html) 将 key 所储存的值加上给定的浮点增量值（increment） 。 |
| 18   | [DECR key](https://www.runoob.com/redis/strings-decr.html) 将 key 中储存的数字值减一。 |
| 19   | [DECRBY key decrement](https://www.runoob.com/redis/strings-decrby.html) key 所储存的值减去给定的减量值（decrement） 。 |
| 20   | [APPEND key value](https://www.runoob.com/redis/strings-append.html) 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。 |

实例

```bash
127.0.0.1:6379> set k1 zhangsan # 设置只当key的值
OK
127.0.0.1:6379> GETRANGE k1 0 5 # 获取范围内的字串
"zhangs"
127.0.0.1:6379> GETSET k1 wangwu # 获取并且修改key
"zhangsan"
127.0.0.1:6379> GETBIT k1 3 # 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
(integer) 1
127.0.0.1:6379> MGET k1 k2 # 获取多个key
1) "wangwu"
2) "lisi"
127.0.0.1:6379> SETEX k1 10 zhangsan # 设置key对应value值生存生时间s
OK
127.0.0.1:6379> ttl k1
(integer) 6
127.0.0.1:6379> SETNX k1 zhangsan # 如果不存在则设置键值
(integer) 1
127.0.0.1:6379> SETNX k1 zhangsan
(integer) 0
127.0.0.1:6379> STRLEN k1 # 获取String类型的长度
(integer) 8
127.0.0.1:6379> SETRANGE k1 4 0 # replace替换字串
(integer) 8
127.0.0.1:6379> get k1
"zhan0san"
127.0.0.1:6379> MSET k3 zhaoliu k4 zhangfei # 设置多个键值
OK
127.0.0.1:6379> MSETNX k3 zhaoliu k5 111 # 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。14
(integer) 0
127.0.0.1:6379> INCR k5 # 自增1
(integer) 11
127.0.0.1:6379> INCRBY k5 10
(integer) 21
127.0.0.1:6379> APPEND k1 hello # 拼接字符串
(integer) 13
127.0.0.1:6379> get k1
"zhanmusihello"
127.0.0.1:6379> set user {name:zhangsan,age:20}  # 设置对象（Json格式）
#高阶用法
mset user:id:age 20 user:id:sex 1
```

### 列表（list）

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [BLPOP key1 [key2 \] timeout](https://www.runoob.com/redis/lists-blpop.html) 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 2    | [BRPOP key1 [key2 \] timeout](https://www.runoob.com/redis/lists-brpop.html) 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 3    | [BRPOPLPUSH source destination timeout](https://www.runoob.com/redis/lists-brpoplpush.html) 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 4    | [LINDEX key index](https://www.runoob.com/redis/lists-lindex.html) 通过索引获取列表中的元素 |
| 5    | [LINSERT key BEFORE\|AFTER pivot value](https://www.runoob.com/redis/lists-linsert.html) 在列表的元素前或者后插入元素 |
| 6    | [LLEN key](https://www.runoob.com/redis/lists-llen.html) 获取列表长度 |
| 7    | [LPOP key](https://www.runoob.com/redis/lists-lpop.html) 移出并获取列表的第一个元素 |
| 8    | [LPUSH key value1 [value2\]](https://www.runoob.com/redis/lists-lpush.html) 将一个或多个值插入到列表头部 |
| 9    | [LPUSHX key value](https://www.runoob.com/redis/lists-lpushx.html) 将一个值插入到已存在的列表头部 |
| 10   | [LRANGE key start stop](https://www.runoob.com/redis/lists-lrange.html) 获取列表指定范围内的元素 |
| 11   | [LREM key count value](https://www.runoob.com/redis/lists-lrem.html) 移除列表元素 |
| 12   | [LSET key index value](https://www.runoob.com/redis/lists-lset.html) 通过索引设置列表元素的值 |
| 13   | [LTRIM key start stop](https://www.runoob.com/redis/lists-ltrim.html) 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。 |
| 14   | [RPOP key](https://www.runoob.com/redis/lists-rpop.html) 移除列表的最后一个元素，返回值为移除的元素。 |
| 15   | [RPOPLPUSH source destination](https://www.runoob.com/redis/lists-rpoplpush.html) 移除列表的最后一个元素，并将该元素添加到另一个列表并返回 |
| 16   | [RPUSH key value1 [value2\]](https://www.runoob.com/redis/lists-rpush.html) 在列表中添加一个或多个值 |
| 17   | [RPUSHX key value](https://www.runoob.com/redis/lists-rpushx.html) 为已存在的列表添加值 |

实例

```bash
127.0.0.1:6379> LPUSH list h #头部边添加元素
(integer) 1
127.0.0.1:6379> LLEN list #获取列表的长度
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1 #遍历列表从0 到 尾
1) "o"
2) "l"
3) "l"
4) "e"
5) "h"
127.0.0.1:6379> lpop list #移除列表最后一个元素
"o"
127.0.0.1:6379> LRANGE list 0 -1
1) "l"
2) "l"
3) "e"
4) "h"
127.0.0.1:6379> LINDEX list 2 #获取下表为2的元素
"l"
127.0.0.1:6379> RPOPLPUSH list mylist #列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它
"o"
127.0.0.1:6379> LTRIM list 3 4 #修建列表，只保留3 到 4 的元素
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "l"
2) "0"
127.0.0.1:6379> LINSERT list after o world # 在指定元素后面或者前面插入元素
(integer) 6
127.0.0.1:6379> LRANGE list 0 -1
1) "h"
2) "e"
3) "l"
4) "l"
5) "o"
6) "world"
127.0.0.1:6379> LPUSHX list 1 #插入已经存在的列表
(integer) 7
127.0.0.1:6379> LRANGE list 0 -1
1) "1"
2) "h"
3) "e"
4) "l"
5) "l"
6) "o"
7) "world"
127.0.0.1:6379> LRANGE mylist 0 -1
(empty array)
127.0.0.1:6379> LSET list 6 redis #修改列表指定下标的值
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "1"
2) "h"
3) "e"
4) "l"
5) "l"
6) "o"
7) "redis"
```

> 小结

- set实际是一个链表，before Node after，left ，right 都可以插入值

- 如果key 不存在，创建新的链表

- 如果key 存在，新增内容

- 如果移除了所有的值，空链表，也代表不存在

- 在两边插入或改动值，效率最高！中间元素，相对来说效率会第一点

  可以用来 消息排队！ 消息队列（Lpush Rpop） 左	进右出 ，栈（Lpush Lpop）左进左出

### 集合（Set）

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [SADD key member1 [member2\]](https://www.runoob.com/redis/sets-sadd.html) 向集合添加一个或多个成员 |
| 2    | [SCARD key](https://www.runoob.com/redis/sets-scard.html) 获取集合的成员数 |
| 3    | [SDIFF key1 [key2\]](https://www.runoob.com/redis/sets-sdiff.html) 返回第一个集合与其他集合之间的差异。 |
| 4    | [SDIFFSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sdiffstore.html) 返回给定所有集合的差集并存储在 destination 中 |
| 5    | [SINTER key1 [key2\]](https://www.runoob.com/redis/sets-sinter.html) 返回给定所有集合的交集 |
| 6    | [SINTERSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sinterstore.html) 返回给定所有集合的交集并存储在 destination 中 |
| 7    | [SISMEMBER key member](https://www.runoob.com/redis/sets-sismember.html) 判断 member 元素是否是集合 key 的成员 |
| 8    | [SMEMBERS key](https://www.runoob.com/redis/sets-smembers.html) 返回集合中的所有成员 |
| 9    | [SMOVE source destination member](https://www.runoob.com/redis/sets-smove.html) 将 member 元素从 source 集合移动到 destination 集合 |
| 10   | [SPOP key](https://www.runoob.com/redis/sets-spop.html) 移除并返回集合中的一个随机元素 |
| 11   | [SRANDMEMBER key [count\]](https://www.runoob.com/redis/sets-srandmember.html) 返回集合中一个或多个随机数 |
| 12   | [SREM key member1 [member2\]](https://www.runoob.com/redis/sets-srem.html) 移除集合中一个或多个成员 |
| 13   | [SUNION key1 [key2\]](https://www.runoob.com/redis/sets-sunion.html) 返回所有给定集合的并集 |
| 14   | [SUNIONSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sunionstore.html) 所有给定集合的并集存储在 destination 集合中 |
| 15   | [SSCAN key cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/sets-sscan.html) 迭代集合中的元素 |

实例

```bash
127.0.0.1:6379> sadd set2 a b d f e		# 给set中添加元素
(integer) 5
127.0.0.1:6379> SDIFF set1 set2			# 返回两个集合的差异
1) "c"
127.0.0.1:6379> Sinter set1 set2		# 返回两个集合的交集
1) "b"
2) "a"
127.0.0.1:6379> SUNION set1 set2		# 返回两个集合的并集
1) "a"
2) "b"
3) "c"
4) "e"
5) "d"
6) "f"
127.0.0.1:6379> SISMEMBER set f 		# 判断元素是否数据集合
(integer) 0
127.0.0.1:6379> SMEMBERS set1  			# 展示集合所有元素
1) "b"
2) "a"
3) "c"
127.0.0.1:6379> SMOVE set2 set1 f  		# 移动集合指定元素到指定集合
(integer) 1
127.0.0.1:6379> SPOP set1				# 随机删除集合元素
"a"
127.0.0.1:6379> SRANDMEMBER set1		# 随机返回一个集合元素
"f"
127.0.0.1:6379> SREM set1 f				# 删除集合指定元素
(integer) 1
127.0.0.1:6379> SMEMBERS set1			
1) "b"
2) "c"
```

> 小结

应用场景

微博 ，A用户将所有的关注的人放在一个set集合中，将他的粉丝也放在一个集合中

共同关注，共同爱好，二度好友，推荐好友！

### 哈希（Hash）

Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [HDEL key field1 [field2\]](https://www.runoob.com/redis/hashes-hdel.html) 删除一个或多个哈希表字段 |
| 2    | [HEXISTS key field](https://www.runoob.com/redis/hashes-hexists.html) 查看哈希表 key 中，指定的字段是否存在。 |
| 3    | [HGET key field](https://www.runoob.com/redis/hashes-hget.html) 获取存储在哈希表中指定字段的值。 |
| 4    | [HGETALL key](https://www.runoob.com/redis/hashes-hgetall.html) 获取在哈希表中指定 key 的所有字段和值 |
| 5    | [HINCRBY key field increment](https://www.runoob.com/redis/hashes-hincrby.html) 为哈希表 key 中的指定字段的整数值加上增量 increment 。 |
| 6    | [HINCRBYFLOAT key field increment](https://www.runoob.com/redis/hashes-hincrbyfloat.html) 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。 |
| 7    | [HKEYS key](https://www.runoob.com/redis/hashes-hkeys.html) 获取所有哈希表中的字段 |
| 8    | [HLEN key](https://www.runoob.com/redis/hashes-hlen.html) 获取哈希表中字段的数量 |
| 9    | [HMGET key field1 [field2\]](https://www.runoob.com/redis/hashes-hmget.html) 获取所有给定字段的值 |
| 10   | [HMSET key field1 value1 [field2 value2 \]](https://www.runoob.com/redis/hashes-hmset.html) 同时将多个 field-value (域-值)对设置到哈希表 key 中。 |
| 11   | [HSET key field value](https://www.runoob.com/redis/hashes-hset.html) 将哈希表 key 中的字段 field 的值设为 value 。 |
| 12   | [HSETNX key field value](https://www.runoob.com/redis/hashes-hsetnx.html) 只有在字段 field 不存在时，设置哈希表字段的值。 |
| 13   | [HVALS key](https://www.runoob.com/redis/hashes-hvals.html) 获取哈希表中所有值。 |
| 14   | [HSCAN key cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/hashes-hscan.html) 迭代哈希表中的键值对。 |

实例

```bash
127.0.0.1:6379> HMSET map name zhangsan password 123456  # 创建hash 设置多个字段
OK
127.0.0.1:6379> HEXISTS map name		# 判断是否存在某个key	
(integer) 1
127.0.0.1:6379> HGET map name		# 获取hash指定字段
"zhangsan"
127.0.0.1:6379> HGETALL map			# 获取所有键值对
1) "name"
2) "zhangsan"
3) "password"
4) "123456"
127.0.0.1:6379> HMSET map age 20	
OK
127.0.0.1:6379> HINCRBY map age 5	# 设置指定字段增加
(integer) 25
127.0.0.1:6379> HKEYS map		# 获取所有key
1) "name"
2) "password"
3) "age"
127.0.0.1:6379> HVALS map		# 获取所有的values
1) "zhangsan"
2) "123456"
3) "25"
127.0.0.1:6379> HSETNX map sex 1		#设置键值如果不存在
(integer) 1

```

### 有序集合（sorted set）

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

> 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)。

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [ZADD key score1 member1 [score2 member2\]](https://www.runoob.com/redis/sorted-sets-zadd.html) 向有序集合添加一个或多个成员，或者更新已存在成员的分数 |
| 2    | [ZCARD key](https://www.runoob.com/redis/sorted-sets-zcard.html) 获取有序集合的成员数 |
| 3    | [ZCOUNT key min max](https://www.runoob.com/redis/sorted-sets-zcount.html) 计算在有序集合中指定区间分数的成员数 |
| 4    | [ZINCRBY key increment member](https://www.runoob.com/redis/sorted-sets-zincrby.html) 有序集合中对指定成员的分数加上增量 increment |
| 5    | [ZINTERSTORE destination numkeys key [key ...\]](https://www.runoob.com/redis/sorted-sets-zinterstore.html) 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中 |
| 6    | [ZLEXCOUNT key min max](https://www.runoob.com/redis/sorted-sets-zlexcount.html) 在有序集合中计算指定字典区间内成员数量 |
| 7    | [ZRANGE key start stop [WITHSCORES\]](https://www.runoob.com/redis/sorted-sets-zrange.html) 通过索引区间返回有序集合指定区间内的成员 |
| 8    | [ZRANGEBYLEX key min max [LIMIT offset count\]](https://www.runoob.com/redis/sorted-sets-zrangebylex.html) 通过字典区间返回有序集合的成员 |
| 9    | [ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT]](https://www.runoob.com/redis/sorted-sets-zrangebyscore.html) 通过分数返回有序集合指定区间内的成员 |
| 10   | [ZRANK key member](https://www.runoob.com/redis/sorted-sets-zrank.html) 返回有序集合中指定成员的索引 |
| 11   | [ZREM key member [member ...\]](https://www.runoob.com/redis/sorted-sets-zrem.html) 移除有序集合中的一个或多个成员 |
| 12   | [ZREMRANGEBYLEX key min max](https://www.runoob.com/redis/sorted-sets-zremrangebylex.html) 移除有序集合中给定的字典区间的所有成员 |
| 13   | [ZREMRANGEBYRANK key start stop](https://www.runoob.com/redis/sorted-sets-zremrangebyrank.html) 移除有序集合中给定的排名区间的所有成员 |
| 14   | [ZREMRANGEBYSCORE key min max](https://www.runoob.com/redis/sorted-sets-zremrangebyscore.html) 移除有序集合中给定的分数区间的所有成员 |
| 15   | [ZREVRANGE key start stop [WITHSCORES\]](https://www.runoob.com/redis/sorted-sets-zrevrange.html) 返回有序集中指定区间内的成员，通过索引，分数从高到低 |
| 16   | [ZREVRANGEBYSCORE key max min [WITHSCORES\]](https://www.runoob.com/redis/sorted-sets-zrevrangebyscore.html) 返回有序集中指定分数区间内的成员，分数从高到低排序 |
| 17   | [ZREVRANK key member](https://www.runoob.com/redis/sorted-sets-zrevrank.html) 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| 18   | [ZSCORE key member](https://www.runoob.com/redis/sorted-sets-zscore.html) 返回有序集中，成员的分数值 |
| 19   | [ZUNIONSTORE destination numkeys key [key ...\]](https://www.runoob.com/redis/sorted-sets-zunionstore.html) 计算给定的一个或多个有序集的并集，并存储在新的 key 中 |
| 20   | [ZSCAN key cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/sorted-sets-zscan.html) 迭代有序集合中的元素（包括元素成员和元素分值） |

# 整合Redis

## 导入Jedis和fastjson的依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.5.2</version>
</dependency>
<!-- fastjson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
</dependency>
```

## 测试Jedis事务

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/7 15:52
 * @description：测试json事务
 */
public class Test {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name","zhangsan");
        jsonObject.put("password", "12345");
        String user = jsonObject.toJSONString();
        Transaction tx = jedis.multi(); //开启事务
        try {
            tx.set("user1",user);
            //tx.incrBy("user1", 10);
            tx.set("user2",user);
            //int i = 1 / 0;
            //执行成功，提交事务
            tx.exec();
        } catch (Exception e) {
            //抛出异常，取消事务
            tx.discard();
            e.printStackTrace();
        } finally {
            //关闭事务
            tx.close();
        }
        System.out.println(jedis.get("user1"));
        System.out.println(jedis.get("user2"));
    }
}

```

## 配置连接远程Linux的Redis

- 注释配置文件中 （#bing 127.0.0.1）
- 如果想要对Redis进行增删改则需要修改protected-mode
  - 关闭protected-mode模式，此时外部网络可以直接访问
  - 开启protected-mode保护模式，需配置bind ip或者设置访问密码
- 设置阿里云安全组
- 重启Redis服务器

```java
Jedis jedis = new Jedis("8.140.116.125", 6379);
jedis.auth("*****");
System.out.println(jedis.set("name","zhaosi"));
```

## Springboot整合

> SpringBoot操作数据：spring-data jpa jdbc mongodb redis

SpringBoot2.x后，原来使用的Jedis被替换为lettuce

Jedis：实现上是直接连接Redis Server，如果在多线程环境下是非线程安全的。每个线程都去拿自己的Jedis实例，当连接数量增多时，资源消耗阶梯式增大，连接成本就较高了。（BIO）

lettuce：基于Netty，Netty是一个多线程、时间驱动的I/O框架。连接实例可以在多个线程中进行共享，不存在线程不安全的情况。当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。通过异步的方式可以让我们更好的利用系统资源，而不用浪费线程等待网络或磁盘I/O。所以 Lettuce 可以帮助我们充分利用异步的优势。（NIO）

>使用连接池，为每个Jedis实例增加物理连接Lettuce的连接是基于Netty的，连接实例（StatefulRedisConnection）可以在多个线程间并发访问，因为StatefulRedisConnection是线程安全的，所以一个连接实例（StatefulRedisConnection）就可以满足多线程环境下的并发访问，当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。

> 存在问题

在存入序列化对象的时候，必须添加对象的无参构造，否则报错SerializationException。

在实体类中得显式定义无参构造器，因为在有参构造器存在的时候，没有了默认的无参构造器redis的这些序列化方式,使用的是无参构造函数进行创建对象，set方法进行赋值。

## [代码实例](https://github.com/dunk-code/springboot_redis)

# Redis持久化

> 数据库状态

Redis是一个键值对数据库服务器，服务器通常包含任意个非空数据库，而每个非空数据库中又可以包含任意个键值对，为了方便期间，我们将服务器中的非空数据库以及他们的键值对统称为数据库状态。如图数据库状态实例

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509180828.png" alt="image-20210509180827771" style="zoom:80%;" />

> 为什么要持久化

因为Redis是内存数据库，它将自己的数据库状态存储在内存中，所以如果不想办法将储存在内存中的数据库状态保存到磁盘里面，那么一旦服务器进程退出，服务器中的数据库状态也会消失不见。

## RDB持久化

> Redis默认使用的是RDB持久化

> RDB持久化既可以手动执行，也可以根据服务器配置选项定期执行，该功能可以将某个时间点上的数据库状态保存到一个RDB文件中

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509181600.png" alt="image-20210509181559637" style="zoom:80%;" />



> RDB文件是一个经过压缩的二进制文件，通过该文件可以还原生成RDB文件时的数据库状态

### RDB文件的创建和载入

> 有两个Redis命令可以用于生成RDB文件，一个是SAVE，另一个是BSAVE

#### 创建

##### SAVE

SAVE命令会阻塞Redis服务器进程，直到RDB文件创建完毕为止，在服务器阻塞期间，服务器不能处理任何命令请求

```bash
redis> SAVE
OK
```

> 服务器状态

只有当服务器执行完SAVE命令，重新开始接受命令之后，客户端发送的命令才会被处理

##### BSAVE

不同于SAVE阻塞服务器进程的做法，BSAVE命令会派生一个子进程，然后由子进程负责创建RDB文件，服务器进程（父进程）继续处理命令请求：

```bash
redis> BSAVE
Background saving started
```

创建RDB文件实际工作伪代码

```python
def SAVE():
    #创建RDB文件
    rdbSAVE()
    
def BSAVE():
    #创建子进程
    pid = fork()
    
    if pid == 0:
        #子进程负责创建RDB文件
        rdbSAVE()
        
        #完成后向父进程发送信号
        singal_parent()
        
	elif pid > 0:
        #父进程继续处理命令请求，并通过轮询等待子进程的信号
        handle_request_and_wait_singal()
        
	else:
        #处理出错情况
        handle_fork_error()
```

> 服务器状态

BSAVE命令执行期间，Redis服务器可以继续处理客户端命令请求，但是，在处理SAVE、BSAVE、BGREWRITEAOF三个命令的方式会和平时不同

- SAVE：拒绝执行，避免父进程和子进程同时执行，防止产生竞争条件
- BSAVE：拒绝执行，防止产生竞争条件
- BGREWRITEAOF：如果BSAVE命令正在执行，那么客户端发送的BGREWRITEAOF命令会被延迟到BGSAVE命令执行之后执行；如果BGREWRITEAOF命令正在执行，那么客户端发送的BSAVE命令将会被服务器拒绝。这两个命令都是由子进程执行的，所以这两个命令并不冲突，不能同时执行他们只是一个性能方面的考虑

#### 载入

> Redis没有专门用于载入RDB文件的命令，RDB文件的载入工作室在服务器启动时自动执行的，只要Redis服务器在启动时检测到了RDB文件的存在，他就会自动载入RDB文件

如果服务器开启了AOF持久化功能，那么服务器会优先使用AOF文件来还原数据库状态。因为AOF文件的更新频率通常比RDB文件的更新频次高。只有在AOF持久化功能关闭的时候，服务器才会用RDB文件来还原数据库状态。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509183125.png" alt="image-20210509183123557" style="zoom:80%;" />

> 服务器状态

服务器在载入RDB文件期间，会一直处于阻塞状态，直到载入工作完成。

### 自动间隔性保存

因为BSAVE命令不会阻塞服务器进程，所以Redis允许用户通过设置服务器配置的save选项，让服务器每隔一段时间自动执行一次命令

```bash
save 900 1
save 300 10
save 60 10000
```

满足以下三个条件之一，BSAVE命令就会被执行

在 900s 内至少有 1个key进行了操作 

在 300s 内至少有 10个key进行了操作 

在 60s 内至少有1w个key进行了操作 

#### 实现原理

如果用户没有主动设置save属性，那么服务器会为save设置默认条件，接着服务器会根据save选项所设置的保存条件，设置服务器状态redisServer结构中的saveparams属性

```c
struct redisServer {
    //....
    
    //记录了保存条件的数组
    struct saveparam *saveparams;
    
    //....
}

struct saveparam {
    //秒数
    time_t seconds;
    
    //修改数
    int change;
}
```

saveparams属性是一个数组，数组每个元素都是一个saveparam结构，每个saveparam都保存了一个save选项设置的保存条件

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509195351.png" alt="image-20210509195347589" style="zoom:80%;" />

#### dirty计数器和lastsave属性

除了saveparams数组之外，服务器维持着一个dirty计数器，以及一个lastsave属性：

- dirty计数器记录上一次成功执行SAVE命令或者BSAVE命令之后，服务器对数据库状态进行了多少次修改（写入、删除、更新）
- lastsave属性是一个UNIX时间戳，记录了服务器上一次成功执行SAVE命令或者BSAVE命令的时间

#### 检查保存条件是否满足

Redis服务器周期操作函数ServerCron默认每隔100ms就会执行一次，该函数用于对正在运行的服务器进行维护，它的其中一项工作就是检查save选项所设置的保存条件是否已经满足，如果满足，就执行BSAVE命令，程序会循环遍历saveparams数组中所有的保存条件，只要有任意一个满足就会执行BSAVE命令

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509200916.png" alt="image-20210509200915894" style="zoom:80%;" />

以上就是Redis服务器根据save选项所设置的保存条件，自行执行BSAVE命令。进行间隔性数据保存的实现原理

### RDB文件结构

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509204827.png" alt="image-20210509201714067" style="zoom:80%;" />

- REDIS：RDB文件最开头的REDIS部分，这个部分长度为5个字节，保存着“REDIS”这五个字符，程序可以在载入文件时，快速检查所载入的文件是否是RDB文件，RDB文件保存额是二进制数据，而不是C字符串，为了方便起见，结尾不带‘\0’
- db_version：记录RDB文件的版本号
- databases：包含零或任意多个数据库，数据库状态为空时，这部分也为空，数据库非空，根据类型不同，这个部分长度也不同
- EOF：常量，长度为1个字节，这个常量标志着RDB文件正文内容的结束，当读入这个值时，证明所有数据库的所有键值对已经全部载入。
- check_sum：是一个8字节长的无符号整数，保存一个校验和，是由前面四个部分的内容计算出来的，用来检查RDB文件是否出错。

#### 内部结构

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210509204828.png" alt="image-20210509203359097" style="zoom:80%;" />

##### database

- SELECTDB：一个字节，当读入程序遇到这个值时，他知道下来将是一个数据库号码
- db_number：保存着数据库号码，根据号码的大小可以是1、2、5个字节，当读入db_number部分后，服务器会调用SELECT命令，切换数据库
- key_value_pairs：保存了数据库中所有的键值对数据

##### key_value_pairs

key_value_pairs部分保存了一个或者以上数量的键值对，如多键值对带有过期时间的话，那么键值对的过期时间也会被保存在内

TYPE记录了value的类型，长度为一个字节，值可以是以下中的一种

- REDIS_RDB_TYPE_STRING		
- REDIS_RDB_TYPE_LIST
- REDIS_RDB_TYPE_SET
- REDIS_RDB_TYPE_ZSET
- REDIS_RDB_TYPE_HASH
- REDIS_RDB_TYPE_LIST_ZIPLIST
- REDIS_RDB_TYPE_SET_INTSET
- REDIS_RDB_TYPE_ZSET_ZIPLIST
- REDIS_RDB_TYPE_HASH-ZIPLIST

对于不同的TYPE，底层的value编码的结构也不同，具体的结构参见《Redis设计与实现》

### 分析RDB文件

调用od命令，打印RDB文件，给定-c参数，可以以ASCII编码方式打印文件，-x可以以十六进制打印输入文件

```bash
$ od -c dump.rdb
$ od -cx dump.rdb
```

空数据库

![image-20210509205551343](https://gitee.com/zhang-songyao/blog-images/raw/master/20210509205552.png)

 插入

```bash
127.0.0.1:6379> set MSG HELLO
OK
127.0.0.1:6379> save
```

![image-20210509205740736](https://gitee.com/zhang-songyao/blog-images/raw/master/20210509205742.png)

### RDB优缺点

> RDB是Redis Database缩写快照
>
> RDB是Redis默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为dump.rdb。通过配置文件中的save参数来定义快照的周期。

#### 优点

- 只有一个文件dump.rdb，方便持久化
- 容灾性好，一个文件可以保存到安全的磁盘
- 性能最大化，fork子进程来完成写操作，让主进程继续处理命令，所以IO最大化，使用单独子进程来进行持久化，主进程不会进行任何IO操作，保证了Redis的高性能
- 相对数据集大时，比AOF启动效率更高

#### 缺点

- 数据安全性低，RDB是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候)
- fork一条进程的时候会占用一定的内存空间

## AOF持久化

> 配置appendonly yes开启AOF持久化

> 当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复

> AOF (Append Only File)，与RDB持久化通过保存数据库中的键值对象来记录数据库状态不同，AOF持久化是通过保存Redis服务器所执行的写命令来记录数据库状态。因为这个模式是只追加的方式，所以没有任何磁盘寻址的开销，所以很快，有点像Mysql中的**binlog**。

### AOF持久化的实现

AOF持久化功能的实现可以分为命令追加（append）、文件写入、文件同步（sync）三个步骤

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510183819.png" alt="image-20210510175945632" style="zoom:80%;" />

#### 命令追加

当AOF持久化功能处于打开状态时，服务器在执行完一个写命令之后，会以协议到格式将被执行的命令追加到服务器状态的aof_buf缓冲区的末尾

```c
struct redisServer {
    //...
    
    //AOF缓冲区
    sds aof_buf;
    
    //...
};
```

#### 文件写入与同步

Redis的服务器进程是一个时间循环（loop），这个循环中的文件事件负责接收客户端的命令请求，以及向客户端发送命令回复，而时间事件则负责执行像serverCon函数这样需要运行的函数

因为服务器在处理文件事件时可能会执行写命令，使得一些内容被追加到aof_buf缓冲区中，所以服务器每次结束一个事件循环之前，都会调用feedAppendOnlyFile函数，考虑是否要将aof_buf中的内容写入和保存到AOF文件中。

```python
def evenLoop():
	while True:
		
		#处理文件事件，接收命令请求以及发送命令回复
        #处理命令请求时可能会有新的内容被追加到aof_buf缓冲区中
        processFileEvents()
        
        #处理时间事件
        processTimeEvents()
        
        #考虑是否要将aof_buf中的内容写入和保存到AOF文件中
        feedAppendOnlyFile()
```

```c
void feedAppendOnlyFile(struct redisCommand *cmd, int dictid, robj **argv, int argc) {
 
    if (dictid != server.appendseldb) { //当前操作的数据库和之前的数据库不一致，则写一条改变数据库的命令
        char seldb[64];
 
        snprintf(seldb,sizeof(seldb),"%d",dictid);
        buf = sdscatprintf(buf,"*2\r\n$6\r\nSELECT\r\n$%lu\r\n%s\r\n",
            (unsigned long)strlen(seldb),seldb);
        server.appendseldb = dictid;
    }
 
     .....
 
    server.aofbuf = sdscatlen(server.aofbuf,buf,sdslen(buf)); //数据放入atobuf中
 
    if (server.bgrewritechildpid != -1)  //如果子进程在进行Aof log rewrite，则同时将数据放入缓冲区bgrewritebuf
        server.bgrewritebuf = sdscatlen(server.bgrewritebuf,buf,sdslen(buf));
 
    sdsfree(buf);
}
```

feedAppendOnlyFile函数的行为由appendsync选项来决定

|  appendsync  | feedAppendOnlyFile函数的行为                                 |
| :----------: | :----------------------------------------------------------- |
|    always    | 将缓存区aof_buf的所有内容写入并同步到AOF文件                 |
| **everysec** | 将缓存区aof_buf的所有内容写入到AOF文件，如果上次同步AOF文件的时间距离现在超过一秒钟，那么再次对AOF文件进行同步，并且这个操作是由一个线程专门负责的 |
|      no      | 将缓存区aof_buf的所有内容写入到AOF文件，何时进行同步，由操作系统决定 |

> 文件的写入和同步

为了提高文件的写入效率，在现代操作系统中，当用户调用write函数，将一些数据写入文件的时候，操作系统通常会将写入的数据暂时保存在一个内存缓冲区中，等待缓冲区被填满后，或者超过了指定的时限后，才真正的将缓冲区中的数据写入磁盘中

这种做法提高了效率，但也为写入数据带来了安全性问题，如果服务器停机，那么保存在内存缓冲区中的数据将会丢失

为此系统提供了fsync和fdatasync两个同步函数，他们可以强制将操作系统立即将缓冲区的数据写入硬盘中，以确保安全性

> AOF持久化效率和安全性

- always：效率是最慢的，但是安全性最高，如果出现故障停机，AOF持久化也只会丢失一个事件循环中所产生的命令数据
- **everysec**：从效率上讲，**everysec**足够快，并且就算出现故障停机，数据库也只是丢失1s的命令数据
- no：写入速度最快，但是这种模式会在系统缓存中积累一段时间的写入数据，所以该模式单次同步时长通常是三种模式最长的，从平坦操作来看，no和**everysec**模式效率相似，但是出现故障停机，no模式将丢失上次同步AOF文件后的所有命令数据

### AOF文件的载入和数据还原

Redis读取AOF文件的详细步骤：

1. 创建一个不带网络连接的伪客户端（fake client）：因为Redis的命令只能在客户端上下文中执行，而载入AOF文件时所需要的命令直接源于AOF文件，不需要网络，伪客户端和带网络连接的客户端执行的命令效果完全一样
2. 从AOF文件中分析并读取出写命令
3. 使用伪客户端执行写命令
4. 重复步骤2和步骤3

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510183820.png" alt="image-20210510183809541" style="zoom:80%;" />

### AOF重写

> 为什么需要重写

对于一个list键的状态，由于用户的多次写入导致AOF文件中出现了数据冗余

```bash
127.0.0.1:6379> RPOP list A B
(error) ERR wrong number of arguments for 'rpop' command
127.0.0.1:6379> RPUSH list A B
(integer) 2
127.0.0.1:6379> RPUSH list C
(integer) 3
127.0.0.1:6379> RPUSH list D E
(integer) 5
127.0.0.1:6379> LPOP list
"A"
127.0.0.1:6379> LPOP list
"B"
127.0.0.1:6379> RPUSH list F G
(integer) 5
```

为了记录这个list状态，AOF需要保存6条命令，所以AOF文件体积会迅速膨胀，为了解决这个问题，Redis提出了AOF文件重写功能（rewrite），Redis创建两个AOF文件，新旧文件保存的数据库状态相同，但是新文件中没有任何浪费空间的冗余命令。

### AOF重写实现

> 为了使用尽量少的命令来记录list键的状态，那么最简单高效的办法不是去读取和分析已有的AOF文件，而是直接从数据库中读取键list的值，然后使用一条命令来代替上文的6条命令

AOF重写代码实现

```c
//后台重写AOF文件
int rewriteAppendOnlyFileBackground(void) {
    if (server.aof_child_pid != -1 || server.rdb_child_pid != -1) return C_ERR;
    if (aofCreatePipes() != C_OK) return C_ERR;//创建父进程与子进程的管道
    openChildInfoPipe();
    start = ustime();
    if ((childpid = fork()) == 0) {
        char tmpfile[256];
        snprintf(tmpfile,256,"temp-rewriteaof-bg-%d.aof", (int) getpid());
        if (rewriteAppendOnlyFile(tmpfile) == C_OK) {
            ……
        } 
    } else {
        /* Parent */ ……
    }
    return C_OK; /* unreached */
}//重写AOF文件的程序
int rewriteAppendOnlyFile(char *filename) {
    snprintf(tmpfile,256,"temp-rewriteaof-%d.aof", (int) getpid());
    server.aof_child_diff = sdsempty();
    rioInitWithFile(&aof,fp);
    if (server.aof_rewrite_incremental_fsync)
        rioSetAutoSync(&aof,AOF_AUTOSYNC_BYTES);
    ……//进行重写操作
    if (rewriteAppendOnlyFileRio(&aof) == C_ERR) goto werr;
    if (fflush(fp) == EOF) goto werr;
    if (fsync(fileno(fp)) == -1) goto werr;
    //重写期间，从父进程的重写缓冲区获取部分写命令
    ……
    if (rename(tmpfile,filename) == -1) {
    }
    return C_OK;
}//重写操作
int rewriteAppendOnlyFileRio(rio *aof) {
    ……// 遍历所有的数据库
    for (j = 0; j < server.dbnum; j++) {
        char selectcmd[] = "*2\r\n$6\r\nSELECT\r\n";
        redisDb *db = server.db+j;
        dict *d = db->dict;
        if (dictSize(d) == 0) continue;
        di = dictGetSafeIterator(d);
        if (rioWrite(aof,selectcmd,sizeof(selectcmd)-1) == 0) goto werr;
        if (rioWriteBulkLongLong(aof,j) == 0) goto werr;
        //遍历dict
        while((de = dictNext(di)) != NULL) {
            ……//检查key-value是否过期，过期就不需要重写到AOF文件
            if (expiretime != -1 && expiretime < now) continue;
            // 根据value类型，进行对应的重写逻辑
            if (o->type == OBJ_STRING) {
                char cmd[]="*3\r\n$3\r\nSET\r\n";
                if (rioWrite(aof,cmd,sizeof(cmd)-1) == 0) goto werr;
                if (rioWriteBulkObject(aof,&key) == 0) goto werr;
                if (rioWriteBulkObject(aof,o) == 0) goto werr;
            } else if (o->type == OBJ_LIST) {
                if (rewriteListObject(aof,&key,o) == 0) goto werr;
            } else if (o->type == OBJ_SET) {
                if (rewriteSetObject(aof,&key,o) == 0) goto werr;
            } else if (o->type == OBJ_ZSET) {
                if (rewriteSortedSetObject(aof,&key,o) == 0) goto werr;
            } else if (o->type == OBJ_HASH) {
                if (rewriteHashObject(aof,&key,o) == 0) goto werr;
            } else if (o->type == OBJ_MODULE) {
                if (rewriteModuleObject(aof,&key,o) == 0) goto werr;
            }//写入key-value的过期时间
            if (expiretime != -1) {
                char cmd[]="*3\r\n$9\r\nPEXPIREAT\r\n";
                if (rioWrite(aof,cmd,sizeof(cmd)-1) == 0) goto werr;
                if (rioWriteBulkObject(aof,&key) == 0) goto werr;
                if (rioWriteBulkLongLong(aof,expiretime) == 0) goto werr;
            }
            ……
        }
        dictReleaseIterator(di);
        di = NULL;
    }
    return C_OK;
}
```

大概思路：遍历所有的数据库，忽略空数据库，遍历数据库中所有的键，忽略已经过期的键，重写命令，新生成的AOF文件只包含还原当前数据库状态所必须的命令，所以新的AOF文件不会浪费任何的硬盘空间。

> 实际中，为了避免客户端缓冲区溢出，重写程序在处理列表、哈希表、集合、有序列表这四种可能包含多个元素的键时，会先检查元素数量，如果超过REDIS_AOF_REWRITE_ITEMS_PER_CMD（64）常量值，就会使用多条命令来记录键值，每条命令最多处理的元素数量不能超过64

### AOF后台重写（BGREWRITEAOF）

AOF重写程序可以很好的创建一个新的AOF文件，但是这个函数会进行大量的写操作，那么在服务器重写AOF文件期间，服务器将无法处理客户端请求，为了解决这种问题，Redis将AOF重写程序放到子进程中执行

- 一方面，主进程可以继续处理命令
- 另一方面，子进程带有服务器进程的数据副本，使用子进程而不是线程，可以避免使用锁的情况下，保证数据安全性。

由于AOF重写在子进程中完成，那么在AOF后台重写的过程中，如果主进程有写入命令，那么就会导致最终数据不一致，为了解决这个问题，Redis服务器设置了一个AOF重写缓冲区，这个缓冲区在服务器创建子进程后开始使用，当Redis服务器执行一个写命令后，他会同时将写命令发送给AOF缓冲区和AOF重写缓冲区

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210510200335.png" alt="image-20210510200332350" style="zoom:80%;" />

- AOF缓冲区的内容会被定期写入和同步到AOF文件，对现有的AOF文件的处理工作会如常执行
- 从创建子进程开始，服务器执行的所有命令都会被记录到AOF重写缓冲区中，当子进程完成重写后，会给父进程发送信号，父进程会调用一个信号处理函数，执行以下工作
  - 将AOF重写缓冲区的所有内容写入到新AOF文件中，这是新AOF文件保存的数据库状态将和当前数据库状态一致
  - 对新的AOF文件进行覆盖（原子地进行）完成新旧AOF文件替换

| 时间 |                   服务器进程（父进程）                    |             子进程             |
| :--: | :-------------------------------------------------------: | :----------------------------: |
|  T1  |                         SET k1 v1                         |                                |
|  T2  |                         SET k1 v2                         |                                |
|  T3  |                         SET k1 v3                         |                                |
|  T4  |                 创建子线程执行AOF文件重写                 |        开始AOF文件重写         |
|  T5  |                       SET k2 10086                        |          执行重写操作          |
|  T6  |                       SET k3 12345                        |          执行重写操作          |
|  T7  |                       SET k4 222222                       | 完成重写操作，向父进程发送信号 |
|  T8  | 接受到子进程发来的信号，将新添加的键值对追加到AOF文件末尾 |                                |
|  T9  |               用新的AOF文件覆盖旧的AOF文件                |                                |

> 在整个AOF后台执行过程中，只有信号处理函数会阻塞当前父进程，其他时候，都不会阻塞父进程，将AOF重写对服务器造成的影响降到最低

### AOF优缺点

#### 优点

- 数据安全，AOF持久化可以配置appendfsync 属性，有 always，每进行一次 命令操作就记录到 aof 文件中一次。
- 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。
- AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令 进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）)

#### 缺点

- AOF文件比RDB文件大，且恢复速度慢
- 数据集大的时候，比RDB启动效率低

## 持久化方式的选择

- 一般来说， 如果想达到足以媲美PostgreSQL的数据安全性，你应该同时使用两种持久化功能。在这种情况下，当 Redis 重启的时候会优先载入AOF文件来恢复原始的数据，因为在**通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整**
- 如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失，那么你可以只使用RDB持久化
- 有很多用户都只使用AOF持久化，但并不推荐这种方式，因为定时生成RDB快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比AOF恢复的速度要快，除此之外，使用RDB还可以避免AOF程序的bug
- 如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式
  

# 独立功能实现

## 事务

> 什么是事务

事务是一个单独的隔离操作，事务中的所有命令都会序列化、按顺序的执行，事务在执行过程中，不会被其他客户端发来的命令请求打断。

事务是一个原子操作：事务中的命令要么全部执行，要么全部不执行。

服务器判断命令是该入队还是该立即执行的过程

### Redis事务的概念

Redis事务的本质是通过MULTI、EXEC、WATCH等一组命令的集合，事务支持一次执行多个命令，一个事务中所有命令都会被序列化，在事务执行过程中，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

> 总结来说，Redis事务就是`一次性`、`顺序性`、`排他性`的执行一个队列中的一系列命令

Redis的事务总是具有ACID中的**一致性**和**隔离性**，其他特性是不支持的，当服务器运行在AOF持久化模式下，并且appendfsync选项的值为always式，事务也具有持久性

Redis是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止，因此，**Redis的事务总是带有隔离性的**，所以没有隔离级别的概念。

Redis中，单条命令是原子执行的，但是事务**不保证原子性**，**不支持回滚**，事务中任意命令执行失败，其余命令仍会被执行

> 作者认为保证了原子性
>
> **其实 Redis 事务真正支持原子性的前提：开发者不要写有逻辑问题的代码！**

### Redis事务的三个阶段

1. 事务开始MUTLI
2. 命令入队
3. 事务执行EXEC

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set key1 v1
QUEUED
127.0.0.1:6379> set key2 v2
QUEUED
127.0.0.1:6379> get key1
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
3) "v1"
```

事务执行过程中，如果服务端收到了EXEC、DISCARD、WATCH、MULTI之外的请求，将会把请求放入队列中排队。

### Redis事务相关命令

#### Redis事务命令

Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的

> MULTI

MULTI命令用于开启一个事务，它总是返回OK。 MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。

> EXEC

EXEC命令执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 nil 。

> DICARD

DISCARD命令，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。

Redis会将一个事务中的所有命令序列化，然后按顺序执行。

- Redis不支持回滚，Redis在事务失败时不进行回滚，而是继续执行余下命令，所以Redis的内部可以保持简单并且快速

> 不支持回滚是因为这种复杂的功能和Redis追求简单高效的设计主旨不符，Redis事务的执行时错误通常都是编程错误产生的，这种错误通常只会在开发环境中。很少混会在实际的生产环境中出现。所以没必要回滚。

- 如果一个事务中的命令入队出现错误（可以理解为编译型异常），那么所有命令都会不执行

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set key1 v1
QUEUED
127.0.0.1:6379> set key2 v2
QUEUED
127.0.0.1:6379> setget key3 v3
(error) ERR unknown command `setget`, with args beginning with: `key3`, `v3`, 
127.0.0.1:6379> set key4 v4
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get key1
(nil)
```

- 如果一个事务中出现运行错误（可以理解为运行时异常），那么正确的命令会被执行

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 "v1"
QUEUED
127.0.0.1:6379> INCR k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
4) "v1"
```

#### Redis锁

> 悲观锁

认为什么时候都会出现问题，无论做什么都会加锁

> 乐观锁

- 认为什么时候都不会出现问题，所以不会上锁、可以更新数据的时候去判断一下，在此期间是否有人修改过这个数据
- 获取 version
- 更新的时候比较 version

> WATCH

WATCH命令是一个乐观锁，可以为Redis事务提供check-and-set（CAS）行为。可以监控一个或者多个键，一旦其中有一个键被修改（或删除），之后的事务都不会执行，监控一直持续到EXEC命令

> UNWATCH

UNWATCH命令可以取消watch对所有key的监控。

![image-20210506210002143](https://gitee.com/zhang-songyao/blog-images/raw/master/20210506210003.png)

| 时间 |             客户端A             |   客户端B    |
| :--: | :-----------------------------: | :----------: |
|  T1  |           WATCH money           |              |
|  T2  |              MULTI              |              |
|  T3  | DECRBY money 100、INCRBY out100 |              |
|  T4  |                                 | SET money 10 |
|  T5  |              EXEC               |              |

#### Redis事务原理

##### 命令入队

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508183811.png" style="zoom: 67%;" />

##### 事务队列（FIFO）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210506212512.png" alt="image-20210506212511837" style="zoom:80%;" />

##### 执行事务

当一个处于事务状态的客户端向服务器发送EXEC命令时，EXEC命令将立即被服务器执行，服务器会遍历这个客户端的事务队列，执行队列中保存的所有命令，最后将执行所得的结果全部返回给客户端。

##### WATCH的执行原理

每个Redis数据库都保存这一个watch_keys字典，这个字典的键是被某个WATCH命令监视的数据库键，而字典的值则是一个链表，链表中记录了所有监视响应数据库键的客户端

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210506214219.png" alt="image-20210506214217801" style="zoom:80%;" />

如图，客户端c1和c2正在监视“name”、客户端c3正在监视age。

如果当前客户端为“cur”执行如下语句，图中虚线由于执行下列语句而添加到字典中的

```bash
WATCH "name" "age"
```

> 监视机制的触发

所有对数据库进行修改的命令：SET、LPUSH、SADD、ZREM、DEL、FLUSHDB等、在执行之后都会调用touchWatch函数对watch_keys字典进行检查，查看是否客户端正在监视刚刚被命令修改多的数据库键，如果有的话，那么touchWatchKey函数会将监视器被修改键的客户端的REDIS_DIRTY_CAS标识打开，标识该客户端的事务安全性已经被破坏了。

当服务器收到一个客户端发来的EXEC命令时，会根据这个客户端是否打开了REDIS_DIRTY_CAS标识来决定是否执行事务

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210506215253.png" alt="image-20210506215251734" style="zoom:80%;" />

### 小结

- 事务提供了一种将多个命令打包，然后一次性、有序执行的机制
- 多个命令会被入队到事务队列中，然后按照先进先出的顺序执行
- 事务在执行过程中不会被中断，当事务队列中的所有命令都被执行完毕之后，事务才会结束
- 带有WATCH命令的事务会将客户端和被监视的键在数据库的watched_keys字典中进行关联，当键被修改时，程序会将所有监视被修改的客户端的REDIS_DIRTY_CAS标识打开
- 只有在客户端的REDIS_DIRTY_CAS标识未打开的时候，服务器才会执行客户端提交的事务
- Redis的事务总是具有ACID中的原子性、**一致性**和**隔离性**，其他特性是不支持的，当服务器运行在AOF持久化模式下，并且appendfsync选项的值为always式，事务也具有持久性。

## Redis发布与订阅

> Redis发布订阅（pub/sub）是一种消息通信模式

通过执行SUBSCRIBE命令，客户端可以定订阅一个或者多个频道，从而成为这些频道的订阅者（subscriber）：每当有其他客户端向被订阅的频道发送消息时，频道的所有订阅者都会收到这条消息。

通过执行PSUBSCRIBE命令，客户端可以订阅一个或者多个模式，从而成为这些模式的订阅者，每当有其他客户端想某个频道发送消息的时候，消息不仅会被发送给这个频道的订阅者，还会被发送者发给与这个频道相配的模式的订阅者

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511142519.png" alt="image-20210511142516817" style="zoom:80%;" />

![image-20210423090614975](https://gitee.com/zhang-songyao/blog-images/raw/master/20210510211233.png)

### 命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | PSUBSCRIBE pattern [[pattern …\]](https://www.runoob.com/redis/pub-sub-psubscribe.html) 订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument …\]]](https://www.runoob.com/redis/pub-sub-pubsub.html) 查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html) 将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern …\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html) 退订所有给定模式的频道。 |
| 5    | SUBSCRIBE channel [channel …\]](https://www.runoob.com/redis/pub-sub-subscribe.html) 订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel …\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html) 指退订给定的频道。 |

```bash
#消息订阅值
127.0.0.1:6379> SUBSCRIBE dunkcode 
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "dunkcode"
3) (integer) 1
1) "message"
2) "dunkcode"
3) "hello"
1) "message"
2) "dunkcode"
3) "redis"

#消息发布者
127.0.0.1:6379> PUBLISH dunkcode hello
(integer) 1
127.0.0.1:6379> PUBLISH dunkcode redis
(integer) 1
```

### 实现原理

Redis将所有频道的订阅关系都保存在服务器状态的pubsub_channels字典中，这个字典的键时某个被订阅的频道，而键的值是一个链表，保存了所有定义这个频道的客户端

#### 订阅频道

如果频道已经有了其他的订阅者，那么它在pubsub_channels字典中必然存在，程序只需要将客户端添加到订阅者链表的末端

如果频道没有订阅者，那么不存在与pubsub_channels字典中，那么程序会为频道创建一个键，并将这个键的值设置为空链表，然后添加订阅者

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511143409.png" alt="image-20210511143404698" style="zoom:80%;" />

#### 退订频道

UNSUBSCRIBE命令和SUBSCRIBE命令相反，当一个客户端退订某个频道的时候，服务器会找到pubsub_channels字典中的的响应频道，找到响应的客户端，删除，如果删除后链表成为一个空链表，那么程序会删除对应的键

#### 模式订阅与退订

与订阅频道类似，服务器将所有的模式与订阅的关系保存在服务器状态的pubsub_patterns属性中，pubsub_patterns属性是一个链表，链表中的每一个节点都保存着一个pubsubPattern结构

```c
typedef struct pubsubPattern {
    
    //订阅模式的客户端
    redisClient *client;
    
    //被订阅的模式
    robj *pattern;
} pubsubPattern;
```

每当客户端订阅某个模式时，服务器会对每个订阅的模式执行以下操作

- 新建一个pubsubPattern结构，将结构属性社会为订阅的模式，client属性社会为订阅模式的客户端
- 将pubsubPattern结构添加到pubsub_patterns链表末尾

每当客户端执行退订订阅模式时，服务器将遍历pubsub_patterns链表，并且删除那些pattern为退订模式的，并且客户端为退订客户端的pubsubPattern节点

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511144521.png" alt="image-20210511144517689" style="zoom:80%;" />

### 发送消息

当一个Redis客户端执行PUBLIST<channel><message>命令将消息发送给频道channel的时候，服务器需要执行以下命令

- 将message消息发送给channel频道的所有订阅者，即在pubsub_channels字典中遍历寻找到channel频道，遍历链表，将消息发送个名单上所有的客户端
- 如果一个或者多个模式pattern与channel频道匹配，那么将消息message发送个pattern模式的订阅者，当PUBLISH命令将消息发送给了所有订阅它的频道后，服务器开始在pubsub_patterns链表中查找是否有被订阅模式与该频道匹配，如果匹配，将message发送个对应的客户端（被订阅者）

### 查看订阅消息

PUBSUB命令是Redis2.8后新增的命令之一，客户端可以通过这个命令来查看频道或者模式的相关信息

#### PUBSUB CHANNELS

PUBSUB CHANNELS [pattern]

- 如果不选择pattern属性，那么服务器返回当前被订阅的所有频道（遍历pubsub_channels字典的所有键）
- 如果给定pattern属性，服务器返回当前被订阅的频道中那些与pattern模式相匹配的频道（遍历pubsub_patterns链表）

#### PUBSUB NUMSUB

接受任意多个频道作为输入参数，返回这些频道的订阅者数量（遍历字典中对应键值的链表的长度）

#### PUBSUB NUMPAT

返回服务器当前被订阅模式的数量（返回pubsub_patterns链表的长度）

# Redis集群

## Redis主从复制

> 为什么需要主从复制

一般来说，要将Redis用于工程项目中，只有一台Redis是完全不够的

- 从结构上，单个 Redis服务器会发生单点故障，井且一台服务器需要处理所有的请求负載，压力较大
- 从容量上，单个 Redis服务器内存容量有限就算一台 Redis服务器内存容量为256G，也不能将所有内存用作 Redis?存储内存一般来说，单台 Redist最大使用内存不应该超过20G

> 主从复制的作用

1. 数据元余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：当主节点出现可题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务(即写 Redis数据时应用连接主节点，读Redis数据时应用连接从节点)，分担服务器负载;尤具是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis服务器的并发量
4. **高可用**(集群)基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Reds高可用的基础

### 复制原理

Slave启动成功连接到 master 后会发送一个sync命令

Master接到命令,启动后台的存盘进程,同时收集所有接收到的用于修改数据集命令,在后台进程执行完毕之后, master将传送整个数据文件到 slave,井完成一次完全同步

全量复制：而 slave服务在接收到数据库文件数据后,将其存盘并加载到内存中

增量复制：Master继续将新的所有收集到的修改命令依次传给 slave,完成同步

但是只要是重新连接 master,一次完全同步(全量复制)将被自动执行

#### 旧版复制功能

Redis的复制功能分为同步（sync）和命令传播（command propagate）两个操作：

- 同步操作主要讲从服务器的数据库状态更新至主服务器当前所处的数据库状态
- 命令传播操作则作用于在主服务器上的数据库状态被修改，导致主服务器和从服务器的数据库状态不同，让主从服务器的数据库状态重新回到一致的状态

> 同步

- 从服务器发送SYNC命令
- 主服务器收到SYNC命令，主服务器执行BSAVE命令，在后台生成一个rdb文件，并使用一个缓冲区记录从现在开始执行的所有写命令
- 当主服务器执行完BSAVE命令后，将RDB文件发送给从服务器从服务器接受并载入这个RDB文件，将自己的数据库状态更新至和主服务器执行BSAVE时一致的状态
- 主服务器将记录在缓冲区中的写命令，发送给从服务器，从服务器执行主服务器发送的命令，一致自己和主服务器的状态

<img src="C:/Users/dell/AppData/Roaming/Typora/typora-user-images/image-20210511151625644.png" alt="image-20210511151625644" style="zoom:80%;" />

> 命令传播

当同步操作执行完毕之后，主从服务器两者的数据库状态将达到一致，但是一致不是一成不变的，每当主服务器执行写命令的时候，主服务器会将自己执行的写命令，发送给从服务器，从服务执行命令后，主从服务器回到一致状态

#### 旧版复制功能的缺陷

从服务器对主服务器的复制只要分为以下两种情况

- 初次复制：从服务器之前没有复制过任何主服务器，或者从服务器当前要复制的主服务器和上次复制的主服务器不同
- 断线后复制：处于命令传播阶段的主从服务器因为网络原因而中断了复制，但从服务器通过自动重连重新连接上了主服务器，并且继续复制主服务器

> 对于旧版的复制功能，虽然也能让服务器重新回到一直状态，但是效率非常低，因为尽管是断线重新连接后，从服务器依然需要重新复制主服务器的所有状态，尽管之前大部分数据都是一致的

SYNC命令是非常耗费资源的命令

- 主服务器执行BSAVE命令生成RDB文件，将耗费主服务器大量的CPU、内存和磁盘IO资源
- 主服务器发送RDB文件将占用大量的网络资源（带宽和流量），以及时间
- 从服务器接受RDB文件后，载入RDB文件，从服务器将被阻塞无法执行任何命令

#### 新版复制功能实现

使用PSYNC代替SYNC命令来执行复制时的同步操作

PSYNC具有完成同步操作和部分同步操作两种模式

- 完成同步操作：和SYNC执行同步的方式相似
- 部分同步操作：用于处理断线重连后重复制的情况，如果条件允许，从服务器只需要执行断开连接期间的主服务器所有的写命令即可

##### 部分同步的实现

部分同步功能由以下三部分构成

- 主服务器的复制偏移量（replication offset）：主服务器和从服务器会分别维护一个复制偏移量，复制偏移量等于当前偏移量+传播数据长度字节数，如果主从服务器的偏移量相等，说明主从服务器处于一致状态
- 主服务器的复制积压缓冲区（replication backlog）：是由主服务器维护的一个固定长度的FIFO队列，默认大小为1MB，保存着一部分最近传播的写命令，并且复制积压缓冲区会为队列中的每个字节记录相应的复制偏移量，方便同步。大小一般是当前服务器断线后重新连上主服务器所需的平均时间second与主服务器平均每秒产生的写命令数据量write_size_per_second乘积的两倍
- 服务器运行ID（run ID）：从服务器初次复制会将自己的ID发送给主服务器，主服务器和从服务器都会保存这个ID，当重新连接时，通过ID，主从服务器可以彼此认出对方

#### PSYNC命令的实现

PYNC命令调用方法有两种

- 如果从服务器以前没有执行过任何主服务器，或者之前执行SALVEOF  no one命令，那么从服务器在开始一次新的复制时向主服务器发送PYNC ？-1命令，主动请求主服务器进行完整重同步
- 相反的，如果从服务器已经复制过了某个主服务器，那么从服务器在开始一次新的复制时将主服务器发送的PYNC<runid><offset>命令，其中runid是上一次复制的主服务器的运行ID，而offset则是从服务器当前的复制偏移量，接受到这个命令的主服务器会通过这两个参数来判断应该对服务器执行那种同步操作
- 如果主服务器返回+FULLRESYNC<runid><offset>回复，那么表示主从服务器将进行完整的同步操作，runid将保存下来，下次发送PSYNC命令时使用，offset则是主服务器当前的复制偏移量，从服务器将这个值作为自己的初始化偏移量
- 如果主服务器返回+CONTINUE，表示执行部分同步操作
- 如果主服务器返回-ERR回复，那么表示服务器版本低于2.8，无法识别命令

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511172920.png" alt="image-20210511172914297" style="zoom:80%;" />

#### 复制的实现

1. 设置主服务器的地址和端口号
2. 建立套接字连接，从服务器根据命令所设置的IP地址和端口，创建连向主服务器的套接字连接
3. 从服务器作为主服务器的客户端后的第一件事是向主服务器发送PING命令，检查套接字连接是否正常
4. 身份验证，在主服务器设置了requirepass选项和从服务器设置masterauth的同时，需要验证主服务器密码
5. 发送端口信息，从服务器发送REPLCONF listening-port 12345，设置监听端口
6. 同步从服务器发送PSYNC命令
7. 命令传播

### 单台Linux服务器搭建Redis主从

> 修改配置文件内容

- 端口号

```bash
port 6380
```

- pid

```bash
pidfile /var/run/redis_6380.pid
```

- log文件名称

```bash
logfile "6380"
```

- dump.rdb文件名称

```bash
dbfilename dump6380.rdb
```

- 通过SLAVEOF命令设置主机信息

通过INFO replication查看当前服务器状态

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511195909.png" alt="image-20210511195905936" style="zoom:80%;" />

通过SLAVEOF host port配置从机

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511195946.png" alt="image-20210511195935353" style="zoom:80%;" />

查看和从机状态从机状态

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511200302.png" alt="image-20210511200258538" style="zoom:80%;" />

### 主机宕机

如果主机宕机了，从机想要变成主机 我们可以通过命令 `slaveof no noe` 让自己变成主机！其他节点就可以手动连接，这个时候主机连接了 还想要 这个主机当老大 那么我们需要手动配置

## 哨兵模式

> 配置哨兵配置文件 sentinel.conf

```bash
sentinel monitor 被监控的名字自定义 ip地址 端口号 1
sentinel monitor myredis 127.0.0.1 6310 1
```

1表示，如果主机宕机了 slave 投票看让谁接替成为主机，票数最多的，就会成为主机

使用`redis-sentinel` 服务来启动 哨兵

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511210318.png" alt="image-20210511210315362" style="zoom:80%;" />

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210511210631.png" alt="image-20210511210628481" style="zoom:80%;" />

#### 哨兵集群的配置文件

```bash
# Example sentinel.conf
# port <sentinel-port> 哨兵的默认端口 默认是 26379 
port 8001
# 守护进程模式
daemonize yes
# 指明日志文件名
logfile "./sentinel1.log"
# 工作路径，sentinel一般指定/tmp比较简单
dir ./
# 哨兵监控这个master，在至少quorum个哨兵实例都认为master down后把master标记为odown
# （objective down客观down；相对应的存在sdown，subjective down，主观down）状态。
# slaves是自动发现，所以你没必要明确指定slaves。
sentinel monitor MyMaster 127.0.0.1 7001 1
# master或slave多长时间（默认30秒）不能使用后标记为s_down状态。
sentinel down-after-milliseconds MyMaster 1500
# 若sentinel在该配置值内未能完成failover操作（即故障时master/slave自动切换），则认为本次failover失败。
sentinel failover-timeout TestMaster 10000
# 设置master和slaves验证密码
sentinel auth-pass TestMaster testmaster123
sentinel config-epoch TestMaster 15
#除了当前哨兵, 还有哪些在监控这个master的哨兵
sentinel known-sentinel TestMaster 127.0.0.1 8002 0aca3a57038e2907c8a07be2b3c0d15171e44da5
sentinel known-sentinel TestMaster 127.0.0.1 8003 ac1ef015411583d4b9f3d81cee830060b2f29862
```

# 缓存

## 缓存穿透

> 概念

缓存穿透概念，用户查询一个数据，发现Redis内存数据库中没有，也就是缓存没有命中，于是向持久层数据库查询，发现也没有，于是本次查询失败，当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库，这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透

> 解决方案

- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截

- 布隆过滤器是一种数据结构，bloomfilter就类似于一个hash set，用于快速判某个元素是否存在于集合中，其典型的应用场景就是快速判断一个key是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于hash算法和容器大小，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202208220013842.jpeg)

- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击

![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202208220013446.jpeg)

如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

> 对于空间的利用到达了一种极致，那就是Bitmap和布隆过滤器(Bloom Filter)。

Bitmap： 典型的就是哈希表
缺点是，Bitmap对于每个元素只能记录1bit信息，如果还想完成额外的功能，恐怕只能靠牺牲更多的空间、时间来完成了。

布隆过滤器（推荐）

就是引入了k(k>1)个相互独立的哈希函数，保证在给定空间、误判率下，完成元素判重的过程。

优点是空间小效率和查询时间都远远高于一般的算法，缺点是有一定的误识别率和删除困难，Bloom-Fliter算法的核心思想就是利用多个不同的hash函数来解决冲突

Hash存在一个冲突（碰撞）的问题，用同一个Hash得到的两个URL的值有可能相同。为了减少冲突，我们可以多引入几个Hash，如果通过其中的一个Hash值我们得出某元素不在集合中，那么该元素肯定不在集合中。只有在所有的Hash函数告诉我们该元素在集合中时，才能确定该元素存在于集合中。这便是Bloom-Filter的基本思想。
Bloom-Filter一般用于在大数据量的集合中判定某元素是否存在。

## 缓存击穿

> 概念

缓存击穿是指缓存中没有数据，但数据库中有数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没有读到数据，又同时去数据库取数据，引起数据库压力瞬间增大，造成压力过大。

> 解决方案

- 设置热点数据永不过期
- 接口限流与熔断、降级
- 加互斥锁

重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些 服务 不可用时候，进行熔断，失败快速返回机制。

## 缓存雪崩

> 概念

缓存雪崩是指，缓存层出现了错误，不能正常工作了，于是所有的请求都会到达存储层，存储层的调用量会暴增，造成存储层挂掉

![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202208220013321.jpeg)

> 解决方案

1. 缓存数据的过期时间设置为随机值，防止同一时间大量数据过期现象发生
2. 一般并发量不是特别多的时候，使用最多的解决方案就是加锁排队
3. 给每一缓存数据增加相应的缓存标记，记录缓存是否失效，如果缓存标记失效，则更新数据缓存

## 三者的区别

> 缓存击穿是缓存中没有数据，去数据库读取数据，用户并发地查询同一条数据，导致数据库压力过大。缓存穿透是指缓存中没有数据，数据库中也没有数据，用户直接访问数据库，导致数据库压力过大。缓存雪崩是指不同数据都过期了，很多数据都查不到从而查数据库

## 缓存预热

**缓存预热**就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

> 解决方案

1. 直接写个缓存刷新页面，上线时手工操作一下；
2. 数据量不大，可以在项目启动的时候自动进行加载；
3. 定时刷新缓存；

## 缓存降级

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；

警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；

错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；

严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。

# [Redis面试题](https://thinkwon.blog.csdn.net/article/details/103522351)

## RDB和AOF的优缺点

### RDB

#### 优点：

他会生成多个数据文件，每个行数据文件分别代表了某一时刻Redis里面的数据，这种方式很适合做冷备，完整的数据运维设置定时任务，定时同步到远端的服务器，比如阿里的云服务器，这样一旦线上挂了，想恢复多少分钟之前的数据，就去远端拷贝一份之前的数据就好了

> RDB对Redis的性能影响非常小，因为在同步数据的时候他只是去fork了一个子进程去做持久化，而且他在数据恢复的时候比AOF文件快

但是**两种机制全部开启的时候，Redis在重启的时候会默认使用AOF去重新构建数据，因为AOF的数据是比RDB更完整的**

#### 缺点：

RDB都是快照文件，都是默认五分钟甚至更久的时间才会生成一次，这意味着你这次同步到下次同步这中间的五分钟数据很可能全部丢掉，AOF最多丢一秒的数据，数据完整性上高下立判

还有就是**RDB**在生成数据快照的时候，如果文件很大，客户端可能会暂停几毫秒甚至几秒

### AOF

#### 优点：

**AOF**是一秒一次去通过一个后台的线程`fsync`操作，那最多丢这一秒的数据

AOF在对日志文件进行操作的时候是以append-only的方式去写的，只是追加方式去写数据，自然减少了很多磁盘寻址的开销，写入性能惊人，文件不容易破坏

#### 缺点

一样的数据，AOF文件比RDB文件大

AOF开启后，Redis支持写的QPS会比RDB支持的写要低，他不是每秒都要去异步刷新一次日志fsync，当然即使这样性能还是很高，我记得**ElasticSearch**也是这样的，异步刷新缓存区的数据去持久化

### 如何选择

小孩子才做选择，**我全都要**，你单独用**RDB**你会丢失很多数据，你单独用**AOF**，你数据恢复没**RDB**来的快，真出什么时候第一时间用**RDB**恢复，然后**AOF**做数据补全，真香！冷备热备一起上，才是互联网时代一个高健壮性系统的王道。

## Redis还有其他保证集群高可用的方式么？

> 哨兵+主从并**不能保证数据不丢失**，但是可以保证集群的**高可用**。

哨兵必须用三个实例去保证自己的健壮性的

如果只有两个哨兵，那么master宕机了，s1和s2；两个哨兵只要有一个认为你宕机了就切换，并且会选举一个哨兵去执行故障转移，但是这个时候需要大多数哨兵都是运行的，如果master和s1都宕机了，那么哨兵就只剩下了s2，一台哨兵，就没有哨兵去允许故障转移了。

### 主观下线

Sentinel会以每秒1次的频率向所有与他穿件命令连接的实例（包括主服务器、从服务器、其他Sentinel）发送PING命令，恢复来判断实例是否在线，如果一个实例连续在down-after-milliseconds毫秒内无效回复，那么该Sentinel会修改该实例的SRI_S_DOWN标识，一次来表示该实例进入主观下线状态

### 客观下线

当Sentinel确定一个主服务器主观下线后，为了确定这个主服务器是否真的下线，他会向同样监视这一主服务器的其他Sentinel哨兵进行询问，当Sentinel从其他Sentinel收到足够数量的下线判断后，Sentinel就会人文该主服务器客观下线

### 选举领头Sentinel

当一个主服务被判断客观下线后，监视这个下线服务器的各个Sentinel会进行协商，选举一个领头的Sentinel，并由领头的Sentinel对下线的主服务器进行故障转移操作

> 选举规则：先到先得，当一个Sentinel的票数超过半数以上的Sentinel的投票后，那么该Sentinel将成为领头Sentinel，所有的Sentinel都会将这个Sentinel设置为自己的领头Sentinel

### 故障转移

选举出领头的Sentinel后，领头的Sentinel将会对已下线的主服务器执行故障转移操作

- 在已下线的主服务器下的所有从服务器中，挑选一个从服务器，将其转换为主服务器
- 让已下线的主服务器的所有从服务器改为复制的新主服务器
- 将已下线的主服务器设置为新主服务器的从服务器，当这个旧的主服务器重新上线时，就会成为新主服务器的从服务器

> 挑选规则：挑选出目前位置最活跃的从服务器（与领头Sentinel成功通信的时间最近），根据服务器优先级进行选择，优先级相同，选取偏移量最大的（保存数据最新），偏移量相同，选取ID最小的从服务器

## Redis过期策略

> **定期删除+惰性删除**两种

定期好理解，默认100s就随机抽一些设置了过期时间的key，去检查是否过期，过期了就删了。

> 为啥不扫描全部设置了过期时间的key呢？

假如Redis里面所有的key都有过期时间，都扫描一遍？那太恐怖了，而且我们线上基本上也都是会设置一定的过期时间的。全扫描跟你去查数据库不带where条件不走索引全表扫描一样,100s一次，消耗太大

> 如果一直没随机到很多key，里面不就存在大量的无效key了？

**惰性删除**，见名知意，惰性嘛，我不主动删，我懒，我等你来查询了我看看你过期没，过期就删了还不给你返回，没过期该怎么样就怎么样。

> 最后就是如果的如果，定期没删，我也没查询，那可咋整？

**内存淘汰机制**！

- noeviction:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）


- allkeys-lru: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。
- volatile-lru: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。
- allkeys-random: 回收随机的键使得新添加的数据有空间存放。
- volatile-random: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。
- volatile-ttl: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

如果没有键满足回收的前提条件的话，策略**volatile-lru**, **volatile-random**以及**volatile-ttl**就和noeviction 差不多了。

## [LRU](https://blog.csdn.net/qq_35190492/article/details/103041932?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162055059816780366543961%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162055059816780366543961&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-3-103041932.nonecase&utm_term=Redis)(最近最少使用算法)

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/13 23:45
 * @description：LRU算法	
 */
public class TestLRU {
    public static void main(String[] args) {}
}

class LRUCache {
    int capacity;
    Map<Integer, Node> map;
    DoubleList list;

    public LRUCache (int cap) {
        this.capacity = cap;
        this.map = new HashMap<>();
        this.list = new DoubleList();
    }

    /**
     * 添加缓存
     * 如果map中包含当前key，则修改当前的val值，并将节点添加到链表头部
     * 否则
     *  如果双端链表的长度小于cap，直接添加到链表头部
     *  否则，移除链表最后一个节点，添加到链表头部，删除map中的映射
     *  最后，添加到map中
     * @param key
     * @param val
     */
    public void put(int key, int val) {
        Node cur = new Node(key, val);
        if(map.containsKey(key)) {
            list.remove(map.get(key));
            list.addFirst(cur);
        } else {
            if(list.size == capacity) {
                Node node = list.removeLast();
                map.remove(node.key);
            }
            list.addFirst(cur);
        }
        map.put(key, cur);
    }

    /**
     * 如果map中不包含key，返回-1
     * 否则，获取当前节点，删除当前节点（map、list），将节点添加到list头部
     * @param key
     * @return
     */
    public int get(int key) {
        if(!map.containsKey(key)) return -1;
        Node target = map.get(key);
        list.remove(target);
        list.addFirst(target);
        return target.val;
    }
}

class Node {
    int key, val;
    Node next, pre;

    public Node(int key, int val) {
        this.key = key;
        this.val = val;
    }

    @Override
    public String toString() {
        return "Node{" +
                "key=" + key +
                ", val=" + val +
                '}';
    }
}

class DoubleList {
    Node head;
    Node tail;
    int size;

    public DoubleList() {
        head = new Node(-1, -1);
        tail = new Node(-1, -1);
        head.next = tail;
        tail.pre = head;
        size = 0;
    }

    public void addFirst(Node node) {
        node.next = head.next;
        head.next = node;
        node.next.pre = node;
        node.pre = head;
        size++;
    }


    public Node remove(Node node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
        size--;
        return node;
    }

    public Node removeLast() {
        Node target = tail.pre;
        tail.pre.pre.next = tail;
        tail.pre = tail.pre.pre;
        size--;
        return target;
    }

    public void list() {
        Node temp = head.next;
        while(temp != tail) {
            System.out.println(temp);
            temp = temp.next;
        }
    }

}
```

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210512195124.jpeg)

找一个数据结构实现下Java版本的LRU还是比较容易的，知道啥原理就好了

使用LinkedHashMap版的LRU算法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/14 11:23
 * @description：使用LinkedHashMap构建LRU页面置换算法
 */
public class LRULinkedHashMap {

    public static void main(String[] args) {
        LRUCache<Integer, Integer> lruCache = new LRUCache<>(3);
        lruCache.put(1,2);
        lruCache.put(2,2);
        lruCache.put(3,2);
        lruCache.get(2);
        //必须这样测试
        //否则会报ConcurrentModificationException异常
        Iterator<Map.Entry<Integer, Integer>> it = lruCache.entrySet().iterator();
        while(it.hasNext()) {
            Map.Entry entry = it.next();
            System.out.println(entry.getKey() + "   " + entry.getValue());
        }
        lruCache.put(4,3);
        lruCache.get(2);
        lruCache.put(4,5);
        it = lruCache.entrySet().iterator();
        while(it.hasNext()) {
            Map.Entry entry = it.next();
            System.out.println(entry.getKey() + "   " + entry.getValue());
        }
    }

    static class LRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int CACHE_SIZE;

        public LRUCache(int cacheSize) {
            //true表示让hashmap按照访问顺序，最新访问的放在头部，最老访问的放在尾部
            super((int)Math.ceil(cacheSize / 0.75f) + 1, 0.75f, true);
            this.CACHE_SIZE = cacheSize;
        }

        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            //当map中数量大于指定缓存数量，就删除最老的数据
            return size() > CACHE_SIZE;
        }
    }

}

```

## LFU(最不经常使用算法)

- set(key, value)：将记录(key, value)插入该结构
- get(key)：返回key对应的value值

但是缓存结构中最多放K条记录，如果新的第K+1条记录要加入，就需要根据策略删掉一条记录，然后才能把新记录加入。这个策略为：在缓存结构的K条记录中，哪一个key从进入缓存结构的时刻开始，被调用set或者get的次数最少，就删掉这个key的记录；

如果调用次数最少的key有多个，上次调用发生最早的key被删除

```java
/**
 * @author ：zsy
 * @date ：Created 2021/7/25 13:31
 * @description：https://www.nowcoder.com/practice/93aacb4a887b46d897b00823f30bfea1?tpId=190&&tqId=35215&rp=1&ru=/activity/oj&qru=/ta/job-code-high-rd/question-ranking
 */
public class NC94LFU缓存结构设计 {
    @Test
    public void test() {
        Solution solution = new Solution();
    }

    public class Solution {
        /**
         * lfu design
         * @param operators int整型二维数组 ops
         * @param k int整型 the k
         * @return int整型一维数组
         */
        public int[] LFU (int[][] operators, int k) {
            // write code here
            LFUCache lfuCache = new LFUCache(k);
            ArrayList<Integer> res = new ArrayList<>();
            for (int[] operator : operators) {
                if (operator[0] == 1) {
                    lfuCache.put(operator[1], operator[2]);
                } else {
                    res.add(lfuCache.get(operator[1]));
                }
            }
            int[] ans = new int[res.size()];
            for (int i = 0; i < res.size(); i++) {
                ans[i] = res.get(i);
            }
            return ans;
        }

        class LFUCache {
            //容量
            int capacity;
            //map
            Map<Integer, Node> map;
            //双向链表
            DoubleList list;

            public LFUCache(int capacity) {
                this.capacity = capacity;
                map = new HashMap<>();
                list = new DoubleList();
            }

            /**
             *  1、已经存在key
             *      -覆盖val，frequency++
             *      -删除链表中的节点，重新加入
             *  2、不存在
             *      -达到了最大容量
             *          删除链表第一个元素，执行插入操作
             *      -未达到
             *          直接插入节点
             * @param key
             * @param val
             */
            public void put(int key, int val) {
                if (map.containsKey(key)) {
                    Node node = map.get(key);
                    list.remove(node);
                    node.val = val;
                    node.frequency++;
                    map.put(key, node);
                    list.add(node);
                } else {
                    Node cur = new Node(key, val, 1);
                    if (list.size >= capacity) {
                        Node node = list.removeFirst();
                        map.remove(node.key);
                    }
                    list.add(cur);
                    map.put(key, cur);
                }
            }

            /**
             * 获取val，frequency++，链表重新赋值
             * @param key
             * @return
             */
            public int get(int key) {
                if (!map.containsKey(key)) return -1;
                Node node = map.get(key);
                list.remove(node);
                node.frequency++;
                list.add(node);
                return node.val;
            }
        }

        //双向链表
        class DoubleList {
            //头尾指针
            Node head, tail;
            //节点数量
            int size;

            public DoubleList() {
                this.head = new Node(-1, -1, -1);
                this.tail = new Node(-1, -1, Integer.MAX_VALUE);
                head.next = tail;
                tail.pre = head;
                size = 0;
            }

            /**
             * 删除节点
             * @param node
             * @return
             */
            private Node remove(Node node) {
                node.pre.next = node.next;
                node.next.pre = node.pre;
                size--;
                return node;
            }

            /**
             * 添加节点，并且维护链表顺序
             *  -频次升序
             *  -相同频次，按操作时间早->晚排序（方便删除）
             * @param node
             */
            private void add(Node node) {
                Node temp = head;
                while (node.frequency >= temp.next.frequency) {
                    temp = temp.next;
                }
                node.next = temp.next;
                temp.next.pre = node;
                temp.next = node;
                node.pre = temp;
                size++;
            }

            /**
             * 删除第一个节点，即调用次数最少，且调用发生最早
             * @return
             */
            private Node removeFirst() {
                Node tar = head.next;
                head.next = tar.next;
                tar.next.pre = head;
                size--;
                return tar;
            }

            /**
             * 展示列表，测试使用
             */
            private void list() {
                Node temp = head.next;
                while (temp != null && temp != tail) {
                    System.out.println(temp);
                    temp = temp.next;
                }
            }
        }

        //Node存放K-V值和当前节点操作的频次
        class Node {
            private int key, val, frequency;
            Node pre, next;

            public Node(int key, int val, int frequency) {
                this.key = key;
                this.val = val;
                this.frequency = frequency;
            }
        }
    }


}

```



## Redis过期键的删除策略

> [删除策略](https://www.cnblogs.com/lukexwang/p/4694094.html)

对于过期键一般的三种删除策略

- 定时删除：在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键的删除操作
- 惰性删除：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，那就返回该键
- 定期删除：每过一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于删除多少过期的键，以及检查多少个数据库，则由算法决定

三种策略优缺点

- 定时删除策略
  - 对内存是最友好的：通过使用定时器，定时删除策略可以保证过期键会尽可能快地被删除，并释放过期键所占的内存；
  - 但另一方面，定时删除策略的缺点是，它对CPU是最不友好的：在过期键比较多的情况下，将CPU时间用在删除和当前任务无关的过期键上，无疑对服务器的响应时间和吞吐量造成影响
- 惰性删除策略
  - 惰性删除策略对CPU时间来说是友好的：程序只会在取出键时才对键进行过期检查是，这可以保证删除过期键的操作只会在非做不可的情况下进行
  - 惰性删除的缺点：它对内存是最不友好的：如果一个键已经过期，而这个键又仍然保留在数据库中，那么只要这个过期键不被删除，他所占用的内存就不会被释放掉
- 定时删除占用太多的CPU时间，影响服务器的响应时间和吞吐量；惰性删除浪费太多的内存，有内存泄漏的危险。定期删除策略是前两种策略的一种整合折中
  - 定期删除策略每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响
  - 通过定期删除过期键，定期删除策略有效地减少了因为过期键带来的内存浪费
  - 定期删除策略的难点是确定删除操作的执行时长和频率

