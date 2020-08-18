# Elastic-Job





## 目标

把**定时任务**通过**集群**的方式进行*管理调度*，并采用**分布式部署**，保证系统的高可用，提高了容错。

- 如何保证定时任务只在集群的某一个节点上执行？？
- 一个任务如何拆分为多个独立的任务项，由分布式的机器去分别执行？？
- 众多的定时任务如何统一管理？？



### 并行任务调度

**并行任务调度 实现靠多线程** ：？？

- 如果有大量任务需要调度，光靠多线程就会有瓶颈？？，仅一台计算机的CPU处理能力是有限的
- 如果将任务调度程序分布式部署，每个节点还可以部署为集群，让多台计算机共同完成任务调度，来提高处理效率



### 高可用

某个实例宕机，不影响其他实例来执行任务

### 弹性扩容



### 任务管理与监测



### 避免任务重复执行

- 分布式锁：Redis、zookeeper
- zookeeper选举











## 基本概念



### 任务调度

系统为了**自动**完成特定任务，在**约定的**特定时刻<u>*去自动的执行*</u> 任务的过程。有了任务调度即可解放更多的人力来由系统自动去执行任务。可以简单的理解为，`定时任务`！！！



### 分布式调度

> 在分布式系统环境下，运行任务调度，称为分布式任务调度

Elastic-Job 是由当当网基于quartz 二次开发之后的**分布式任务调度**解决方案 ， 由两个相对独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成 。

Elastic-Job 主要的设计理念是**无中心化**的**分布式定时调度框架**，思路来源于Quartz的基于DB(数据库)的高可用方案，Quartz按照日历 CronTrigger表达式来进行任务调度。

**但数据库没有分布式协调功能** 

所以在高可用方案的基础上增加了**弹性扩容**和**数据分片**的思路，以便于更大限度的利用分布式服务器的资源。





### 分片

任务的分布式执行：

将**一个任务**拆分为`多个独立的`**任务项**，然后由分布式的服务器分别执行某一个或几个分片项。



例如：有一个遍历数据库某张表的作业，现有2台服务器。 为了快速的执行作业，那么每台服务器应执行作业的50%。

- 为满足此需求，可将作业分成2片，每台服务器执行1片。
  作业遍历数据的逻辑可以为：服务器A遍历ID以奇数结尾的数据；服务器B遍历ID以偶数结尾的数据。

- 如果分成10片，则服务器A被分配到分片项0,1,2,3,4；服务器B被分配到分片项5,6,7,8,9。
  作业遍历数据的逻辑可以为：服务器A遍历ID以0-4结尾的数据；服务器B遍历ID以5-9结尾的数据



### 分片项与业务处理解耦

Elastic-Job并不直接提供数据处理的功能，==框架只会将分片项分配至各个运行中的作业服务器==，开发者需要自行处理**分片项**与**真实数据**的`对应关系`。

需要关注两个部分：

1. 以上面例子分成10片为例，框架只负责 使服务器分配得到哪些分片项，由**作业分配策略**决定
2. 但是**每个分片处理哪一部分数据**，比如第一个分片处理id以0-4结尾的数据，是由开发者去决定和处理的。







### 中心化

xxl-job是中心化设计

所有定时任务的执行：

1. 在**调度中心**，判断作业是否到了执行时间
2. 然后通知业务系统去执行

> 即使是作业节点，也并不知道自己应该什么时候执行定时任务？？，只能通过调度中心去决定作业的执行





### 去中心化

elastic-job是去中心化设计

作业**调度**中心节点，各个作业节点是自治的？？，作业框架的程序 <u>在到达相应时间点时</u> **各自触发调度**？？

怎么个自洽法？？

每个作业以自己为中心？？

缺点是可能会存在各个作业服务器的时间不一致的问题。




