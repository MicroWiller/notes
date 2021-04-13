

# JVM和GC



[MAT（Memory Analyzer Tool） 工具简介](https://juejin.cn/post/6908665391136899079)     





## 对象内存分配流程

一个对象刚刚new出来，会尝试在`栈`上分配

> 前提：栈上分配不下，才存到共享区 



![](https://raw.githubusercontent.com/MicroWiller/photobed/master/ObjectAllocated.png)

- 这里**考虑的是无法栈上分配需要共享的对象** 
- 对于 HotSpot JVM 实现，所有的 GC 算法的实现都是一种对于堆内存的管理，也就是都实现了一种堆的抽象，它们都实现了接口 CollectedHeap
- 当分配一个对象堆内存空间时，在 CollectedHeap 上首先都会检查是否启用了 TLAB
- 如果启用了，则会尝试 TLAB 分配
- 如果当前线程的 TLAB 大小足够，那么从线程当前的 TLAB 中分配
- 如果不够，但是当前 TLAB 剩余空间小于**最大浪费空间限制**，则从堆上（一般是 Eden 区） **重新申请**一个新的 TLAB 进行分配
- 否则，直接在 TLAB 外进行分配



参考：[全网最硬核 JVM TLAB 分析](https://juejin.cn/post/6925217498723778568#heading-0)	









##  JVM体系结构

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/JVMSystem.png)

Java8以后的JVM
![](_v_images/20200121175013850_26836.png )

<img src="JVM和GC.assets/image-20200517094615981.png" alt="image-20200517094615981" style="zoom:50%;" />



##  GC 流程图

<img src="JVM和GC.assets/image-20200517104747335.png" alt="image-20200517104747335" style="zoom:80%;" />

## 垃圾回收算法

>什么是垃圾？？
`简单说`，就是内存中已经不再被使用的空间


1. 复制拷贝：新生代
    【Eden==>Survivor，把有用的拷贝到幸存区】
    【SurvivorForm ==> SurvivorTo】
    
    ​	
    
    <img src="_v_images/20200123085137923_16851.png" style="zoom:60%;" />
    优点：速度快
    缺点：浪费内存
    
2. 标记清除
    <img src="JVM和GC.assets/image-20200517094751023.png" alt="image-20200517094751023" style="zoom:60%;" />
    缺点：导致内存碎片

3. 标记整理
    <img src="_v_images/20200121180926308_1093.png" style="zoom:70%;" />
    优点：既不浪费空间，也不产生碎片
    缺点：耗时间

GC算法总体概述：
<img src="_v_images/20200122223625185_9683.png" style="zoom:60%;" />

<img src="_v_images/20200516182904870_19025.png" style="zoom:67%;" />

##  GC Roots

> 怎么找到垃圾
>
> - 引用计数
> - 可达性分析


> 为什么用 可达性分析 枚举根节点（根搜索路径）？？
<img src="JVM和GC.assets/image-20200517094938371.png" alt="image-20200517094938371" style="zoom:67%;" />

<img src="_v_images/20200121195952068_25064.png" style="zoom:40%;" />


有哪些可以作为GC Roots 的对象 ---> `set` ：
* 方法区中 类静态属性 引用的对象
* 方法区中 常量 引用的对象
* 本地方法栈中 JNI (Java本地接口) 中引用的对象 （线程里面的start() 就是Native方法）
* 虚拟机栈中 局部变量表 中引用的对象

代码实例：
```java
public class GCRootDemo {
    private byte[] byteArray = new byte[100 * 1024 * 1024];

    private static GCRootDemo2 g2;                                // 方法区中 类静态属性 引用的对象
    private static final GCRootDemo3 g3 = new GCRootDemo3();      // 方法区中 常量 引用的对象
    
    public static void m1(){
        GCRootDemo g1 = new GCRootDemo();                     // 虚拟机栈中 局部变量表 中引用的对象
        System.gc();
        System.out.println("第一次GC完成");
    }
    public static void main(String[] args) {
        m1();
    }
}
```



##  JVM调优

####  查看命令

>如何查看一个正在运行中的java程序，它的某个JVM参数是否开启？具体值是多少？

查看参数盘点家底：
1. `jps`（查看正在运行的虚拟机金灿灿）  


`jinfo`（实时查看和调整虚拟机的参数）： jinfo -flag PrintGCDetails 进程号（查看该进程是否开启打印细节）
****
    `jstat` (监视虚拟机的各种运行状态信息，具体包括类装载，内存，垃圾收集，JIT编译等运行数据，主要用来在运行期定位虚拟机性能问题) ：jstat -gc 进程号 【查看该进程的gc信息】
    
    `jstack` (Java堆栈跟踪工具)  jstack 进程号【把该进程所有的线程全列出来】


​    
​    
2. 命令

   - java -XX:+PrintFlags`Initial` [- version]    （主要查看初始默认值,JVM出厂设置）

   -  java -XX:+PrintFlags`Final` [- version]        （主要查看修改更新）
        注意区分：
        
        1. =(没改过) 
        2.  :=(人为更新/不同JVM加载不一样)
           <img src="_v_images/20200121225955282_3228.png" style="zoom:67%;" />
        
        
        
   - java -XX:+Print`CommandLine`Flags -version    （方便查看垃圾回收器的种类）

   - -XX: +UseCompressedClassPointers    (对象的内存布局)

   - -XX: +UseCompressedOops        (使用压缩的普通对象指针)

###  JVM的参数类型

1. 标配参数
    - -version
    - -help
    - java -showversion
2. X参数 （了解）
    - -Xint        解释执行
    - -Xcomp    第一次使用就编译成本地代码
    - -Xmixed    混合模式
3. `XX参数`
    1. Boolean类型
       
        - 公式：-XX: +/- 某个属性值    （+表示开启，-表示关闭）
        - 是否打印GC收集细节         -XX: +/- PrintGCDetails
        - 是否使用串行垃圾回收器    -XX: +/- UseSerialGC
        
    2. KV设值类型
        - 公式：-XX: 属性key = 属性值value
        - -XX : MetaspaceSize = 128m
        - -XX : MaxTenuringThreshold = 15
        
        
        
        - [ ] jinfo举例，如何查看当前运行程序的配置
        
        - 公式：jinfo -flag 配置项 进程编号
        - 查看进程全部信息：jinfo -flag`s` 进程编号
        - 查看进程具体某个信息：jinfo -flag InitialHeapSize 进程编号
        
        - [ ] 坑题
          <img src="JVM和GC.assets/image-20200517095324904.png" alt="image-20200517095324904" style="zoom:50%;" />
              -Xms1024m  -Xmx1024m 两个值`最好相同`（根据物理内存：Xms默认为1/64 Xmx默认为1/4）
        
        - -Xss ：-XX:ThreadStackSize


### 常用基本参数配置

<img src="_v_images/20200122081332314_25851.png" style="zoom:60%;" />



#### -Xms/ -Xmx

<img src="_v_images/20200122081621807_12866.png" style="zoom:67%;" />

> -Xms和-Xmx 内存设置成相同，`扩大和缩小`内存都会浪费资源

#### -Xss

<img src="_v_images/20200122081735191_32115.png" style="zoom:67%;" />
windows下依赖于虚拟内存(0)，Linux/x64默认为1024K

#### -Xmn : 设置年轻代大小



#### 小总结

<img src="_v_images/20200121232246229_10245.png" style="zoom:60%;" />

#### -XX : MetaspaceSize

![](JVM和GC.assets/20200122083505077_18881.png )
元空间默认的是`二十多兆`，而不管是几个G的内存

#### 典型设置案例

<img src="JVM和GC.assets/image-20200517101347006.png" alt="image-20200517101347006" style="zoom:67%;" />



#### -XX:+PeintGCDetails

- 输出详细GC收集日志信息
- GC：新生代
![](JVM和GC.assets/20200122085944959_10820.png )
- FullGC：老年代
![](JVM和GC.assets/20200122090843315_13816.png )

#### -XX:SurvivorRatio

<img src="JVM和GC.assets/image-20200517101716165.png" alt="image-20200517101716165" style="zoom:50%;" />

<img src="JVM和GC.assets/image-20200517101803401.png" alt="image-20200517101803401" style="zoom:60%;" />

#### -XX:NewRatio

<img src="JVM和GC.assets/image-20200517101922999.png" alt="image-20200517101922999" style="zoom:50%;" />

<img src="JVM和GC.assets/image-20200517101949909.png" alt="image-20200517101949909" style="zoom:67%;" />

#### -XX:MaxTenuringThreshold

设置垃圾最大年龄

<img src="JVM和GC.assets/image-20200517100159285.png" alt="image-20200517100159285" style="zoom:60%;" />

#### -XX:PreTenureSizeThreshold

大对象到达多大，直接扔到老年代



###  调优

####  调优的基础概念

<img src="JVM和GC.assets/image-20200518095817873.png" alt="image-20200518095817873" style="zoom:50%;" />

#### 什么是调优

1. 根据需求进行JVM规划和预调优，？？到底用多少内存，机器什么样的算力
2. 优化运行JVM运行环境（慢，卡顿）？？优化虚拟机参数
3. 解决JVM运行过程中出现的各种问题(OOM)

> CPU飙高怎么检查？？
>
> 1. 先找出哪个线程：jstack / arthas(thread)，找出哪个线程的CPU用得最高
> 2. 如果该线程为GC线程，得看GC日志，是否频繁GC，如果是，表示有些对象回收不掉，也可能是OOM问题导致CPU飙高
> 3. 如果是业务线程，到线程里面查看调用了什么方法，查看方法有什么问题

> 内存泄漏OOM怎么排查？？
>
> 测试环境：jmap / heapdump 谁占用的内存比较大
>
> 线上环境：只能找日志，怀疑哪个类，用arthas跟踪它 

####  获取虚拟机内存总量

![](JVM和GC.assets/20200121232701200_29051.png )


## 引用类型

### 整体架构

![](JVM和GC.assets/20200122143627716_22443.png )

### 强引用

对于强引用的对象，就算出现了OOM也不会对该对象进行回收，`死都不收`。


```java
Object obj = new Object();
```

### 软引用

被软引用关联的对象，`内存够用时保留，不够用回收`。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
System.out.println(sf.get());
```
软引用，通常用在对内存敏感的程序中；比如高速缓存(hashMap)，MyBatis缓存中 都用到软引用

### 弱引用

被弱引用关联的对象，只要`一运行gc一定会被回收`，不管JVM的内存空间是否足够，都会回收该对象占用的内存

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;    // 使对象只被弱引用关联
System.out.println(wf.get());
```

### 虚引用
> 监控 对象的回收情况（`死刑之前做点什么`），在被真正回收前`需要被`引用队列保存下
>
> <img src="JVM和GC.assets/20200122164416808_30454.png" style="zoom:67%;" />

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
obj = null;
```
### 软/弱的适用场景
<img src="JVM和GC.assets/20200122155400546_9526.png" style="zoom:67%;" />

解决缓存清空，保证性能 ---> WeakHashMap（只要一gc就清除） --->高速缓存

死缓：
<img src="JVM和GC.assets/image-20200517102427535.png" alt="image-20200517102427535" style="zoom:67%;" />



## GCRoots 和 四大引用
![](_v_images/20200122171629934_17599.png =450x)

## OOM

### 架构
![](_v_images/20200122173136333_20561.png =400x)

### Error
![](_v_images/20200122172252756_13509.png =500x)

- StackOverFlowError    栈溢出，递归深度过多，（Linux下1024K）
- OutOfMemoryError: Java heap space    创建的对象过多/太大
- OutOfMemoryError: GC overhead limit exceeded    GC回收时间过长，超过98%的时间来做GC并且回收了不到2%的堆内存
    ![](_v_images/20200122182248238_17027.png =600x)
    场景：
    ![](_v_images/20200122183130612_18013.png =500x)

- OutOfMemoryError: Direct buffer memory
    ![](_v_images/20200122184050093_28848.png =600x)
    场景：
    ![](_v_images/20200122184835521_16514.png =500x)

- OutOfMemoryError: `unable to create new native thread`    （高并发）
    ![](_v_images/20200122203506248_28882.png =700x)
    实例：
    ![](_v_images/20200122204415463_18903.png =500x)
    服务器级别参数调优，修改线程数量：
    ![](_v_images/20200122211007832_23278.png =500x)

- OutOfMemoryError: Metaspace
    ![](_v_images/20200122212154382_17443.png =600x)


## 垃圾回收器

![](_v_images/20200122213859606_8263.png =500x)

![](_v_images/20200122214252106_15616.png =600x)


### 四种回收算法

<img src="JVM和GC.assets/image-20200517095453632.png" alt="image-20200517095453632" style="zoom:50%;" />

- Serial：串行垃圾回收器（单线程）
    ![](JVM和GC.assets/20200122221029784_26253.png )
    
- Parallel：并行垃圾回收器（多线程）
    ![](JVM和GC.assets/20200122220828368_2479.png )
    
- CMS：Concurrent Mark Sweep
    ![](_v_images/20200122221444302_19811.png )
    
    a mostly Concurrent, low-pause collecter
    
    1. initial mark	2. concurrent mark	3. remark	4. concurrent sweep

<img src="JVM和GC.assets/image-20200517095602385.png" alt="image-20200517095602385" style="zoom:67%;" />

- G1：垃圾回收器
    ![](JVM和GC.assets/20200122222443919_6278.png )

> 总结：
>
> 垃圾回收算器的发展：随着内存越来越大的过程而演进的
>
> Serial算法：几十兆；Parallel算法：几个G；CMS算法：几十个G


### 配置垃圾回收器

- 查看服务器默认垃圾收集器： java -XX:+PrintCommandLineFlags -version
- 配置使用串行回收器：java -XX:+UseSerialGC

![](JVM和GC.assets/20200122225921570_364.png )

<img src="_v_images/20200122231511712_28695.png" style="zoom:67%;" />

<img src="_v_images/20200123080731515_25899.png" style="zoom:67%;" />

### 七大垃圾收集器

<img src="JVM和GC.assets/image-20200517105354978.png" alt="image-20200517105354978" style="zoom:60%;" />

<img src="_v_images/20200123081517673_29407.png" style="zoom:50%;" />

> 1.8默认使用 useParaGC

#### 新生代

##### 串行GC(Serial)：

DefNew一个，Tenured一个

![](JVM和GC.assets/20200123084202028_14209.png )

##### 并行GC(ParNew)：

ParNew和Parallel Scavenge算法上没有区别，只是连接的老年代不同

<img src="JVM和GC.assets/20200123090533739_25418.png" style="zoom:67%;" />

<img src="JVM和GC.assets/20200123090606821_7371.png" style="zoom:67%;" />

##### 并行回收GC(Parallel Scavenge)

PSYoungGen多个，ParOldGen多个

<img src="JVM和GC.assets/20200123091359674_14600.png" style="zoom:80%;" />

<img src="JVM和GC.assets/20200123092217275_25285.png" style="zoom:60%;" />



#### 老年代

##### 串行GC(Serial Old)

![](JVM和GC.assets/20200123162312659_29801.png )



##### 并行GC(Parallel Old)

![](JVM和GC.assets/20200123153847756_6181.png )



##### 并发标记清除GC(CMS)

a mostly Concurrent, low-pause collecter

1. initial mark	2. concurrent mark	3. remark	4. concurrent sweep

   > 1. 初始标记：只找到最根上的对象，*main方法里面new的对象*，时间短
   >
   > 2. 并发标记：工作线程和垃圾回收线程并发，三色标记法
   >
   > ​	误删：并发标记时被标记的垃圾，后面运行过程中有*其他的引用连上了它*，不是垃圾了
   >
   > 3. 重新标记：由于前面阶段会有漏标、误标的标记产生，所以需要做重新标记，做修正，也必须是STW
   > 4. 并发清理：垃圾回收线程清理的时候，工作线程可能产生新的垃圾，浮动垃圾，下一轮才能清理掉
   >
   > <img src="JVM和GC.assets/20200123161017759_2056.png" style="zoom:50%;" />

![](JVM和GC.assets/20200123155027628_23730.png )

虽然是并发，但两次标记过程还是`需要停顿一点点`
![](JVM和GC.assets/20200123160330121_5792.png )

CMS的问题：

1. Concurrent Mode Failure
2. 浮动垃圾
3. 碎片化特别严重时候，用单线程Serial Old来清理整个内存，卡顿更严重  

优点：并发收集停顿低
缺点：并发执行对CPU资源大；采用的标记-清除算法会导致`大量碎片`



##### 三色标记法

<img src="JVM和GC.assets/image-20200517222857781.png" alt="image-20200517222857781" style="zoom:60%;" />

[视频解说](https://www.bilibili.com/video/BV1AZ4y147fD?p=7)

CMS： Incremental Update	

G1：SATB	

ZGC：Colored Pointers【算法涉及到操作系统里面虚拟内存的映射】

> 都是怎么用最快效率的方式，同时标记的时候能*找到漏标和误标的标记*



###### 问题：

<img src="JVM和GC.assets/image-20200519093648702.png" alt="image-20200519093648702" style="zoom:50%;" />

###### 第1种情况：B=>D 消失

<img src="JVM和GC.assets/image-20200519103329527.png" style="zoom:40%;" />

###### 第2种情况：B=>D 消失，A=>D增加

<img src="JVM和GC.assets/image-20200519103243574.png" alt="image-20200519103243574" style="zoom:43%;" />

> 很严重的问题：D会被当做垃圾回收掉

###### 解决方法1

<img src="JVM和GC.assets/image-20200519103034204.png" alt="image-20200519103034204" style="zoom:50%;" />

- 将黑色A 变成灰色，下次再扫描的时候，找A的孩子

  > 什么时候变灰色？？
  >
  > - 写后内存屏障，JVM会根据一些操作来 增加自己特定的一些操作
  >
  > 当一个引用发生改变的时候，JVM跟踪该引用，如果发现是黑色对象指向白色对象，这时黑色对象变成灰色

  

###### 解决方法2

<img src="JVM和GC.assets/image-20200519105005925.png" alt="image-20200519105005925" style="zoom:50%;" />

###### 解决方法3

<img src="JVM和GC.assets/image-20200519105638734.png" alt="image-20200519105638734" style="zoom:50%;" />

- 64位的虚拟机，每一个引用占64位，每个指针的长度都是64位
- 42位，代表指向这个对象的真正地址
- 18位，没有用
- Marked 0 ，Marked 1，代表不同的标记状态
- Remapped，该对象挪了位置，正在重新定位的过程
- Finalizable，正在回收

> 读内存屏障，读对象之前，看看该对象是什么状态



```java
 A a = new A();
```

之前都是在 new 出来的对象上动手脚，回收到哪儿去了？？ 

> 颜色指针：
>
> 在 指针a 上动手脚
>
> 操作系统的内存地址映射，内存的虚拟空间映射到物理地址中的物理空间
>
> <img src="JVM和GC.assets/image-20200519110859920.png" alt="image-20200519110859920" style="zoom:40%;" />





##### 补充

<img src="_v_images/20200123082343426_5593.png" style="zoom:70%;" />



sever和client的区别：
<img src="_v_images/20200123083047493_6305.png" style="zoom:67%;" />

### 如何选择垃圾收集器

<img src="JVM和GC.assets/image-20200517110316239.png" alt="image-20200517110316239" style="zoom:67%;" />

> CMS毛病很多，如果调不了最优，直接升G1（虽然空间占得比较多，但响应时间好很多）

![](JVM和GC.assets/20200123171632050_23609.png )

### G1


#### 以前收集器特点

<img src="_v_images/20200123172707820_29894.png" style="zoom:67%;" />

#### G1是什么

<img src="JVM和GC.assets/image-20200517110612649.png" alt="image-20200517110612649" style="zoom:80%;" />

![](JVM和GC.assets/20200123173652149_4672.png )

> 逻辑分代，物理不分代

<img src="JVM和GC.assets/20200123174345218_214.png" style="zoom:80%;" />

<img src="JVM和GC.assets/image-20200518092622236.png" alt="image-20200518092622236" style="zoom:50%;" />



特点：
![](JVM和GC.assets/20200123175534100_8366.png )

最大的好处：化整为零，避免全内存扫描，只需要按照区域来进行扫描即可

#### 底层原理

- `Region区域化`垃圾收集器
    <img src="JVM和GC.assets/image-20200517110804915.png" alt="image-20200517110804915" style="zoom:80%;" />

    ![](JVM和GC.assets/20200123182346280_17205.png )
    
    
    
    Region的回收算法：
    <img src="_v_images/20200123182804185_2191.png" style="zoom:80%;" />
    
    Humongous：巨型对象
    ![](_v_images/20200123183055916_13358.png )

回收步骤
    G1收集器下的Young GC：
![](JVM和GC.assets/20200123184448884_14359.png )

​					 <img src="_v_images/20200123184542868_17392.png" style="zoom:70%;" />
​    

<img src="_v_images/20200123184400559_13593.png" style="zoom:70%;" />

 4 步过程
    <img src="_v_images/20200123185117962_16093.png" style="zoom:70%;" />



#### 常用配置参数

<img src="JVM和GC.assets/image-20200517111427777.png" alt="image-20200517111427777" style="zoom:80%;" />

![](JVM和GC.assets/20200123204939845_15695.png )

#### 和CMS比 有什么优势

![](JVM和GC.assets/20200123205308698_4359.png )

#### 历史由来

![](JVM和GC.assets/20200123174234219_26865.png )

## 10. SpringBoot微服务优化

![](_v_images/20200123210807666_25952.png )

外部启动：
![](_v_images/20200123210500999_23401.png )

![](_v_images/20200123211011832_16014.png )




