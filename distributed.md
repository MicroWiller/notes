# 分布式





## 概念


1. 分布式是`系统部署方式`，服务是分散部署在不同的机器上
2. 微服务是`架构设计风格`，很小的服务，小到一个服务只对应一个单一的功能，只做一件事

区别：

- 生产环境下的微服务肯定是分布式部署的，分布式部署的应用不一定是微服务架构的

- 比如`集群部署`，它是把相同应用复制到不同服务器上，**但是逻辑功能上还是单体应用** 



- 在《分布式系统原理与范型》一书中有如下定义：“分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”；





## 分布式基础





## 分布式事务

锁是让谁去做？？事务是操作时候的具体细节？？



### 分布式锁

|            单机场景            |                      分布式场景                      |
| :---------------------------: | :-------------------------------------------------: |
| 使用`语言的内置锁`来实现进程同步 | 使用`分布式锁`来实现可能位于不同节点（机器）上的进程同步 |

***

阻塞锁通常使用互斥量来实现：

| 互斥量 | 0（false） | 1（true） |
| :----: | :--------: | :-------: |
|  状态  |    锁定    |  未锁定   |
`互斥量`的表示：1 和 0 可以用一个**整型值**表示，也可以用**某个数据是否存在**表示

***

某个数据是否存在表示：

|     |  数据库的唯一索引  |    Redis 的 SETNX 指令    | Redis 的 RedLock 算法 |                Zookeeper 的有序节点                |
| --- | ---------------- | ------------------------ | -------------------- | ------------------------------------------------- |
| 实现 | 某一条记录是否存在 | 插入一个键值对 key是否存在 | 多个 Redis 实例       | 是否为当前子节点列表中 `序号最小的`临时且有序的子节点 |









## 分布式服务











## 分布式存储









## 消息队列



`AMQP`和`JMS` 两种实现标准

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/AMQPDetails.png)



![](https://raw.githubusercontent.com/MicroWiller/photobed/master/P2P.png)



![](https://raw.githubusercontent.com/MicroWiller/photobed/master/PushAndOrder.png)



[RocketMQ源码](https://github.com/apache/rocketmq)





### 顺序消费，怎么保证时序性



时间上的有序？？业务上的相对有序？？



分布式很难保证时序性：分布式系统缺乏全局时钟





### 消息幂等

也就是，消息不会被重复消费









## 分布式缓存









## 分布式高可用




