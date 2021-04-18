# ComputerTheory



## RSA



 [RSA算法原理一](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html): `欧拉函数`/ `欧拉定理` / `模反元素` 

 [RSA算法原理二](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)：`为啥可靠` / `加密和解密`

[GPG入门](http://www.ruanyifeng.com/blog/2013/07/gpg.html)：怎么生成和使用公私钥

[深入理解SSH](https://mp.weixin.qq.com/s/Cu1MkcybjJWomqE11rALpA): `跳板机` / `端口转发` / `指纹匹配` 





## 堡垒机

[堡垒机简介](https://juejin.cn/post/6925212420067557390): 保障网络和数据不受来自外部和内部用户的入侵和破坏，而运用各种技术手段监控和记录运维人员对网络内的服务器、网络设备、安全设备、数据库等设备的操作行为，以便集中报警、及时处理及审计定责。



[开源堡垒机：JumpServer ](https://docs.jumpserver.org/zh/master/) 







## KeePass

密码管理器

https://keepass.info/

 https://zh.wikipedia.org/wiki/KeePass

 https://zhuanlan.zhihu.com/p/39645975







## 进程/线程





- 它们在不同操作系统上也有不同的实现，但是在大多数的实现中线程都属于进程
- Linux 在调度器上，不区分进程和线程的调度
- 现代操作系统都是直接调度线程，不会调度进程



 3 种资源是：计算资源（CPU）、内存资源和文件资源 

|          | 进程               | 线程                |
| -------- | ------------------ | ------------------- |
| 操作系统 | 分配资源的基础单位 | (CPU)调度的基本单元 |
|          |                    |                     |
|          |                    |                     |





### 进程

进程（Process），顾名思义就是**正在执行**的<u>*应用程序*</u>， **<u>*是软件在内存中的执行副本*</u>** 



进程是分配资源的基础单位







### 线程



- 操作系统（CPU）调度时的最基本单元
- 仅仅被分配 CPU 资源

- 很长一段时间被称作**轻量级进程**，是程序执行的基本单位







参考：[进程的开销比线程大在了哪里？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4625) 

【解析】 Linux 中创建一个进程自然会创建一个线程，也就是主线程。创建进程需要为进程划分出一块完整的内存空间，有大量的初始化操作，比如要把内存分段（堆栈、正文区等）。创建线程则简单得多，只需要确定 PC 指针和寄存器的值，并且给线程分配一个栈用于执行程序，同一个进程的多个线程间可以复用堆栈。因此，创建进程比创建线程慢，而且进程的内存开销更大。





### 进程间通信



1. 管道，shell里面的管道
2. 内存共享： Linux 中有 POSIX 内存共享库——shmem；实现原理是以虚拟文件系统的形式，从内存中划分出一块区域，供两个进程共同使用
3. 本地消息/队列：本质上，这些都是收/发消息的模式；在消息体量庞大的情况下，也可以构造生产者队列和消费者队列，用并发技术进行处理
4. 远程调用：通过本地程序调用来封装远程服务请求





参考：[进程间通信都有哪些方法？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4630)

【解析】 你可以从单机和分布式角度给面试管阐述。

- 如果考虑单机模型，有管道、内存共享、消息队列。这三个模型中，内存共享程序最难写，但是性能最高。管道程序最好写，有标准接口。消息队列程序也比较好写，比如用发布/订阅模式实现具体的程序。

- 如果考虑分布式模型，就有远程调用、消息队列和网络请求。直接发送网络请求程序不好写，不如直接用实现好的 RPC 调用框架。RPC 框架会增加系统的耦合，可以考虑 消息队列，以及发布订阅事件的模式，这样可以减少系统间的耦合。







## 内存管理



应用程序一般都有 3 个区域（3 个段）——**正文段**（程序）、**数据段**（常量、全局变量）、**堆栈段** 



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/PortationOfProcess.png" style="zoom:67%;" />





### 虚拟内存



虚拟化技术中，操作系统设计了虚拟内存（理论上可以无限大的空间），受限于 CPU 的处理能力，通常 64bit CPU，就是 264 个地址。

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/VirtualMemory.png)



1. 虚拟化技术中，应用使用的是虚拟内存，**操作系统** <u>*管理*</u> 虚拟内存和真实内存之间的映射
2. 操作系统 将虚拟内存分成整齐小块，每个小块称为一个页
   - 一方面应用使用内存是以页为单位，整齐的页能够避免内存碎片问题。
   - 另一方面，每个应用都有高频使用的数据和低频使用的数据。
   - 这样做，操作系统就不必从应用角度去思考哪个进程是高频的，仅需思考哪些页被高频使用、哪些页被低频使用。如果是低频使用，就将它们保存到硬盘上；如果是高频使用，就让它们保留在真实内存中。







### 页 / 页表



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/PageAndPageChart.png" style="zoom:67%;" />



- 操作系统将虚拟内存分块，每个小块称为一个页（Page）
- 真实内存也需要分块，每个小块我们称为一个 Frame
- Page 到 Frame 的映射，需要一种叫作页表的结构
- Page 大小和 Frame 大小通常相等，页表中记录的某个 Page 对应的 Frame 编号





### MMU



Memory Management Unit 



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/MMU.png" style="zoom:67%;" />



当 CPU 需要执行一条指令时，如果指令中涉及内存读写操作，CPU 会把虚拟地址给 MMU，MMU 自动完成虚拟地址到真实地址的计算；然后，MMU 连接了地址总线，帮助 CPU 操作真实地址。



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/MemoryManagerUnit.png" style="zoom:60%;" />





### TLB



在 MMU 中往往还有一个微型的设备，叫作**转置检测缓冲区**（Translation Lookaside Buffer，TLB）



TLB 是硬件实现的，因此速度很快，通常是一张表，所以 TLB 也称作快表。





































