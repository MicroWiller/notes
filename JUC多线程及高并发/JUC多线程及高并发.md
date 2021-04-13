# JUC多线程及高并发



高并发最nb的不是加锁，而是`控制` 



## 面试题



 [如何优雅的让3个线程打印ABC](https://mp.weixin.qq.com/s/oEWfJ5ExWUHpB7xQ21LD-Q)：BlockingQueue，类比Go里面的Channel



- 为什么 wait 方法必须在 synchronized 保护的同步代码中使用？

- 为什么 wait/notify/notifyAll 被定义在 Object 类中，而 sleep 定义在 Thread 类中？

- wait/notify 和 sleep 方法的异同？

 [wait/notify/notifyAll](#wait/notify/notifyAll)



- 谈谈你对`volatile` 的理解？？ [volatile](#volatile)  

- `volatile`的可见性和禁止指令重排序是如何实现的？？[内存屏障](#内存屏障) 

- 请描述一下对象的创建过程？？ [对象的创建过程](#对象的创建过程) 

- 对象在内存中的`内存布局`？？

- `Obeject o = new Object()`在内存中占了多少字节？？ [16字节](https://blog.csdn.net/u011727756/article/details/106546178/) 

- `DCL单例`为什么要加`volatile`？？[DCL](#DCL) 

  

- `CAS`你知道么？？ [CAS](#CAS（CompareAndSwap）) 

- 原子类`AtomicInteger`的`ABA`问题谈谈？？`原子更新引用`知道么？？ [AtomicInteger](#AtomicInteger) 

  

- ` ArrayList`是线程不安全的，请编写一个` 不安全`的代码？？

  ```java
  public static void main(String[] args) {
      final List<String> list = new ArrayList<String>();
      for (int i = 0; i < 30; i++) {
          new Thread(new Runnable() {
              public void run() {
                  list.add(UUID.randomUUID().toString().substring(0, 8));
                  System.out.println(list.toString());
              }
          }).start();
      }
  }
  ```

  

- `公平锁` / `非公平锁` / `可重入锁` / `递归锁` / `自旋锁` 谈谈你的理解？？

- 请描述`Synchrnoized`和`ReentrantLock`的异同？？

- 请描述`Synchrnoized`和`ReentrantLock`的底层实现及`重入的底层原理`？？
  
- [Synchronized](#Synchronized)  [ReentrantLock](#Synchronized / ReentrantLock) [重入](#可重入锁（递归锁）) 
  
- `CountDownLatch` / `CyclicBarrier` / `Semaphore` 使用过么？？
- 请描述锁的`四种状态`和`升级过程`？？
- 请描述一下`锁的分类`以及`JDK中的应用`？？
- `自旋锁`一定比`重量级锁`效率高吗？？ [自旋锁](#自旋锁) 
- 打开偏向锁是否效率一定会提升？？为什么？？



- `阻塞队列`知道么？？
- `线程池`用过么？？`ThreadPoolExecutor`谈谈你的理解？？
  - 自定义线程池，七大参数
- 生产上，你如何设置`合理参数`的？？



- `死锁编码`以及`定位分析`谈谈？？



- 请谈一下`AQS`，为什么`AQS`的底层是`CAS + volatile` 



- 聊聊你对`as-if-serial`和`happens-before`语义的理解？？



- 你了解`ThreadLocal`吗？？你知道`ThreadLocal`中如何解决内存泄漏问题吗？？ [ThreadLocal](#ThreadLocal) 













## JMM

[从 JMM 透析 volatile 与 synchronized 原理](https://mp.weixin.qq.com/s?__biz=MzU2NDg0OTgyMA==&mid=2247495689&idx=2&sn=87e3df8b713978ee6d9732c3b194e6cb&chksm=fc460dfacb3184ec07904d964a8eeff2dd67f40849487c2bbaae93874cdcaf807307e9744fb6&mpshare=1&scene=1&srcid=1109mbQPuMqxAQy5vYFOfTea&sharer_sharetime=1604937209814&sharer_shareid=af15e3d5261cd956e28a355a03c10c40&key=516e28ba4c9b5ddf18aa93805270f9aa6bb26c25361c24e449e4d8eca9c8e1f0a1247d3018fe201384c6fc355aa3f1d3e00905ca1e584a70ba621329b001cf3a1f16ce92afdc61ad1c87efecc36f0e170b2d53fb0887b21ce4590dd7d2080c645620c106b44c2570dfedb044cb0c1afc993657e2cc39ef55aa415dc3375b4929&ascene=1&devicetype=Windows+10+x64&version=63000039&lang=zh_CN&exportkey=A86ZuDMKEZZFBGTivef2w6A%3D&pass_ticket=lr%2BSexr5Km%2F3qIPXzb2wzVGwiEEQ%2BFFMZLTDCmv3j4p5edu%2BzBIjpqQSzCn%2FC3XR&wx_header=0#)	



### 定义

Java内存模型， 不是“真实存在”，

而是一种*抽象定义*，是和多线程相关的一组“规范”，

需要每个 JVM 的实现都要遵守这样的“规范”，

主要为了<u>并发编程</u> **安全的访问数据**。





- 每创建一个线程，虚拟机都会为其开辟一个`工作内存`（Java虚拟机栈空间）；
- 每个线程**操作** 主内存的数据时，都需要将数据`拷贝复制`到线程自己的**工作内存**中，修改完了再将值写回主内存中



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/JMM.png" style="zoom:57%;" />

### 虚拟机栈



每个线程拥有一个「虚拟机栈」，每个「虚拟机栈」拥有多个「栈帧」，而栈帧则对应着一个方法。

每个「栈帧」包含局部变量表、操作数栈、动态链接、方法返回地址。

方法运行结束则意味着该「栈帧」出栈。

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/JVMStack.png" style="zoom:47%;" />



- Java 内存模型中的线程的`工作内存（working memory）`是 cpu 的`寄存器`和`高速缓存`的**<u>*抽象描述*</u>**

-  JVM 的静态内存储模型`（JVM 内存模型）`**<u>*是一种对内存的物理划分*</u>**，它只`局限在内存`，而且只局限在 JVM 的内存



### 特性



> JMM需要满足三个特性：原子性，有序性，可见性


2. 原子性：逻辑上的最小单位，成功一起成功，失败一起失败，类似事务的原子性
2. 有序性：按照编译后的代码顺序执行
3. 可见性：其中一个线程修改后的值写回主内存时，主内存马上通知其他线程最新的值



### 有序性

> 系统底层如何保证有序性？？   

1. 内存屏障sfence mfence lfence等系统CPU原语
2. 锁总线



 [锁总线的视频详细解说 66:30](https://www.bilibili.com/video/BV12Q4y1N75Q?p=5)

















## volatile





### 乱序执行

==原因：==

**CPU乱序执行：**CPU速度比内存快很多，等着从内存取得数据，绝大部分时间片空转了

CPU在进行`读等待`的同时`执行其他指令`，**<u>*是CPU乱序的根源*</u>**

这种实现，是为了`提高效率`

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/messExecute.png" style="zoom:67%;" />







### 内存屏障

==但，有些时候，我们并不需要== 

> **底层**的内存屏障 和 **JVM级别**的内存屏障  `不一样`  【从顶至低有 Java/Byte/JVM/CPU等层级】
>
> [内存屏障的视频解释 49:19](https://www.bilibili.com/video/BV12Q4y1N75Q?p=5)

JVM级别 要求任何虚拟机都必须实现 **JSR标准**：<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/MemoryLoad.png" style="zoom:67%;" />

> Load：读；Store：写	



JVM层级，volatile的**实现细节**：<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/JvmVolatileDetail.png" style="zoom:67%;" />



CPU底层的实现，如X86 CPU内存屏障：![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/X86MemoryLoad.png)

万能的方法：锁总线



JVM层别，有些底层不可以进行重排序（了解）：

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ResortJVM.png" style="zoom:67%;" />







### 对象的创建过程

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ObjectCreatProcess.png" style="zoom:67%;" />

- new ：向内存里申请一块空间 ==> 0 new #2 <T>，
- dup：设默认值
- invokespecial 调用 T 的构造方法
- astore_1 ：小t 指向 new出来的空间



### 定义

是轻量级的同步策略，相当于`乞丐版`的synchronized



**修饰变量时：**

- 线程间可见 【其中一个线程修改后的值写回主内存时，主内存马上通知其他线程最新的值】
- 不保证原子性
- 禁止指令重排    （内存屏障）





### 底层实现

>加入volatile关键字时，汇编后会多出一个`lock`前缀指令。lock前缀指令其实就相当于一个内存屏障。

- JVM层级，特别精巧的情况下，根据系统**CPU**是否支持**缓存一致性协议**，如果支持，volatile使用缓存一致性协议实现，不支持再使用其他方式
- 作为volatile指令来讲，在JVM层级， 有JVM级别的内存屏障
- hotspot偷懒，直接使用最终极的方式，**锁总线**



### DCL

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/DCLneedVolatile.png" style="zoom:67%;" />

如果发生指令重排，线程2判断 `t != null` ，然后使用对象，这时候的对象的<u>***成员属性为默认值0/null***</u>





### n++

**n++，n被volatitle修饰：**

1. 不能保证原子性，被拆分成了`3个`指令：getfield、iadd、putfield
2. 但会禁止指令重排，并保证可见性



可以用 JUC中的 `Atomic包` 来保证原子性(AtomicInteger、、、)：

```java
private volatile AtomicInteger value;
```

底层是CAS





###  缓存一致性协议

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/MESICache.png" style="zoom:67%;" />

MESI (Intel CPU)

缓存一致性协议 ，跟Java的volatile底层实现 **没有一毛钱关系**

根据不同的CPU有不同的实现方法







## CAS（CompareAndSwap）

==自旋锁，可以保证原子性==



### 原理

CAS的全称是`Compare-And-Swap`，**<u>*它是一条CPU并发原语*</u>**

- 它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值，***<u>这个过程是原子的</u>***
- compareAndSwap，这个过程是`原子操作`，*是一个CPU并发源语，偏硬件，天生就是连续的，不会被打断*



```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```


- expect: 期望和之前的值相同，update：现在想更新的值
- return false; 表示期望值和真实值不相同，交换失败
- return true;  表示期望值和真实值相同，交换成功




`相比下，什么不用synchronized??`
- synchronized 加锁，同一时间段只允许一个线程访问，一致性得到了保障，但`并发性下降`
- CAS 没有加锁，反复通过CAS比较，直到比较成功一刻为止，**既保证了一致性又提高了并发性**，其缺点`消耗CPU资源`





### AtomicInteger

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    // 得到unsafe类
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            // 对象的内存偏移量
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
}
```



```java
/**
 * Atomically increments by one the current value.
 *
 * @return the previous value
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

```java
// unsafe.getAndAddInt
// var1 当前对象本身
// var2 当前对象值的引用地址
// var4 需要变动的数量
// var5 通过var1和var2找出的 当前主内存中的值
public final int getAndAddInt (Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while (this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```



[后面的源码视频追踪](https://www.bilibili.com/video/BV12Q4y1N75Q?p=2)

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/CompareAndSwapInt.png)



以上代码的底层汇编：
![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/UnsafeOriginal.png)



**atomic_linux_x86.inline.hpp：**Java层面的CAS对应汇编层级上的`cmpxchg`指令，但该指令并不是原子性的

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/atomic_linux_x86.png)



`LOCK_IF_MP`，保证了 cmpxchg1 的原子性，MP表示 mutil processor 多个CPU

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/Lock_if_Mp.png)



**指令：** *基本上Java层面所有的锁，最终的实现都和该指令有关*

```assembly
 lock cmpxchg
```

- 硬件： lock指令，在执行后面指令的时候，**锁定一个北桥信号** （不采用锁总线的方式，会更轻量级）
- 一个CPU 要去内存访问一个值的时候 **要经过**系统总线





### CAS缺点

1. 循环时间长开销很大，`自旋`频繁

  <img src="JUC多线程及高并发.assets/20200117214147271_7297.png" style="zoom:60%;" />

  

2. 只能保证`一个`共享变量的原子操作

  <img src="JUC多线程及高并发.assets/image-20200519155344914.png" alt="image-20200519155344914" style="zoom:60%;" />

  

3. ABA问题


### ABA

**ABA问题：**狸猫换太子，首尾一样，但中间有多次猫腻的机会

<img src="JUC多线程及高并发.assets/20200117220613001_13429.png" style="zoom:60%;" />

### 原子引用

<img src="JUC多线程及高并发.assets/20200117222210458_2656.png" style="zoom:80%;" />

Reference，表示自定义的类型（User、Student、、、）

### 带标记的原子引用
stamped
理解原子引用+`修改版本号`(类似时间戳)

<img src="JUC多线程及高并发.assets/20200117224207676_619.png" style="zoom:67%;" />



***

## Unsafe
JDk 娘胎里就自带的基础类，rt.jar包下的原生类
![](_v_images/20200117074634953_10518.png =500x)

![](_v_images/20200117081059064_7405.png =800x)
![](_v_images/20200117085007928_26887.png =800x)

Unsafe类（像C的指针） + CAS思想（CPU并发源语）= 自旋









## 锁



### 锁的 7 大分类

`多种多样的分类`，是评价一个事物的多种标准

- 比如评价一个城市，标准有人口多少、经济发达与否、城市面积大小等
- 而一个城市可能同时占据多个标准，以北京而言，人口多，经济发达，同时城市面积还很大



> 同理，对于 Java 中的锁而言，一把锁也有可能同时占有多个标准，符合多种分类
>
> 比如 ReentrantLock 既是可中断锁，又是可重入锁



 7 大类别，分别是：

- 偏向锁/轻量级锁/重量级锁：`Synchronized`
- 可重入锁/非可重入锁：`ReentrantLock` 
- 共享锁/独占锁：`读锁` / `写锁`  
- 公平锁/非公平锁：`是否插队`
- 悲观锁/乐观锁：`独占` / `CAS` 
- 自旋锁/非自旋：`CAS` 
- 可中断锁/不可中断锁：
  - ReentrantLock，是一种典型的可中断锁；`lockInterruptibly` 方法在获取锁的过程中，突然不想获取了，那么也可以在中断之后去做其他的事情，不需要一直傻等到获取到锁才离开
  - synchronized 关键字修饰的锁代表的是不可中断锁，一旦线程申请了锁，`就没有回头路了`，只能等到拿到锁以后才能进行其他的逻辑处理











### 竞争条件

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/CompeteContition.png" style="zoom:60%;" />

- 上面的程序执行了两次i++，但`最终i的值为 1`；把上面的程序想象成两个自动提款机，***<u>用户的余额就可能会被算错</u>***

- `i++` 这段程序访问了**共享资源 i**，这种访问共享资源的程序片段我们称为**<u>*临界区*</u>**

- 在临界区，*<u>**程序片段会访问共享资源**</u>*，`造成竞争条件`，也就是`共享资源的值最终取决于程序执行的时序`，因此这个值***不是确定的***



**两类解决方案：**

- 一种方案就是**避免程序同时进入临界区**，这个方案叫作`互斥`

  - 核心就是给每个线程一个变量i，比如利用 ThreadLocal，这样线程之间就不存在竞争关系了

  - Monitor对象 实现了生产者、消费者模型；提供了互斥，还提供了线程间的通信，避免了使用自旋锁，还简化了程序设计。

    

- 还有一些方案旨在**避免竞争条件**，比如 ThreadLocal、 cas 指令以及乐观锁。

















### Synchronized





#### monitor

synchronized 属于独占式悲观锁，是通过` JVM 隐式实现`的，synchronized 只允许同一时刻只有一个线程操作资源



> 在 Java 中每个对象都隐式包含一个 monitor（监视器）对象



加锁的过程，其实就是竞争 monitor 的过程：

- 当线程进入字节码 monitorenter 指令之后，线程将持有 monitor 对象，执行 monitorexit 时释放 monitor 对象 
- 当其他线程没有拿到 monitor 对象时，则需要阻塞等待获取该对象



获取和释放 monitor 锁的时机：

- 每个 Java 对象都可以用作一个实现同步的锁，这个锁也被称为内置锁或 monitor 锁
- 获得 monitor 锁的唯一途径就是进入由这个锁保护的同步代码块或同步方法
- 线程在进入被 synchronized 保护的代码块`之前`，会自动获取锁，并且***无论是正常路径退出***，还是通过***抛出异常退出***，在退出的时候都会`自动释放锁` 



解析`synchronized代码块`：使用monitorenter和monitorexit实现的

```java
public class SynTest {
    public void synBlock() {
        synchronized (this) {
            System.out.println("lagou");
        }
    }
}
```

```java
public void synBlock();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2       // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3       // String lagou
         9: invokevirtual #4       // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
```

- monitorenter
  - 执行 monitorenter 的线程尝试获得 monitor 的所有权，会发生以下这三种情况之一：
  - 如果该 monitor 的计数为 0，则线程获得该 monitor 并将其计数设置为 1。然后，该线程就是这个 monitor 的所有者
  - 如果线程已经拥有了这个 monitor ，则它将重新进入，并且累加计数
  - 如果其他线程已经拥有了这个 monitor，那个这个线程就会被阻塞，直到这个 monitor 的计数变成为 0，代表这个 monitor 已经被释放了，于是当前这个线程就会再次尝试获取这个 monitor
- monitorexit
  - monitorexit 的作用是将 monitor 的计数器减 1，直到减为 0 为止
  - 代表这个 monitor 已经被释放了，已经没有任何线程拥有它了，也就代表着解锁
  - 所以，其他正在等待这个 monitor 的线程，此时便可以再次尝试获取这个 monitor 的所有权



解析`synchronized方法`：使用falgs实现的

```java
public synchronized void synMethod() {
    
}
```

```java
public synchronized void synMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 16: 0
```

- 被 synchronized 修饰的方法会有一个 ACC_SYNCHRONIZED 标志，表示这是一个同步方法 







#### 用户态与内核态



JDK早期，synchronized叫做重量级锁，因为申请资源必须通过**Kernel**，系统(OS)调用



linux进程有4GB地址空间，如图所示：

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/StatusInuserOrKernel.png" style="zoom:80%;" />

[详细文档URL](https://blog.csdn.net/qq_39823627/article/details/78736650) 

- 3G-4G大部分是共享的，是内核态的地址空间。这里存放**整个内核的代码**和**所有的内核模块**以及**内核所维护的数据**。



进程：一个应用程序启动后会在内存中创建一个执行副本，这就是进程。

内核进程：Linux 的内核是一个 Monolithic Kernel（宏内核），可以看作一个进程。也就是开机的时候，磁盘的**内核镜像**被导入内存作为一个执行副本，成为内核进程。

> 用户态进程通常是应用程序的副本，内核态进程就是内核本身的进程。



用户态：当一个进程在执行**用户自己的代码**时处于用户运行态

内核态：当一个进程因为系统调用**陷入内核代码**中执行时处于内核运行态

> 用户态和内核态的切换：
>
> 当在系统中执行一个程序时，大部分时间是运行在用户态下的，在其需要操作系统帮助完成一些用户态自己没有特权和能力完成的操作时就会切换到内核态。



线程：在现代操作系统中，执行程序的单位

程序：就是存储在内存中的指令集合





#### 对象在内存中的存储布局

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ObjectInMemory.png" style="zoom:80%;" />

- markWord，8个字节(Byte)，64位(Bit)
- 类型指针，该对象**属于哪个类的** 
- 实例数据，对象的成员属性/方法 【int m;  这里就有四个字节】
- 对齐，如果一个对象前面三块合起来 不能被8字节整除的话，**补齐** 【JVM读数据是按照**块**(8字节)来读的】











#### markWord

- 锁信息存在了markword 里面，`加锁`就是修改markword

- `锁升级的过程`，只需要观察markword 的变化

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/LockStatus.png" style="zoom:67%;" />

**无锁态：** **hashCode**，叫 identityHashCode，当有人调这个hashCode才会往里面装，否则没有值

**分代年龄：** 一次GC时，对象没被回收，年龄+1，年龄就记在这儿，**4位最高就是15**，不能再往上调

**Epoch：** 批量撤销

**Lock Record：** 锁记录，`记录锁了多少次`。每个线程想上自旋锁时，线程栈都会生成Lock Record，多线程的竞争往markWord写的就是该Lock Record的指针，【谁能把markWord写上自己栈中Lock Record的指针，该锁就属于谁】

**重量级锁：** C++层面生成一对象，ObjectMonitor()

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ObjectMonitor.png)



上锁过程：

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/monitorObject.png)

- Monitor（监视器锁）本质是依赖于底层的操作系统的 Mutex Lock（互斥锁）来实现的
- Mutex Lock 的切换需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间
- 所以 synchronized 是 Java 语言中的一个重量级操作









#### 锁升级



![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/UpgradeLock.png)

> 偏向锁和轻量级锁只需要在**用户空间**就可以完成

- `偏向锁`：**偏向**于某个线程，将当前的线程ID记录在markword上，不需要向OS申请资源(不再进行任何同步操作，如 Locking、Unlocking 等)，直到另一个线程尝试获取此锁的时候，偏向锁模式才会结束，偏向锁可以提高带有同步但无竞争的程序性能。
  - 匿名偏向：刚new完，还没有任何人持有这把锁
- `自旋锁`：多个线程竞争，**谁先**成功修改markword（使用CAS锁总线），谁就持有这边锁，把自己的信息记录到markWord上；对比的是线程和对象的 MarkWord，如果更新成功则表示当前线程成功拥有此锁；如果失败，虚拟机会先检查对象的 MarkWord 是否指向当前线程的栈帧，如果是，则说明当前线程已经拥有此锁，否则，则说明此锁已经被其他线程占用了
- `重量级锁`：通过操作系统的互斥量（mutex lock）来实现的，这种实现方式需要在用户态和核心态之间做转换，有很大的性能消耗；经过OS的调度后，如果OS提供给你一把锁，并且这把锁会提供 这把锁上的一些**队列**(waitSet)，没有抢到锁的线程都进入**等待队列**，从而不占用CPU资源



> 轻量级锁 什么时候升级为重量级锁？？
>
> JDK 1.6之前，情况1：自旋**次数**超过10次 ; 情况二：等待自旋的线程**超过**CPU核数的二分之一
>
> 1.6后的优化，[自适应自旋](#自适应自旋锁)，再也不用调上面的两个参数了



**偏向锁：**

```shell
-XX:BiasedLockingStartupDelay=40000     # 偏向锁默认打开, 时延4秒
```

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/BiasedLock.png" style="zoom:67%;" />

**why? 时延4秒** 

- 因为JVM虚拟机自己有一些默认启动的线程，里面有好多sync代码
- 这些sync代码启动时就知道**肯定会有竞争**
- 如果使用偏向锁，*就会造成偏向锁不断的进行**锁撤销和锁升级**的操作*，效率较低

总之，等JVM启动完后，再启动偏向锁











#### 自适应自旋锁



JDK 1.6 引入了自适应式自旋锁意味着**自旋的时间**不再是固定的时间了

比如在同一个锁对象上，如果通过自旋等待成功获取了锁，那么虚拟机就会认为，它下一次很有可能也会成功 (通过自旋获取到锁)，因此允许自旋等待的时间会==相对的比较长==

而当某个锁通过自旋**很少成功**获得过锁，那么以后在获取该锁时，可能会直接忽略掉自旋的过程，以避免浪费 CPU 的资源





#### 锁消除



锁粗化功能是默认打开的，用` -XX:-EliminateLocks` 可以关闭该功能



```java
public class Person {
    private String name;
    private int age;
    public Person(String personName, int personAge) {
        name = personName;
        age = personAge;
    }
    public Person(Person p) {
        this(p.getName(), p.getAge());
    }

    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }
}

class Employee {
    private Person person;
    // makes a defensive copy to protect against modifications by caller
    public Person getPerson() {
        return new Person(person);
    }
    public void printEmployeeDetail(Employee emp) {
        Person person = emp.getPerson();
        // this caller does not modify the object, so defensive copy was unnecessary
       System.out.println("Employee's name: " + person.getName() + "; age: " + person.getAge());
    }
}
```

- 如果编译器可以确定最开始的 person 对象`不会被修改的话`，它可能会**<u>*优化并且消除这个新建  person 的过程*</u>**



==根据这样的思想==：

​	经过逃逸分析之后，如果发现某些对象不可能被其他线程访问到，那么就可以把它们`当成栈上数据`，<u>栈上数据由于只有本线程可以访问</u>，自然是线程安全的，也就`无需加锁`，所以会把这样的锁给自动去除掉。



#### 锁粗化

```java
public void lockCoarsening() {
    synchronized (this) {
        //do something
    }
    synchronized (this) {
        //do something
    }
    synchronized (this) {
        //do something
    }
}
```

- 这种释放和重新获取锁是完全没有必要的
- 如果我们把同步区域扩大，也就是只在最开始加一次锁，并且在最后直接解锁，那么就可以把中间这些无意义的解锁和加锁的过程消除，相当于是**<u>*把几个 synchronized 块合并为一个较大的同步块*</u>**
- 这样做的好处在于在线程执行这些代码时，就`无须频繁申请与释放锁`了，这样就减少了性能开销



```java
for (int i = 0; i < 1000; i++) {
    synchronized (this) {
        //do something
    }
}
```

- 锁粗化不适用于循环的场景，仅适用于非循环的场景







#### Synchronized / ReentrantLock



##### 相同点

1. synchronized 和 Lock 都是用来保护资源线程安全的
2. 都可以保证可见性
3. synchronized 和 ReentrantLock 都拥有`可重入`的特点

Synchronized / ReentrantLock 默认是 `独占的非公平的可重入锁`



##### 不同点

- 用法不同
  - synchronized 关键字可以加在方法上，不需要指定锁对象（此时的锁对象为 this），也可以新建一个同步代码块并且自定义 monitor 锁对象
  -  Lock 接口必须显示用 Lock 锁对象开始加锁 lock() 和解锁 unlock()，并且一般会在 finally 块中确保用 unlock() 来解锁，以防发生死锁
  - 与 Lock 显式的加锁和解锁不同的是 synchronized 的加解锁是隐式的
- 加解锁顺序不同
  - Lock 可以不完全按照加锁的反序解锁
  - synchronized 解锁的顺序和加锁的顺序必须完全相反
- synchronized 锁不够灵活
  - 一旦 synchronized 锁已经被某个线程获得了，此时其他线程如果还想获得，那它`只能被阻塞`，直到持有锁的线程运行完毕或者发生异常从而释放这个锁
  - Lock 类在等锁的过程中，如果使用的是 `lockInterruptibly` 方法，那么如果觉得等待的时间太长了不想再继续等待，***可以中断退出***，也可以用 tryLock() 等方法尝试获取锁，***<u>如果获取不到锁也可以做别的事</u>***，更加灵活

- synchronized 锁只能同时被一个线程拥有，但是 Lock 锁没有这个限制
  - 在读写锁中的读锁，是可以同时被多个线程持有的，可是 synchronized 做不到
- 原理区别
  - synchronized 是内置锁，由 JVM 实现获取锁和释放锁的原理，还分为偏向锁、轻量级锁、重量级锁
  - Lock 根据实现不同，有不同的原理，例如 ReentrantLock 内部是通过 AQS 来获取和释放锁的
- 是否可以设置公平/非公平
  - ReentrantLock 等 Lock 实现类可以根据自己的需要来设置公平或非公平，synchronized 则不能设置
- 性能区别
  - 在 Java 5 以及之前，synchronized 的性能比较低，但是到了 Java 6 以后，发生了变化，因为 JDK 对 synchronized 进行了很多优化，比如自适应自旋、锁消除、锁粗化、轻量级锁、偏向锁等









[两者间的重要区别02:03](https://www.bilibili.com/video/BV12Q4y1N75Q?p=7)：

- Synchronized 的notifyAll()浪费资源 ；有且仅有一个waitSet队列，生产者和消费者都放在同个队列
- ReentrantLock 中的Condition是**队列**；有多少个condition就有多少个队列，生产者和消费者可以放在不同的队列





### 唤醒与等待

- LockSupport下的park 和 unpark 使用更方便，之前醒着的线程也可以使用unpark 
- synchronized，wait，notify：调用一个对象的wait()，不是让对象wait，**而是让持有这个对象锁的线程去wait**，该线程让出锁
- lock中condition的 signal() 和 await()
- TransferQueue 阻塞的容量为0的队列







### 可重入锁（递归锁）

`公寓大门`的锁 和`厕所门栓`上的锁，**<u>当获得大门的锁</u>**，可以使用厕所门栓

Synchronized：

```java
class Phone {
    public synchronizd void sendSMS() throws Exception {
        System.out.println(Thread.currentThread().getName() + "\t invoked sendSMS()");
        sendEmail();
    }
    public synchronizd void sendEmail () throws Exception {
        System.out.println(Thread.currentThread().getName() + "\t invoked sendEmail()");
    }
}
```



ReentrantLock：

```java
public  void get() {
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName() + "\t invoked get()");
        set();
    } finally {
        lock.unlock();
    }
}
public void set() {
    try {
        System.out.println(Thread.currentThread().getName() + "\t invoked set()");
    } finally {
        lock.unlock();
    }
}
```

- 可重入锁的最大作用是 `避免死锁`
- 线程可以进入它已经拥有锁的方法，如果该方法内部`还有其他`**锁方法**，***<u>不用获取其他锁也可以进入</u>***



```java
public  void get() {
    lock.lock();
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName() + "\t invoked get()");
        set();
    } finally {
        lock.unlock();
        lock.unlock();
    }
}
```

> 只要lock和unlock两两配对，`加多少`把都可以；若没有配对，将会卡死，出不来







### 悲观锁/乐观锁

- 悲观锁：synchronized 关键字和 Lock 接口

  - Java 中悲观锁的实现包括 synchronized 关键字和 Lock 相关类等
  - 以 Lock 接口为例，例如 Lock 的实现类 ReentrantLock，类中的 lock() 等方法就是执行加锁，而 unlock() 方法是执行解锁。处理资源之前必须要先加锁并拿到锁，等到处理完了之后再解开锁，这就是非常典型的悲观锁思想。

- 乐观锁：原子类

  - 乐观锁的典型案例就是原子类，例如 AtomicInteger 在更新数据时，就使用了乐观锁的思想，多个线程可以同时操作同一个原子变量。

- 大喜大悲：数据库

  - 数据库中同时拥有悲观锁和乐观锁的思想

  - 例如，我们如果在 MySQL 选择 `select for update` 语句，那就是悲观锁，在提交之前不允许第三方来修改该数据，这当然会造成一定的性能损耗，在高并发的情况下是不可取的

  - 使用版本 version 字段在数据库中实现乐观锁

    ```mysql
    UPDATE student
        SET 
            name = ‘小李’,
            version= 2
        WHERE   id= 100
            AND version= 1
    ```



悲观锁适合用于并发写入多、临界区代码复杂、竞争激烈等场景，这种场景下悲观锁可以避免大量的无用的反复尝试等消耗。

乐观锁适用于大部分是读取，少部分是修改的场景，也适合虽然读写都很多，但是并发并不激烈的场景。在这些场景下，乐观锁不加锁的特点能让性能大幅提高。











### 自旋锁

==Unsafe类（像C的指针） + CAS思想（CPU并发源语）= 自旋== 



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/SpinAndUnspin.png" style="zoom:57%;" />

- 阻塞和唤醒线程都是需要`高昂的开销`
- 如果同步代码块中的内容不复杂，那么可能转换线程带来的开销`比`实际业务代码执行的开销还要大
- 自旋锁用循环去不停地尝试获取锁，`让线程始终处于 Runnable 状态`，***<u>节省了线程状态切换带来的开销</u>***



自旋锁（spinlock）：指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程`上下文切换`的消耗，缺点是`循环会消耗CPU` 

```java
// unsafe.getAndAddInt
// var1 当前对象本身
// var2 当前对象值的引用地址
// var4 需要变动的数量
// var5 通过var1和var2找出的 当前主内存中的值
public final int getAndAddInt (Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while (this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

- 好处：不会立即阻塞
- 坏处：陷入死循环，严重消耗CPU，性能下降



手写一个自旋锁：

```java
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.locks.Lock;
 
/**
 * 描述：     实现一个可重入的自旋锁
 */
public class ReentrantSpinLock  {
 
    private AtomicReference<Thread> owner = new AtomicReference<>();
 
    //重入次数
    private int count = 0;
 
    public void lock() {
        Thread t = Thread.currentThread();
        if (t == owner.get()) {
            ++count;
            return;
        }
        //自旋获取锁
        while (!owner.compareAndSet(null, t)) {
            System.out.println("自旋了");
        }
    }
 
    public void unlock() {
        Thread t = Thread.currentThread();
        //只有持有锁的线程才能解锁
        if (t == owner.get()) {
            if (count > 0) {
                --count;
            } else {
                //此处无需CAS操作，因为没有竞争，因为只有线程持有者才能解锁
                owner.set(null);
            }
        }
    }
 
    public static void main(String[] args) {
        ReentrantSpinLock spinLock = new ReentrantSpinLock();
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                spinLock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + "获取到了自旋锁");
                    Thread.sleep(4000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    spinLock.unlock();
                    System.out.println(Thread.currentThread().getName() + "释放了了自旋锁");
                }
            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
    }
}
```







### 读写锁(ReadWriteLock)

- 独占锁：指该锁一次只能被`一个`线程所持有；对ReentrantLock和Synchronized而言都是独占锁

- 共享锁：该锁可以被`多个`线程所持有
    - 读-读 能共存
    - 读-写 不能共存
    - 写-写 不能共存
>写操作：原子+独占，整个过程必须是一个完整的统一体，中间不许被分割、被打断



```java
public class CachedData {

    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

    void processCachedData() {
        rwl.readLock().lock();
        
        if (!cacheValid) {
            
            //在获取写锁之前，必须首先释放读锁。
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                //这里需要再次判断数据的有效性,因为在我们释放读锁和获取写锁的空隙之内，可能有其他线程修改了数据。
                if (!cacheValid) {
                    data = new Object();
                    cacheValid = true;
                }
                //在不释放写锁的情况下，直接获取读锁，这就是读写锁的降级。
                rwl.readLock().lock();
            } finally {
                //释放了写锁，但是依然持有读锁
                rwl.writeLock().unlock();
            }
        }
        try {
            System.out.println(data);
        } finally {
            //释放读锁
            rwl.readLock().unlock();
        }
    }
}

```



对于 ReentrantReadWriteLock 而言

- `插队策略`
  
  - 公平策略下，只要队列里有线程已经在排队，就不允许插队
  - 非公平策略下：
    - 如果允许读锁插队，那么由于读锁可以同时被多个线程持有，所以可能造成源源不断的后面的线程一直插队成功，导致读锁一直不能完全释放，从而导致写锁一直等待，为了防止“饥饿”，**<u>*在等待队列的头结点是尝试获取写锁的线程的时候，不允许读锁插队*</u>**
    - `写锁可以随时插队`，因为写锁并不容易插队成功，写锁只有在当前没有任何其他线程持有读锁和写锁的时候，才能插队成功，同时写锁一旦插队失败就会进入等待队列，所以很难造成“饥饿”的情况，允许写锁插队是为了提高效率。
  
  
  
- `升降级策略`：只能从写锁降级为读锁，不能从读锁升级为写锁



```java
/**
 * 描述：     演示读写锁用法
 */
public class ReadWriteLockDemo {

    private static final ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(
            false);
    private static final ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock
            .readLock();
    private static final ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock
            .writeLock();

    private static void read() {
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void write() {
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> read()).start();
        new Thread(() -> read()).start();
        new Thread(() -> write()).start();
        new Thread(() -> write()).start();
    }
}
// Thread-0得到读锁，正在读取
// Thread-1得到读锁，正在读取
// Thread-0释放读锁
// Thread-1释放读锁
// Thread-2得到写锁，正在写入
// Thread-2释放写锁
// Thread-3得到写锁，正在写入
// Thread-3释放写锁
```



写锁降级为读锁：

```java
public class CachedData {
 
    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 
    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            //在获取写锁之前，必须首先释放读锁。
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                //这里需要再次判断数据的有效性,因为在我们释放读锁和获取写锁的空隙之内，可能有其他线程修改了数据。
                if (!cacheValid) {
                    data = new Object();
                    cacheValid = true;
                }
                //在不释放写锁的情况下，直接获取读锁，这就是读写锁的降级。
                rwl.readLock().lock();
            } finally {
                //释放了写锁，但是依然持有读锁
                rwl.writeLock().unlock();
            }
        }
         try {
            System.out.println(data);
        } finally {
            //释放读锁
            rwl.readLock().unlock();
        }
    }
}
```



读锁不支持升级为写锁：

```java
final static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 
public static void main(String[] args) {
    upgrade();
}
 
public static void upgrade() {
    rwl.readLock().lock();
    System.out.println("获取到了读锁");
    rwl.writeLock().lock();
    System.out.println("成功升级");
}
```









### 公平锁和非公平锁

* FairLock：指多个线程按照 申请锁的时间顺序 来依次获取锁，类似队列，**先来先得** ==》避免死锁
* NonfairSync：指多个线程 `不`按照 申请锁的时间顺序来依次获取锁，**<u>*在合适的时机插队*</u>** 

  * 有可能后申请的线程比先申请的线程优先获取锁，一上来就抢 抢不到锁再排到队尾
  * *在高并发的情况下*，非公平锁有可能会造成**优先级反转**或者**饥饿现象** (按公平排的最后一位一直没有得到锁）
  * 合适的时机：
    * 假设当前线程在请求获取锁的时候，恰巧前一个持有锁的线程释放了这把锁，那么当前申请锁的线程就可以不顾已经等待的线程而选择立刻插队
    * 但如果当前线程请求的时候，前一个线程并没有在那一时刻释放锁，那么当前线程还是一样会进入等待队列

> 唤醒队列头排队的线程是需要很大开销的，很有可能在唤醒之前，新来排队的线程(没有进入等待队列) 已经拿到了这把锁并且执行完任务释放了这把锁
>
> **挂起**和**恢复**都存在一定的开销





```java
// 公平锁
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() && // 公平锁和非公平锁的核心区别
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

```java
// 非公平锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 无论是否有线程在排队了，都回尝试去获取锁；如果获取不到，再去排队
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```



|          |                          优势                          |                        劣势                        |
| :------: | :----------------------------------------------------: | :------------------------------------------------: |
|  公平锁  | 各线程公平平等，每个线程在等待一段时间后，总有机会执行 |                  更慢，吞吐量更小                  |
| 非公平锁 |                    更快，吞吐量更大                    | 有可能产生线程饥饿，某些线程长时间内始终得不到执行 |



```java
/**
 * 描述：演示公平锁，分别展示公平和不公平的情况，非公平锁会让现在持有锁的线程优先再次获取到锁。代码借鉴自Java并发编程实战手册2.7。
 */
public class FairAndUnfair {
    public static void main(String args[]) {
        PrintQueue printQueue = new PrintQueue();
        Thread thread[] = new Thread[10];
        for (int i = 0; i < 10; i++) {
            thread[i] = new Thread(new Job(printQueue), "Thread " + i);
        }
        for (int i = 0; i < 10; i++) {
            thread[i].start();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Job implements Runnable {
    private PrintQueue printQueue;
    public Job(PrintQueue printQueue) {
        this.printQueue = printQueue;
    }
    @Override
    public void run() {
        System.out.printf("%s: Going to print a job\n", Thread.currentThread().getName());
        printQueue.printJob(new Object());
        System.out.printf("%s: The document has been printed\n", Thread.currentThread().getName());
    }
}

class PrintQueue {
    private final Lock queueLock = new ReentrantLock(false);
    public void printJob(Object document) {
        queueLock.lock();
        try {
            Long duration = (long) (Math.random() * 10000);
            System.out.printf("%s: PrintQueue: Printing a Job during %d seconds\n",
                    Thread.currentThread().getName(), (duration / 1000));
            Thread.sleep(duration);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
        }

        queueLock.lock();
        try {
            Long duration = (long) (Math.random() * 10000);
            System.out.printf("%s: PrintQueue: Printing a Job during %d seconds\n",
                    Thread.currentThread().getName(), (duration / 1000));
            Thread.sleep(duration);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
            }
    }
}
```







## JUC - AQS



java.util.concurrent (JUC) 大大提高了并发性能，`AQS(AbstractQueuedSynchronizer)` 被认为是 JUC 的核心



### Lock

当使用 synchronized 不合适或不足以满足要求的时候，Lock 可以用来提供`更高级功能` 

```java
package java.util.concurrent.locks;
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

```java
Lock lock = ...;
lock.lock();
try{
    //获取到了被本锁保护的资源，处理任务
    //捕获异常
}finally{
    lock.unlock();   //释放锁
}
```

lock() 方法不能被中断，一旦陷入死锁，`lock()` 就会陷入永久等待，所以一般用 `tryLock()` 等方法来代替 lock()

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则做其他事情
}
```

```java
// 避免死锁的方法
public void tryLock(Lock lock1, Lock lock2) throws InterruptedException {
    while (true) {
        if (lock1.tryLock()) {
            try {
                if (lock2.tryLock()) {
                    try {
                        System.out.println("获取到了两把锁，完成业务逻辑");
                        return;
                    } finally {
                        lock2.unlock();
                    }
                }
            } finally {
                lock1.unlock();
            }
        } else {
            Thread.sleep(new Random().nextInt(1000));
        }
    }
}
```



`lockInterruptibly()`：

- 这个方法的作用就是去获取锁，如果这个锁当前是可以获得的，那么这个方法会立刻返回，但是如果这个锁当前是不能获得的（被其他线程持有），那么当前线程便会开始等待，除非它等到了这把锁或者是在等待的过程中被中断了，否则这个线程便会一直在这里执行这行代码。
- 一句话总结就是，**<u>*除非当前线程在获取锁期间被中断，否则便会一直尝试获取直到获取到为止*</u>** 
- 可以把这个方法理解为`超时时间是无穷长的 tryLock(long time, TimeUnit unit)`，因为 tryLock(long time, TimeUnit unit) 和 lockInterruptibly() 都能响应中断，只不过 lockInterruptibly() 永远不会超时

```java
public void lockInterruptibly() {
    try {
        lock.lockInterruptibly();
        try {
            System.out.println("操作资源");
        } finally {
            lock.unlock();
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```









### ReentrantLock



**ReentrantLock中Lock()的非公平实现：** 

```java
final void lock() {

    if (compareAndSetState(0, 1))
        // 将当前线程设置为此锁的持有者
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

 非公平的抢占逻辑：

1. **某线程**  <u>*在发送请求的同时*</u>  该锁的状态恰好变成了可用，那么此线程就可以跳过队列中所有排队的线程直接拥有该锁

2. tryAcquire 方法中 尝试获取锁，如果获取锁失败，则把它加入到阻塞队列中 

3. acquireQueued，阻塞队列中的线程尝试获取锁，失败则会被挂起 

第二、三步的流程图：

<img src="JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91.assets/1605412835121.png" alt="1605412835121" style="zoom:67%;" />





**ReentrantLock中Lock()的公平实现：** 

```java
final void lock() {

    acquire(1);
}
```

> 可以看出非公平锁比公平锁只是多了一行 compareAndSetState 方法：
>
> 该方法是尝试将 state 值由 0 置换为 1，如果设置成功的话，则说明当前没有其他线程持有该锁，不用再去排队了，可直接占用该锁；
>
> 否则，则需要通过 acquire 方法去排队。 











### CountDownLatch

简单描述：火箭发射**倒计时**

班长锁门case：
<img src="JUC多线程及高并发.assets/20200119092129757_7966.png" style="zoom:67%;" />



### CyclicBarrier

简单描述：集齐⑦龙珠，召唤神龙

（Cyclic) 可**循环**使用的 (Barrier)**屏障**：直到最后一个线程到达屏障时，才会打开屏障，所有被拦截的线程才能继续干活

> 线程进入屏障通过  `CyclicBarrier.await();`





### Semaphore

==可伸缩，可变化，多个线程抢占多个资源==

> 多个共享资源间的 互斥使用，并发线程数的控制
>
> 加锁有四种：synchronized / Lock / ReentrantLock / Semaphore

简单描述：`争车位`
<img src="JUC多线程及高并发.assets/20200119104651006_619.png" style="zoom:60%;" />






## 阻塞队列 (BlockingQueue)



### 原理

阻塞队列，首先它是一个`队列`，满足`先进先出(FIFO)`；其次它是`阻塞`的，通过`ReentrantLock`来实现

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/BlockingQueue.png" style="zoom:60%;" />

- 当阻塞队列为**空**时，Thread2从队列中获取元素的操作将会被阻塞
- 当阻塞队列为**满**时，Thread1往队列里添加元素的操作将会被阻塞



### 为什么用？有什么好处？

1. 为什么使用阻塞队列？？

   ​	不得不堵塞



2. 堵塞队列有没有好的一面？？    

   ​	**好处是：**我们`不再关心` 什么时候需要阻塞(**wait**)线程，什么时候需要唤醒(**notify**)线程，这一切 BlockingQueue 都一手包办了

   

3. 不得不堵塞，如何管理？？

   ？？

> 在多线程领域，所谓阻塞：
>
> 在**某些情况下**会`挂起`线程（即阻塞）,一旦条件满足，被挂起的线程又会自动`被唤醒`



### BlockingQueue架构



<img src="JUC多线程及高并发.assets/20200119140316541_14718.png" style="zoom:67%;" />

 <img src="JUC多线程及高并发.assets/20200119135759467_6213.png" style="zoom:50%;" />



SynchronousQueue

> SynchronousQueue： `定制版`，有且仅有一个，`0库存`，没有容量的阻塞队列
>
> 与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue
>
> 每个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然





### BlockingQueue的核心方法
![](JUC多线程及高并发.assets/20200119140920516_312.png )
![](JUC多线程及高并发.assets/20200119142808424_31693.png )

**最好使用：超时方法类型** 
![](JUC多线程及高并发.assets/20200119143120893_23284.png )



|      |      |      |      |      |
| ---- | ---- | ---- | ---- | ---- |
|      |      |      |      |      |
|      |      |      |      |      |
|      |      |      |      |      |











## 生产者消费者模式





#### 传统版: 生产/消费者模式 1.0 和 2.0 



ProdConsumer_Tradition：


**多线程并发**状态下，**高内聚、低耦合**的前提下：

1. 资源类凭什么被操作？？

   ​	因为资源类 `自身高内聚`，带着对资源操作的方法【空调自身带着两个方法：制冷/制热，如下面的`MyResource`】

   

2. 所有java操作？？

   ​	一切`皆`对象，先操作对象，再操作对象里面的变量

   

3. 虚假唤醒？？

  ​	不能用if，用while循环判断，被唤醒起来`要重新`再过一次判断

 



#### 阻塞队列版：生产/消费者模式 3.0



ProdConsumer_BlockQueue：

> 运用到了 volatitl / CAS / AtomicInteger / 原子引用 / 线程交互 / BlockQueue

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 资源类凭什么被操作？？
 * 因为资源类 `自身高内聚`，带着对资源操作的方法
 * 列如：
 *      空调自身带着两个方法：制冷/制热
 * */
class MyResource{
    // 默认开启，进行生产和消费；volatile: 线程间交互，保证可见性
    private volatile boolean FLAG = true;

    // 保证原子性，稳定输出
    private AtomicInteger atomicInteger = new AtomicInteger();

    // 架构程序：通顺、适配和通用，传接口，不能传具体的类 ==> UML聚和
    // 右边为null，必须把口子放得宽，只要实现了BlockingQueue接口的子类都要满足 ==> 多态
    BlockingQueue<String> blockingQueue = null;

    // 1.set/get设值注入 2.构造注入 ---> 传接口，程序就通用了
    // 高手都是传接口 ==> 面向接口编程
    public MyResource(BlockingQueue<String> blockingQueue) {    // 构造注入
        this.blockingQueue = blockingQueue;

        // 明确的知道是谁在干活--->log调试
        // 写，足够的抽象，往高处写
        // 查，往细节落地
        System.out.println(blockingQueue.getClass().getName());
    }

    public void myProd() throws Exception{ // 生产者
        String data = null;
        boolean retValue;
        // 判断
        while (FLAG){
            // ++i变string ---> 阻塞队列放string类
            data = atomicInteger.incrementAndGet() +"";

            // 往阻塞队列里面放数据
            retValue = blockingQueue.offer(data,2L, TimeUnit.SECONDS);
            if (retValue){
                System.out.println(Thread.currentThread().getName() + "\t 插入队列" + data + "成功");
            }else {
                System.out.println(Thread.currentThread().getName() + "\t 插入队列" + data + "失败");
            }
        }
        System.out.println(Thread.currentThread().getName() + "\t 叫停了，falg=false; 生产动作结束");
    }

    public void myConsumer() throws Exception{ // 消费者
        String res = null;
        while (FLAG){
            res = blockingQueue.poll(2L,TimeUnit.SECONDS);  // 2s中取不到 就不取了
            if (null == res || res.equalsIgnoreCase("")){
                FLAG = false;
                System.out.println(Thread.currentThread().getName() + "\t 超过2s没有取到数据，消费退出");

                // 怎么退出 ---> 否则生产已经停了，仍然在等
                return;
            }
            System.out.println(Thread.currentThread().getName() + "\t 消费队列" + res + "成功");
        }
    }

    public  void  stop() throws  Exception{
        this.FLAG = false;
    }
}


/**
 * volatitl/CAS/AtomicInteger/原子引用/线程交互/BlockQueue
 */
public class ProdConsumer_BlockQueueDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource(new ArrayBlockingQueue<>(10));

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 生产线程启动");
            try {
                myResource.myProd();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "Prod").start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 消费线程启动");
            try {
                myResource.myConsumer();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "Consumer").start();

        try { TimeUnit.SECONDS.sleep(5); }catch (InterruptedException e){ e.printStackTrace();}
        System.out.println("5s时间到，调用stop方法，活动结束");
        try {
            myResource.stop();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```





1. 



## 线程池

线程池模型：一堆线程 + 一个队列



### 好处

1. 线程池可以解决线程生命周期的系统开销问题，同时还可以加快响应速度。因为线程池中的线程是可以复用的，我们只用少量的线程去执行大量的任务，这就大大减小了线程生命周期的开销。而且线程通常不是等接到任务后再临时创建，而是已经创建好时刻准备执行任务，这样就消除了线程创建所带来的延迟，提升了响应速度，增强了用户体验
2. 线程池可以统筹内存和 CPU 的使用，避免资源使用不当。线程池会根据配置和任务数量灵活地控制线程数量，不够的时候就创建，太多的时候就回收，避免线程过多导致内存溢出，或线程太少导致 CPU 资源浪费，达到了一个完美的平衡
3. 线程池可以统一管理资源。比如线程池可以统一管理任务队列和线程，可以统一开始或结束任务，比单个线程逐一处理任务要更方便、更易于管理，同时也有利于数据统计，比如我们可以很方便地统计出已经执行过的任务的数量。







### 线程池架构

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ThreadArchiteture.png" style="zoom:67%;" />

- 底层：ThreadPoolExecutor





### 常见线程池

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/OftenThreadPool.png" style="zoom:57%;" />



### ScheduledThreadPool

> 周期性任务的线程池：ScheduledThreadPoolExecutor

```java
// Executors.class
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

```java
// ScheduledExecutorService.class
public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);
public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);
// 这两个方法，对fixed的时间定义不一样：
// scheduleAtFixedRate 是以任务开始的时间为时间起点开始计时，时间到就开始执行，第二次任务，而不管任务需要花多久执行
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
// cheduleWithFixedDelay 方法以任务结束的时间为下一次循环的时间起点开始计时
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
```





> SpringBoot中的ThreadPoolTaskScheduler，在其基础上封装， 可以很方便的对重复执行的任务进行调度管理 

 [ThreadPoolTaskScheduler介绍](https://www.cnblogs.com/toiletgg/p/10647436.html)  







### Executor存在的问题

- **CachedThreadPool**：`带缓存，可扩容`，一池多线程，短期异步的小程序 或者负载较轻的服务；
  
    ```java
    public static ExecutorService newCachedThreadPool() {
            return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                          60L, TimeUnit.SECONDS,
                                          new SynchronousQueue<Runnable>());
        }
    ```
    
    也就是说，来任务了就创建线程；当线程空闲超过60秒，就销毁线程



- **FixedThreadPool**：一池`固定`线程，执行长期任务，性能好很多；
  
    ```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
            return new ThreadPoolExecutor(nThreads, nThreads,
                                          0L, TimeUnit.MILLISECONDS,
                                          new LinkedBlockingQueue<Runnable>());
        }
    ```
    
    



- **SingleThreadExecutor**：相当于大小为 `1` 的 FixedThreadPool，一池单线程，一个任务一个任务的执行
  
    ```java
    public static ExecutorService newSingleThreadExecutor() {
            return new FinalizableDelegatedExecutorService
                (new ThreadPoolExecutor(1, 1,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>()));
        }
    ```
    
    

然而实际生产中，一个都不用，`自己自定义`

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ThreadPoolExecutor.png" style="zoom:60%;" />





### 线程池7大参数



```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

1. corePoolSize：线程池中的`常驻`核心线程数
    - 近似理解为`今日当值`线程
    - 当线程数目达到 corePoolSize后，就会把到达的任务（需要线程）放`缓存队列`当中
2. maximumPoolSize：线程池*能够容纳* **同时执行**的`最大线程数`，此值必须 >=1
    - 当corePoolSize满了 && 阻塞队列也满了 ---> 线程池开始`扩容`（加班窗口）
3. keepAliveTime：多余的空闲线程的存活时间（当线程数大于corePoolSize才起作用）
    1. 当前线程池数量超过 corePoolSize时 && 空闲时间达到 keepAliveTime值时
    2. 多余空闲线程会被销毁直到**只剩下  corePoolSize个**线程为止
4. unit：keepAliveTime的单位
5. workQueue：`任务队列`，**被提交但尚未被执行**的任务
    - 类似银行里面的候客区
6. threadFactory：表示**生成**线程池中**工作线程**的*线程工厂* 
    - 用于创建线程，一般用默认的即可
    - 类似于各个银行网点的**标准**
7. handler：拒绝策略
    - 当 阻塞队列 满了 && （工作线程>=线程池的max线程数） ==> 启动线程的拒绝



### 底层工作原理图

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ThreadPoolTheory.png" style="zoom:67%;" />

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ThreadPoolExecutorSteps.png)





### 自定义线程池

 [为什么不应该自动创建线程池？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=16#/detail/pc?id=252) 



```java
ExecutorService threadPool = new ThreadPoolExecutor(
    2, // corePoolSize
    5, // maximumPoolSize
    2L, // keepAliveTime
    TimeUnit.SECONDS, // unit
    new LinkedBlockingQueue<>(3), // workQueue
    Executors.defaultThreadFactory(), // threadFactory
    new ThreadPoolExecutor.AbortPolicy()); // handler

```



**合理配置线程池，你是如何考虑的？？**



1. 首先获得系统的CPU核数：

```java
Runtime.getRuntime().availableProcessors()
```

2. 区分类型
    1. CPU密集型：计算得多

     最佳的线程数为 CPU 核心数的 1~2 倍

     

   2. IO密集型：等待得多

      线程数 = CPU 核心数 *（1+平均等待时间/平均工作时间）



> 可以进行压测，**监控 JVM 的线程**情况以及 **CPU 的负载**情况，根据实际情况衡量应该创建的线程数，合理并充分利用资源。





###  ForkJoinPool

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ForkJoinPool.png" style="zoom:47%;" />

有一个 Task，这个 Task 可以产生三个子任务，三个子任务**并行执行**完毕后**将结果汇总**给 Result

```java
class Fibonacci extends RecursiveTask<Integer> { 
 
    int n;
 
    public Fibonacci(int n) { 
        this.n = n;
    } 
 
    @Override
    public Integer compute() { 
        if (n <= 1) { 
            return n;
        } 
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork();
        Fibonacci f2 = new Fibonacci(n - 2);
        f2.fork();
        return f1.join() + f2.join();
    } 
    
    public static void main(String[] args) throws ExecutionException, InterruptedException { 
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        for (int i = 0; i < 10; i++) { 
            ForkJoinTask task = forkJoinPool.submit(new Fibonacci(i));
            System.out.println(task.get());
        } 
	}
 }

```





ForkJoinPool 线程池中每个线程都有自己`独立的任务队列 `

除此之外，每个线程还有一个对应的双端队列 `deque`，这时一旦线程中的任务被 Fork 分裂了，分裂出来的子任务放入线程自己的 deque 里，而不是放入公共的任务队列中

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ForkJoinPoolStruct.png" style="zoom:40%;" />



`work-stealing`：

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ForkResult.png" style="zoom:50%;" />

- 程 t1 获取任务的逻辑是后进先出（先进后出），也就是LIFO（Last In Frist Out）
- 线程 t0 在“steal”偷线程 t1 的 deque 中的任务的逻辑是先进先出，也就是FIFO（Fast In Frist Out）





### 线程复用

原理：

- 让每个线程去执行一个“循环任务”，在这个“循环任务”中，不停地检查是否还有任务等待被执行
- 如果有则直接去执行这个任务，也就是调用任务的 run 方法，把 run 方法当作和普通方法一样的地位去调用
- 相当于把每个任务的 run() 方法串联了起来，所以线程数量并不增加



```java
// ThreadPoolExecutor.class
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    // 判断当前线程数是否小于核心线程数
    if (workerCountOf(c) < corePoolSize) {
        // addWorker 方法的主要作用是在线程池中创建一个线程并执行第一个参数传入的任务
        // 第二个参数是个布尔值，如果布尔值传入 true 代表增加线程时判断当前线程是否少于 corePoolSize，小于则增加新线程，大于等于则不增加
        // 如果传入 false 代表增加线程时判断当前线程是否少于 maxPoolSize，小于则增加新线程，大于等于则不增加
        // 布尔值的含义是以核心线程数为界限还是以最大线程数为界限进行是否新增线程的判断
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    
    // 代码执行到这里，说明当前线程数大于或等于核心线程数或者 addWorker 失败了
    // 检查线程池状态是否为 Running
    // 如果为true，通过workQueue.offer(command)放入队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        
        // 如果线程池已经不处于 Running 状态，说明线程池被关闭，那么就移除刚刚添加到任务队列中的任务
        if (! isRunning(recheck) && remove(command))
            // 并执行拒绝策略
            reject(command);
        
        // 能进入这个 else 说明前面判断到线程池状态为 Running
        // 当任务被添加进来之后，需要防止没有可执行线程的情况发生，比如之前的线程被回收了或意外终止了
        else if (workerCountOf(recheck) == 0)
            // 如果检查当前线程数为 0，执行 addWorker() 方法新建线程
            addWorker(null, false);
    }
    // 执行到这里，说明 线程池不是 Running 状态或线程数大于或等于核心线程数并且任务队列已经满了
    // 此时就会再次调用 addWorker 方法并将第二个参数传入 false
    // 代表增加线程时判断当前线程数是否少于 maxPoolSize，小于则增加新线程，大于等于则不增加
    // addWorker 方法如果返回 true 代表添加成功
    // 如果返回 false 代表任务添加失败，说明当前线程数已经达到 maxPoolSize，然后执行拒绝策略 reject 方法
    else if (!addWorker(command, false))
        reject(command);
}
```

```java
// Work.class ：线程的包装类
 private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
     
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
     
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }
 }
```

```java
// runWorker() 方法的简化代码
final void runWorker(Worker w) {
    Runnable task = w.firstTask;
    while (task != null || (task = getTask()) != null) {
        try {
            task.run();
        } finally {
            task = null;
        }
    }
}
```







## 线程



### 本质只有一种

创建线程只有一种方式：`构造Thread类`



实现线程 “运行内容” 的两种方式：

- `实现Runnable接口`的Run方式，并把Runnable实列作为target对象，传给Thread类，最终调用target.run()

  ```java
  public Thread(Runnable target) {
      init(null, target, "Thread-" + nextThreadNum(), 0);
  }
  
  private void init(ThreadGroup g, Runnable target, String name,long stackSize) {
      init(g, target, name, stackSize, null);
  }
  private void init(ThreadGroup g, Runnable target, String name,
                    long stackSize, AccessControlContext acc) {
      if (name == null) {
          throw new NullPointerException("name cannot be null");
      }
  
      this.name = name;
  
      Thread parent = currentThread();
      SecurityManager security = System.getSecurityManager();
      if (g == null) {
          /* Determine if it's an applet or not */
  
          /* If there is a security manager, ask the security manager
                 what to do. */
          if (security != null) {
              g = security.getThreadGroup();
          }
  
          /* If the security doesn't have a strong opinion of the matter
                 use the parent thread group. */
          if (g == null) {
              g = parent.getThreadGroup();
          }
      }
  
      /* checkAccess regardless of whether or not threadgroup is
             explicitly passed in. */
      g.checkAccess();
  
      /*
           * Do we have the required permissions?
           */
      if (security != null) {
          if (isCCLOverridden(getClass())) {
              security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
          }
      }
  
      g.addUnstarted();
  
      this.group = g;
      this.daemon = parent.isDaemon();
      this.priority = parent.getPriority();
      if (security == null || isCCLOverridden(parent.getClass()))
          this.contextClassLoader = parent.getContextClassLoader();
      else
          this.contextClassLoader = parent.contextClassLoader;
      this.inheritedAccessControlContext =
          acc != null ? acc : AccessController.getContext();
      // 把Runnable实列作为target对象
      this.target = target;
      setPriority(priority);
      if (parent.inheritableThreadLocals != null)
          this.inheritableThreadLocals =
          ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
      /* Stash the specified stack size in case the VM cares */
      this.stackSize = stackSize;
  
      /* Set thread ID */
      tid = nextThreadID();
  }
  
  public void run() {
      if (target != null) {
          target.run();
      }
  }
  ```

  

- `继承Thread类`，重写Thread的run()方法；Thread.start() 会执行 run() 



实现`Runnable`接口，要比继承`Thread`类更好：

1. 可以报不同的内容进行解耦，权责分明
2. 某些情况可以提升性能，减少开销
3. 继承`Thread`类相当于限制了代码未来的可扩展性





### Callable/FutureTask



问题：

- **相比**Runnable，Callable`有返回值`，且需要`抛异常`

- 但是，Thread的构造方法 <u>没有传</u> Callable接口的 
- 不过可以使用 **FutureTask** 来`适配`



解决一：

可以使用`ExecutorService`的`submit()`方法

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ExecutorService.png" style="zoom:50%;" />

```java
class CallableTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return new Random().nextInt();
    }
}

// 创建线程池
ExecutorService service = Executors.newFixedThreadPool(10);
// 提交任务，并且FUture提交返回结果
Future<Integer> future = service.submit(new CallableTask());
```





解决二：

- FutureTask类 的`构造引入` 

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/FutureTask.png)

- 因为 FutureTask类 `实现了` Runnable接口

```java
public class FutureTask<V> implements RunnableFuture<V>{}
public interface RunnableFuture<V> extends Runnable, Future<V>
```



**使用实例：**

封装了 Callable接口 的FutureTask接口（适配模式），`传入`线程并启动

```java
FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>(){...} );
new Thread(futureTask).start();

// 方式一：
// 获得 Callable接口的返回值，建议放在最后
// get() 要求获得Callable线程的计算结果，如果没有计算完成就强求去要，会导致堵塞，直到计算完成
System.out.println(futureTask.get()); 

// 方式二：
// 什么时候算完才获得，更精确，类似自旋锁
while( !futureTask.isDone()){

}
```



> FutureTask 可用于==异步获取执行结果==或==取消执行任务==的场景

1. 当一个计算任务需要执行*很长时间* ，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后`再去`获取结果

2. 多个线程抢同一个FutureTask，计算结果只算一次；要想`多算`，得起多个FutureTask

   





### 停止线程



#### interrupt

想写一个健壮性很好，能够安全应对各种场景的程序时，正确停止线程就显得格外重要

- 对于 Java 而言，最正确的停止线程的方式是使用 `interrupt`
- 但 interrupt 仅仅`起到通知`被停止线程的作用
- 对于被停止的线程而言，它拥有`完全的自主权`，它既可以选择立即停止，也可以选择一段时间后停止，也可以选择压根不停止



标准格式：

```java
while (!Thread.currentThread().isInterrupted() && more work to do) {
    do more work
}
```



#### sleep 期间能否感受到中断

```java
public class StopDuringSleep {
 
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            try {
                while (!Thread.currentThread().isInterrupted() && num <= 1000) {
                    System.out.println(num);
                    num++;
                    Thread.sleep(1000000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(5);
        thread.interrupt();
    }
}
```

如果 `sleep`、`wait` 等可以让线程进入阻塞的方法使线程休眠了，而`处于休眠中的线程被中断`，那么***线程是可以感受到中断信号的***，并且会抛出一个 `InterruptedException` 异常，同时`清除中断信号`，将中断标记位设置成 false。





```java
void subTas() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        // 在这里不处理该异常是非常不好的, 中断信号被隐藏了
    }
}
```



#### **两种最佳处理方式** 



- 方法签名抛异常，run() 强制 try/catch

  ```java
  void subTask2() throws InterruptedException {
      Thread.sleep(1000);
  }
  ```

  - 调用方要不使用 try/catch 并在 catch 中正确处理异常，要不将异常声明到方法签名中
  - 如果每层逻辑都遵守规范，便可以将中断信号层层传递到顶层，最终让 run() 方法可以捕获到异常

  

- 再次中断

  ```java
  private void reInterrupt() {
      try {
          Thread.sleep(2000);
      } catch (InterruptedException e) {
          // 在 catch 语句中再次中断线程
          Thread.currentThread().interrupt();
          e.printStackTrace();
      }
  }
  ```

  - 如果线程在休眠期间被中断，那么会自动清除中断信号
  - 如果这时手动添加中断信号，中断信号依然可以被捕捉到
  - 这样后续执行的方法依然可以检测到这里发生过中断，可以做出相应的处理，整个线程可以正常退出



#### 错误的停止方法

 stop()，suspend() 和 resume()，这些方法已经被 Java 直接标记为 @Deprecated

-  stop() 会直接把线程停止

  - 没有给线程足够的时间来处理想要在停止前保存数据的逻辑，任务戛然而止，会导致出现数据完整性等问题

  

-  suspend() 和 resume() 

  - 线程调用 suspend()，它并不会释放锁，就开始进入休眠，但此时有可能仍持有锁，这样就容易`导致死锁`问题，因为这把锁在线程被 resume() 之前，是不会被释放的
  - 实列：假设线程 A 调用了 suspend() 方法让线程 B 挂起，线程 B 进入休眠，而线程 B 又刚好持有一把锁，此时假设线程 A 想访问线程 B 持有的锁，但由于线程 B 并没有释放锁就进入休眠了，所以对于线程 A 而言，此时拿不到锁，也会陷入阻塞，那么线程 A 和线程 B 就都无法继续向下执行。



-  volatile 标记位的停止方法也是错误的

  - volatile 修饰标记位`适用的场景`

    ```java
    public class VolatileCanStop implements Runnable {
     
        private volatile boolean canceled = false;
     
        @Override
        public void run() {
            int num = 0;
            try {
                while (!canceled && num <= 1000000) {
                    if (num % 10 == 0) {
                        System.out.println(num + "是10的倍数。");
                    }
                    num++;
                    Thread.sleep(1);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
     
        public static void main(String[] args) throws InterruptedException {
            VolatileCanStop r = new VolatileCanStop();
            Thread thread = new Thread(r);
            thread.start();
            Thread.sleep(3000);
            r.canceled = true;
        }
    }
    // 用 volatile 修饰的布尔值的标记位设置成 true，正在运行的线程就会在下一次 while 循环中判断出 canceled 的值已经变成 true 了，这样就不再满足 while 的判断条件，跳出整个 while 循环，线程就停止了，这种情况是演示 volatile 修饰的标记位可以正常工作的情况
    ```

    

  - volatile 修饰标记位不适用的场景

    ```java
    public class VolatileCanStop {
    	public static void main(String[] args) throws InterruptedException {
            ArrayBlockingQueue storage = new ArrayBlockingQueue(8);
    
            Producer producer = new Producer(storage);
            Thread producerThread = new Thread(producer);
            producerThread.start();
            Thread.sleep(500);
    
            Consumer consumer = new Consumer(storage);
            while (consumer.needMoreNums()) {
                System.out.println(consumer.storage.take() + "被消费了");
                Thread.sleep(100);
            }
            System.out.println("消费者不需要更多数据了。");
    
            //一旦消费不需要更多数据了，我们应该让生产者也停下来，但是实际情况却停不下来
            producer.canceled = true;
            System.out.println(producer.canceled);
        }
    }
    class Producer implements Runnable {
        public volatile boolean canceled = false;
        BlockingQueue storage;
        public Producer(BlockingQueue storage) {
            this.storage = storage;
        }
     
        @Override
        public void run() {
            int num = 0;
            try {
                while (num <= 100000 && !canceled) {
                    if (num % 50 == 0) {
                        storage.put(num);
                        System.out.println(num + "是50的倍数,被放到仓库中了。");
                    }
                    num++;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("生产者结束运行");
            }
        }
    }
    class Consumer {
        BlockingQueue storage;
        public Consumer(BlockingQueue storage) {
            this.storage = storage;
        }
        public boolean needMoreNums() {
            if (Math.random() > 0.97) {
                return false;
            }
            return true;
        }
    }
    ```

    <img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/VolatileNotCanStop.png" style="zoom:50%;" />

  - 尽管已经把 canceled 设置成 true，但生产者仍然没有停止

  - 因为在这种情况下，生产者在执行 storage.put(num) 时`发生阻塞`，在它被叫醒之前是没有办法进入下一次循环判断 canceled 的值的

  - 这种情况下用 volatile 是没有办法让生产者停下来的

  - 如果用 `interrupt` 语句来中断，即使生产者处于阻塞状态，仍然能够感受到中断信号，并做响应处理







### 线程6种状态

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ThreadStatus.png)

- New 表示线程被`创建但尚未启动`的状态
  - 当我们用 new Thread() 新建一个线程时，如果线程没有开始运行 start() 方法，所以也没有开始执行 run() 方法里面的代码，那么此时它的状态就是 New
  - 一旦线程调用了 start()，它的状态就会从 New 变成 Runnable
- Java 中的 `Runable` 状态**<u>*对应操作系统线程状态*</u>** 中的两种状态： `Running` 和 `Ready`
  - 也就是说，Java 中处于 Runnable 状态的线程：`有可能正在执行`
  - ==也有可能没有正在执行==，`正在等待被分配 CPU 资源`  （时间片）
  - 如：一个正在运行的线程是 Runnable 状态，当它运行到任务的一半时，执行该线程的 CPU 被调度去做其他事情，导致该线程暂时不运行，它的状态依然不变，还是 Runnable，因为它有可能随时被调度回来继续执行任务
- `Blocked`(被阻塞）、`Waiting`(等待）、`Timed Waiting`(计时等待），这三 种状态统称为`阻塞状态` 
  - Blocked 与 Waiting 的区别是：
  - Blocked 在`等待其他线程释放 monitor 锁` 
  -  Waiting 则是在`等待某个条件`，比如 join 的线程执行完毕，或者是 notify()/notifyAll()
- Terminated 终止状态
  - 要想进入这个状态有两种可能：
  - run() 方法`执行完毕`，线程正常退出
  - 出现一个`没有捕获的异常`，终止了 run() 方法，最终导致意外终止





### wait/notify/notifyAll



为什么 wait 方法必须在 synchronized 保护的同步代码中使用？

```java
// 为了保证 “判断-执行” 是原子操作
// 如果中间被打断，是线程不安全的，如生产者-消费者
synchronized (obj) {
    while (condition does not hold)
        obj.wait();
    ... // Perform action appropriate to condition
}
```

- `虚假唤醒`(spurious wakeup)：线程可能在既没有被`notify/notifyAll`，也没有被`中断`或者`超时`的情况下被唤醒
- 所以需要采用`while循环`的结构，即便被虚假唤醒了，也会再次检查while里面的条件，如果不满足条件，就会继续wait，也就消除了虚假唤醒的风险





为什么 wait/notify/notifyAll 被定义在 Object 类中，而 sleep 定义在 Thread 类中？

1.  Java 中每个对象都有一把称之为 `monitor` 监视器的锁，由于每个对象都可以上锁，这就要求在对象头中有一个用来保存锁信息的位置。这个锁是对象级别的，而非线程级别的，wait/notify/notifyAll 也都是锁级别的操作，它们的锁属于对象，所以把它们定义在 Object 类中是最合适，因为 Object 类是所有对象的父类。
2. 如果把 wait/notify/notifyAll 方法定义在 Thread 类中，会带来很大的局限性，比如一个线程可能持有多把锁，以便实现相互配合的复杂逻辑，假设此时 wait 方法定义在 Thread 类中，如何实现让一个线程持有多把锁呢？又如何明确线程等待的是哪把锁呢？
   - 既然我们是让当前线程去等待某个对象的锁，自然应该通过操作对象来实现，而**<u>不是操作线程</u>**。





wait/notify 和 sleep 方法的异同？

- 相同点：
  - 它们都可以让线程阻塞
  - 它们都可以响应 interrupt 中断：在等待的过程中如果收到中断信号，都可以进行响应，并抛出 InterruptedException 异常
- 不同点：

  - wait 方法必须在 synchronized 保护的代码中使用，而 sleep 方法并没有这个要求
  - 在同步代码中执行 sleep 方法时，并不会释放 monitor 锁，但执行 wait 方法时会主动释放 monitor 锁
  - sleep 方法中会要求必须定义一个时间，时间到期后会主动恢复，而对于没有参数的 wait 方法而言，意味着永久等待，直到被中断或被唤醒才能恢复，它并不会主动恢复
  - wait/notify 是 Object 类的方法，而 sleep 是 Thread 类的方法







### 线程安全

定义：

- 如果某个对象是线程安全的，那么对于使用者而言：
- `不需要考虑方法间的协调问题`，比如不需要考虑不能同时写入或读写不能并行的问题
- `不需要考虑任何额外的同步问题`，比如不需要额外自己加 synchronized 锁



一共有3 种典型的线程安全问题:

- 运行结果错误，如，两个线程同时+1
- 发布和初始化导致线程安全问题，如，线程的初始化操作还未完成，其他线程就开始调用改线程的属性==》NUll
- 活跃性问题，程序始终得不到运行的最终结果
  - 最典型的有三种，分别为死锁、活锁和饥饿



需要额外注意线程安全问题：

1. 访问共享变量或资源
2. 依赖时序的操作，如：判断-执行的组合操作（不是原子操作）
3. 不同数据之间存在绑定关系
4. `被调用方`没有声明自己是线程安全的





多线程可能会带来性能问题：

- 线程调度
  - 调度开销：
  - `上下文切换` 会挂起当前正在执行的线程并保存当前的状态，然后寻找下一处即将恢复执行的代码，唤醒下一个线程，以此类推，反复执行
  - `缓存失效` 一旦进行了线程调度，切换到其他线程，CPU就会去执行不同的代码，原有的缓存就很可能失效了，需要重新缓存新的数据，这也会造成一定的开销
- 线程协作
  - 线程之间如果有共享数据，为了避免数据错乱，为了保证线程安全，编译器和 CPU 就有可能对其进行`禁止重排序`等优化
  - 也可能出于同步的目的，反复把线程工作内存的数据 flush 到主存中，然后再从主内存 refresh 到其他线程的工作内存中

其实性能问题有许多的表现形式，比如`服务器的响应慢`、`吞吐量低`、`内存占用过多`就属于性能问题









### 基础线程机制



#### Executor

- Executor 管理多个异步任务的执行，而程序员`无需显式`地管理线程的生命周期。

- 这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程，`带缓存，可扩容`；
- FixedThreadPool：所有任务只能使用`固定`大小的线程；
- SingleThreadExecutor：相当于大小为 `1` 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

还有两种Executor：

- ScheduledThreadPool：设置时间参数，池里面的请求每间隔多少秒执行一次
- WorkStealingPool：(java8) 使用目前机器上可用的处理器作为它的并行级别

**有两种提交请求：** 

1. execute() ：`Executor` 接口带着的
2. submit()：`ExecutorService` 接口重新定义的



#### Daemon

守护线程是程序运行时在后台提供服务的线程，`不属于`程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会`杀死所有`守护线程。

main() 属于非守护线程。

在线程启动之前使用 `setDaemon()` 方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```



#### sleep()

Thread.sleep(millisec) 方法会`休眠`当前正在执行的线程，millisec 单位为毫秒。

如果线程这时候手上有锁，`不会`释放。

sleep() 可能会抛出 `InterruptedException`，因为异常不能跨线程传播回 main() 中，因此**<u>*必须在本地进行处理*</u>** 

- 线程中抛出的其它异常，也同样需要在本地进行处理

```java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



#### yield()

- 对静态方法 Thread.yield() 的调用**声明了**当前线程已经完成了生命周期中最重要的部分，可以**切换**给其它线程来执行。
- 该方法只是对线程调度器的`一个建议`，而且也只是建议***<u>具有相同优先级</u>*** 的其它线程可以运行。

```java
public void run() {
    Thread.yield();
}
```









## ThreadLocal



[应用场景/源码](https://juejin.cn/post/6854573219916021767#heading-0)	



[ThreadLocal夺命4问](https://juejin.cn/post/6917793256977891335)	









## 死锁编码及定位分析

死锁：指`两个或两个以上`的进程在执行过程中，因`争夺资源`而造成的一种`互相等待`的现象

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/DeadLock.png" style="zoom:50%;" />



**1. 产生死锁的原因** 

**2. 代码实例**

```java
class holdLock implements Runnable{
    private String lock1;
    private String lock2;
    public holdLock(String lock1, String lock2) {
        this.lock1 = lock1;
        this.lock2 = lock2;
    }
    @Override
    public void run() {
        synchronized (lock1){
            System.out.println(Thread.currentThread().getName() + "\t 已经持有"+lock1+", 想持有"+ lock2);
            synchronized (lock2){
                System.out.println(Thread.currentThread().getName() + "\t 已经持有"+lock2+", 想持有"+ lock1);
            }
        }
    }
}
public class DeadLockDemo {
    public static void main(String[] args) {
        String lock1 = "lockA";
        String lock2 = "lockB";
        new Thread(new holdLock(lock1,lock2),"AAA").start();
        new Thread(new holdLock(lock2,lock1),"BBB").start();
    }
}
```



**3. 定位分析**

1. jps：查看java的后台运行程序 ---> `定位进程号`
   <img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/FindJavaProcess.png" style="zoom:47%;" />

  

2. jstack：java的异常栈消息

   ```shell
   jstack + 进程编号
   ```

   

## 集合类

java.util.`ConcurrentModificationException` 





### ArrayList

ArrayList的添加方法，相比Vector 没有`synchronized`修饰



**1. 故障现象**
<img src="JUC多线程及高并发.assets/20200118142329482_6916.png" style="zoom:80%;" />



**2. 导致原因**

在迭代期间，不允许编辑

```java
// ArrayList 源码里的 ListItr 的 next 方法中有一个 checkForComodification 方法
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```



**3. 解决方案**

```java
new Vector<>(); // 不推荐使用
Collections.synchronizedList(new ArrayList<>());    //借用集合工具类，包书壳
new CopyOnWriteArrayList();   // 读写分离，写时复制
```



**4. 优化建议**

CopyOnWriteArrayList，即写时复制的容器:

​		`适用场景`：读多写少；读操作可以尽可能的快，而写即使慢一些也没关系

> 可以对 CopyOnWrite 容器进行`并发的读`，*而不需要加锁* 
>
> 因为当前容器不会添加任何元素，所以 CopyOnWrite 容器也是一种读写分离的思想，`读和写 在不同的容器`



缺点：

- 内存占用问题
- 在元素较多或者复杂的情况下，复制的开销很大
- 数据一致性问题



源码分析：

```java
// 数据结构：
/** 可重入锁对象 */
final transient ReentrantLock lock = new ReentrantLock();
/** CopyOnWriteArrayList底层由数组实现，volatile修饰，保证数组的可见性 */
private transient volatile Object[] array;
// 得到数组
final Object[] getArray() {
    return array;
}
// 设置数组
final void setArray(Object[] a) {
    array = a;
}
// 初始化CopyOnWriteArrayList相当于初始化数组
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}
```



```java
// CopyOnWriteArrayList.add() 
public boolean add(E e) {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 得到原数组的长度和元素
        Object[] elements = getArray();
        int len = elements.length;
        // 复制出一个新数组
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 添加时，将新元素添加到新数组中
        newElements[len] = e;
        // 将volatile Object[] array 的指向替换成新数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

1. 往一个容器**添加元素**的时候，`不直接`往当前容器object[] 添加，而是先将当前容器object[] 进行copy**复制**出一个`新的容器`object[] newElements
2. 然后`往新的`容器object[] newElements里面添加元素
3. 添加完元素之后，再将原容器的`引用指向新的容器` setArray(newElements);





### HashSet

**1. 故障现象**

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ConcurrentOfHashSet.png)

**2. 导致原因**





**3. 解决方案**

```java
Collections.synchronizedSet(new HashSet<>());  // 包书皮
new CopyOnWriteArraySet<>();  // 读写分离，写时复制
```



**4. 优化建议** 

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;
    private final CopyOnWriteArrayList<E> al;

    // Creates an empty set.
    // 底层的数据结构是 CopyOnWriteArrayList
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }

```



`类似` HashSet 的底层 数据结构是HashMap

```java
 /**
  * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
  * default initial capacity (16) and load factor (0.75).
  */
public HashSet() {
    map = new HashMap<>();
}
// HashSet只关心key，value都是object类型的 present常量
```





### HashMap

**1. 故障现象**

![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/CurrentOfHashmap.png)



**2. 导致原因**

不允许在迭代期间修改内容，其原理是检测 `modCount` 变量

```java
// HashMap&KeySet.class
public final void forEach(Consumer<? super K> action) {
    Node<K,V>[] tab;
    if (action == null)
        throw new NullPointerException();
    if (size > 0 && (tab = table) != null) {
        int mc = modCount;
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next)
                action.accept(e.key);
        }
        // mc (expectedModCount)
        if (modCount != mc)  
            throw new ConcurrentModificationException();
    }
}
```



**3. 解决方案**

```java
new HashTable();
Collections.synchronizedMap(new HashMap<>());
new ConcurrentHashMap<>();
```

**4. 优化建议**（同样的错误不犯第二次）





## 高并发心法



### 问题

**1、对数据化的指标没有概念：**不清楚选择什么样的指标来衡量高并发系统？分不清并发量和QPS，甚至不知道自己系统的总用户量、活跃用户量，平峰和高峰时的QPS和TPS等关键数据。

**2、设计了一些方案，但是细节掌握不透彻：**讲不出该方案要关注的技术点和可能带来的副作用。比如读性能有瓶颈会引入缓存，但是忽视了缓存命中率、热点key、数据一致性等问题。

**3、理解片面，把高并发设计等同于性能优化：**大谈并发编程、多级缓存、异步化、水平扩容，却忽视高可用设计、服务治理和运维保障。

**4、掌握大方案，却忽视最基本的东西：**能讲清楚垂直分层、水平分区、缓存等大思路，却没意识去分析数据结构是否合理，算法是否高效，没想过从最根本的IO和计算两个维度去做细节优化。



### 目标

 [高并发心法](https://mp.weixin.qq.com/s/P81b6LLJ5EL1kYmX627KJw)：高性能、高可用，以及高可扩展



高扩展性需要考虑：服务集群、数据库、缓存和消息队列等中间件、负载均衡、带宽、依赖的第三方等，当并发达到某一个量级后，上述每个因素都可能成为扩展的瓶颈点。











