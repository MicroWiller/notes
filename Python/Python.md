# Python



## Hello World

```python
print("Hello World")
# 可以不写分号

# input输入函数，类似C中的 scanf()函数
# 用于接收用户的输入并且返回
temp = input("请输入内容：") 
```

```c
// C的输出函数
printf("Hello World");

int n;
printf("请输入字符的个数：");
scanf("%d", &n);
```

```java
// Java的输出函数
System.out.println("Hello World");
```



## 特殊语法



```python
x = 3
y = 5
x, y = y, x  
# 交换，python中的语法糖 
```



## 字符串



```python
print('will')
print("will")
# 单引号 和 双引号 都表示字符串

# 长字符串：
print('''will''')
print("""will""")
# 长字符串的发明，主要是为了实现“跨行字符串”
```



制表符：

<img src="Python.assets/1602247447072.png" alt="1602247447072" style="zoom:67%;" />



```python
print(r"D:\Code\python")
# r 在这里表示转义
# D:\Code\python
# \ 放在末尾，表示这事还没完！！！
```



```python
# 字符串的加法和乘法
# 字符串可以直接加，也就是拼接在一起
'520' + '1314'
# '5201314'

# 乘法，表示字符串的复制
print("I love U" * 3000)
```



## 数字类型

> 在内存中的存储方式不一样

整数：长度不受限制，可以进行大数计算

浮点数：小数在python中，以浮点数的形式存放；具有误差(IEEE754)

复数：包括实部和虚部，如：`1+2j` 

```python
x = 1 + 2j
realPart = x.real
imagPart = x.imag
```



<img src="Python.assets/1602564474014.png" alt="1602564474014" style="zoom:70%;" />

```python
pow(2, 3, 5)

2 ** 3 % 5

# 两个表达式的结果相同：3
```





## 运算符优先级

![1602650456972](Python.assets/1602650456972.png)











