# Redis



## 部署运维

```shell
docker run -p 6379:6379 
-v /will/myredis/data:/data 
-v /will/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf  
-d redis redis-server /usr/local/etc/redis/redis.conf
--appendonly yes
```

```shell
// 连接进入Redis
docker exec -it redis_name /bin/bash
./redis-cli

# 查询所有的键
keys * 

# 获取键对应的value的类型
type key 

# 删除指定的key value
del key
```





## 遗留问题

- Redis其实也是多线程？？，只不过是用单线程？？来接收请求，在客户端看起来？？是串行接收执行，所以效果上就是单线程？？。但是`IO多路复用`？？才是Redis能**高并发**的底层保证。？？













## 面试题





> Redis 属于单线程还是多线程？

- Redis 是单线程的，主要是指 Redis 的**网络 I/O** 线程，以及键值的 SET 和 GET 等**读写操作**都是由一个线程来完成

- 但Redis 的持久化、集群同步等操作，则是由另外的线程来执行的
- 可以在无锁的情况下完成所有操作，不存在死锁和线程切换带来的性能和时间上的开销
- 但同时单线程也不能发挥多核 CPU 的性能



[文件事件处理器](#文件事件处理器) 



[需要掌握的Redis原理](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=592#/detail/pc?id=6063) 

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisOfHigh.png" style="zoom:50%;" />



> 单线程的redis为什么这么快

- 纯内存操作，采用了高效的数据结构，比如哈希表和跳表
- 单线程操作，避免了频繁的上下文切换
- 采用了非阻塞I/O多路复用机制，因为基于非阻塞的 I/O 模型，就意味着 I/O 的读写流程不再阻塞
- Redis 4.0 版本之后，Redis 添加了多线程的支持，但这时的多线程主要体现在大数据的异步删除功能上，例如 unlink key、flushdb async、flushall async 等
- Redis 6.0 版本之后，为了更好地提高 Redis 的性能，新增了多线程 I/O 的读写并发能力，这是因为随着网络硬件的性能提升，Redis 的性能瓶颈有时会出现在网络 I/O 的处理上，所以为了提高网络请求处理的并行度，Redis 6.0 对于网络请求采用多线程来处理。
- 但是对于读写命令，Redis 仍然使用单线程来处理。





> Reids 为什么先执行命令，再把数据写入日志呢？（写后日志，正好和Mysql的“写前日志”相反）

- 因为 ，Redis 在写入日志之前，不对命令进行**语法检查**；

- 所以，只记录执行成功的命令，避免了出现记录错误命令的情况； 

- 并且，在命令执行完之后再记录，**不会阻塞**当前的写操作



这样做也会带来风险：

- 数据可能会丢失 
- 可能阻塞其他操作： 虽然 AOF 是写后日志，避免阻塞当前命令的执行，但因为 AOF 日志也是在主线程中执行，所以当 Redis 把日志文件写入磁盘的时候，还是**会阻塞后续**的操作无法执行。







> RDB 做快照时会阻塞线程吗？

- 因为 Redis 的单线程模型决定了它所有操作都要尽量避免阻塞主线程，所以对于 RDB 快照也不例外，这关系到是否会降低 Redis 的性能。
- 为了解决这个问题，Redis 提供了两个命令来生成 RDB 快照文件，分别是 **save** 和 **bgsave**。save 命令在主线程中执行，会导致阻塞。而 bgsave 命令则会*创建一个子进程* ，用于写入 RDB 文件的操作，避免了对主线程的阻塞，这也是 Redis RDB 的默认配置。





> RDB 做快照的时候数据能修改吗？



**这个问题非常重要**，考察候选人对 RDB 的技术掌握得够不够深。你可以思考一下，如果在执行快照的过程中，数据如果能被修改或者不能被修改都会带来什么影响？

1. 如果此时可以执行写操作：意味着 Redis 还能正常处理写操作，就可能出现正在执行快照的数据是已经被修改了的情况；
2. 如果此时不可以执行写操作：意味着 Redis 的所有写操作都得等到快照执行完成之后才能执行，那么就又出现了阻塞主线程的问题。



那Redis 是如何解决这个问题的呢？ 它利用了 bgsave 的子进程，具体操作如下：

- 如果主线程执行*读操作*，则主线程和 bgsave 子进程互相不影响；
- 如果主线程执行*写操作*，则被修改的数据会复制一份副本，然后 bgsave 子进程会把该副本数据写入 RDB 文件，在这个过程中，主线程仍然可以直接修改原来的数据。

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RDB_ProcessOfRedis.png" style="zoom:67%;" />







> redis的常用数据结构

- string/hash/list/set/sortedsort



> zset的数据结构

- set 关联一个double类型的分数 ==>有序集合



> zset的中插入一个元素的时间复杂度

- 向有序集合添加 score和element 的时间复杂度为 O(logN)   ||  删除或获取element的时间复杂度都为O(1)



> redis如何保证高可用？

- redis持久化机制【dump.rdb/appendonly.aof】、主从复制、哨兵机制



> 什么是redis的哨兵模式？

- 通过后台监测主机是否宕机，如果宕机了按投票选举，票数多的从机成为新的主机，整个过程是自动的



> 缓存穿透、缓存击穿、缓存雪崩区别和解决方案

- [详细内容](https://blog.csdn.net/kongtiao5/article/details/82771694)	

  









## redis的应用场景

 **通过降低数据的安全性，减少对事务的支持，减少对复杂查询的支持，来获取性能上的提升** 

•	缓存（数据查询、短连接、新闻内容、商品内容等等）
•	聊天室的在线好友列表
•	任务队列（秒杀、抢购、12306等等）
•	应用排行榜  (zsort)
•	网站访问统计
•	数据过期处理（可以精确到毫秒)
•	分布式集群架构中的session分离





## redis数据结构

```java
// 类似
Map<string, value>;    

// key都是字符串
// value有5种不同的数据结构，结构不同，但存储的都是字符串？？
```



**value的数据结构：**      

| 类型     |    类似    |                   备注                    |
| :------- | :--------: | :---------------------------------------: |
| 字符串   |   string   | 可以存放数字，并加减；但需进行转换成int64 |
| 哈希     |    hash    |              map<key, value>              |
| 列表     | linkedlist |          双端节点, 支持重复元素           |
| 集合     |  hashSet   |              不允许重复元素               |
| 有序集合 | sortedset  |       不允许重复元素，且元素有顺序        |



Redis的数据结构：
		![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/DataStructureOfRedis.png)











## 文件事件处理器



 [Redis 到底是单线程还是多线程？](https://www.cnblogs.com/javastack/p/12848446.html) 

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/SingleThreadOfRedis.png" style="zoom:57%;" />

- 文件事件处理器：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。
-  因为文件事件分派器队列的**消费**是单线程的，所以Redis才叫单线程模型。













## 穿透/击穿/雪崩



[小故事](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247486528&idx=1&sn=3f7b09eb21969fdb16f5b0805ff69fed&scene=21#wechat_redirect)	



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/BUfferDesign.png" style="zoom:67%;" />



- **缓存击穿**：缓存中没有，但数据库中有的数据



- **缓存穿透**：缓存和数据库中都没有的数据，并发查同一条数据
  - 解决：
  - 最常见的则是采用*布隆过滤器*，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力
  - 还有一个更为**简单粗暴的方法**，如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟



- **缓存雪崩**：缓存中数据大批量到过期时间
  - 解决：
  - 大多数系统设计者考虑用加锁（ 最多的解决方案）或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上
  - 还有一个简单方案就是<u>*分散开*</u> 缓存失效时间







## 缓存过期/内存淘汰



[过期淘汰策略和内存淘汰策略有什么区别](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1779)	



[个中原由](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247486528&idx=1&sn=3f7b09eb21969fdb16f5b0805ff69fed&scene=21#wechat_redirect)	



### 过期淘汰策略



**新增** Redis 缓存时可以**设置缓存的过期时间**，该时间保证了数据在规定的时间内失效，可以借助这个特性来实现很多功能

- 超时时间有了，该在什么时候去干这个清理的活呢？ ？
- 不能一口气把所有过期的都给删除掉，redis里面存了大量的数据，要全面扫一遍的话那不知道要花多久时间，会严重影响接待新的客户请求！！ 



时间紧任务重，只好随机选择一部分来清理，能缓解内存压力就行！！！



**缓存过期策略：**

- 惰性删除： 原来逃脱了redis 随机选择算法的键值，*<u>一旦遇到查询请求，被发现已经超期了，立即删除</u>*； 是被动式触发的，不查询就不会发生；导致缓存中出现`大量的过期key无法被删除`
- 定时删除：为每个设置了过期时间的key，都需要*<u>创建一个定时器，到过期时间就会立即清除</u>*；但需要消耗大量的CPU资源去清除过期的数据，`可能会影响缓存服务的性能` 
- 定期过期：添加一个`即将过期的缓存字典`，*<u>每隔一定的时间，会扫描一定数量的key</u>*，并清除其中已过期的key







### 内存淘汰策略



```shell
# 查看当前 Redis 的内存淘汰策略
config get maxmemory-policy

# 修改内存淘汰策略的方式，无需重启 Redis 服务器，次重启 Redis 服务器之后设置的内存淘汰策略就会丢失
config set maxmemory-policy noeviction
# 也可以通过配置文件来修改，redis.conf 对应的配置项是“maxmemory-policy noeviction”
```



- noeviction：不淘汰任何数据，当内存不足时，执行缓存新增操作会报错，它是 Redis 默认内存淘汰策略
- allkeys-lru：淘汰整个键值中最久未使用的键值
- allkeys-random：随机淘汰任意键值
- volatile-lru：淘汰所有设置了过期时间的键值中最久未使用的键值
- volatile-random：随机淘汰设置了过期时间的任意键值
- volatile-ttl：优先淘汰更早过期的键值
- volatile-lfu，淘汰所有设置了过期时间的键值中最少使用的键值
- allkeys-lfu，淘汰整个键值中最少使用的键值

> 小贴士：从以上内存淘汰策略中可以看出，allkeys-xxx 表示从所有的键值中淘汰数据，而 volatile-xxx 表示从设置了过期键的键值中淘汰数据。



**内存淘汰算法：** 

LRU（ Least Recently Used，最近最少使用）淘汰算法：是一种常用的页面置换算法，也就是说`最久没有使用`的缓存将会被淘汰

LFU（Least Frequently Used，最不常用的）淘汰算法：最不常用的算法是根据`总访问次数`来淘汰数据的，它的核心思想是***<u>“如果数据过去被访问多次，那么将来被访问的频率也更高”</u>***。

LFU 相对来说比 LRU 更“智能”，因为它解决了使用频率很低的缓存，只是最近被访问了一次就不会被删除的问题。

LRU？？可以继承 LinkedHashMap来实现







## 高可用

[高可用](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1782)	





### 持久化



[RDB / AOF](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247486926&idx=1&sn=58e99f81d6d6ee31c9a5c8f93122e108&scene=21#wechat_redirect)	



#### RDB

Redis DataBase（**快照**方式）：按照一定的 <u>*时间周期策略*</u> 把内存的数据以快照的形式保存到硬盘的二进制文件。



实现：

1. 单独fork()创建一个子进程
2. 将当前父进程的全部数据复制到子进程的内存中，然后由子进程写入到临时文件中(需上一个持久化的过程结束了)
3. 再用这个**临时文件** <u>*覆盖*</u> 上次的快照文件
4. 然后子进程退出，内存释放



在一定的间隔时间中，**检测key的变化情况**，然后持久化数据



1. 编辑redis.windwos.conf文件
	```shell
	# after 900 sec (15 min) if at least 1 key changed
	save 900 1
	
	# after 300 sec (5 min) if at least 10 keys changed
	save 300 10
	
	# after 60 sec if at least 10000 keys changed
	save 60 10000
	```
	
	
	
2. 重新启动redis服务器，并指定配置文件名称
	...\redis\windows-64\redis-2.8.9>redis-server.exe redis.windows.conf	
	
	

- RDB 默认的保存文件为 dump.rdb
- 优点是以二进制存储的，因此占用的空间更小、数据存储更紧凑
- 与 AOF 相比，RDB 具备更快的重启恢复能力
- RDB对Redis对外提供的读写服务，影响非常小，可以让Redis保持高性能，因为Redis主进程只需要fork一个子进程，让子进程执行磁盘IO操作来进行RDB持久化即可





#### AOF



Append Only File（文件追加方式）：将每一个**收到的写命令**都通过Write函数追加到文件最后，类似于MySQL的binlog

- 当Redis重启，会通过重新执行文件中，保存的写命令来在内存中重建整个数据内容



```shell
appendonly no（关闭aof） --> appendonly yes （开启aof）

appendfsync always ： 每一次操作都进行持久化

appendfsync everysec ： 每隔一秒进行一次持久化

appendfsync no	 ： 不进行持久化
```

- AOF 默认的保存文件为 appendonly.aof
- 优点是存储频率更高，因此丢失数据的风险就越低，并且 AOF 并不是以二进制存储，存储信息更易懂
- 缺点是占用空间大，重启之后的数据恢复速度比较慢 （大到一定程度，redis会进行 rewrite操作）





##### 优点

1. AOF可以更好的保护数据不丢失，一般每隔1S，通过一个后台线程执行一次**fsync**操作，保证 os cache中的数据写入磁盘，最多丢失1秒数据
2. AOF日志文件以append-only模式写入，所以没有任何**磁盘寻址**开销，写入性能高，而且文件不容易破损；即使文件破损，也有工具可以恢复
3. AOF文件过大时候，会出现**rewrite**操作，但并不影响客户端的读写；因为在rewrite log的时候，会对其中的数据进行压缩，创建出一份需要恢复数据的最小日志出来；在创建新日志文件的时候，老的日志文件还是照常写入，当新的merge后的日志文件ready的时候，再交换新老日志文件即可；基于redis当前内存中最新的数据进行**指令的重新构建** 
4. AOF日志文件的命令通过，可读的方式进行记录，这个特性非常适合做灾难新的误删除的紧急恢复





##### 缺点

1. 占用磁盘空间更多
2. 做数据恢复比较慢，冷备不太方便





##### Rewrite过程

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/AOF Rewrite.png" alt="AOF Rewrite" style="zoom:67%;" />





##### AOF文件修复



如果redis **在append数据到AOF文件时**，*机器宕机了*，可能会导致AOF文件破损(只保存到完整命令的一部分)

```shell
# 修复方法
./redis-check-aof [--fix] <file.aof> 
```











#### 混合模式

Redis 4.0 推出了**混合**持久化



混合持久化的功能指的是 Redis 可以使用 RDB + AOF 两种格式来进行数据持久化 



AOF 和 RDB 混合的数据持久化机制： 把数据以 RDB 的方式写入文件，再将后续的操作命令以 AOF 的格式存入文件，既保证了 Redis 重启速度，又降低数据丢失风险。

```shell
127.0.0.1:6379> config get aof-use-rdb-preamble
1) "aof-use-rdb-preamble"
2) "yes"

# 打开
config set aof-use-rdb-preamble yes
```



- Redis 混合持久化的存储模式是，开始的数据以 RDB 的格式进行存储，因此只会占用少量的空间
- 之后的命令会以 AOF 的方式进行数据追加，这样就可以减低数据丢失的风险，同时可以提高数据恢复的速度
- 当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复



![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/AOF和RDB同时工作.png)







Redis企业级备份：

![	](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisBackup.png)





### 主从复制



> Redis主从架构 -> 读写分离架构 -> 可支持水平扩展的读高并发架构

- 写多读少的业务，得考虑MQ，做异步分离





#### 复制原理



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ReplicationTheroyOfRedis.png" style="zoom:80%;" />





[Redis主从复制/哨兵模式的原理解析](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247487533&idx=1&sn=49b600ef7eac342dad1f5a8048361099&scene=21#wechat_redirect)  

全量复制使用`snyc`命令来实现，其流程是：

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisSnyc.png" style="zoom:67%;" />

旧版本全量复制功能，其最大的问题是从服务器断线重连时，**即便**在从服务器上已经有一部分数据了，也需要进行全量复制，这样做的`效率很低`，于是新版本的redis在这部分做了改进？？**序列号**

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisNumber.png" style="zoom:67%;" />

新版本redis使用==psync==命令来代替sync命令，该命令既可以实现完整全同步也可以实现部分同步。

1.  **复制偏移量**：
    - 执行复制的双方，主从服务器，分别会维护一个复制偏移量：
    - 主服务器每次向从服务器同步了N字节数据之后，将修改自己的复制偏移量+N。
    - 从服务器每次从主服务器同步了N字节数据之后，将修改自己的复制偏移量+N。
2. **复制积压缓冲区**：
    - 主服务器内部维护了一个固定长度的先进先出队列做为复制积压缓冲区，其默认大小为1MB。
    - 在主服务器进行命令传播时，不仅会将写命令同步到从服务器，还会将写命令写入复制积压缓冲区。





### 哨兵模式



 Redis 哨兵模块存储在 Redis 的 src/redis-sentinel 目录下 

```shell
# 启动哨兵功能
./src/redis-sentinel sentinel.conf
```





三个监控任务：



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisSentinel;.png" style="zoom:67%;" />



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisBetweenSentinel.png" style="zoom:50%;" />

> 每隔2秒，每个哨兵节点将会向redis数据节点的__sentinel__:hello频道同步自身得到的主节点信息以及当前哨兵节点的信息
>
> 由于其他哨兵节点也订阅了这个频道，因此实际上这个操作可以**交换**哨兵节点之间关于主节点以及哨兵节点的信息。





<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/RedisSentinelTheory.png" style="zoom:57%;" />

> 哨兵的工作原理是每个哨兵会以每秒钟 1 次的频率，向已知的主服务器和从服务器，发送一个 PING 命令。
>
> 如果最后一次有效回复 PING 命令的时间，超过了配置的最大下线时间（Down-After-Milliseconds）时，默认是 30s，那么这个实例会被哨兵标记为主观下线。 
>
> 如果一个主服务器被标记为主观下线，那么正在监视这个主服务器的所有哨兵节点，要以每秒 1 次的频率确认主服务器是否进入了主观下线的状态。
>
> 如果有足够数量（quorum 配置值）的哨兵证实该主服务器为主观下线，那么这个主服务器被标记为客观下线。
>
> 此时所有的哨兵会按照规则（协商）自动选出新的主节点服务器，并自动完成主服务器的自动切换功能，而整个过程都是无须人工干预的。 







发现新的哨兵节点：

如果有新的哨兵节点加入，此时保存下来这个新哨兵节点的信息，后续与该哨兵节点建立连接。 
交换主节点的状态信息，作为后续客观判断主节点下线的依据。







### Redis Cluster





Redis Cluster 是一种分布式去中心化的运行模式，是在 Redis 3.0 版本中推出的 Redis 集群方案，它将数据分布在不同的服务器上，以此来降低系统对单主节点的依赖，从而提高 Redis 服务的读写性能。



<img src="Redis.assets/image-20210424232749103.png" alt="image-20210424232749103" style="zoom:67%;" />



Redis Cluster 方案采用哈希槽（Hash Slot），来处理数据和实例之间的映射关系。在 Redis Cluster 方案中，一个切片集群共有 16384 个哈希槽，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中，具体执行过程分为两大步。

- 根据键值对的 key，按照 CRC16 算法计算一个 16 bit 的值。

- 再用 16bit 值对 16384 取模，得到 0~16383 范围内的模数，每个模数代表一个相应编号的哈希槽







剩下的一个问题就是，这些哈希槽怎么被映射到具体的 Redis 实例上的呢？有两种方案。

- 平均分配： 在使用 cluster create 命令创建 Redis 集群时，Redis 会自动把所有哈希槽平均分布到集群实例上。比如集群中有 9 个实例，则每个实例上槽的个数为 16384/9 个。

- 手动分配： 可以使用 cluster meet 命令手动建立实例间的连接，组成集群，再使用 cluster addslots 命令，指定每个实例上的哈希槽个数，



数据、哈希槽，以及实例三者的映射分布关系：









## 分布式锁

[来龙去脉](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1780)	



分布式锁是*控制*  **分布式系统之间** <u>同步访问</u> **共享资源** 的一种方式。

是为了解决分布式系统中，不同的系统或是同一个系统的不同主机**共享同一个资源**的问题，它通常会采用**互斥**来保证程序的一致性，这就是分布式锁的用途以及执行原理。



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/DistributeOfRedis.png" style="zoom:57%;" />



分布式锁的常见实现方式有四种：

- 基于 MySQL 的悲观锁来实现分布式锁，这种方式使用的最少，因为这种实现方式的性能不好，且容易造成死锁
- 基于 Memcached 实现分布式锁，可使用 add 方法来实现，如果添加成功了则表示分布式锁创建成功
- 基于 Redis 实现分布式锁，这也是要介绍的重点，使用 `setnx` 方法来实现，set if not exists（不存在则创建）
- 基于 ZooKeeper 实现分布式锁，利用 ZooKeeper **顺序临时节点**来实现



**分布式锁得以实现的基本前提：** 

之所以可以使用以上四种方式来实现分布式锁，是因为以上四种方式都属于程序调用的“`外部系统`”，而分布式的程序是需要**共享**“外部系统”的



## Redis 实现分布式锁



一、

```shell
127.0.0.1:6379> setnx lock true
(integer) 1 #创建锁成功

#逻辑业务处理...

127.0.0.1:6379> del lock
(integer) 1 #释放锁
```

以上代码有一个问题，就是**没有设置锁的超时时间** 

如果出现异常情况，会导致锁未被释放，而其他线程又在排队等待此锁就会导致程序不可用



二、

```shell
127.0.0.1:6379> setnx lock true
(integer) 1 #创建锁成功

127.0.0.1:6379> expire lock 30 #设置锁的(过期)超时时间为 30s
(integer) 1 

#逻辑业务处理...

127.0.0.1:6379> del lock
(integer) 1 #释放锁
```

 这样执行仍然会有问题，因为 setnx lock true 和 expire lock 30 命令是**非原子的**  

 如果在 setnx 命令执行完之后，发生了异常情况，那么就会导致 expire 命令不会执行，因此依然没有解决死锁的问题



三、

 Redis 2.6.12 中我们可以使用一条 set 命令来执行键值存储，并且可以判断键是否存在以及设置超时时间了，如下代码所示： 

```shell
127.0.0.1:6379> set lock true ex 30 nx
OK #创建锁成功
```

-  ex 是用来设置超时时间的，expire
-  nx 是 not exists 的意思，用来判断键是否存在 
-  “OK”则表示创建锁成功，否则表示此锁有人在使用 



四、

 锁误删：

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/20210220182119.png" style="zoom:67%;" /> 



锁被误删的解决方案是在使用 set 命令创建锁时，给 value 值设置一个归属标识。

例如，在 value 中插入一个 UUID，每次在删除之前先要判断 UUID 是不是属于当前的线程，如果属于再删除，这样就避免了锁被误删的问题。 



五、

 注意：在锁的归属判断和删除的过程中，*不能* 先判断锁再删除锁，如下代码所示： 

```java
if(uuid.equals(uuid)){ // 判断是否是自己的锁
	del(luck); // 删除锁
}
```



 把判断和删除放到一个**原子单元**中去执行，因此需要借助 Lua 脚本来执行 

```java
/**
 * 释放分布式锁
 * @param jedis Redis客户端
 * @param lockKey 锁的 key
 * @param flagId 锁归属标识
 * @return 是否释放成功
 */
public static boolean unLock(Jedis jedis, String lockKey, String flagId) {

    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";

    Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(flagId));

    if ("1L".equals(result)) { // 判断执行结果
        return true;
    }
    return false;
}
```



六、

锁超时可以通过两种方案来解决：

- 把执行耗时的方法从锁中剔除，减少锁中代码的执行时间，保证锁在超时之前，代码一定可以执行完；
- 把锁的超时时间设置的长一些，正常情况下我们在使用完锁之后，会调用删除的方法手动删除锁，因此可以把超时时间设置的稍微长一些。







## Redis 实现消息队列

[Redis如何实现消息队列？？实现的方式有几种？？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1781)	



| 实现方式                        | 持久化 | 重复消费 | 主题订阅 | 消费消息确认 | 延迟消息队列 |
| ------------------------------- | :----: | :------: | :------: | :----------: | :----------: |
| List (2.0 之前)                 |   Y    |    N     |    N     |      N       |      N       |
| Zset (2.0 之前)                 |   Y    |    N     |    N     |      N       |      Y       |
| Publisher/Subscriber (2.0 之后) |   N    |    N     |    Y     |      N       |              |
| Stream (5.0 之后)               |   Y    |    Y     |    Y     |      Y       |              |



- List和Zset 可以借助 Redis 本身的持久化功能，AOF 或者是 RDB 或混合持久化的方式，用于把数据保存至磁盘













##  Jedis



#### 步骤

1. 下载jedis的jar包
2. 使用

```java
            //1. 获取连接
    		Jedis jedis = new Jedis("localhost",6379);
   			//2. 操作
   			jedis.set("username","zhangsan");
    		//3. 关闭连接
    		jedis.close();
```



#### string

```java
            //1. 获取连接
	        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
	        //2. 操作
	        //存储
	        jedis.set("username","zhangsan");
	        //获取
	        String username = jedis.get("username");
	        System.out.println(username);
	
	        //可以使用setex()方法存储可以 "指定过期时间的 key value"
	        jedis.setex("activecode",20,"hehe");//将activecode：hehe键值对存入redis，并且20秒后自动删除该键值对
	
	        //3. 关闭连接
	        jedis.close();
```

#### hash 

 map格式  

			hset
			hget
			hgetAll

```java
            //1. 获取连接
	        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
	        //2. 操作
	        // 存储hash
	        jedis.hset("user","name","lisi");
	        jedis.hset("user","age","23");
	        jedis.hset("user","gender","female");
	
	        // 获取hash
	        String name = jedis.hget("user", "name");
	        System.out.println(name);
	
	
	        // 获取hash的所有map中的数据
	        // Map结构中存储的全是 string
	        Map<String, String> user = jedis.hgetAll("user");
	
	        // 得到所有keyset（field）
	        Set<String> keySet = user.keySet();
	        for (String key : keySet) {
	            //获取value
	            String value = user.get(key);
	            System.out.println(key + ":" + value);
	        }
	
	        //3. 关闭连接
	        jedis.close();
```

#### list 

linkedlist格式。支持重复元素

```java
			 //1. 获取连接
	        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
	        //2. 操作
	        // list 存储
	        jedis.lpush("mylist","a","b","c");//从左边存
	        jedis.rpush("mylist","a","b","c");//从右边存
	
	        // list 范围获取
	        List<String> mylist = jedis.lrange("mylist", 0, -1);
	        System.out.println(mylist);
	        
	        // list 弹出
	        String element1 = jedis.lpop("mylist");//c
	        System.out.println(element1);
	
	        String element2 = jedis.rpop("mylist");//c
	        System.out.println(element2);
	
	        // list 范围获取
	        List<String> mylist2 = jedis.lrange("mylist", 0, -1);
	        System.out.println(mylist2);
	
	        //3. 关闭连接
	        jedis.close();
```

#### set  

 不允许重复元素

```java
			//1. 获取连接
	        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
	        //2. 操作
	
	
	        // set 存储
	        jedis.sadd("myset","java","php","c++");
	
	        // set 获取
	        Set<String> myset = jedis.smembers("myset");
	        System.out.println(myset);
	
	        //3. 关闭连接
	        jedis.close();
```
#### sortedset

不允许重复元素，且元素有顺序, 分数可以覆盖（排行榜，热搜）

```java 
			//1. 获取连接
	        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
	        //2. 操作
	        // sortedset 存储
	        jedis.zadd("mysortedset",3,"亚瑟");
	        jedis.zadd("mysortedset",30,"后裔");
	        jedis.zadd("mysortedset",55,"孙悟空");
	
	        // sortedset 获取
	        Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);
	
	        System.out.println(mysortedset);
	
	
	        //3. 关闭连接
	        jedis.close();
```


​	
####  JedisPool
1. 创建JedisPool连接池对象
2. 调用方法 getResource()方法获取Jedis连接

```java
				//0.创建一个配置对象
		        JedisPoolConfig config = new JedisPoolConfig();
		        config.setMaxTotal(50);
		        config.setMaxIdle(10);
		
		        //1.创建Jedis连接池对象
		        JedisPool jedisPool = new JedisPool(config,"localhost",6379);
		
		        //2.获取连接
		        Jedis jedis = jedisPool.getResource();
		        //3. 使用
		        jedis.set("hehe","heihei");
		
		
		        //4. 关闭 归还到连接池中
		        jedis.close();
```



#### 连接池工具类

```java
public class JedisPoolUtils {

			    private static JedisPool jedisPool;
			
			    static{
			        //读取配置文件
			        InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
			        //创建Properties对象
			        Properties pro = new Properties();
			        //关联文件
			        try {
			            pro.load(is);
			        } catch (IOException e) {
			            e.printStackTrace();
			        }
			        //获取数据，设置到JedisPoolConfig中
			        JedisPoolConfig config = new JedisPoolConfig();
			        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
			        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));
			
			        //初始化JedisPool
			        jedisPool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));
			
			
			
			    }
			
			
			    /**
			     * 获取连接方法
			     */
			    public static Jedis getJedis(){
			        return jedisPool.getResource();
			    }
			}
```



## redis.conf
```cnof
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf
 
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
################################## INCLUDES ###################################
 
# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
 
################################## NETWORK #####################################
 
# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 lookback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#bind 127.0.0.1
 
# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes
 
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
 
# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511
 
# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700
 
# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0
 
# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300
 
################################# GENERAL #####################################
 
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
#daemonize no
 
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no
 
# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid
 
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
 
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""
 
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no
 
# Specify the syslog identity.
# syslog-ident redis
 
# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
 
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
 
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""
 
save 120 1
save 300 10
save 60 10000
 
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
 
# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes
 
# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes
 
# The filename where to dump the DB
dbfilename dump.rdb
 
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./
 
################################# REPLICATION #################################
 
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of slaves.
# 2) Redis slaves are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition slaves automatically try to reconnect to masters
#    and resynchronize with them.
#
# slaveof <masterip> <masterport>
 
# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
# masterauth <master-password>
 
# When a slave loses its connection with the master, or when the replication
# is still in progress, the slave can act in two different ways:
#
# 1) if slave-serve-stale-data is set to 'yes' (the default) the slave will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#
# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO and SLAVEOF.
#
slave-serve-stale-data yes
 
# You can configure a slave instance to accept writes or not. Writing against
# a slave instance may be useful to store some ephemeral data (because data
# written on a slave will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
#
# Since Redis 2.6 by default slaves are read-only.
#
# Note: read only slaves are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only slave exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only slaves using 'rename-command' to shadow all the
# administrative / dangerous commands.
slave-read-only yes
 
# Replication SYNC strategy: disk or socket.
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
# -------------------------------------------------------
#
# New slaves and reconnecting slaves that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the slaves.
# The transmission can happen in two different ways:
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the slaves incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to slave sockets, without touching the disk at all.
#
# With disk-backed replication, while the RDB file is generated, more slaves
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new slaves arriving will be queued and a new transfer
# will start when the current one terminates.
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple slaves
# will arrive and the transfer can be parallelized.
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
repl-diskless-sync no
 
# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the slaves.
#
# This is important since once the transfer starts, it is not possible to serve
# new slaves arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more slaves arrive.
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
repl-diskless-sync-delay 5
 
# Slaves send PINGs to server in a predefined interval. It's possible to change
# this interval with the repl_ping_slave_period option. The default value is 10
# seconds.
#
# repl-ping-slave-period 10
 
# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of slave.
# 2) Master timeout from the point of view of slaves (data, pings).
# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-slave-period otherwise a timeout will be detected
# every time there is low traffic between the master and the slave.
#
# repl-timeout 60
 
# Disable TCP_NODELAY on the slave socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to slaves. But this can add a delay for
# the data to appear on the slave side, up to 40 milliseconds with
# Linux kernels using a default configuration.
#
# If you select "no" the delay for data to appear on the slave side will
# be reduced but more bandwidth will be used for replication.
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and slaves are many hops away, turning this to "yes" may
# be a good idea.
repl-disable-tcp-nodelay no
 
# Set the replication backlog size. The backlog is a buffer that accumulates
# slave data when slaves are disconnected for some time, so that when a slave
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the slave missed while
# disconnected.
#
# The bigger the replication backlog, the longer the time the slave can be
# disconnected and later be able to perform a partial resynchronization.
#
# The backlog is only allocated once there is at least a slave connected.
#
# repl-backlog-size 1mb
 
# After a master has no longer connected slaves for some time, the backlog
# will be freed. The following option configures the amount of seconds that
# need to elapse, starting from the time the last slave disconnected, for
# the backlog buffer to be freed.
#
# A value of 0 means to never release the backlog.
#
# repl-backlog-ttl 3600
 
# The slave priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a slave to promote into a
# master if the master is no longer working correctly.
#
# A slave with a low priority number is considered better for promotion, so
# for instance if there are three slaves with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the slave as not able to perform the
# role of master, so a slave with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
slave-priority 100
 
# It is possible for a master to stop accepting writes if there are less than
# N slaves connected, having a lag less or equal than M seconds.
#
# The N slaves need to be in "online" state.
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the slave, that is usually sent every second.
#
# This option does not GUARANTEE that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough slaves
# are available, to the specified number of seconds.
#
# For example to require at least 3 slaves with a lag <= 10 seconds use:
#
# min-slaves-to-write 3
# min-slaves-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.
 
# A Redis master is able to list the address and port of the attached
# slaves in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover slave instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a masteer.
#
# The listed IP and address normally reported by a slave is obtained
# in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the slave to connect with the master.
#
#   Port: The port is communicated by the slave during the replication
#   handshake, and is normally the port that the slave is using to
#   list for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the slave may be actually reachable via different IP and port
# pairs. The following two options can be used by a slave in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
#
# slave-announce-ip 5.5.5.5
# slave-announce-port 1234
 
################################## SECURITY ###################################
 
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
 
# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.
 
################################### LIMITS ####################################
 
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000
 
# Don't use more memory than the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU cache, or to set
# a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
 
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# allkeys-lru -> remove any key according to the LRU algorithm
# volatile-random -> remove a random key with an expire set
# allkeys-random -> remove a random key, any key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# noeviction -> don't expire at all, just return an error on write operations
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction
 
# LRU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs a bit more CPU. 3 is very fast but not very accurate.
#
# maxmemory-samples 5
 
############################## APPEND ONLY MODE ###############################
 
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.
 
appendonly no
 
# The name of the append only file (default: "appendonly.aof")
 
appendfilename "appendonly.aof"
 
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".
 
# appendfsync always
appendfsync everysec
# appendfsync no
 
# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.
 
no-appendfsync-on-rewrite no
 
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.
 
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
 
# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes
 
################################ LUA SCRIPTING  ###############################
 
# Max execution time of a Lua script in milliseconds.
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
lua-time-limit 5000
 
################################ REDIS CLUSTER  ###############################
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes
 
# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
#
# cluster-config-file nodes-6379.conf
 
# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
#
# cluster-node-timeout 15000
 
# A slave of a failing master will avoid to start a failover if its data
# looks too old.
#
# There is no simple way for a slave to actually have a exact measure of
# its "data age", so the following two checks are performed:
#
# 1) If there are multiple slaves able to failover, they exchange messages
#    in order to try to give an advantage to the slave with the best
#    replication offset (more data from the master processed).
#    Slaves will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#
# 2) Every single slave computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the slave will not try to failover
#    at all.
#
# The point "2" can be tuned by user. Specifically a slave will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
#
#   (node-timeout * slave-validity-factor) + repl-ping-slave-period
#
# So for example if node-timeout is 30 seconds, and the slave-validity-factor
# is 10, and assuming a default repl-ping-slave-period of 10 seconds, the
# slave will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
#
# A large slave-validity-factor may allow slaves with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a slave at all.
#
# For maximum availability, it is possible to set the slave-validity-factor
# to a value of 0, which means, that slaves will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
#
# cluster-slave-validity-factor 10
 
# Cluster slaves are able to migrate to orphaned masters, that are masters
# that are left without working slaves. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working slaves.
#
# Slaves migrate to orphaned masters only if there are still at least a
# given number of other working slaves for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a slave
# will migrate only if there is at least 1 other working slave for its master
# and so forth. It usually reflects the number of slaves you want for every
# master in your cluster.
#
# Default is 1 (slaves migrate only if their masters remain with at least
# one slave). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
#
# cluster-migration-barrier 1
 
# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#
# cluster-require-full-coverage yes
 
# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.
 
################################## SLOW LOG ###################################
 
# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.
 
# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000
 
# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128
 
################################ LATENCY MONITOR ##############################
 
# The Redis latency monitoring subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports.
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
latency-monitor-threshold 0
 
############################# EVENT NOTIFICATION ##############################
 
# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events ""
 
############################### ADVANCED CONFIG ###############################
 
# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
 
# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2
 
# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0
 
# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512
 
# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
 
# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
hll-sparse-max-bytes 3000
 
# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
activerehashing yes
 
# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
#
# The limit can be set differently for the three different classes of clients:
#
# normal -> normal clients including MONITOR clients
# slave  -> slave clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
#
# Instead there is a default limit for pubsub and slave clients, since
# subscribers and slaves receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
 
# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10
 
# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
aof-rewrite-incremental-fsync yes
```