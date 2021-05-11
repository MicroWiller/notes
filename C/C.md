# C



## Hello word

```c
#include <stdio.h>

int main() {
	printf("Hello World!!!");
	return 0;
}
```



## 基础



### 变量

> 标志着，一个内存位置的名称；通过这个名称，可以找到这个位置的号码，更改里面的值

![1603597011069](C.assets/1603597011069.png)



使用const关键字修饰变量：只读

```c
const int price = 17;
```



const修饰指针：

```c
const cnum = 520;
const *pc = &cnum;
```

不可以修改指针指向的地址的值

```c
*pc = 1024;
// error: assignment of read-only location '*pc'
```

指针pc，可以更改pc指向的地址

```c
const num = 666;
pc = &num;
```





### 关键字



<img src="C.assets/1601789352245.png" alt="1601789352245" style="zoom:67%;" />



### 数据类型

<img src="C.assets/1601792135022.png" alt="1601792135022" style="zoom:67%;" />



<img src="C.assets/1601789438649.png" alt="1601789438649" style="zoom:67%;" />

Java中：

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/\~

<img src="C.assets/1601792208740.png" alt="1601792208740" style="zoom:50%;" />

[printf 格式化输出不同的数据类型](https://fishc.com.cn/forum.php?mod=viewthread&tid=66471&extra=page%3D1%26filter%3Dtypeid%26typeid%3D583)   



### 内存中数据类型

![1601875189814](C.assets/1601875189814.png)



<img src="C.assets/1601875290208.png" alt="1601875290208" style="zoom:80%;" />



### 常量

> 不可以改变的值



```c
// 常量指针，指针本身不可以改变
int * const p = &num;
```



<img src="C.assets/1601791421516.png" alt="1601791421516" style="zoom:67%;" />



- 定义符号常量

  <img src="C.assets/1601791486489.png" alt="1601791486489" style="zoom:67%;" />





- 取值范围

  [signed](https://www.bilibili.com/video/av27744141?p=7) / unsigned

  [视频解说](https://www.bilibili.com/video/av27744141?p=7)		[补码原理](https://fishc.com.cn/forum.php?mod=viewthread&tid=67124&extra=page%3D1%26filter%3Dtypeid%26typeid%3D571)		





### 字符串

==字符串就是指向字符的指针== 

```c
char 变量名[数量];
变量名[索引号] = 字符;
// 定义字符串
char name[5] = {'W', 'i', 'l', 'l', '\0'};
// 或者直接写中括号
char name[] = {'W', 'i', 'l', 'l', '\0'};
// 或者字符串常量
char name[] = {"will"};
// 最后去掉大括号
char name[] = "will";
```



> 对比Java：Java有单独的String数据类型



> 字符串处理函数：
>
> - 获取字符串长度：strlen (区别于sizeof函数)
> - 拷贝字符串：strcpy / strcpy(限制源字符串拷贝过去的长度，并且不会自动追加一个结束符)
> - 连接字符串：strcat / strncat (这里可以自动追加一个结束符，catenate)
> - 比较字符串：strcmp / strncmp



### 数组

初始化：

​	需要注意的是，可以只给一部分元素赋值，==未被赋值的元素自动初始化为0== 

```c
int a[10] = {1, 2, 3, 4, 5, 6};
// 表示为前面6个元素赋值， 后面4个元素系统自动初始化为0
```

```java
// Java中，定义数组的方式： 
int[] a = new int[]{1, 2, 3, 4, 5, 6};
```

```go
// go中，定义数组的方式
var array [5]int = [5]int{1, 2, 3, 4, 5}
// 或者
array := [5]int{1, 2, 3, 4, 5}
```

```python
# python 数组中可以存放不同的数据类型
array = [1,2,3,'a']
```



> C语言中的数组比较特别:
>
> 通过**函数传递**一个数组的时候基于引用语义，
>
> 但是在结构体中**定义数组变量**的时候基于值语义（表现在为结构体赋值的时候，该数组会被完整地复制）。



- 二维数组在内存中的存储方式

  <img src="C.assets/1602040750202.png" alt="1602040750202" style="zoom:67%;" />





### 算数运算符

<img src="C.assets/1601802478240.png" alt="1601802478240" style="zoom:67%;" />





### 优先级



![1603106694361](C.assets/1603106694361.png)



## 指针



### 内存中

> 普通变量存放的是数据，指针变量存放的是地址
>
> 指针占4个字节，来存放一个地址（内存中）



<img src="C.assets/1601875493146.png" alt="1601875493146" style="zoom:67%;" />

> 变量pa 对应 地址11000，该地址里面装的是 **变量a**对应的`地址10000`；即，`地址10000` 称为指针，`pa` 称为指针变量



### 定义



```c
// 类型名 *指针变量名

char *pa;	// 定义一个指向字符型的指针变量

int *pb;	// 定义一个指向整型的指针变量

// 指针变量(pa)中存放的地址(10000), 指向的内存单元(地址10000中存放的值)的数据类型
```



go语言的指针定义不一样：

```go
var pb *int
```





### 取地址运算符和 &

如果需要获取某个变量的地址，可以使用 **&** 

```c
char *pa = &a;
int *pb = &f;
```





### 取值运算符 *

如果需要访问指针变量指向的数据，可以使用 ***** 

```c
printf("%c, %d\n", *pa, *pb);
```



取值运算符也称为间接运算符



```c
#include <stdio.h>

int main () {
	char a = 'F';
	int f = 14;
	
	char *pa = &a; 		// char *pa (定义指针)；&a (取变量a的地址) 
	int *pf = &f;		// char *pf (定义指针)；&f (取变量f的地址)
	 
	printf("pa = %c\n", *pa);
	printf("pf = %d\n", *pf);
	
	*pa = 'W';
	*pf = 7;
	
	printf("now, pa = %c\n", *pa);
	printf("now, pf = %d\n", *pf);
	
	printf("now, a = %c\n", a);
	printf("now, f = %d\n", f);
	
	printf("sizeof pa = %d\n", sizeof(pa));
	printf("sizeof pf = %d\n", sizeof(pf));
	
	printf("the addr of a is:  %p\n", pa);
	printf("the addr of f is:  %p\n", pf);
	
	return 0;
}
// pa = F
// pf = 14
// now, pa = W
// now, pf = 7
// now, a = W
// now, f = 7
// sizeof pa = 8
// sizeof pf = 8
// the addr of a is:  000000000062FE0F
// the addr of f is:  000000000062FE08
```



### 避免访问未初始化的指针

```c
#include <stdio.h>

int main() {
	
	int *a;
	
	*a = 1234;
	
	return 0;
}
```

> a, 野指针，对随机的地址进行赋值，很危险

> 对比Java：Java中的对象引用，默认初始化为null；



### 间接访问

```c
#include <stdio.h>

int main() {
	int a;
	int *p = &a;
	
	printf("请输入一个整数：");
	scanf("%d", &a);
	printf("a = %d\n", a);
	
	printf("请再输入一个整数：");
	// 间接改变 变量a的值 
	scanf("%d", p);
	printf("a = %d\n", a);
	
	return 0; 
} 
```



### 数组名的真实身份

```c
#include <stdio.h>

int main(){
	
	char str[128];
	
	printf("请输入你的名字：");
	
	scanf("%s", str);	// 这里并没有使用 &来取str的地址 
	
	printf("你的名字是：%s\n", str);
	
    	
	// %p : 用来打印地址的 
	printf("str 的地址是：%p\n", str);
	// 第一个元素是字符，所有要用&取其地址 
	printf("str 第一个元素的地址是：%p\n", &str[0]);
    
	return 0;
} 

请输入你的名字：will
你的名字是：will
str 的地址是：000000000062FDA0
str 第一个元素的地址是：000000000062FDA0
```

> 数组名，其实是数组第一个元素的地址，是地址常量，不可以修改的 (不是lvalue)
>
> ？？数组的第一个元素？？是指针？？

数组名只是一个地址，而指针是一个左值(可以进行自增等操作)





### 指针数组/数组指针

```c
// 指针数组, 是数组, 每个数组元素存放一个指针变量
// []的优先级 > *的优先级
int *p1[5];

// 数组指针, 是指针，？？指向数组
int (*p2)[5];

// 看最后两个字
```



#### 指针数组

```c
#include <stdio.h>
int main() {
	
	int a = 1;
	int b = 2;
	int c = 3;
	int d = 4;
	
   
	int *p[4] = {&a, &b, &c, &d};
	
	for(int i = 0; i <4; i++) {
		printf("%d\n", *p[i]);
	}
	return 0;
}
```

```c
#include <stdio.h>
int main(){
	
	char *p[4] = {
		"micro",
		"will",
		"is",
		"man"
	};
    
    // int *p[4] = {1, 2, 3, 4};   error: [Error] invalid conversion from 'int' to 'int*' [-fpermissive]	
	
	for(int i = 0; i < 4; i++) {
		// 这里p前面，不用加* 
		printf("%s\n", p[i]);
	}
	
	return 0;
} 

```

> **%s** 指向的是地址，和%d、%c 不同？？？



#### 数组指针

```c
#include <stdio.h>

int main() {
	
	int array[5] = {1, 2, 3, 4, 5};
    
	// 指针p 指向数组第一个元素的地址，指向的是一个变量 ；
    
    // 不是数组指针，没有指向数组
	int *p = array;
	
	for(int i = 0; i < 5; i++) {
		printf("%d\n", *(p + i));
	} 
	
	return 0;
} 
```



```c
#include <stdio.h>

int main() {
	
	int array[5] = {1, 2, 3, 4, 5};
	
    // 数组指针，指向的是数组
	int (*p)[5] = &array;
	
	for (int i = 0; i < 5; i++) {
        // 取两次地址？？
        // 第一次，表示数组的地址
        // 第二次，取出该地址的值
		printf("%d\n", *(*p + i));
	}
	
	return 0;
} 
```



### 二维数组



```c
array[4][5] = {};

// array，数组名，相当于数组第一个元素(跨度是五个元素)的指针(地址)，指向包含五个元素的数组的指针
```

![1603290035554](C.assets/1603290035554.png)

```c
#include <stdio.h>

int main() {
	
	int array[4][5] = {0};
	
	printf("sizeof int: %d\n", sizeof(int));
	printf("array: %p\n", array);
	printf("array + 1: %p\n", array + 1);
	return 0;
} 
```



```c
*(array + 1) = array[1];

// 取值运算符(*)作用于一个地址之上，把地址的值取出来，称为？ ？解引用

// 这里取出来的值，仍然是地址
```

<img src="C.assets/1603361836438.png" alt="1603361836438" style="zoom:67%;" />

```c
#include <stdio.h> 

int main() {
	
	int array[4][5] = {0};
	int v = 0;
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 5; j++) {
			array[i][j] = ++v;
		}
	}
	
	printf(" *(array + 1) : %p\n", *(array + 1));
	printf(" array[1] : %p\n", array[1]);
	printf(" &array[1][0] : %p\n", &array[1][0]);
	printf(" **(array + 1) : %d\n", **(array + 1));
	return 0;
} 
 // *(array + 1) : 000000000062FDC4
 // array[1] : 000000000062FDC4
 // &array[1][0] : 000000000062FDC4
 // **(array + 1) : 6

```



![1603533842719](C.assets/1603533842719.png)

```c
#include <stdio.h> 

int main() {
	
	int array[4][5] = {0};
	int v = 0;
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 5; j++) {
			array[i][j] = ++v;
		}
	}
	printf(" *(*(array + 1)+3) : %p\n", *(*(array + 1)+3));
	printf(" array[1][3] : %p\n", array[1][3]);
	return 0;
} 
// *(*(array + 1)+3) : 0000000000000009
// array[1][3] : 0000000000000009
```



### 二维数组/数组指针

```c
#include <stdio.h> 

int main() {
	
	int array[2][3] = {{0, 1, 2}, {3, 4, 5}};
	
	// 数组指针 指向二维数组的第一个区域元素{0,1,2} 
	int (*p)[3] = array;
	
	printf(" *(*(p+1)+1) : %d\n", *(*(p+1)+1));
	printf(" *(*(array + 1)+1) : %d\n", *(*(array + 1)+1));
	printf(" array[1][1] : %d\n", array[1][1]);
	return 0;
} 
// *(*(p+1)+1) : 4
// *(*(array + 1)+1) : 4
// array[1][1] : 4
```



### Void指针



Void指针，通常称为通用指针，就是可以指向任意类型的数据

也就是说，==任何类型的指针==都可以赋值给void指针



```c
#include <stdio.h>

int main() {
	
	int num = 7;
	
	// 初始化两个指针：一个int，一个char 
	int *pi = &num;
	char *ps = "will";
	
	// 初始化void指针
	void *pv;
	
	// 将void指针，指向int类型指针 
	pv = pi;
	printf("pi: %p, pv:%p\n", pi, pv);
	
	// 将void指针，指向char类型指针 
	pv = ps;
	printf("ps: %p, pv:%p\n", ps, pv);
	
	return 0;
}
// 可以将任何类型的指针，转换成void指针
// pi: 000000000062FE04, pv:000000000062FE04
// ps: 0000000000404000, pv:0000000000404000

// 如果void指针，转换成其他类型指针，需要强制转换
```



### void指针的转换

```c
#include <stdio.h>

int main() {
	
	int num = 7;
	
	// 初始化两个指针：一个int，一个char 
	int *pi = &num;
	char *ps = "will";
	
	// 初始化void指针
	void *pv;
	
	// 将void指针，指向int类型指针 
	pv = pi;
	printf("pi: %p, pv:%p\n", pi, pv);
    // 将void指针，强制转换成int指针
	printf("pv:%d\n", *(int *)pv);
	
	// 将void指针，指向char类型指针 
	pv = ps;
	printf("ps: %p, pv:%p\n", ps, pv);
    // 将void指针，强制转换成char指针
	printf("pv:%s\n", (char *)pv);
	
	return 0;
}
```



```c
printf(" pv: %d\n", *pv);
// warning: dereferencing 'void *' pointer
// 因为并不知道，void指针的类型，也就是不知道内存地址的宽度

// 但如果void指针，指向的是字符串，不会报错
printf(" pv: %s\n", *pv);
// 因为字符串在C语言中有特殊的约定：
// 只需要指向这个字符串的起始地址，然后会一个字节一个字节的读下去，直到读到 '\0'(特殊的空字符)为止才算结束
```





### NULL指针

> 一个指针，不指向任何数据，称为NULL指针

```c
#define NULL ((void *)0)
// 该指针，不指向任何东西
```

> 如果还不清楚，将指针初始化为什么地址时：
>
> 请先将它初始化为NULL
>
> 在对指针进行解引用时，先检查该指针是否为NULL



```c
#include <stdio.h>

int main() {
	
	int *p1;
	int *p2 = NULL;
	
    // 指针p1的默认值是随机的，p1常被称为野指针
	printf("p1: %d\n", *p1);
    
    // 对一个NULL指针，进行解引用，是非法的，将会报错
    // Segmentation fault
	printf("p2: %d\n", *p2);
	
	return 0;
}
```







### 指向指针的指针

pp：

![1603593639591](C.assets/1603593639591.png)

> C语言定义中，已经交代了，**进行多少次**==解引用==才能取到值

如：

```c
int *p = &num;    // 进行一次解引用取到，int类型num变量中的值
int **pp = &p;    // 进行两次解引用取到，int类型num变量中的值
```

> 看定义中，使用了多少个星号(*)



### 应用

[指针数组和指向指针的指针 05:14](https://www.bilibili.com/video/BV17s411N78s?p=26)	

[数组指针和二维数组 13:51](https://www.bilibili.com/video/BV17s411N78s?p=26)	：指向指针的指针，不适用于二维数组，因为指针和数组的**跨度**不一样；==数组指针可以解决这个跨度问题==

[指向常量的指针 02:47](https://www.bilibili.com/video/BV17s411N78s?p=27)	

<img src="C.assets/1603599261335.png" alt="1603599261335" style="zoom:67%;" />

[常量指针 07:24](https://www.bilibili.com/video/BV17s411N78s?p=27)	

```c
int * const p = &num;
```

<img src="C.assets/1603599596725.png" alt="1603599596725" style="zoom:67%;" />













