# IO

[理论基础](https://www.bilibili.com/video/BV1Af4y117ZK?t=7&p=2)	：

<img src="IO.assets/image-20200730155347922.png" alt="image-20200730155347922" style="zoom:67%;" />



`时钟中断` ==> 保护现场【现在的执行程序】 ==> CPU执行内核的程序调度 ==> 恢复现场【调度到的程序 之前的寄存器状态】

`程序的切换过程`：上下文 、保护现场、恢复现场、内存和CPU间的吞吐量

`中断（INT x80）`、`系统调用`：用户空间 **想**系统调用 内核空间时，所用到的方法【实际上是，CPU来调用】

- 【保护模式：用户空间不能直接调用内核空间】





## 磁盘操作：File
       File 类可以用于**表示文件和目录的信息**，但是它`不表示`文件的内容。
 * 从 Java7 开始，可以使用 Paths 和 Files 代替 File。
```java
import java.nio.file.Files;
import java.nio.file.Paths;
import java.nio.file.Path;    // Path就是取代File的，Path用于来表示文件路径和文件
```





## 字节操作：InputStream 和 OutputStream

```java
public abstract int read() throws IOException;
public int read(byte b[], int off, int len) throws IOException{...}
public int read(byte b[]) throws IOException {
        return read(b, 0, b.length);
    }
```
1. 无参的read()方法返回的**int类型**，是表示数据**下一个字节的字节码**，如果已经到达流的最后面了，那就返回-1
2. 两个带参数read()方法返回的是`实际`**读取的字节数**。



### 文件复制

```java
public static void copyFile(String src, String dist) throws IOException {
    FileInputStream in = new FileInputStream(src);
    FileOutputStream out = new FileOutputStream(dist);

    byte[] buffer = new byte[20 * 1024];
    int cnt;

    // 带参数的 read() 最多读取 buffer.length 个字节
    // 返回的是实际读取的个数
    // 返回 -1 的时候表示读到 eof，即文件尾
    while ((cnt = in.read(buffer, 0, buffer.length)) != -1) {
        out.write(buffer, 0, cnt);
    }

    in.close();
    out.close();
}
```
### 装饰者模式
Java I/O 使用了装饰者模式来实现。以 InputStream 为例：
* InputStream 是抽象组件；
* FileInputStream 是 InputStream 的子类，属于**具体组件**，提供了字节流的输入操作；
* FilterInputStream 属于**抽象装饰者**，装饰者用于装饰组件，`为组件提供额外的功能`。
* 例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。
* DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。

![](IO.assets/20200105105938629_14783.png )

#### 具有缓存功能的字节流对象：
```java
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream); 
// 只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可
```

## 字符操作：Reader 和 Writer
### 编码与解码
    编码就是把“字符转换为字节”
    
    而解码是把“字节重新组合成字符”
* GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
* UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
* UTF-16be 编码中，中文字符和英文字符都占 2 个字节。
    * **Java 的内存编码**使用双字节编码 UTF-16be，**这不是指** Java 只支持这一种编码方式，**而是说** char 这种类型使用 UTF-16be 进行编码。
    * char 类型占 16 位---->两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文`都能使用一个 char 来存储`。
    * UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。
> 位(bit)、字节(byte)、字符(word)、字

### String 的编码方式
- String 可以看成一个字符序列：
    * 可以指定一个编码方式将它编码为字节序列
    * 也可以指定一个编码方式将一个字节序列解码为 String
```java
    String str1 = "中文";
    byte[] bytes = str1.getBytes("UTF-8");
    String str2 = new String(bytes, "UTF-8");
    System.out.println(str2);
```

- 在调用无参数 getBytes() 方法时，`默认的编码`方式不是 UTF-16be
```java
byte[] bytes = str1.getBytes();
```
> 双字节编码的好处是可以`使用一个 char 存储中文和英文`
> 而将 String 转为 bytes[] 字节数组就不再需要这个好处，因此也就不再需要双字节编码。
         getBytes() 的默认编码方式与平台有关，一般为 UTF-8。

### Reader 与 Writer
    不管是磁盘还是网络传输，`最小的存储单元都是字节`，而不是字符。
    但是在程序中操作的`通常是`字符形式的数据，因此需要提供对字符进行操作的方法。
- InputStreamReader 实现从`字节流`解码成`字符流`
- OutputStreamWriter 实现`字符流`编码成为`字节流`

## 对象操作：Serializable
### 序列化
> 序列化就是将一个`对象`转换成`字节序列`，方便存储和传输。
- 序列化：ObjectOutputStream.writeObject()
- 反序列化：ObjectInputStream.readObject()
> 不会对静态变量进行序列化，因为序列化**只是保存对象的状态**，`静态变量属于类的状态`



### Serializable

> 序列化的类需要实现 Serializable 接口
> 它只是一个标准，`没有任何方法需要实现`
> 但是如果不去实现它的话而进行序列化，会抛出异常。



### transient

> transient 关键字可以`使一些属性不会被序列化`
```java
private transient Object[] elementData;
// ArrayList 中存储数据的数组 elementData 是用 transient 修饰的
// 因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化
```
通过`重写`序列化和反序列化方法，使得可以`只序列化`数组中有内容的那部分数据



## 网络操作：Socket



### Java 中的网络支持



#### InetAddress

用于表示网络上的硬件资源，即 IP 地址

没有公有的构造函数，只能通过`静态方法来创建`实例
```java
InetAddress.getByName(String host);
InetAddress.getByAddress(byte[] address);
```
#### URL

统一资源定位符

可以直接从 URL 中读取字节流数据。
```java
public static void main(String[] args) throws IOException {

    URL url = new URL("http://www.baidu.com");

    /* 字节流 */
    InputStream is = url.openStream();

    /* 字符流 */
    InputStreamReader isr = new InputStreamReader(is, "utf-8");

    /* 提供缓存功能 */
    BufferedReader br = new BufferedReader(isr);

    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }

    br.close();
}
```

#### Sockets

使用 TCP 协议实现网络通信

>ServerSocket：`服务器`端类
>Socket：`客户`端类
>服务器和客户端通过 `InputStream` 和 `OutputStream` 进行输入输出

![](IO.assets/20200105163630488_8722.png )
两个流：网络流 和 本地流



#### Datagram

使用 UDP 协议实现网络通信

- DatagramSocket：通信类
- DatagramPacket：数据包类


## NIO
- 新的输入/输出 (NIO) 库是在 `JDK 1.4 `中引入的，弥补了原来的 I/O 的不足，提供了`高速的、面向块`的 I/O。
> 块、通道、缓冲区


### 流与块

|                    |          I/O          |           NIO            |
| ------------------ | --------------------- | ------------------------ |
| 数据打包和传输的方式 | 以`流`的方式处理数据    | 以`块`的方式处理数据      |
| 优点                | 创建过滤器非常容易      | 处理数据比按流处理要快得多 |
| 缺点                | 面向流的 I/O 通常相当慢 | 缺少面向流的优雅性和简单性 |
***
- java.io.* 已经以 NIO 为基础`重新实现`了，所以现在它可以利用 NIO 的一些特性
- 例如，java.io.* 包中的一些类`包含以块的形式`读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。





### 通道
> 通道 Channel 是对原 I/O 包中的流的`模拟`，可以通过它读取和写入数据

|                              流                               |             通道             |
| :-----------------------------------------------------------: | :-------------------------: |
|                            `单向`                             |           `双向 `            |
| 只能在一个方向上移动(必须是 InputStream 或者 OutputStream 的子类 | 可以用于读、写或者同时读写 |

>通道包括以下类型：
- FileChannel：从`文件`中读写数据；
- DatagramChannel：通过 `UDP` 读写网络中数据；
- SocketChannel：通过 `TCP` 读写网络中数据；
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。



### 缓冲区

> 发送给一个通道的所有数据都必须首`先放到缓冲区`中，同样地，从通道中读取的任何数据都要先读到缓冲区中。
> `不会直接`对通道进行读写数据，而是要先经过缓冲区。



缓冲区`实质上是一个数组`，但它`不仅仅`是一个数组



> 缓冲区提供了对数据的`结构化访问`，而且还可以跟`踪系统的读/写进程`。
> capacity：最大容量；
> position：当前已经读写的字节数；
> limit：还可以读写的字节数。

缓冲区包括以下类型：
- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer



### 文件 NIO 实例

```java
public static void fastCopy(String src, String dist) throws IOException {

    /* 获得源文件的输入字节流 */
    FileInputStream fin = new FileInputStream(src);

    /* 获取输入字节流的文件通道 */
    FileChannel fcin = fin.getChannel();

    /* 获取目标文件的输出字节流 */
    FileOutputStream fout = new FileOutputStream(dist);

    /* 获取输出字节流的文件通道 */
    FileChannel fcout = fout.getChannel();

    /* 为缓冲区分配 1024 个字节 */
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

    while (true) {

        /* 从输入通道中读取数据到缓冲区中 */
        int r = fcin.read(buffer);

        /* read() 返回 -1 表示 EOF */
        if (r == -1) {
            break;
        }

        /* 切换读写 */
        buffer.flip();

        /* 把缓冲区的内容写入输出文件中 */
        fcout.write(buffer);

        /* 清空缓冲区 */
        buffer.clear();
    }
}
```




### 选择器









## Linux





### 输入输出流

[视频讲解](https://www.bilibili.com/video/BV1Af4y117ZK)	

<img src="IO.assets/image-20200719102805850.png" alt="image-20200719102805850" style="zoom:70%;" />

- 8 相当于 Channel，可以用于读、写或者同时读写
- <> ：表示输入输出流；【如果输入输出流后面接的是文件描述符，需要在 <> 后面加一个 &，如上图所示】
- 实例：> 到文件；>& 到文件描述符
- 0 ：标准输入
- 1 ：标准输出
- 2 ：报错流
- `"GET / HTTP/1.0\n"` ：应用层中Http协议的请求头
- `"HTTP/1.0 200 OK"` ：应用层中Http协议的返回头

<img src="IO.assets/image-20200719103332067.png" alt="image-20200719103332067" style="zoom:60%;" />

- $$ ：表示<u>当前所在位置</u>的**解释程序**
- fd ：文件描述符，？？类似Java中变量引用



### strace

```shell
strace -ff  -o out 待执行的程序 待执行的文件
# -ff ：未来程序所有的线程
# -o：输出
# out：以out三个字母开头的，每个线程的日志文件
```






