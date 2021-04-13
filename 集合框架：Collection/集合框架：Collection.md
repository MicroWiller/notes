# 集合框架：Collection



## 结构图

![](_v_images/20200216115103549_22386.png)

- list，有序，可以存放重复的值和null值；有索引
- set，无序，不能存放重复的值，且只能放一个null值；没有索引







## 数组的局限性

>数组有定长特性，必须定义长度，`长度一旦指定，不可更改`
```java
    int[] arr = new int[3];// 动态初始化 
    int[] arr = new int[]{1,2,3,4,5}; // 静态初始化 
    int[] arr = {1,2,3,4,5}; // 静态初始化 省略形式
```

注意：
<img src="集合框架：Collection.assets/20200216113847793_29149.png" style="zoom:70%;" />



- array[1]：array变量保存的是首地址，1是`偏移量`





## 扩容 ==> ArrayList

> 为了解决数组的局限性：扩容------>`ArrayList` 数组和列表的结合
> `size`： 真实的元素个数
> `length`：ArrayList的长度

### 初始长度

[ArrayList list = new ArrayList(20);中的list扩充几次](https://blog.csdn.net/u014236541/article/details/51211644)

**ArrayList的三个构造方法：**（1.8）
```java
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```



- 初始长度为0，`头一次添加`才为10，每次扩容1.5倍
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114616665_5671.png )
- 参数>0，添加多少就为多少
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114657192_19655.png )

### ArrayList 扩容的两种方法

1. 长度+长度 >>1(右移1，除2) 1.5倍
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114253194_1021.png )

2. 长度==新数组长度 
    - 如果扩容1.5倍还不够：按照传递的数值给容量
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114427185_4612.png )


### 总结

![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114753590_14011.png )

![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200217100558746_4148.png )

### 引入泛型Generic

![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216114923721_2705.png )
> 规范了类型，只能包装类型

- 泛型的简写:
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216115011856_26733.png )



## List

### ArrayList和LinkedList的区别

> ArrayList 底层数组，查找比linkedlist快
![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125158721_29322.png )

> LinkedList 底层是链表，里面的Node，相当于一个一个的车厢
![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216173607749_23736.png )



### Vector 和 ArrayList的区别

- Vector 可以当做ArrayList来看
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125309430_8951.png )

- 无参构造直接传成10
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125350871_7680.png )

- vector 和 ArrayList不一样的是
    ![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125435748_5030.png)


## set

> `set特性`：不允许有重复的元素放入
![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125617666_7005.png )

![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125658723_13168.png )

![](%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%EF%BC%9ACollection.assets/20200216125729386_25090.png )

1. TreeSet 是 SortSet 接口的唯一实现， 可以确保集合元素处于排序状态。TreeSet 不是根据元素插入顺序进行排序的，而是`根据元素的值`来排序的。TreeSet 支持两种排序方法：*自然排序* 和*定制排序* 。
2. LinkedHashSet，其集合也是根据元素 `hashCode` 值来决定元素的存储位置，但它同时用`链表`来维护元素的次序，这样使得元素**看起来**是以插入的顺序保存的
3. 所以LinkedHashSet 的性能略低于 HashSet，但在*迭代访问全部元素时* 将有很好的性能，因为它以链表来维护内部顺序。
4. HashSet 的性能比 Treeset 好，因为 TreeSet 需要额外的红黑树算法来维护集合元素的次序，只有当需要一个保持排序的 Set 时，才会用 TreeSet。
5. EnumSet 是性能最好的，但它只能保存枚举值。


## Collections

Collections是一个类，“容器”的工具类 , 就如同Arrays是数组的工具类
