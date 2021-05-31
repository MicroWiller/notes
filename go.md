# Go



## 面试题

 [nil切片和空切片](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483777&idx=1&sn=d3d108abe68c282a4d291a432559f2cc&scene=21#wechat_redirect)：`数组指针地址都分别指向哪儿` 

 [json包变量不加tag会怎么样](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483698&idx=1&sn=352a5cddf20fe95f5ec26bfc9a6de64b&scene=21#wechat_redirect)：`Marshal` [Struct Tag](#Struct Tag) 

 [什么是内存逃逸](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483686&idx=1&sn=e48c51107191f02da5751a19a54f7d41&chksm=9aee288fad99a199c126d5ff735af7320356ce4bb5753ae59ac6231e596354499414b5705b79&token=2092782362&lang=zh_CN&scene=21#wechat_redirect)：`从栈逃到堆上` / `典型场景` 

 [怎么避免内存逃逸](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483692&idx=1&sn=d5d34fad7a4553e0b9d5714385b7af48&chksm=9aee2885ad99a193253c1e57bd361b3f5af643d3ba14f56f25c0c5551990c848c6f30a5ca23e&token=961196008&lang=zh_CN&scene=21#wechat_redirect)：`noescape` 

 [字符串转成byte数组](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483669&idx=1&sn=88f754ddabc04eb3f66ba8ac37ee1461&chksm=9aee28bcad99a1aa1ada41cfccaffc7ef4719a9bc11c1bef45b7d1b5427c1faa12d8d0c3156f&token=2092782362&lang=zh_CN&scene=21#wechat_redirect)： [零拷贝](#零拷贝)  

 [对已经关闭的的chan进行读写，会怎么样](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483735&idx=1&sn=6ab8ac319dc1857acf44c9bb9d21c7c3&scene=21#wechat_redirect)：`读：关闭前是否有值` / `写：Panic`  

[对未初始化的的 chan 进行读写，会怎么样](https://mp.weixin.qq.com/s?__biz=MzAwMDAxNjU4Mg==&mid=2247483717&idx=1&sn=0c0385234cd70d72d424eb7097ebfe9a&scene=21#wechat_redirect)：`都会阻塞` 



 [go黑暗角落](https://mp.weixin.qq.com/s/XLvWg9kRoiR4WPhjXE5X_g)：For循环迭代器变量被重用





## Go实战

 [今日头条Go建千亿级微服务的实践](https://zhuanlan.zhihu.com/p/26695984)：`并发控制` / `超时控制` / 

**wait：** 

```go
func main() {
   wg := sync.WaitGroup{}
	wg.Add(3)
	go func() {
		defer wg.Done()
		// do ...
	}()
	go func() {
		defer wg.Done()
		// do ...
	}()
	go func() {
		defer wg.Done()
		// do ...
	}()
	wg.Wait()
}
```

Cancle：

```go
func main() {
   ctx := context.Background()
	ctx, cancle := context.WithCancel(ctx)
	go Proc(ctx)
	go Proc(ctx)
	go Proc(ctx)
	// Cancel after 1s
	time.Sleep(time.Second)
	cancle()
}
func Proc(ctx context.Context) {
	for {
		select {
		case <- ctx.Done():
			return
		default:
			// do ...
		}
	}
}
```

```go
func Handle(r *http.Request)  {
	timeoutStr := r.FormValue("timeout")
	timeout, _ := strconv.Atoi(timeoutStr)
	ctx, cancel := context.WithTimeout(context.Background(), time.Duration(timeout))
	defer cancel()
	done := make(chan struct{}, 1)
	go func() {
		RPC(ctx, ...)
        // ？？万一是其他协程传递的呢？？ 
		done <- struct{}{}
	}()
	select {
	case <- done:
		// nice ... 
	case <- ctx.Done():
		// timeout
	}
}
```





[go并发中一些有趣的现象](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247485030&idx=1&sn=6eb805b2ecbc123e984f563bde734812&chksm=f904133bce739a2df5c3c800539d6d4a15e8d900b25f8ffb4ed08685a4135212f521825e96bc&cur_album_id=1515516076481101825&scene=190#rd)：

```go
func main() {
 wg := sync.WaitGroup{}
 wg.Add(5)
 for i := 0; i < 5; i++ {
  go func() {
   defer wg.Done()
   fmt.Println(i)
  }()
 }
 wg.Wait()
}
// 主要是变量 i 没有作为形参传入
```

// 第一次输出：5 5 5 5 5

// 多输出几次：5 3 5 5 5



```go
func main() {
 wg := sync.WaitGroup{}
 wg.Add(5)
 for i := 0; i < 5; i++ {
  go func(i int) {
   defer wg.Done()
   fmt.Println(i)
  }(i)
 }
 wg.Wait()
}
```

// 第一次输出：0 1 2 4 3

// 第二次输出：4 0 1 2 3

（输出的值都是无序且不稳定的）





其整个程序扭转实质上分为了多个阶段，也就是各自运行的时间线并不同，可以其拆分为：

- 先创建：`for-loop` 循环创建 `goroutine`。
- 再调度：协程`goroutine` 开始调度执行。
- 才执行：开始执行 `goroutine` 内的输出。













## GOPROXY

```shell
# 开启module模式
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct

# 设置不走 proxy 的私有仓库，多个用逗号相隔（可选）
go env -w GOPRIVATE=*.corp.example.com
```





## 数组 / 切片



### 数组

```go
array:=[5]string{"a","b","c","d","e"}
array:=[...]string{"a","b","c","d","e"}

// slice := []string{"a","b","c","d","e"}
```

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ArrayOfGo.png" style="zoom:60%;" />

```go
// 数组循环
for i,v:=range array{
    fmt.Printf("数组索引:%d,对应值:%s\n", i, v)
}
```











### 切片

```go
slice := array[2:5]
```

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/SliceOfGo.png" style="zoom:60%;" />

- array[2:5] 获取到的是 c、d、e 这三个元素；左闭右开区间



```go
// 还可以使用make 函数传入一个容量参数：
slice:=make([]string,4)
```

```go
// 可以指定新创建的切片 []string 容量为 8：
slice := make([]string,4,8)
```

- Go 语言在内存上划分了一块容量为 8 的内容空间（容量为 8）
- 但是只有 4 个内存空间才有元素（长度为 4），其他的内存空间处于空闲状态
- 当通过 append 函数往切片中追加元素的时候，会追加到空闲的内存上，当切片的长度要超过容量的时候，会进行扩容



```go
// 通过字面量的方式声明和初始化：      (和数组的区别只是，没有容量个数)
slice := []string{"a","b","c","d","e"}
```



通过内置的 append 函数对一个切片追加元素，返回新切片，如下面的代码所示：

```go
//追加一个元素
slice2:=append(slice1,"f")

//多加多个元素
slice2:=append(slice1,"f","g")

//追加另一个切片
slice2:=append(slice1,slice...)
```





小技巧：在创建新切片的时候，最好要让新切片的**长度==容量**，这样在追加操作的时候就会**生成新的**底层数组，从而和原有数组分离，就不会因为共用底层数组导致修改内容的时候影响多个切片。









## fmt

`fmt.Print`有几个变种：

```go
Print:  输出到控制台,不接受任何格式化操作
Println: 输出到控制台并换行
Printf : 只可以打印出格式化的字符串。只可以直接输出字符串类型的变量（不可以输出别的类型）
Sprintf：格式化并返回一个字符串而不带任何输出
Fprintf：来格式化并输出到 io.Writers 而不是 os.Stdout
```

通用的占位符

```go
%v     值的默认格式。
%+v   类似%v，但输出结构体时会添加字段名
%#v　 相应值的Go语法表示 
%T    相应值的类型的Go语法表示 
%%    百分号,字面上的%,非占位符含义
```



参考：[Go中的fmt几种输出的区别和格式化方式](https://www.cnblogs.com/rickiyang/p/11074171.html) 



## Context

[深度解析context](https://zhuanlan.zhihu.com/p/68792989): 不完美，但很高效解决了问题



==任何调用的默认超时时间的设置是非常有必要的== 









## 指针



### 指针结构图

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/PointerValue.png" style="zoom:67%;" />

- 指针是一种数据类型，用来存储一个内存地址，该地址指向存储在该内存中的对象
- 这个对象可以是字符串、整数、函数或者你自定义的结构体
- 指针变量的值就是 **它(内存地址)** 所指向数据的内存地址，普通变量的值就是我们具体存放的数据
- 指针类型非常廉价，只占用 4 个或者 8 个字节的内存大小



### 指针变量/普通变量

```go
func main() {
   name:="will"
   nameP:=&name		// &, 表示取地址；指针的类型是 *string
   fmt.Println("name变量的值为:",name)	// name变量的值为: will
   fmt.Println("name变量的内存地址为:",nameP)	// name变量的内存地址为: 0xc000010200
}
```

-  Go 语言中使用类型名称前加 * 的方式，即可表示一个对应的指针类型
- 比如 int 类型的指针类型是 *int，float64 类型的指针类型是 *float64，自定义结构体 A 的指针类型是 *A



### 定义方式

```go
// 也可以通过 var 关键字定义
var intP *int

//指针类型不同，无法赋值 :
intP = &name 	
//  Cannot use '&name' (type *string) as type *int in assignment

// 指针类型的变量如果没有分配内存，默认零值 nil，它没有指向的内存；所以无法使用，强行使用就会得到 nil 指针错误
*intP =10
// ipanic: runtime error: invalid memory address or nil pointer dereference
```

- 通过 var 声明的指针变量不能直接赋值和取值的，因为它`仅仅是个变量`，*还没有对应的内存地址*，它的值是 `nil`

- 而对于==值类型==来说，`即使只声明一个变量`，没有对其初始化，**<u>*该变量也会有分配好的内存*</u>**

  ```go
  func main() {
     var s string
     fmt.Printf("%p\n",&s)
  }
  // 声明了一个变量 s，并没有对其初始化，但是可以通过 &s 获取它的内存地址
  // 这其实是 Go 语言帮我们做的，可以直接使用
  ```

  

```go
// 只需要通过 new 函数给它分配一块内存就可以了
var intP *int = new(int)

// 更推荐简短声明法: 
intP := new(int)
```



**和普通类型不一样的是**，指针类型还可以通过内置的 new 函数来声明: 

```go
intP1 := new(int)
// 内置的 new 函数有一个参数，可以传递类型给它,它会返回对应的指针类型
```



### 获取指针指向的值

```go
nameV := *nameP
fmt.Println("nameP指针指向的值为:",nameV)
```

- 要获取指针指向的值，只需要在指针变量前加 `*` 号即可
- `*` 直接，映射到 指针中存储的内存地址 指向的指





### 修改指针指向的值

```go
//修改指针指向的值
*nameP = "will micro" 
fmt.Println("nameP指针指向的值为:",*nameP)
fmt.Println("name变量的值为:",name)
// 运行结果:
// nameP指针指向的值为: will micro
// name变量的值为: will micro
```

==对 *nameP 赋值等于修改了指针 nameP 指向的值== 

不光 nameP 指针指向的值被改变了，变量 name 的值也被改变了，这就是指针的作用



### 指针参数

```go
age:=18
modifyAge(age)
fmt.Println("age的值为:",age)

func modifyAge(age int)  {
   age = 20
}
```

modifyAge 中的 age 只是实参 age 的一份拷贝，所以修改它不会改变实参 age 的值

**要达到修改年龄，需要使用指针:**  

```go
age:=18
modifyAge(&age)
fmt.Println("age的值为:",age)
func modifyAge(age *int)  {
   *age = 20
}
```

> 需要在函数中通过形参改变实参的值时，需要使用指针类型的参数



### 指针的优点

1. 可以**修改**指向数据的值
2. 在`变量赋值`，`参数传值`的时候可以**节省内存**



Go 语言作为一种高级语言，在指针的使用上还是比较**克制**的。

它在设计的时候就对指针进行了诸多限制，比如`指针不能进行运算`，也`不能获取常量的指针`。



### 指针的建议

1. 不要对 map、slice、channel 这类引用类型使用指针
2. 如果需要修改`方法接收者`***<u>内部的</u>*** 数据或者状态时，需要使用指针
3. 如果需要修改`参数的值`或者`内部数据`时，也需要使用指针类型的参数
4. 如果是`比较大的结构体`，每次参数传递或者调用方法都要内存拷贝，内存占用多，这时候可以考虑使用指针
5. 像 int、bool 这样的小数据类型没必要使用指针
6. 如果需要`并发安全`，则尽可能地不要使用指针，使用指针一定要保证并发安全
7. 指针最好`不要嵌套`，也就是不要使用一个指向指针的指针，虽然 Go 语言允许这么做，但是这会使你的代码变得异常复杂





## 值类型/指针类型



### 指针的指向

```go
// 定义一个地址结构体
type address struct {
   province string
   city string
}

// 值类型 address 作为接收者实现了接口 fmt.Stringer
func (addr address) String()  string{
   return fmt.Sprintf("the addr is %s%s",addr.province,addr.city)
}
// 那么指针类型 *address 也实现了接口 fmt.Stringer
```



```go
func main() {
   add := address{province: "北京", city: "北京"}
   printString(add)
   printString(&add)
}
func printString(s fmt.Stringer) {
   fmt.Println(s.String())
}
// 正常运行：
// 证明了当值类型作为接收者实现了某接口时，它的指针类型也同样实现了该接口
```



```go
func main() {
    var si fmt.Stringer = address{province: "上海",city: "上海"}
    printString(si)
    sip := &si
    printString(sip)
}
// 错误信息：
// ./main.go:42:13: cannot use sip (type *fmt.Stringer) as type fmt.Stringer in argument to printString:
//	*fmt.Stringer is pointer to interface, not interface
```



总结：**指向`具体类型`的指针可以实现一个接口，但是指向`接口`的指针永远不可能实现该接口** 





### 内存与不同类型



#### 概述

一个变量必须要经过`声明`、`内存分配`才能赋值，*才可以在声明的时候进行初始化* ：

- 如果要对一个变量赋值，这个变量`必须有对应的分配好的内存`，这样才可以对这块内存操作，完成赋值的目的

- **不止赋值操作**，`对于指针变量`，如果没有分配内存，*取值操作一样会报 nil 异常*，`因为没有可以操作的内存`

  

`指针类型`在声明的时候，`Go 语言并没有自动分配内存`，所以不能对其进行赋值操作，这和`值类型`不一样

`slice`、`chan`和 `map` 也一样，因为它们*<u>**本质上也是指针类型**</u>*





#### new 函数

```go
func main() {
   var sp *string  // 没有分配到内存，不能对其进行赋值操作
    
   sp = new(string)	// 关键点，这可以分配到一块内存
   *sp = "will"
   fmt.Println(*sp)
}
```

内置函数 new 的源代码:

```go
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```

> 翻译：根据传入的类型申请一块内存，然后返回指向这块内存的指针，指针指向的数据就是该类型的零值。



指针变量初始化：（工厂函数）

```go
pp := NewPerson("will",20)

func NewPerson(name string,age int) *person{
   p:=new(person)
   p.name = name
   p.age = age
   return p
}
```





#### make 函数

使用 make 函数创建 map 的时候，其实调用的是 makemap 函数

```go
// makemap implements Go map creation for make(map[k]v, hint).
func makemap(t *maptype, hint int, h *hmap) *hmap{
  //省略无关代码
}
```

makemap 函数返回的是 *hmap 类型，而 hmap 是一个结构体，它的定义如下面的代码所示：

```go
// src/runtime/map.go
// A header for a Go map.
type hmap struct {
   // Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
   // Make sure this stays in sync with the compiler's definition.
   count     int // # live cells == size of map.  Must be first (used by len() builtin)
   flags     uint8
   B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
   noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
   hash0     uint32 // hash seed
   buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
   oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
   nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
   extra *mapextra // optional fields
}
```

- map 关键字其实非常复杂，它包含 map 的大小 count、存储桶 buckets 等

- 要想使用这样的 hmap，不是简单地通过 new 函数返回一个 *hmap 就可以，`还需要对其进行初始化`，这就是 make 函数要做的事情



其实 make 函数就是 map 类型的工厂函数，它可以根据传递它的 K-V 键值对类型，创建不同类型的 map，同时可以初始化 map 的大小。

> 提示：make 函数不只是 map 类型的工厂函数，还是 chan、slice 的工厂函数。它同时可以用于 slice、chan 和 map 这三种类型的初始化。

- [map](#Map) 
- [chan](#chan) 
- [slice](#SliceHeader) 



#### 总结

- `new 函数只用于分配内存，并且把内存清零`，也就是返回一个指向对应类型零值的指针。

  - new 函数一般用于需要显式地返回指针的情况，不是太常用。

  

- `make 函数只用于 slice、chan 和 map 这三种内置类型的创建和初始化`，因为这三种类型的结构比较复杂。

  - 比如 slice 要提前初始化好内部元素的类型，slice 的长度和容量等，这样才可以更好地使用它们。





## 引用类型





**引用类型的零值都是 nil：** 

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/AllOfTypeInGo.png" style="zoom:50%;" />





在 Go 语言中，函数的参数传递`只有值传递`，而且传递的 *实参* `都是原始数据的一份拷贝`

- 如果拷贝的内容是值类型的，那么在函数中就`无法修改`原始数据

- 如果拷贝的内容是指针（或者可以理解为引用类型 map、chan 等），那么在函数中就`可以修改`原始数据











## Map



### 引用类型

```go
func main() {
   m:=make(map[string]int)
   m["will"] = 18
   fmt.Println("will的年龄为",m["will"])
   modifyMap(m)
   fmt.Println("will的年龄为",m["will"])
}
func modifyMap(p map[string]int)  {
   p["will"] =20
}
// 成功修改：
// will的年龄为 18
// will的年龄为 20
```



### if 判断

```go
// 判断key是否存在

if value, ok := map[key]; ok {
    // 存在
}
if value, ok := map[key]; !ok {
    // 不存在
}
```







### for 循环

```go
// 不关心索引和数据的情况
for range m {}

// 只关心索引的情况,只遍历键, 无须将值改为匿名变量形式，忽略值即可
for kwy := range m {}

// 关心索引和数据的情况
for kwy, value := range m {}


// 如果需要特定顺序的遍历结果，正确的做法是先排序

```





### hmap



用`字面量`或者`make函数`的方式创建 map，都转换成 makemap 函数的调用，这个转换是 **Go 语言编译器自动做的** 

```go
// src/runtime/map.go
// makemap implements Go map creation for make(map[k]v, hint).
func makemap(t *maptype, hint int, h *hmap) *hmap{
  //省略无关代码
}
```

 Go 语言的 map 类型本质上就是 `*hmap`, **本质上就是个指针** 











## chan



Go 语言并发模块中的 `channel`, **本质上也是个指针** : 

创建的 chan 其实是个 *hchan，所以它在参数传递中也和 map 一样

```go
func makechan(t *chantype, size int64) *hchan {
    //省略无关代码
}
```

严格来说，Go 语言没有引用类型，但是我们可以把 map、chan 称为**引用类型**，**为了便于理解**。

除了 map、chan 之外，Go 语言中的`函数`、`接口`、`slice 切片`都可以称为**引用类型** 









## 切片&数组





### 数组

```go
a1:=[1]string{"张三"}
a2:=[2]string{"李四"}
```

- 变量 a1 的大小是 `1`，内部元素的类型是 string，也就是说 a1 `最多只能存储 1 个类型为 string 的元素`
- 变量 a2 的大小是 `2`，内部元素的类型也是 string，所以 a2 `最多可以存储 2 个类型为 string 的元素` 



数组由两部分构成：`数组的大小`和`数组内的元素类型` 

```go
//数组结构伪代码表示
array{
  len
  item type
}
```



**数组的限制：** 

- 一旦一个数组被声明，它的`大小`和`内部元素的类型`就**不能改变**，**不能随意**地向数组添加任意多个元素

- 函数间的传参是`值传递`，数组作为参数在各个函数之间被传递的时候，同样的`内容就会被一遍遍地复制`，这就会造成大量的内存浪费





### slice 切片

​		`为了解决数组以上的限制`，Go 语言创造了 slice，也就是切片。

> 切片是 对数组的 **抽象** 和 **封装** 
>
> 底层是一个数组存储所有的元素，但是它`可以动态地添加元素`，容量不足时还可以`自动扩容`



切片，就是`动态数组` 

```go
s := []string{"张三", "李四"}
// 区别于数组，没有容量个数
// a := [2]string{"张三", "李四"}
```





### 动态扩容

```go
func main() {
   // 定义slice并初始化
   ss := []string{"will","张三"}
   fmt.Println("切片ss长度为",len(ss),",容量为",cap(ss))
    
   // 动态扩容
   ss=append(ss,"李四","王五")
   fmt.Println("切片ss长度为",len(ss),",容量为",cap(ss))
    
   fmt.Println(ss)
}
// 切片ss长度为 2 ,容量为 2
// 切片ss长度为 4 ,容量为 4
// [will 张三 李四 王五]
```

**append 自动扩容的原理**：

1. `新建`一个底层数组
2. 把原来切片内的元素`拷贝`到新数组中
3. 最后返回一个`指向新数组`的切片





### 数据结构

```go
type SliceHeader struct {
   Data uintptr
   Len  int
   Cap  int
}
```

SliceHeader 是切片`在运行时`的表现形式：

- Data 用来`指向`存储切片元素的数组
- Len 代表切片的`长度` 
- Cap 代表切片的`容量 ` 



`不同切片`对应的<u>底层 Data</u> `指向`的可能是`同一个数组`：

```go
func main() {
   // 定义并初始化数组
   a1:=[2]string{"will","张三"}
   
   // 根据数组定义slice
   s1:=a1[0:1]
   s2:=a1[:]
    
   //打印出s1和s2的Data值，是一样的
   fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&s1)).Data)
   fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&s2)).Data)
}
// 824634150744
// 824634150744
```

> 注意：
>
> 多个切片共用一个底层数组虽然可以减少内存占用，但是如果有一个切片修改内部的元素，其他切片也会受影响。
>
> 所以在切片作为参数在函数间传递的时候要小心，**<u>*尽可能不要修改原切片内的元素*</u>**。



### for range



对于数组和切片来说，Go 语言有三种不同的遍历方式：



```go
// 不关心索引和数据的情况
for range a {}

// 只关心索引的情况
for i := range a {}

// 关心索引和数据的情况
for i, elem := range a {}
```









### 高效的原因

- 从`集合类型`的角度考虑
  - 数组、切片和 map 都是集合类型，它们都可以存放元素，但是数组和切片的取值和赋值操作要更高效，因为它们是`连续的内存`操作，***通过索引就可以快速地找到元素存储的地址*** 
  - 当然 map 的价值也非常大，因为它的 Key 可以是很多类型，比如 int、int64、string 等
  - 数组和切片的`索引`只能是***整数***



- 从`内存大小`的角度考虑
  - 在数组和切片中，切片又是高效的，因为它在赋值、函数传参的时候，并不会把所有的元素都复制一遍，而只是复制 SliceHeader 的三个字段就可以了，共用的还是同一个底层数组
  - 切片的高效还体现在 for range 循环中，因为`循环得到的临时变量也是个值拷贝`，所以在遍历大的数组时，切片的效率更高



总结：切片`基于指针的封装`是它效率高的根本原因，因为可以减少内存的占用，以及减少内存复制时的时间消耗



### string <=> byte



#### 强制转换

 []byte(s) 和 string(b) 这种`强制转换`**会重新拷贝一份字符串** 

```go
s := "will"
fmt.Printf("s的内存地址：%d\n", (*reflect.StringHeader)(unsafe.Pointer(&s)).Data)

b := []byte(s)
fmt.Printf("b的内存地址：%d\n",(*reflect.SliceHeader)(unsafe.Pointer(&b)).Data)

s3 := string(b)
fmt.Printf("s3的内存地址：%d\n", (*reflect.StringHeader)(unsafe.Pointer(&s3)).Data)

fmt.Println(s,string(b),s3)		// 都是 "will"

// 打印出的内存地址都不一样
```

> 提示：
>
> 可以通过查看 runtime.stringtoslicebyte 和 runtime.slicebytetostring 这两个函数的源代码，了解关于 string 和 []byte 类型互转的具体实现。





```go
// StringHeader is the runtime representation of a string.
type StringHeader struct {
   Data uintptr		// 用于存放指向真实内容的指针
   Len  int
}
```

- 在程序运行的时候，字符串和切片本质上就是 StringHeader 和 SliceHeader



#### 零拷贝

```go
s:="will"
b:=[]byte(s)
// s3:=string(b)

// s4 没有申请新内存（零拷贝）,和变量 b 使用的是同一块内存，因为它们的底层 Data 字段值相同
s4 := *(*string)(unsafe.Pointer(&b))
```



 字符串转成byte数组：

```go
a := "aaa"
ssh := *(*reflect.StringHeader)(unsafe.Pointer(&a))
b := *(*[]byte)(unsafe.Pointer(&ssh))  
fmt.Printf("%v",b)
```

- `unsafe.Pointer(&a)`方法可以得到变量`a`的地址
- `(*reflect.StringHeader)(unsafe.Pointer(&a))` 可以把字符串a转成底层结构的形式
- `(*[]byte)(unsafe.Pointer(&ssh))` 可以把ssh底层结构体转成byte的切片的指针
-  再通过 `*`转为指针指向的实际内容



#### SliceHeader/StringHeader

- SliceHeader 有 Data、Len、Cap 三个字段

- StringHeader 有 Data、Len 两个字段

- 所以 *SliceHeader 通过 unsafe.Pointer 转为 *StringHeader 的时候没有问题，因为 *SliceHeader 可以提供 *StringHeader 所需的 Data 和 Len 字段的值

- 但是反过来却不行了，因为 *StringHeader 缺少 *SliceHeader 所需的 Cap 字段，需要我们自己补上一个默认值



```go
s:="will"
//b:=[]byte(s)

sh:=(*reflect.SliceHeader)(unsafe.Pointer(&s))
sh.Cap = sh.Len

b1:=*(*[]byte)(unsafe.Pointer(sh))
```



> 注意：
>
> 通过 unsafe.Pointer 把 string 转为 []byte 后，不能对 []byte 修改
>
> 比如不可以进行 b1[0]=12 这种操作，会报异常，导致程序崩溃。
>
> 这是因为在 Go 语言中 string 内存是只读的。









## reflect



### 概述

```go
func main() {
   i := 3
    
   // reflect.Value 是一个结构体
   iv := reflect.ValueOf(i)
    
   // reflect.Type 是一个接口
   it := reflect.TypeOf(i)
    
   fmt.Println(iv,it)
   // 3 int
}
```



### Value

 reflect.Value 和 int 类型互转：

```go
func main() {
   i := 3
   // int to reflect.Value
   iv := reflect.ValueOf(i)
    
   // reflect.Value to int
   i1 := iv.Interface().(int)
    
   fmt.Println(i1)
}
```

> interface{} 是空接口，可以表示任何类型，也就是说你可以把任何类型转换为空接口，它通常用于反射、类型断言，以减少重复代码，简化编程。



修改对应的值：

```go
func main() {
   i := 3
    
   // 因为 reflect.ValueOf 函数返回的是一份值的拷贝，所以要传入变量的指针才可以
   ipv := reflect.ValueOf(&i)
   
    //  因为传递的是一个指针，所以需要调用 Elem 方法找到这个指针指向的值，最后就可以使用 SetInt 方法修改值
   ipv.Elem().SetInt(4)
    
   fmt.Println(i)
}
```



那么如何修改 `struct 结构体字段` 的值呢？

1. 传递一个 struct 结构体的指针，获取对应的 reflect.Value；

2. 通过 Elem 方法获取指针指向的值；

3. 通过 Field 方法获取要修改的字段；

4. 通过 Set 系列方法修改成对应的值。

如：

```go
func main() {
   p := person{Name: "will",Age: 20}
   ppv := reflect.ValueOf(&p)
   ppv.Elem().Field(0).SetString("张三")
   fmt.Println(p)
}
type person struct {
   Name string
   Age int
}
```



通过反射修改一个值的规则：

1. 可被寻址，通俗地讲就是要向 reflect.ValueOf 函数`传递一个指针作为参数` 

2. 如果要修改 struct 结构体字段值的话，该字段需要是可导出的，而不是私有的，也就是`该字段的首字母为大写`

3. 记得使用 `Elem` 方法`获得指针指向的值`，这样才能调用` Set 系列`方法进行修改



获取对应的底层类型：

```go
func main() {
   p := person{Name: "will",Age: 20}
    
   pv := reflect.ValueOf(p)
   fmt.Println(pv.Kind())
   // struct
   
   ppv := reflect.ValueOf(&p)
   fmt.Println(ppv.Kind())
   // ptr
    
}
// 变量 p 的实际类型是 person，但是 person 对应的底层类型是 struct 这个结构体类型，而 &p 对应的则是指针类型
```





### Type

和 reflect.Value 不同，reflect.Type `是一个接口`，而不是一个结构体，所以只能使用它的方法。

```go
type Type interface {

   Implements(u Type) bool		// 是否实现了接口 u
   AssignableTo(u Type) bool	// 是否可以使用 =
   ConvertibleTo(u Type) bool	// 是否可以进行类型转换
   Comparable() bool			// 是否可以使用关系运算符进行比较
   
   // 以下这些方法和Value结构体的功能相同
   Kind() Kind
   Method(int) Method
   MethodByName(string) (Method, bool)
   NumMethod() int
   Elem() Type
   Field(i int) StructField
   FieldByIndex(index []int) StructField
   FieldByName(name string) (StructField, bool)
   FieldByNameFunc(match func(string) bool) (StructField, bool)
   NumField() int
}  
```

```go
type person struct {
   Name string
   Age int
}
func (p person) String() string{
   return fmt.Sprintf("Name is %s,Age is %d", p.Name, p.Age)
}
```

**遍历结构体的字段和方法**：

```go
func main() {
   p := person{Name: "will",Age: 20}
   pt := reflect.TypeOf(p)
    
   //遍历person的字段
   for i:=0; i<pt.NumField(); i++{
      fmt.Println("字段：",pt.Field(i).Name)
   }
    
   //遍历person的方法
   for i:=0; i<pt.NumMethod(); i++{
      fmt.Println("方法：",pt.Method(i).Name)
   }
}
// 输出：
// 字段： Name
// 字段： Age
// 方法： String
```

> 可以通过 FieldByName 方法获取指定的字段，也可以通过 MethodByName 方法获取指定的方法，获取某个特定的字段或者方法时非常高效，而不是使用遍历。



**是否实现某接口**：

```go
func main() {
   p := person{Name: "will",Age: 20}
   pt := reflect.TypeOf(p)
    
   // 判断是否实现了接口 fmt.Stringer 和 io.Writer
   // elem 获得指针指向的值
   stringerType := reflect.TypeOf((*fmt.Stringer)(nil)).Elem()
   writerType := reflect.TypeOf((*io.Writer)(nil)).Elem()
    
   fmt.Println("是否实现了fmt.Stringer：",pt.Implements(stringerType))
   fmt.Println("是否实现了io.Writer：",pt.Implements(writerType))
}
// 是否实现了fmt.Stringer： true
// 是否实现了io.Writer： false
```

> 尽可能通过类型断言的方式判断是否实现了某接口，而不是通过反射







### JSON <=> Struct 

```go
func main() {
   p := person{Name: "will",Age: 20}
    
   //struct to json
   jsonB,err := json.Marshal(p)
   if err == nil {
      fmt.Println(string(jsonB))
   }
    
   //json to struct
   respJSON:="{\"Name\":\"李四\",\"Age\":40}"
   json.Unmarshal([]byte(respJSON), &p)
   fmt.Println(p)
}
// {"Name":"飞雪无情","Age":20}
// Name is 李四,Age is 40
```

- JSON 字符串的 Key 和 struct 结构体的字段名称一样，比如示例中的 Name 和 Age
- 想把输出的 json 字符串的 Key 改为小写的 name 和 age，可以通过为 struct 字段添加 tag 的方式





### Struct Tag



struct tag 是一个`添加在 struct 字段上的标记` ，使用它进行辅助，可以完成一些额外的操作



比如 json 和 struct 互转：

```go
type person struct {
   Name string `json:"name"`
   Age int `json:"age"`
}
```

> 重点：
>
> json 作为 Key，是 Go 语言自带的 json 包解析 JSON 的一种==约定==
>
> 它会通过 json 这个 Key 找到对应的值，*用于*  <u>JSON 的 Key 值</u>



 tag 就像是我们为 struct 字段起的别名，那么 json 包是如何获得这个 tag 的呢？

- 反射

```go
//遍历person字段中key为json的tag
for i:=0; i<pt.NumField(); i++{
    
   //  Field 方法返回一个 StructField 结构体，有一个字段是 Tag，存有字段的所有 tag
   sf:=pt.Field(i)
   
   fmt.Printf("字段%s上,json tag为%s\n",sf.Name, sf.Tag.Get("json"))
}
```



结构体的字段可以有多个 tag，用于不同的场景

- 比如 json 转换、bson 转换、orm 解析等
- 如果有多个 tag，要使用`空格分隔`
- 采用不同的 Key 可以获得不同的 tag

```go
type person struct {
   Name string `json:"name" bson:"b_name"`
   Age int `json:"age" bson:"b_age"`
}

// 遍历person字段中key为json、bson的tag
for i:=0; i<pt.NumField(); i++{
   sf := pt.Field(i)
   fmt.Printf("字段%s上,json tag为%s\n",sf.Name, sf.Tag.Get("json"))
   fmt.Printf("字段%s上,bson tag为%s\n",sf.Name, sf.Tag.Get("bson"))
}
// 字段Name上,key为json的tag为name
// 字段Name上,key为bson的tag为b_name
// 字段Age上,key为json的tag为age
// 字段Age上,key为bson的tag为b_age
```



**实现 Struct 转 JSON**：

```go
func main() {
   p := person{Name: "will",Age: 20}
   pv := reflect.ValueOf(p)
   pt := reflect.TypeOf(p)
    
   // 自己实现的struct to json
   jsonBuilder := strings.Builder{}
   jsonBuilder.WriteString("{")
   
   num := pt.NumField()
    
   for i := 0; i<num; i++{
      jsonTag := pt.Field(i).Tag.Get("json") //获取json tag
      jsonBuilder.WriteString("\""+jsonTag+"\"")
      jsonBuilder.WriteString(":")
      //获取字段的值
      jsonBuilder.WriteString(fmt.Sprintf("\"%v\"",pv.Field(i)))
      if i<num-1{
         jsonBuilder.WriteString(",")
      }
   }
   jsonBuilder.WriteString("}")
   fmt.Println(jsonBuilder.String())//打印json字符串
}
// {"name":"飞雪无情","age":"20"}
```



json 字符串的转换只是 struct tag 的一个应用场景

- 完全可以把 struct tag 当成结构体中字段的元数据配置，使用它来做想做的任何事情

- 比如 orm 映射、xml 转换、生成 swagger 文档等









### 反射定律



[反射的三大定律](https://blog.golang.org/laws-of-reflection)：

1. 任何接口值，interface{} 都可以反射出反射对象，也就是 reflect.Value 和 reflect.Type，通过函数 reflect.ValueOf 和 reflect.TypeOf 获得。

2. 反射对象也可以还原为 interface{} 变量，也就是第 1 条定律的`可逆性`，通过 reflect.Value 结构体的 Interface 方法获得。

3. 要修改反射的对象，该值必须可设置，也就是`可寻址`。



> 提示：
>
> 任何类型的变量都可以转换为空接口 intferface{}，所以第 1 条定律中函数 reflect.ValueOf 和 reflect.TypeOf 的参数就是 interface{}，表示可以把任何类型的变量转换为反射对象。
>
> 在第 2 条定律中，reflect.Value 结构体的 Interface 方法返回的值也是 interface{}，表示可以把反射对象还原为对应的类型变量。



### 总结



在反射中：

- reflect.Value 对应的是`变量的值`，如果你需要进行和变量的***值有关*** 的操作，应该优先使用 reflect.Value比如获取变量的值、修改变量的值等。
- reflect.Type 对应的是`变量的类型`，如果你需要进行和变量的***类型本身有关*** 的操作，应该优先使用 reflect.Type，比如获取结构体内的字段、类型拥有的方法集等。







## Unsafe



原因：

- Go 的设计者为了编写方便、提高效率且降低复杂度，将其设计成一门`强类型`的`静态`语言
- 强类型意味着，一旦定义了，类型就不能改变
- 静态意味着，在运行前，做了类型检查
- 同时出于安全考虑，Go 语言是`不允许两个指针类型进行转换` 



### unsafe.Pointer

**如 *int 不能转为 *float64** ：

```go
func main() {
   i:= 10
   ip:=&i
   var fp *float64 = (*float64)(ip)
   fmt.Println(fp)
}
// cannot convert ip (type * int) to type * float64
```



**但如果还是需要转换呢？？？** 

`unsafe.Pointer` 是一种特殊意义的指针，==可以表示任意类型的地址==，类似 C 语言里的 void* 指针，**<u>*是全能型的*</u>**

```go
func main() {
   i:= 10
   ip:=&i
    
   var fp *float64 = (*float64)(unsafe.Pointer(ip))
    
   *fp = *fp * 3
   fmt.Println(i)
}
// 30
// 原来变量 i 的值也被改变了，变为 30
```





### uintptr

```go
// uintptr is an integer type that is large enough 
// to hold the bit pattern of any pointer.
// 足够大，可以表示任何指针
type uintptr uintptr
```

> uintptr 可以对`指针偏移`*进行计算*，这样就**可以访问特定的内存**，*达到对特定内存读写的目的*，真正内存级别的操作



**通过指针偏移修改 struct 结构体内的字段**：

```go
func main() {
   p := new(person)
    
   // Name是person的第一个字段不用偏移，即可通过指针修改
   pName := (*string)(unsafe.Pointer(p))
   *pName = "will"
    
   // Age并不是person的第一个字段，所以需要进行偏移，这样才能正确定位到Age字段这块内存，才可以正确的修改
   pAge := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(p))+unsafe.Offsetof(p.Age)))
   *pAge = 20
   fmt.Println(*p)
}
type person struct {
   Name string
   Age int
}
```

- 因为 Age 字段不是 person 的第一个字段，要修改它必须要进行指针偏移运算
- 所以需要先把指针变量 p 通过 unsafe.Pointer 转换为 uintptr，这样才能进行地址运算
- 偏移量可以通过函数 unsafe.Offsetof 计算出来，该函数返回的是一个 uintptr 类型的偏移量





### unsafe.Sizeof

Sizeof 函数可以返回`一个类型所占用的内存大小` ，这个大小**只与类型有关**，*和类型对应的变量存储的内容大小无关*

- 比如 bool 型占用一个字节、int8 也占用一个字节



```go
fmt.Println(unsafe.Sizeof(true))
fmt.Println(unsafe.Sizeof(int8(0)))
fmt.Println(unsafe.Sizeof(int16(10)))
fmt.Println(unsafe.Sizeof(int32(10000000)))
fmt.Println(unsafe.Sizeof(int64(10000000000000)))
fmt.Println(unsafe.Sizeof(int(10000000000000000)))
fmt.Println(unsafe.Sizeof(string("will")))
fmt.Println(unsafe.Sizeof([]string{"李四","张三"}))
```



> 一个 struct 结构体的内存占用大小，等于它包含的字段类型内存占用大小之和。







### 指针转换规则

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/PointerExchange.png)

1. 任何类型的 *T 都可以转换为 unsafe.Pointer
2. unsafe.Pointer 也可以转换为任何类型的 *T
3. unsafe.Pointer 可以转换为 uintptr
4. uintptr 也可以转换为 unsafe.Pointer



- unsafe.Pointer 主要用于指针类型的转换，是各个`指针类型转换`的桥梁
- uintptr 主要用于`指针运算`，尤其是通过偏移量定位不同的内存





## atomic

 [atomic.Value 的前世今生](https://blog.betacat.io/post/golang-atomic-value-exploration/)：“原子的"存储（Store）和加载（Load）**任意类型**的值

```go
// A Value provides an atomic load and store of a consistently typed value.
// The zero value for a Value returns nil from Load.
// Once Store has been called, a Value must not be copied.
//
// A Value must not be copied after first use.
type Value struct {
	v interface{}
}

// Load returns the value set by the most recent Store.
// It returns nil if there has been no call to Store for this Value.
func (v *Value) Load() (x interface{}) {
	......
}

// Store sets the value of the Value to x.
// All calls to Store for a given Value must use values of the same concrete type.
// Store of an inconsistent type panics, as does Store(nil).
func (v *Value) Store(x interface{}) {
    .....
}
```











## 单元测试



### 实列

```go
// ch18/main.go
func Fibonacci(n int) int {
   if n < 0 {
      return 0
   }
   if n == 0 {
      return 0
   }
   if n == 1 {
      return 1
   }
   return Fibonacci(n-1) + Fibonacci(n-2)
}
```

```go
// ch18/main_test.go
func TestFibonacci(t *testing.T) {
   //预先定义的一组斐波那契数列作为测试用例
   fsMap := map[int]int{}
   fsMap[0] = 0
   fsMap[1] = 1
   fsMap[2] = 1
   fsMap[3] = 2
   fsMap[4] = 3
   fsMap[5] = 5
   fsMap[6] = 8
   fsMap[7] = 13
   fsMap[8] = 21
   fsMap[9] = 34
   for k, v := range fsMap {
      fib := Fibonacci(k)
      if v == fib {
         t.Logf("结果正确:n为%d,值为%d", k, fib)
      } else {
         t.Errorf("结果错误：期望%d,但是计算的值是%d", v, fib)
      }
   }
}
```

```shell
 # 运行如下命令，进行单元测试：
 go test -v ./ch18
```

```shell
➜ go test -v ./ch18 
=== RUN   TestFibonacci
    main_test.go:21: 结果正确:n为0,值为0
    main_test.go:21: 结果正确:n为1,值为1
    main_test.go:21: 结果正确:n为6,值为8
    main_test.go:21: 结果正确:n为8,值为21
    main_test.go:21: 结果正确:n为9,值为34
    main_test.go:21: 结果正确:n为2,值为1
    main_test.go:21: 结果正确:n为3,值为2
    main_test.go:21: 结果正确:n为4,值为3
    main_test.go:21: 结果正确:n为5,值为5
    main_test.go:21: 结果正确:n为7,值为13
--- PASS: TestFibonacci (0.00s)
PASS
ok      gotour/ch18     (cached)
```





### 五点规则



1. 含有单元测试代码的 go 文件必须`以 _test.go 结尾`，Go 语言测试工具只认符合这个规则的文件
2. 单元测试文件名 _test.go `前面的部分`，最好是被测试的函数所在的 go 文件的文件名
   - 比如以上示例中单元测试文件叫 main_test.go，因为测试的 Fibonacci 函数在 main.go 文件里

3. 单元测试的函数名必须`以 Test 开头`，是可导出的、公开的函数
4. 测试函数的签名必须`接收一个指向 testing.T 类型的指针`，并且`不能返回任何值`

5. 函数名最好是 `Test + 要测试的函数名`
   - 比如例子中是 TestFibonacci，表示测试的是 Fibonacci 这个函数。



### 覆盖率



```shell
go test -v --coverprofile=ch18.cover ./ch18
```

- 可以得到一个单元测试覆盖率文件，运行这行命令还可以同时看到测试覆盖率

```shell
PASS
coverage: 85.7% of statements
ok      gotour/ch18     0.367s  coverage: 85.7% of statements
```



**查看详细的单元测试覆盖率报告**：

```shell
go tool cover -html=ch18.cover -o=ch18.html
```

命令运行后，会在当前目录下生成一个 ch18.html 文件，使用浏览器打开它，查看内容





###  race 检测



 ```shell
go run -race main.go 
 ```



- 编译器会通过探测所有的内存访问，监听其内存地址的访问（读或写）
- 在应用运行时就能够发现对共享变量的访问和操作，进而发现问题并打印出相关的警告信息



需要注意的一点是，`go run -race` 是运行时检测，并不是编译时。且 race 存在明确的性能开销，通常是正常程序的**十倍**，因此不要想不开在生产环境打开这个配置，很容易翻车。







### 基准测试

```go
// ch18/main_test.go
func BenchmarkFibonacci(b *testing.B){
   for i:=0;i<b.N;i++{
      Fibonacci(10)
   }
}
```

1. 基准测试函数必须`以 Benchmark 开头`，必须是可导出的
2. 函数的签名必须接收一个`指向 testing.B 类型的指针`，并且不能返回任何值
3. 最后的 `for 循环` 很重要，*被测试的代码要放到循环里*
4. `b.N` 是基准测试框架提供的，表示循环的次数，因为需要反复调用测试的代码，才可以评估性能



```shell
#  -bench 表示，接受一个表达式作为参数，以匹配基准测试的函数
# "." 表示运行所有基准测试
➜ go test -bench=. ./ch18
goos: darwin
goarch: amd64
pkg: gotour/ch18
BenchmarkFibonacci-8     3461616               343 ns/op
PASS
ok      gotour/ch18     2.230s
```

- BenchmarkFibonacci-8 ，后面的 -8 ，表示运行基准测试时对应的 GOMAXPROCS 的值

- 基准测试的时间默认是 1 秒，也就是 1 秒调用 3461616 次、每次调用花费 343 纳秒

- 如果想让测试运行的时间更长，可以通过 -benchtime 指定，比如 3 秒

  ```shell
  go test -bench=. -benchtime=3s ./ch18
  ```

  



### 计时方法

> 避免因为准备数据耗时造成的干扰

进行基准测试之前会做一些准备，比如构建测试数据等，这些准备也需要消耗时间，所以需要`把这部分时间排除在外`

```shell
func BenchmarkFibonacci(b *testing.B) {
   n := 10
   b.ResetTimer() //重置计时器
   for i := 0; i < b.N; i++ {
      Fibonacci(n)
   }
}
```

- 除了 ResetTimer 方法外，还有 StartTimer 和 StopTimer 方法，灵活地控制什么时候开始计时、什么时候停止计时。



### 内存统计

统计每次操作分配内存的次数，以及每次操作分配的字节数

```go
func BenchmarkFibonacci(b *testing.B) {
   n := 10
   b.ReportAllocs() //开启内存统计
   b.ResetTimer() //重置计时器
   for i := 0; i < b.N; i++ {
      Fibonacci(n)
   }
}
```

```shell
➜ go test -bench=.  ./ch18
goos: darwin
goarch: amd64
pkg: gotour/ch18
BenchmarkFibonacci-8  2486265  486 ns/op  0 B/op  0 allocs/op
PASS
ok      gotour/ch18     2.533s
```

- ` 0 B/op `，表示每次操作分配了多少字节的内存
- `0 allocs/op`，表示每次操作分配内存的次数





### 逃逸分析

```go
// ch19/main.go
func newString() *string{
   s:=new(string)
   *s = "will"
   return s
}
```

- 通过 new 函数申请了一块内存；

- 然后把它赋值给了指针变量 s；

- 最后通过 return 关键字返回。



```shell
➜ go build -gcflags="-m -l" ./ch19/main.go
# command-line-arguments
ch19/main.go:16:8: new(string) escapes to heap
```

- -m 表示打印出逃逸分析信息
- -l 表示禁止内联
- 指针作为函数返回值的时候，一定会发生逃逸





**优化后的函数代码如下：** 

```go
func newString() string{
   s:=new(string)
   *s = "will"
   return *s
}
```

再次通过命令查看以上代码的逃逸分析，命令如下：

```shell
➜ go build -gcflags="-m -l" ./ch19/main.go
# command-line-arguments
ch19/main.go:14:8: new(string) does not escape
```





==并不是不使用指针就不会发生逃逸==

```go
fmt.Println("will")
```

```shell
➜ go build -gcflags="-m -l" ./ch19/main.go
# command-line-arguments
ch19/main.go:13:13: ... argument does not escape
ch19/main.go:13:14: "will" escapes to heap
ch19/main.go:17:8: new(string) does not escape
```

「will」**逃逸**到了堆上，这是因为「will」这个字符串**被已经逃逸的指针变量引用**，所以它也跟着逃逸了

```go
func (p *pp) printArg(arg interface{}, verb rune) {
   p.arg = arg
   //省略其他无关代码
}
```

- 被已经逃逸的指针引用的变量也会发生逃逸
-  被 slice、map 和 chan，这三种类型引用的指针也会发生逃逸，

```go
func main() {
   m:=map[int]*string{}
   s:="will"
   m[0] = &s
}
```

```shell
➜  gotour go build -gcflags="-m -l" ./ch19/main.go
# command-line-arguments
ch19/main.go:16:2: moved to heap: s
ch19/main.go:15:20: map[int]*string literal does not escape
```

- 变量 m 没有逃逸，**反而**被变量 m 引用的变量 s 逃逸到了堆上
- 所以被map、slice 和 chan 这三种类型引用的指针一定会发生逃逸的



> 指针虽然可以减少内存的拷贝，但它同样会引起逃逸，所以要根据实际情况选择是否使用指针。



#### 优化技巧



1. 尽可能避免逃逸，因为栈内存效率更高，还不用 GC。比如**小对象的传参**，array 要比 slice 效果好。
2. 如果避免不了逃逸，还是在堆上分配了内存，那么对于频繁的内存申请操作，我们要学会**重用内存**，比如使用 sync.Pool
3. 选用合适的算法，达到高性能的目的，比如**空间换时间** 
4. 尽可能避免使用锁、并发加锁的范围要尽可能小
5. 使用 StringBuilder 做 string 和 [ ] byte 之间的转换
6. defer 嵌套不要太多

> 小提示：性能优化的时候，要结合基准测试，来验证自己的优化是否有提升。











### 并发基准测试

```go
func BenchmarkFibonacciRunParallel(b *testing.B) {
   n := 10
   b.RunParallel(func(pb *testing.PB) {
      for pb.Next() {
         Fibonacci(n)
      }
   })
}
```

- 通过 RunParallel 方法运行并发基准测试
- RunParallel 方法会创建多个 goroutine，并将 b.N 分配给这些 goroutine 执行





### 接口+依赖注入



```go
type transaction struct {
    ID       string
    BuyerID  int
    SellerID int
    Amount   float64
    createdAt time.Time
    Status TransactionStatus
}

func (t *transaction) Execute() bool {
    if t.Status == Executed {
        return true
    }
    if time.Now() - t.createdAt > 24.hours { // 交易有有效期
        t.Status = Expired
        return false
    }
    client := BankClient.New(config.token) // 调用银行的 SDK 执行转账
    if err := client.TransferMoney(id, t.BuyerID, t.SellerID, t.Amount); err != nil {
        t.Status = Failed
        return false
    }
    t.Status = Executed
    return true
}
```

最重要的功能集中在`Execute()`函数中，但它却不好测试，因为它有两个外部依赖：

1. 行为不确定的`time.Now`函数，每一次调用都会产生不同的结果
2. 银行提供的转账 SDK，我们不可能每次测试都去真的调用一下，那测试成本也忒高了



```go
// 先将代码里使用到的方法抽象成一个接口
type Transferer interface {
    TransferMoney(id int, buyerID int, sellerID int, amount float64) error
}
```

放到`transaction`的成员属性中，重构后的`transaction`类及其构造函数就变成了这样：

```go
type transaction struct {
    ID       string
    BuyerID  int
    SellerID int
    Amount   float64
    createdAt time.Time
    Status TransactionStatus
    // 增加了一个存放接口的属性
    transferer Transferer
}

func New(buyerID, sellerID int, amount float64, transferer Transferer) *transaction {
    return &transaction{
        ID:         IdGenerator.generate(),
        BuyerID:    buyerID,
        SellerID:   sellerID,
        Amount:     amount,
        createdAt:  time.Now(),
        Status:     TO_BE_EXECUTD,
        transferer: transferer, // 注入进 transaction 类中
    }
}

func (t *transaction) Execute() bool {
    //...
    //不直接创建，而是使用别人注入的接口实例
    t.transferer.TransferMoney(id, t.BuyerID, t.SellerID, t.Amount)
    //...
}
```

现在，可以在单元测试中就能够很方便的替换掉那个成本高昂的支付接口的调用了

```go
// 定义一个满足 Transferer 接口的 mock 类
type MockedClient struct {
    responseError error // 实例化的时候可以将期望的返回值保存进来
}

func (m *MockedClient) TransferMoney(id int, buyerID int, sellerID int, amount float64) error {
    return m.responseError
}

func Test_transaction_Execute(t *testing.T) {
    // 实例化一个可以自由控制结果的 client
    transferer := &MockedClient{
        responseError: errors.New("insufficient balance"),
    }
    tnx := New(buyerID, sellerID, amount, transferer)
    if succeeded := tnx.Execute(); succeeded != false {
        t.Errorf("Execute() = %v, want %v", succeeded, false)
    }
}
```



```go
var nowFn = time.Now //一个全局变量，用来解耦time.Now的生产者和消费者

func (t *transaction) Execute() bool {
    if t.Status == Executed {
        return true
    }
    if nowFn() - t.createdAt > 24.hours { // 不直接调用time.Now()
        t.Status = Expired
        return false
    }
    //...
}
```

这样，单元测试就能随心所欲的改变“当前时间”了

```go
func Test_expired_transaction_Execute(t *testing.T) {
    // 用同样的函数签名改写业务中需要的时间函数
    // 这里能改变私有的全局变量是因为测试代码跟业务代码处于同一个包中
    nowFn = func() time.Time {
        return time.Now().Add(-24 * time.Hour)
    }
    // 依旧需要实例化一个假的的 client
    transferer := &MockedClient{
        responseError: nil,
    }
    tnx := New(buyerID, sellerID, amount, transferer)
    //...
}
```

隐式的复用了`time.Now`的**函数签名**，将它当做一种“接口类型”来使用



参考：[编写可测试 Go 代码的一种模式](https://blog.betacat.io/post/2020/03/a-pattern-for-writing-testable-go-code/#mock-timenow) 













## 服务监控







[pprof](https://segmentfault.com/a/1190000016412013)：runtime / net



[go-torch](https://github.com/uber-archive/go-torch) 



 [golang性能诊断](https://mp.weixin.qq.com/s/yZUE8N-Qb-AB81DgA1cV_w)：`pprof` / `trace` / `linux`





















## 网络编程 





[embed](https://www.flysnow.org/2021/02/28/golang-embed-for-web.html)：1.16 新增的embed在各流行Web框架中的应用





[Gin实战](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1362784031968149504&__biz=MzI3MjU4Njk3Ng==#wechat_redirect)：Go 语言如何玩转 RESTful API 服务？？



 [Restful API](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484393&idx=1&sn=4a27e735672c820ac266484333a806b7&chksm=eb310266dc468b7081405477779b75d2b565ec8255143338f6b037f1725f91a7a468b890f7b1&scene=178&cur_album_id=1362784031968149504#rd) ：`net/http`请求的URL需要一个个去注册，Gin提供了URL路由的**模糊匹配**，比如URL路径中的参数

 [路由参数](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484397&idx=1&sn=05d25b404f117cca76dd2097d3befab3&chksm=eb310262dc468b74724a03d345e9b3ef21b39a38682836f4b00683d7b04fc7264a41649d6a0e&scene=178&cur_album_id=1362784031968149504#rd)：`/users/id` 通过这个`id`参数，获取对应的用户信息

```go
func main() {
    r := gin.Default()
    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        c.String(200, "The user id is  %s", id)
    })
    r.Run(":8080")
}
// 输入 http://localhost:8080/users/123
// The user id is  123
```

```go
Pattern: /users/:id

/users/123          匹配
/users/哈哈        匹配
/users/123/go      不匹配
/users/             不匹配
```



```go
Pattern: /users/*id

/users/123         匹配
/users/哈哈        匹配
/users/123/go      匹配
/users/            匹配
```



 [HTTP路由httprouter](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484116&idx=1&sn=8abf9049e47386d503d7b653541cce16&chksm=eb31035bdc468a4d75b42d2f3ad82154b88665aeb46b083f466499942bb41f3483117edc1e24&scene=21#wechat_redirect)：自定义路由

 

 [URL查询参数的获取和原理分析](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484405&idx=1&sn=39eead58bac34c9b93e37f0832537e53&chksm=eb31027adc468b6c9f6a8df901fc45dca2c2ad1d528e6c93a6193e15c6e8df322c72a4cdc811&scene=178&cur_album_id=1362784031968149504#rd)：Gin获取查询参数 `Query`源码分析 



 [接收数组和 Map](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484410&idx=1&sn=807f89e1d6ba0f96558a66b82b08358b&chksm=eb310275dc468b63fd5f94a772413edb0c223f0bae213055f14494dc0ff76e99a66b98ffe3ac&scene=178&cur_album_id=1362784031968149504#rd)：`QueryArray`  / `QueryMap` 

 

 [获取Form表单参数](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484425&idx=1&sn=2a604e79d19c091410291d690d308dd5&chksm=eb310586dc468c90eb0de31503ca6c2da9fa2b29ccb22eb5f3e37ed26d3b9a60fdbc7d6fba59&scene=178&cur_album_id=1362784031968149504#rd)：

| 查询参数      | Form表单         | 说明                                    |
| :------------ | :--------------- | :-------------------------------------- |
| Query         | PostForm         | 获取key对应的值，不存在为空字符串       |
| GetQuery      | GetPostForm      | 多返回一个key是否存在的结果             |
| QueryArray    | PostFormArray    | 获取key对应的数组，不存在返回一个空数组 |
| GetQueryArray | GetPostFormArray | 多返回一个key是否存在的结果             |
| QueryMap      | PostFormMap      | 获取key对应的map，不存在返回空map       |
| GetQueryMap   | GetPostFormMap   | 多返回一个key是否存在的结果             |
| DefaultQuery  | DefaultPostForm  | key不存在的话，可以指定返回的默认值     |



 [分组路由源代码分析](https://mp.weixin.qq.com/s?__biz=MzI3MjU4Njk3Ng==&mid=2247484434&idx=1&sn=0bd93e3ab59e4bf5f9ed2a965395647e&chksm=eb31059ddc468c8b4bd003f680aaacf9a7cf2e9d539a5e20a7e4dca97e53cf1bf3d323aac923&scene=178&cur_album_id=1362784031968149504#rd)：路由中间件 / 分组路由嵌套











## gRPC



编译Proto文件，并生成语句：

```shell
protoc --go_out=plugins=grpc:. ./proto/*.proto
```











参考：

[gRPC 的使用和了解](https://golang2.eddycjy.com/posts/ch3/03-simple-grpc/) 

[gRPC服务注册发现及负载均衡的实现方案与源码解析](https://juejin.cn/post/6887388610367553549) ：`Resolver` / `Balancer` / `RoundRobin` 









## GMP



### 定义



1. G — 表示 Goroutine，它是一个待执行的任务；
2. M — 表示操作系统的线程，它由*操作系统*的“调度器” 调度和管理；
3. P — 表示处理器，它可以被看做运行在线程上的**本地调度器**；



可以在程序中使用 [`runtime.GOMAXPROCS`](https://draveness.me/golang/tree/runtime.GOMAXPROCS) 来改变最大的活跃线程数

<img src="go.assets/image-20210524211104374.png" alt="image-20210524211104374" style="zoom:67%;" />



- 默认情况下，四核机器会创建四个活跃的操作系统线程，每一个线程都对应一个运行时中的 [`runtime.m`](https://draveness.me/golang/tree/runtime.m) 结构体



### P与 G / M 

```c
struct P {
	Lock;

	uint32	status;
	P*	link;
	uint32	tick;
	M*	m;				// 反向持有一个线程
	MCache*	mcache;

	G**	runq;			// 可运行的 Goroutine 组成的环形的运行队列
	int32	runqhead;
	int32	runqtail;
	int32	runqsize;

	G*	gfree;
	int32	gfreecnt;
};
```





 <img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/GMPOfGolang.png" style="zoom:67%;" />



- 调度器在调度时，会从处理器(P)的队列中，选择*队列头*的 Goroutine 放到线程 M 上执行









### GMP & Thread



- 调度器通过使用与 CPU 数量相等的线程（CPU数量 == 线程数量）减少线程频繁切换的内存开销

- 同时，在每一个线程上，执行额外开销更低的 `Goroutine` 来，**降低**操作系统和硬件的负载











 [Goroutine 数量控制在多少合适，会影响 GC 和调度？](https://mp.weixin.qq.com/s/uWP2X6iFu7BtwjIv5H55vw) 













## 内存分配





<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/GoWithMemoryLayout.png" style="zoom:67%;" />



对应的数据结构 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan)、[`runtime.mcache`](https://draveness.me/golang/tree/runtime.mcache)、[`runtime.mcentral`](https://draveness.me/golang/tree/runtime.mcentral) 和 [`runtime.mheap`](https://draveness.me/golang/tree/runtime.mheap) 







## 垃圾收集







参考： [7.2 垃圾收集器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/#72-垃圾收集器) 



## 面向对象



 [golang面向对象分析](https://www.cnblogs.com/457220157-FTD/p/14703692.html)：封装 / 继承 / 多态



封装：struct





继承：

- ​    匿名组合    (Pseudo is-a)
- 非匿名组合    (has-a)





多态：接口







