# `zookeeper`



`zookeeper`，作为一个分布式协调服务，用途挺多，服务注册与发现、分布式锁、集群选举、配置中心等







## 面试

 [从面试题了解`ZooKeeper`](https://zhuanlan.zhihu.com/p/223341260)  







## 核心概念

  [注册中心优劣分析](https://qiankunpingtai.cn/article/1590111892883) 

主流注册中心产品：

|                  | Nacos                      | Eureka      | Consul            | CoreDNS    | Zookeeper  |
| ---------------- | -------------------------- | ----------- | ----------------- | ---------- | ---------- |
| 一致性协议       | CP+AP                      | AP          | CP                | —          | CP         |
| 健康检查         | TCP/HTTP/MYSQL/Client Beat | Client Beat | TCP/HTTP/gRPC/Cmd | —          | Keep Alive |
| 负载均衡策略     | 权重 /metadata/Selector    | Ribbon      | Fabio             | RoundRobin | —          |
| 雪崩保护         | 有                         | 有          | 无                | 无         | 无         |
| 自动注销实例     | 支持                       | 支持        | 不支持            | 不支持     | 支持       |
| 访问协议         | HTTP/DNS                   | HTTP        | HTTP/DNS          | DNS        | TCP        |
| 监听支持         | 支持                       | 支持        | 支持              | 不支持     | 支持       |
| 多数据中心       | 支持                       | 支持        | 支持              | 不支持     | 不支持     |
| 跨注册中心同步   | 支持                       | 不支持      | 支持              | 不支持     | 不支持     |
| SpringCloud 集成 | 支持                       | 支持        | 支持              | 不支持     | 不支持     |
| Dubbo 集成       | 支持                       | 不支持      | 不支持            | 不支持     | 支持       |
| K8S 集成         | 支持                       | 不支持      | 支持              | 支持       | 不支持     |

[consul使用手册](https://blog.csdn.net/liuzhuchen/article/details/81913562)：`consul template` 



 [服务注册与发现](https://juejin.cn/post/6844903829113143310)：代码demo



[Session / Watch / 数据模型](https://juejin.cn/post/6918688918481141774)	





## 分布式锁

 [分布式锁实现的正确打开方式](https://www.cnblogs.com/zhangyinhua/p/14504717.html) 





## 集群

[集群间的数据同步](https://mp.weixin.qq.com/s/HKNWcC9XY88Dgar_mEqH-g) 







