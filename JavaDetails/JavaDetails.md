# Java Detials





## 阶段图

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/java_class_load_theory.png)

1. 编译器将Java源代码编译成字节码class文件
2. 类加载到JVM里面后，执行引擎把字节码转为可执行代码
3. 执行的过程，再把可执行代码转为机器码，由底层的操作系统完成执行。





## 类加载器

> Java 中的类遵循`按需加载`

类加载器：就是用于加载Java类到JVM中的组件，它负责读取Java字节码，**并转换为java.lang.Class类的一个实例**，使字节码 **.class** 文件得以运行。

- 一般类加载器负责根据一个指导的类找到对应的字节码，然后根据这些字节码定义一个Java类
- 它还可以加载资源，包括图像文件和配置文件



类加载器在实际使用中给我们带来的**好处**是：

可以是Java类动态的加载到JVM中 并运行，即可在程序运行时再加载类（懒加载？？）

- 启动类加载器（BootStrap ClassLoader）：加载对象是Java核心库
  - 把一些核心的Java类加载进JVM中，该加载器使用 C/C++ 实现，**并不是**继承 java.lang.ClassLoader
  - 启动类加载器是所有其他类加载器的`最终父加载器`，负责加载**<JAVA_HOME>/jre/lib** 目录下JVM指定的类库
  - 启动类加载器属于JVM整体的一部分，JVM一启动就将这些指定的类加载到内存中，生成一个个的对象，避免以后过多的I/O操作，提高系统的运行效率
  - 启动类加载器**无法**被Java查询直接使用
- 扩展类加载器（Extension ClassLoader）：加载的对象为Java的扩展库
  - 即加载**<JAVA_HOME>/jre/lib/ext** 目录里面的类
  - 扩展类加载器 由启动类加载器加载
  - 因为启动类加载器并非Java实现，已经脱离Java体系，使用调用扩展类加载器的getParent()方法 获取父加载器会得到null
  - 然而，扩展类加载器的父类加载器 是启动类加载器
- 应用程序类加载器（Application ClassLoader）：也叫系统类加载器（System ClassLoader）
  - 负责加载用户类路径（CLASSPATH）指定的类库
  - 如果程序没有自己定义类加载器，就默认使用应用程序类加载器
  - 应用程序类加载器 也是由启动类加载器加载，但它的父加载器被设置为扩展类加载器
  - 使用这个加载器，可以通过 ClassLoader.getSystemClassLoader()获取







## 双亲委派

双亲委派模型会在类加载器 加载类时：

- 首先委托给父类加载器加载

- 除非父类加载器不能加载才自己加载

  

[打破双亲委派](https://www.bilibili.com/video/BV1ri4y1s7VF?p=9)	：

- 通过 **重写 loadClass()** 方法，打破双亲委派

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/loadClass.png)



不同的classLoader实例，加载同一个class文件，clazz对象不同【clazz前带有classLoader实例标识】

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/deferent_clazz.png)







## 生成doc文档

```java
javadoc demo.java
```





## Java日常开发的坑



[21基础知识点的坑](https://mp.weixin.qq.com/s/r__Lv_3lfL10xYM1Nz7sTw)



[Optional类](https://www.runoob.com/java/java8-optional-class.html): null指针



## 第三方工具包



[Hutool](https://www.hutool.cn/)	

[Guava](https://github.com/google/guava)	







