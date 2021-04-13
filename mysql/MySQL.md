# MySQL



## 实际生产

 [Mysql6000w数据表的查询优化到0.023S](https://juejin.cn/post/6907115764545748999)	



[如何一步步存储一条数据](https://juejin.cn/post/6919263740122628109#heading-6)：`表空间` / `页结构`  



[简单明了的介绍MySql重要的概念](https://juejin.cn/post/6923788859712995336#heading-1)：`不同维度的索引类型`/ `索引结构的演进` / `回表` / `覆盖索引的结构树` / `索引下推`



 [Select count(*)和Count(1)的区别和执行方式](https://www.cnblogs.com/zhurong/p/10062315.html)：并无区别，展示了详细的执行计划



 [MySQL海量数据优化](https://juejin.cn/post/6924591827286753288)：`很精彩的案列`/ `limit优化` 

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/paging.png)







## 面试题

> 聚簇索引和非聚簇索引的 [区别](https://my.oschina.net/xiaoyoung/blog/3046779)

数据和索引是否存储在一起
辅助索引的叶子节点存储不同



> 为什么主键通常建议使用自增id

聚簇索引的数据的物理存放顺序与索引顺序是一致的，即：只要索引是相邻的，那么对应的数据在磁盘上存放也一定是相邻的。



> Innodb和MyISAM的[区别](https://blog.csdn.net/qq_35642036/article/details/82820178)

Innodb支持事务，行[表]锁，聚簇索引，外键，必须有主键
MyISAM不支持事务，表锁，非聚簇索引,不支持外键，可以没有主键
ISAM: Indexed Sequential Access Method 有索引的顺序访问方法
行锁：InnoDB行锁是通过给索引项加锁来实现的，即只有通过索引条件查找数据，InnoDB才使用行级锁，否则将使用表锁！



> InnoDB[如何](https://www.cnblogs.com/jianzh5/p/11643151.html)保证事务特性

A：undo log    C：undo log + redo log    I：锁    D：redo log



> 如何排查慢查询？一条 SQL 语句执行得很慢的原因有哪些？ 

就像要考你计算机网络的知识时，问你“输入URL回车之后，究竟发生了什么”一样，看看你能说出多少了。
[总分两种情况](https://www.sohu.com/a/312754751_355142) 











## 运维部署

```conf
docker run -p 12345:3306 --name mysql -v /will/mysql/conf:/etc/mysql/conf.d -v /will/mysql/logs:/logs -v /will/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```
- -p 12345:3306：将主机的12345端口映射到docker容器的3306端口
- --name mysql ：运行服务的名字
- -v /will/mysql/conf:/etc/mysql/conf.d：将主机`/will/mysql`目录下的 `conf`目录，挂载到容器的`/etc/mysql/conf.d` 
- -v /will/mysql/logs:/logs：将主机`/will/mysql`目录下的 `logs`目录，挂载到容器的`/logs`  
- -v /will/mysql/data:/var/lib/mysql：将主机`/will/mysql`目录下的 `data`目录，挂载到容器的`/var/lib/mysql`  
- -e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
- -d mysql:5.6：后台程序运行mysql5.6



```shell
# 查看docker的运行程序
docker ps

ps -ef

# 登录MySQL
mysql -uroot -p
```



数据备份：

```shell
docker exec mysql服务容器ID sh -c 'exec mysqldump --all-databases -uroot -p"123456" ' > /will/all-databases.sql
```



| 路径              |           解释            |             备注             |
| :---------------- | :-----------------------: | :--------------------------: |
| /var/lib/mysql    | mysql数据库文件的存放路径 | /var/lib/mysql/will.test.pid |
| /usr/share/mysql  |       配置文件目录        |  mysql.server命令及配置文件  |
| /usr/bin          |       相关命令目录        |  mysqladmin mysqldump等命令  |
| /etc/init.d/mysql |       启停相关脚本        |                              |



主要配置文件：
<img src="MySQL.assets/1605670006458.png" alt="1605670006458" style="zoom:80%;" />



## 架构介绍

![](https://raw.githubusercontent.com/MicroWiller/photobed/master/MysqlArchitecture.png)



5.6版本：

<img src="MySQL.assets/1606041895189.png" alt="1606041895189" style="zoom:67%;" />

①通过客户端/服务器通信协议与 MySQL 建立连接。

②查询缓存，这是 MySQL 的一个可优化查询的地方，如果开启了 Query Cache 且在查询缓存过程中查询到完全相同的 SQL 语句，则将查询结果直接返回给客户端；如果没有开启Query Cache 或者没有查询到完全相同的 SQL 语句则会由解析器进行语法语义解析，并生成解析树。

③预处理器生成新的解析树。

④查询优化器生成执行计划。

⑤查询执行引擎执行 SQL 语句，此时查询执行引擎会根据 SQL 语句中表的存储引擎类型，以及对应的 API 接口与底层存储引擎缓存或者物理文件的交互情况，得到查询结果，由MySQL Server 过滤后将查询结果缓存并返回给客户端。

⑤若开启了 Query Cache，这时也会将SQL 语句和结果完整地保存到 Query Cache 中，以后若有相同的 SQL 语句执行则直接返回结果。



 查询缓存的利弊：任何一条对此表的更新操作都会把和这个表关联的所有查询缓存全部清空



## InnoDB



### 架构图

![1605701087248](MySQL.assets/1605701087248.png)







<img src="MySQL.assets/1606053853390.png" alt="1606053853390" style="zoom:67%;" />





### InnoDB/MyISAM

<img src="MySQL.assets/1606053551057.png" alt="1606053551057" style="zoom:67%;" />

InnoDB 表最大还可以支持 64TB，支持聚簇索引、支持压缩数据存储，支持数据加密，支持查询/索引/数据高速缓存，支持自适应hash索引、空间索引，支持热备份和恢复等，如下图所示。

<img src="MySQL.assets/1606053678228.png" alt="1606053678228" style="zoom:67%;" />





### 查询状态

```mysql
show engine innodb status\G;
```

详细的 InnoDB 运行态信息，分段记录的，包括内存、线程、信号、锁、事务等









## 事务



ACID：

- 原子性(Atomicity)：事务的所有操作，要么全部完成，要么全部不完成，不会结束在某个中间环节；undo log回滚日志保证事务的原子性
- 一致性(Consistency)：事务开始之前和事务结束之后，数据库的**完整性限制**未被破坏；列如银行转账，事务的执行不应该改变A/B两个账户的金额总和；undo log+redo log保证事务的一致性
- 隔离性(Isolation)：一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对其他的并发事务是隔离的；锁和多版本控制就符合隔离性；锁（共享、排他）用来保证事务的隔离性
- 持久性(Durability)：事务完成之后，数据库将保存其所做的修改，不丢失；redo log重做日志用来保证事务的持久性





隔离级别：

- 读未提交：也就是一个事务还没有提交时，它做的变更就能被其他事务看到
- 读已提交：指的是一个事务只有提交了之后，其他事务才能看得到它的变更
- 可重复读：此方式为默认的隔离级别，它是指一个事务在执行过程中（从开始到结束）看到的数据都是一致的，在这个过程中未提交的变更对其他事务也是不可见的
- 串行化：是指对同一行记录的读、写都会添加读锁和写锁，后面访问的事务必须等前一个事务执行完成之后才能继续执行，所以这种事务的执行效率很低





## ARIES 三原则



ARIES 三原则，是指 Write Ahead Logging（WAL）

1. 先写日志后写磁盘，日志成功写入后事务就不会丢失，后续由 checkpoint 机制来保证磁盘物理文件与 Redo 日志达到一致性；

2. 利用 Redo 记录变更后的数据，即 Redo 记录事务数据变更后的值；

3. 利用 Undo 记录变更前的数据，即 Undo 记录事务数据变更前的值，用于回滚和其他事务多版本读。





## 多版本控制-MVCC



多版本控制也叫作 MVCC，

是指在数据库中，为了实现高并发的数据访问，对数据进行多版本处理，并通过**事务的可见性**来保证事务能看到**自己应该**看到的数据版本

> 那个多版本是如何生成的呢？

每一次对数据库的修改，都会在 Undo 日志中记录当前修改记录的事务号及修改前数据状态的存储地址（即 ROLL_PTR），以便在必要的时候可以回滚到老的数据版本。

例如，一个读事务查询到当前记录，而最新的事务还未提交，根据原子性，读事务看不到最新数据，但可以去回滚段中找到老版本的数据，这样就生成了多个版本。



多版本控制很巧妙地将稀缺资源的独占互斥转换为并发，大大提高了数据库的吞吐量及读写性能。







## SQL分析



### SQL执行顺序



 **手写**  

<img src="MySQL.assets/20200210141117444_466.png" style="zoom:67%;" />

**机读** 

<img src="MySQL.assets/20200210142804712_29437.png" style="zoom:67%;" />



![](MySQL.assets/20200210143108960_8138.png )



- select 的字段只能是**分组的字段类别**以及使用**聚合函数**如，max(),min(),count()的字段
- group by，即*以其中一个字段的值* 来分组
  where在前，group by在后，注意group by紧跟在where`最后一个限制条件后面`
  - 不能被夹在where限制条件之间的原因：
    - 要先用where过滤掉不进行分组的数据，然后在对剩下满足条件的数据进行分组
- having是在**分好组后**找出特定的分组，通常是以筛选聚合函数的结果
  - 如sum(a) > 100等
  - 且having必须在group by 后面
  - 使用了having必须使用group by，但是使用group by 不一定使用having
- 不允许 使用双重聚合函数，所以在对分组进行筛选的时候可以用order by 排序，然后用limit也可以找到极值





### Join查询



<img src="MySQL.assets/1601430800695.png" alt="1601430800695" style="zoom:67%;" />

```SQL
SELECT * FROM TableA A INNER JOIN TableB B ON A.Key=B.Key;    // 内连接

SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key=B.Key;    // 左连接

SELECT * FROM TableA A RIGHT JOIN TableB B ON A.Key=B.Key;    // 右连接

SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key=B.Key WHERE B.Key IS NULL;

SELECT * FROM TableA A RIGHT JOIN TableB B ON A.Key=B.Key WHERE A.Key IS NULL;

SELECT * FROM TableA A FULL OUTER JOIN TableB B ON A.Key=B.Key;    // 全连接 (MySQL 不支持)
SELECT * FROM TableA A FULL OUTER JOIN TableB B ON A.Key=B.Key WHERE A.Key IS NULL OR B.Key IS NULL;

// 联合、去重 union
SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key=B.Key
union
SELECT * FROM TableB B RIGHT JOIN TableB B ON A.Key=B.Key;

SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key=B.Key WHERE B.Key IS NULL
union
SELECT * FROM TableA A RIGHT JOIN TableB B ON A.Key=B.Key WHERE A.Key IS NULL;
```


![](MySQL.assets/20200210152913545_10920.png )

![](MySQL.assets/20200210153222209_9857.png )







## 索引简介





#### 是什么

索引是数据结构，目的在于提高查找效率，可以类比字典

可以简单理解为：==排好序的 (可以)快速查找(的) 数据结构== 

> 索引会影响到 `where后的查找` 和  `order by 后面的排序`

<img src="MySQL.assets/image-20200430224434235.png" alt="image-20200430224434235" style="zoom:67%;" />

> 增删改慢，除了修改数据，`还要改变索引的指向`
>
> 互联网公司，不能真正的删除数据：1.大数据分析  2.索引





#### 优势/劣势



优势：

- 类似大学图书馆的书目索引，提高**数据检索**的效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低**数据排序**的成本，降低CPU的消耗

劣势：

- 实际上索引也是一张表，该表保存了主键和索引字段，并指向实体表的记录，所以索引列也是**要占空间**的
- 虽然索引大大提高了查询速度，但同时会**降低更新表的速度**，如对表进行INSERT/UPDATE/DELETE
- 更新表时，MYSQL**不仅要保存数据**，**还要保存**索引文件 每次更新 添加索引列的字段
- 都会调整，因为更新所带来的键值变化后的索引信息

> 索引只是提高查询速度的一个因素
>
> 如果MySQL有大数据量的表，就需要花时间建立最优秀的索引，或者优化查询语句

#### 



#### mysql索引分类



- 单值索引：即一个索引只包含单个列，一个表可以有多个单值索引

- 唯一索引：索引列的值，必须唯一，但允许有空值
- 复合索引：即一个索引包含多个列

```mysql
# 创建
CREATE [UNIQUE] INDEX indexName ON mytable(columnname(length));
ALTER mytable ADD [UNIQUE] INDEX [indexName] ON (columnname(length));
# 删除
DROP INDEX [indexName] ON mytable;
# 查看
SHOW INDEX FROM tableName\G
```

```mysql
# 添加数据表的四种方式
ALTER TABLE mytable ADD PRIMARY KEY (column_list);	# 添加主键，索引值必须是唯一的，且不能为null
ALTER TABLE mytable ADD UNIQUE index_name (column_list); # 添加唯一索引，索引值也必须是唯一的，可以为null，可能会出现多次
ALTER TABLE mytable ADD INDEX index_name (column_list);	# 添加普通索引，索引值可以出现多次
ALTER TABLE mytable ADD FULLTEXT index_name (column_list);	# 添加全文索引
```

> 一张表的索引 最多不要超过5个





#### 联合索引/覆盖索引

根据**索引列个数**和**功能描述**不同索引也可以分为：联合索引和覆盖索引

- 联合索引是指在多个字段联合组建索引的。

- 通过索引**就可以查询到所有记录**，**不需要回表**到聚簇索引时，这类索引也叫作覆盖索引。
  - 回表查询：如果 SQL 需要查询**辅助索引**中不包含的数据列时，就需要先通过辅助索引查找到主键值，然后**再回表**通过主键查询到其他数据列



> 主键查询是天然的覆盖索引，==联合索引可以是覆盖索引== 







#### mysql索引结构

> 实质：数据库存数据，SQL取数据，设计一个可以快速查找的数据结构，减少磁盘的IO操作，提升性能



`筛选`索引的数据结构：

- hash：不支持 **部分索引**查询 和 **范围**查询，
- 红黑树：
  - 读取磁盘的次数过多，读取一页[16K]绝大部分的内容都没有用，**很浪费**
  - 那为什么又可以用在HashMap的查找里面？？纯内存读取很快
- N叉树 
  - 层级太多
  - 连续空间，读取一页[16K]绝大部分的内容都用上

`B树：` 基于二叉查找[排序]树，**N叉的排序树**	【B-Tree	B+Tree	B*Tree】

- M阶的Btree的几个重要特性：
  1. 节点最多含有m颗子树(指针)，m-1个关键字(存的数据，空间)  (m>=2)
  2. 除根节点和叶子节点外，其他每个节点至少还有ceil(m / 2) 个子节点，ceil为向上取整
  3. 若根节点不是叶子节点，则至少有两颗子树



`分裂：` 当不满足以上性质时发生		【红黑树，变颜色左旋右旋】

- 同一磁盘块中，关键字的插入，使用插入排序

- 分裂的时候，从中间[m/2] 分开，分成两棵子树

- <img src="MySQL.assets/image-20200430234706659.png" alt="image-20200430234706659" style="zoom:50%;" />

  **为什么不继续用B-Tree做索引：**1. 范围查找索引失效	2. 空间开销大，除了数据本身 还得存数据的内存空间

![image-20200501225255835](MySQL.assets/image-20200501225255835.png)



> 真实的情况是，3层的b+树可以表示 ==上百万的数据，只需要三次IO==



#### 为什么是B+树

[详细解析](https://mp.weixin.qq.com/s/GHLbBd9rJsNmhAjNHcB5xA)	





#### 哪些情况不要建索引

1. 表记录太少，300万以上建
2. 经常**增删改**的表，索引在提高查询速度的同时，会降低更新表的速度
3. 数据**包含许多重复**且**分布均匀**的表字段，建索引没有太大的实际效果



## 性能分析



[优化方案](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=59#/detail/pc?id=1777)	

MySQL 的优化方案有哪些？

- SQL 和索引优化、数据库结构优化、系统硬件优化	



#### MySQL Query Optimizer



MySql中有专门负责优化Select语句的优化器模块

主要功能：

通过计算分析系统中收集到的统计信息，为客户端请求的Query提供它认为最优的执行计划

但是，它认为最优的数据检索方式，不见得是DBA认为最优的，这部分最耗时间



过程：

1. 当客户端向Mysql请求一条Query，命令解析器模块完成请求分类，区别出是Select并转发给Mysql Query Optimizer
2. Mysql Query Optimizer 首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值；
3. 并对Query中的查询条件进行简化和转换；如去掉一些无用或显而易见的条件、结构调整等
4. 然后分析Query中的Hint信息(如果有)，看显示Hint信息是否可以完全确定该Query的执行计划
5. 如果没有Hint或者Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析
6. 然后再得出最后的执行计划



#### MySQL常见瓶颈



![](MySQL.assets/20200210195847557_12868.png )





#### Explain



##### 是什么

可以模拟优化器执行SQL查询语句，从而知道Mysql是如何处理你的SQL语句的。

分析你的查询语句或是表结构的性能瓶颈



##### 能干嘛



- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以被使用
- 哪些索引被实际使用
- 表之间的关系
- 每张表有多少行被优化器查询



##### 怎么玩

```mysql
explain + sql语句;
# 如：
explain select * from person where uname = 'Java';
```




##### 各字段解释

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
| ---- | ----------- | ----- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |
|      |             |       |      |               |      |         |      |      |       |
<img src="MySQL.assets/1606035919343.png" alt="1606035919343" style="zoom:67%;" />



######  id：表的读取顺序
- id 相同，执行顺序由上至下
 ![](MySQL.assets/20200210202224211_17083.png )
- id 不同，如果是子查询，id的序列化会递增，id值越大优先级越高，越先被执行
![](MySQL.assets/20200210203001434_23912.png )
- id相同不同，同时存在
![](MySQL.assets/20200210203421536_4737.png )



######  select_type：读取数据的操作类型

![](MySQL.assets/20200210203722429_23559.png )

![](MySQL.assets/20200210203905730_7287.png )



######  table：显示这一行的数据是关于哪张表的



######  <u>*type*</u>：访问类型

访问类型排列：
![](MySQL.assets/20200210210644944_17600.png )


从最好到最差依次为：system》const》eq_ref》ref》range》index》ALL

![](MySQL.assets/20200211143318011_29688.png )



######  possible_keys

显示可能应用在这张表中的索引，一个或者多个

查询涉及到的字段上，若存在索引，则该索引将被列出，**但不一定被查询实际使用** 



######  key：实际使用的索引

实际使用到的索引；如果为null，则没有使用索引



查询中，若使用了覆盖索引，则该索引和查询的select字段重叠
![](MySQL.assets/20200210214559813_32618.png )



######  key_len

表示**索引中使用到的字节数**，可通过该列计算 查询中使用到的索引的长度

在不损失精确度的情况下，长度**越短越好** 

key_len显示的值为索引字段的最大可能长度，**并非实际使用长度**， 即key_len是根据**表定义**计算而得，不是通过表内检索出的


耗费得越多，精度越高：
![](MySQL.assets/20200210215530575_20868.png )





######  ref：显示`索引的哪一列(字段)`被使用了

如果可能的话，是一个常数(constant)；
- 哪些列或常量被用于查找索引列上的值
    ![](MySQL.assets/20200211143522061_5752.png)



######  rows：越少越好

每张表**有多少行**被优化器查询



根据表统计信息及索引选用情况，大致估算出，找到所需的记录所需要读取的行数

![](MySQL.assets/20200211144715122_25420.png )





######  Extra：不适合在其他列显示，但十分重要的额外信息

- Using filesort：又排了次序(折腾)，文件内排序
  
   <img src="MySQL.assets/20200211150821977_31227.png" style="zoom:60%;" />
   
   
   
   
   
- Using temporary：新建了一个内部临时表
  
    <img src="MySQL.assets/20200211152320077_14118.png" style="zoom:60%;" />
    
    

> order by / group by 一定要和 `索引的个数、顺序`**按需来**

- USING index：相应的select操作中使用了 `覆盖索引` 避免了访问表的数据行
    <img src="MySQL.assets/20200211153935041_29579.png" style="zoom:60%;" />
    

    
    ![](MySQL.assets/20200211153745156_28884.png )
    
    
    
- Using where：表明使用了where过滤

- using join buffer：使用了连接缓存

- impossible where：where子句的值总是false，不能用来获取任何元素
        ![](MySQL.assets/20200211154610986_21330.png )
    
- select tables optimized away：在没有groupBY子句的情况下，对索引优化做的操作

- distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作



##### 热身case

bug：SQL语句不全

![](MySQL.assets/20200211155447224_4563.png )

![](MySQL.assets/20200211155719718_1361.png )





## 索引优化

#### 索引分析

##### 单表
![](_v_images/20200211164234852_19905.png =700x)
> from 前面的 字段(column) 最好是`等于常量`，`不要范围`

如果实在避免不了：
![](_v_images/20200211165648365_14988.png =900x)

##### 两表

![](_v_images/20200211172835729_20464.png =800x)
> `左连接，右索引`（右表搜索行，左边一定都有）；`右连接，左索引`

##### 三表
`左连接，右索引`（右表搜索行，左边一定都有）；`右连接，左索引`

![](_v_images/20200211182903018_4929.png =900x)

![](_v_images/20200211183907016_14454.png =800x)

当`无法保证` 被驱动表的 join条件字段被索引 且内存资源充足的条件下，不要太吝啬JoinBuffer的设置

#### 索引失效

![](_v_images/20200211201854783_10970.png =800x)

索引多列，要遵守*最左前缀法则*；查询从索引的`最左前列`开始 并且`不跳过索引中的列` ==》带头大哥不能死，中间兄弟不能断
![](_v_images/20200211200555547_24326.png =500x)


![](_v_images/20200211201435660_16046.png =700x)


![](_v_images/20200211203001753_1675.png =800x)
![](_v_images/20200211203046694_2571.png =600x)


尽量使用覆盖索引（ 只访问索引的查询（*索引列和查询列一致*）），`减少select*`
![](_v_images/20200211210938194_24113.png =700x)

![](_v_images/20200211211406786_28312.png =700x)

![](_v_images/20200211211749955_15083.png =700x)

% like 加右边
![](_v_images/20200211212251112_13407.png =700x)
![](_v_images/20200211212634218_9475.png =400x)
覆盖索引来解决


varchar类型绝对`不能失去单引号` ==》隐式的类型转换，全表扫描
![](_v_images/20200211214450574_8206.png =700x)

![](_v_images/20200211214758353_6258.png =600x)

##### 小总结
![](MySQL.assets/20200212110832007_11172.png )

![](MySQL.assets/20200212111058887_14977.png )





#### 一般性建议

```mysql
 create index inx_test03_c1234 on test03(c1,c2,c3,c4);
```
![](_v_images/20200212095703446_20072.png =800x)

![](_v_images/20200212101123420_11963.png =800x)

![](_v_images/20200212102221930_19708.png =800x)
>索引使用和建的顺序 尽量跟 排序一致（`充分利用已有的排序`，**避免**MySQL内排序）

![](_v_images/20200212103341805_31945.png =800x)

![](_v_images/20200212104524203_19123.png =800x)
>OrderBY 只要没有跟 索引的顺序一致 *基本上都会产生filesort*，`除非`排序字段已经是一个常量了


![](_v_images/20200212105134296_30163.png =1000x)
> GroupBy ：如果和索引的顺序不一样，`分组之前必排序`，会有`临时表`创建
> 定值为常量，范围之后失效，最终看排序，一般OrderBy 是给个范围

![](_v_images/20200212105902621_9612.png =800x)


## 查询截取分析

0. 收到监控系统报警，模拟运行环境
1. 慢查询的开启和捕获
2. explain + 慢SQL分析
3. show profile 查询SQL 在MySQL服务器里面的执行细节和生命周期
4. SQL数据服务器的参数调优

### 查询优化

#### 永远小表驱动大表

>数据库**最耗费资源**的是 `数据库连接` 

in 和 exist 的区别：

```mysql
#当 t_order表的数据集小于t_user表时,用 in 优化 exist,使用 in,两表执行顺序是先查 t_order 表,再查t_user表
select * from t_user where id in (select user_id from t_order)
 
# 当 t_user 表的数据集小于 t_order 表时，用 exist 优化 in,使用 exists,两表执行顺序是先查 t_user  表,再查 t_order  表
select * from t_user where exists (select 1 from B where t_order.user_id= t_user.id)
```





#### OrderBy 关键字优化

`为排序使用索引`

##### 最佳左前缀原则
只要带头大哥在，按索引顺序OrderBy ==》就不会Using filesort
![](_v_images/20200212144703052_28776.png)

![](_v_images/20200212150838994_11693.png =800x)
![](_v_images/20200212151104225_13825.png =800x)

![](_v_images/20200212145538492_25910.png =900x)
![](_v_images/20200212150335520_22692.png =900x)
> `同升或者同降`==》Using Index

##### 不在索引列上，filesort有两种算法
![](_v_images/20200212152710488_13085.png =1000x)

![](_v_images/20200212153259857_16825.png =900x)

##### 小总结

![](_v_images/20200212154436922_18322.png =600x) 

#### GroupBy 关键字优化

![](_v_images/20200212164205376_18576.png =700x)


### 慢查询日志
```xml
    slow_query_log
    long_query_time
```

#### 是什么
![](_v_images/20200212164610612_8467.png =800x)

#### 怎么玩

默认情况下：
![](_v_images/20200212164753463_8295.png =800x)

开启慢查询日志：
![](_v_images/20200212165359146_10436.png =500x)
![](_v_images/20200212165651132_19359.png =800x)

修改默认的阀值：
![](_v_images/20200212165930411_28712.png =600x)
```mysql
set global long_query_time = 3;    // 修改为阀值到3秒的就是慢SQL
```
![](_v_images/20200212170629890_11886.png =600x)

记录慢SQL并后续分析：
![](_v_images/20200212171247138_13935.png =800x)


```mysql
show global status like '%Slow_queries%';    // 查询当前系统中有多少条慢查询记录
```

永久生效：在配置文件配置
![](_v_images/20200212171754710_7017.png =300x)

#### 日志分析工具 mysqldumpslow
![](_v_images/20200212172137573_5610.png =700x)
![](_v_images/20200212172433064_10793.png =600x)



### show profile

分析 当前会话中 语句执行的资源消耗情况 的明细条
默认情况下，参数处于关闭状态，并保存最近15次的运行结果

![](_v_images/20200213194638139_16013.png =700x)

![](_v_images/20200212182909139_1991.png =300x)

```mysql
// SQL导致服务器慢，要么CPU运算复杂，要么就是频繁IO
show profile cpu, block io for query Query_ID;
```
![](_v_images/20200213201014392_11266.png =500x)



### 全局查询日志

只能在测试环境用，绝不能在生产环境用
![](MySQL.assets/20200213201857044_30680.png )

配置启用：
![](MySQL.assets/20200213201939144_517.png )

编码启用：
![](MySQL.assets/20200213202250528_3100.png )





### 慢查询日志



可以通过设置“slow_query_log=1”来开启慢查询，它的开启方式有两种：



1. 命令行的模式进行开启

```mysql
set global slow_query_log=1

# 重启 MySQL 服务之后就会失效
```



2. 修改 MySQL 配置文件的方式进行开启

- 配置 my.cnf 中的“slow_query_log=1”即可
- 并且可以通过设置“slow_query_log_file=/tmp/mysql_slow.log”来配置慢查询日志的存储目录
- 但这种方式配置完成之后需要重启 MySQL 服务器才可生效。



> 需要注意的是，在开启慢日志功能之后，会对 MySQL 的性能造成一定的影响，因此在生产环境中要慎用此功能。



## MySQL锁机制

![](_v_images/20200213203045844_24303.png =300x)
![](_v_images/20200213203120633_4708.png =800x)

![](_v_images/20200213203528140_13996.png =600x)

```mysql
lock table 表名字 read / write , 表名字2 read / write , 其他;
show open tables;    // 查看表上加过的锁
unlock tables;       // 释放表
```
### 表锁(偏读)

#### 加读锁
![](_v_images/20200213205614501_1680.png =900x)
![](_v_images/20200213205822672_13981.png =900x)
![](_v_images/20200213205853640_1400.png =900x)
![](_v_images/20200213205928510_336.png =900x)

#### 加写锁

![](_v_images/20200213211149680_18149.png =800x)
![](_v_images/20200213211042190_16223.png =800x)

#### 总结
![](_v_images/20200213211541027_1763.png =900x)

![](_v_images/20200213212504095_31147.png =800x)

- 读锁：自己可以读，别人也可以读；自己不能写，别人也不能写；自己不能对其他表操作，别人可以操作
- 写锁：自己可以读，别人不可以读；自己可以写，别人不能写；自己不能对其他表操作，别人可以操作


#### 表锁分析
![](_v_images/20200213213720910_31620.png =200x)
```mysql
show open tables;
```

![](_v_images/20200213213735027_4026.png =600x)
```mysql
show status like 'table%';
```

![](_v_images/20200213214020721_15558.png =900x)

![](_v_images/20200213214252845_5066.png =900x)


### 行锁(偏写)

![](_v_images/20200213214545167_25268.png =1000x)



#### 行锁定演示
![](_v_images/20200214142621739_11006.png =900x)


#### 无索引 行锁升级为表锁

索引失效(varchar类型字段没加单引号) ==》行锁升级为表锁
![](_v_images/20200214145033514_14244.png =700x)

#### 间隙锁危害

![](_v_images/20200214155554460_6142.png =700x)
![](_v_images/20200214160323837_7104.png =700x)

![](_v_images/20200214160419749_32160.png =800x)

#### 如何锁定一行(面试题)

![](_v_images/20200214160700357_26458.png =900x)

```mysql
select * from test_table where  <where_condition> for update;    // 手动锁定某一行
```
#### 总结

![](_v_images/20200214163959658_22284.png =802x)

#### 行锁分析
![](_v_images/20200214164141356_12417.png =700x)
```mysql
show status like 'innodb_row_lock%';
```

![](_v_images/20200214164627045_23796.png =500x)
![](_v_images/20200214164829003_21973.png =800x)


#### 优化建议
![](_v_images/20200214165031245_13942.png =600x)

### 页锁

![](_v_images/20200214165215722_8434.png =800x)


## 主从复制



### 复制的基本原理

slave会从master读取`bin-log`来进行数据同步

![](MySQL.assets/20200214173955144_28132.png )



### 复制的基本原则

![](MySQL.assets/20200214174103257_3718.png )



### 复制的最大问题

延时



### 一主 一从

![](MySQL.assets/20200214174955496_2896.png )

![](MySQL.assets/20200214175438582_8532.png )

![](MySQL.assets/20200214175709049_210.png )

![](MySQL.assets/20200214180001571_27049.png)
![](MySQL.assets/20200214180038534_5473.png )

![](MySQL.assets/20200214180553287_10997.png )
```mysql
show master status;
```
![](MySQL.assets/20200214180628962_32181.png )

![](MySQL.assets/20200214181421275_29166.png )

![](MySQL.assets/20200214181927152_6918.png )

```mysql
stop slave;    // 停止从机的复制
```





## 备份

[全量备份](https://cloud.tencent.com/developer/article/1398937) : `percona-xtrabackup` 





## 批量数据脚本

### 报错

```java
// 有返回值
public String getUserInfo1() {
    ......
    return user.toString();
}

//无返回值
public void getUserInfo2() {
    ......
}
```

创建函数如果报错：**Thies function has none of DETERMINISTIC……**

- 由于开启过`慢查询日志`，因为开启了`bin-log`, 就必须为`function` 指定一个参数

  ```mysql
  show variables like 'log_bin_trust_function_creators';
  
  set global log_bin_trust_function_creators = 1;
  ```

- MySql重启后，上述参数会消失，永久方法：

  - windows下，my.ini[mysqld] 加上 `log_bin_trust_function_creators=1` 
  - linux 下，/etc/my.cnf下 my.conf[mysqld] 加上`log_bin_trust_function_creators=1` 





### 函数

**随机字符串：**

```mysql
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURN VARCHAR(255)
BEGIN
  DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
  DECLARE return_str VARCHAR(255) DEFAULT '';
  DECLARE i INT DEFAULT 0;
  WHILE i < n DO
  SET return_str =CONCAT(return_str,SUBSTRING(chars_str, FLOOR(1+RAND()*52),1));
  SET i = i + 1;
  END WHILE;
  RETURN return_str;
END $$
```



**随机编号：**

```mysql
DELIMITER $$
CREATE FUNCTION rand_num( ) RETURN INT(5)
BEGIN
  DECLARE i INT DEFAULT 0;
  SET i = FLOOR(100+RAND()*10);
  RETURN i;
END $$
```



**删除函数：**

```mysql
drop function rand_num;    //假如要删除
```



### 过程

**插入数据的过程：**

```mysql
DELIMITER ;;
CREATE PROCEDURE insert_emp(IN START INT(10), IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0; # 自动提交，设置为0

# 循环体
REPEAT  
SET i = i + 1;

# 插入数据时，可以调用函数
INSERT INTO emp(empno, ename, create_time) VALUES ((START+i), rand_string(6), now());
UNTIL i = max_num
END REPEAT ;

commit;
END;;
```

```mysql
DELIMITER ;;
CREATE PROCEDURE insert_dept(IN START INT(10), IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0; # 自动提交，设置为0

# 循环体
REPEAT  
SET i = i + 1;

# 插入数据时，可以调用函数
INSERT INTO dept(deptno, dname, loc) VALUES ((START+i), rand_string(10), rand_string(8));
UNTIL i = max_num
END REPEAT ;

commit;
END;;
```

```mysql
DELIMITER ;    
CALL insert_dept(100, 10);    // 调用过程用call
CALL insert_emp(100001,500000);
```









## hacker

 [MySQL 漏洞利用与提权](https://www.sqlsec.com/2020/11/mysql.html#toc-heading-1)	





## 内核参数

```shell
#基础配置
datadir=/data/datafile
socket=/var/lib/mysql/mysql.sock
log-error=/data/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
character_set_server=utf8
#允许任意IP访问
bind-address = 0.0.0.0
#是否支持符号链接，即数据库或表可以存储在my.cnf中指定datadir之外的分区或目录，为0不开启
#symbolic-links=0
#支持大小写
lower_case_table_names=1
#二进制配置
server-id = 1
log-bin = /data/log/mysql-bin.log
log-bin-index =/data/log/binlog.index
log_bin_trust_function_creators=1
expire_logs_days=7
#sql_mode定义了mysql应该支持的sql语法，数据校验等
#mysql5.0以上版本支持三种sql_mode模式：ANSI、TRADITIONAL和STRICT_TRANS_TABLES。
#ANSI模式：宽松模式，对插入数据进行校验，如果不符合定义类型或长度，对数据类型调整或截断保存，报warning警告。
#TRADITIONAL模式：严格模式，当向mysql数据库插入数据时，进行数据的严格校验，保证错误数据不能插入，报error错误。用于事物时，会进行事物的回滚。
#STRICT_TRANS_TABLES模式：严格模式，进行数据的严格校验，错误数据不能插入，报error错误。
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#InnoDB存储数据字典、内部数据结构的缓冲池，16MB已经足够大了。
innodb_additional_mem_pool_size = 16M
#InnoDB用于缓存数据、索引、锁、插入缓冲、数据字典等
#如果是专用的DB服务器，且以InnoDB引擎为主的场景，通常可设置物理内存的60%
#如果是非专用DB服务器，可以先尝试设置成内存的1/4
innodb_buffer_pool_size = 4G
#InnoDB的log buffer，通常设置为 64MB 就足够了
innodb_log_buffer_size = 64M
#InnoDB redo log大小，通常设置256MB 就足够了
innodb_log_file_size = 256M
#InnoDB redo log文件组，通常设置为 2 就足够了
innodb_log_files_in_group = 2
#共享表空间:某一个数据库的所有的表数据，索引文件全部放在一个文件中，默认这个共享表空间的文件路径在data目录下。默认的文件名为:ibdata1 初始化为10M。
#独占表空间:每一个表都将会生成以独立的文件方式来进行存储，每一个表都有一个.frm表描述文件，还有一个.ibd文件。其中这个文件包括了单独一个表的数据内容以及索引内容，默认情况下它的存储位置也是在表的位置之中。
#设置参数为1启用InnoDB的独立表空间模式，便于管理
innodb_file_per_table = 1
#InnoDB共享表空间初始化大小，默认是 10MB，改成 1GB，并且自动扩展
innodb_data_file_path = ibdata1:1G:autoextend
#设置临时表空间最大4G
innodb_temp_data_file_path=ibtmp1:500M:autoextend:max:4096M
#启用InnoDB的status file，便于管理员查看以及监控
innodb_status_file = 1
#当设置为0，该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
#当设置为1，该模式是最安全的，但也是最慢的一种方式。在mysqld 服务崩溃或者服务器主机crash的情况下，binary log 只有可能丢失最多一个语句或者一个事务。
#当设置为2，该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。
innodb_flush_log_at_trx_commit = 1
#设置事务隔离级别为 READ-COMMITED，提高事务效率，通常都满足事务一致性要求
#transaction_isolation = READ-COMMITTED
#max_connections：针对所有的账号所有的客户端并行连接到MYSQL服务的最大并行连接数。简单说是指MYSQL服务能够同时接受的最大并行连接数。
#max_user_connections : 针对某一个账号的所有客户端并行连接到MYSQL服务的最大并行连接数。简单说是指同一个账号能够同时连接到mysql服务的最大连接数。设置为0表示不限制。
#max_connect_errors：针对某一个IP主机连接中断与mysql服务连接的次数，如果超过这个值，这个IP主机将会阻止从这个IP主机发送出去的连接请求。遇到这种情况，需执行flush hosts。
#执行flush host或者 mysqladmin flush-hosts，其目的是为了清空host cache里的信息。可适当加大，防止频繁连接错误后，前端host被mysql拒绝掉
#在 show global 里有个系统状态Max_used_connections,它是指从这次mysql服务启动到现在，同一时刻并行连接数的最大值。它不是指当前的连接情况，而是一个比较值。如果在过去某一个时刻，MYSQL服务同时有10
00个请求连接过来，而之后再也没有出现这么大的并发请求时，则Max_used_connections=1000.请注意与show variables 里的max_user_connections的区别。#Max_used_connections / max_connections * 100% ≈ 85%
max_connections=600
max_connect_errors=1000
max_user_connections=400
#设置临时表最大值，这是每次连接都会分配，不宜设置过大 max_heap_table_size 和 tmp_table_size 要设置一样大
max_heap_table_size = 100M
tmp_table_size = 100M
#每个连接都会分配的一些排序、连接等缓冲，一般设置为 2MB 就足够了
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 2M
#建议关闭query cache，有些时候对性能反而是一种损害
query_cache_size = 0
#如果是以InnoDB引擎为主的DB，专用于MyISAM引擎的 key_buffer_size 可以设置较小，8MB 已足够
#如果是以MyISAM引擎为主，可设置较大，但不能超过4G
key_buffer_size = 8M
#设置连接超时阀值，如果前端程序采用短连接，建议缩短这2个值，如果前端程序采用长连接，可直接注释掉这两个选项，是用默认配置(8小时)
#interactive_timeout = 120
#wait_timeout = 120
#InnoDB使用后台线程处理数据页上读写I/0请求的数量,允许值的范围是1-64
#假设CPU是2颗4核的，且数据库读操作比写操作多，可设置
#innodb_read_io_threads=5
#innodb_write_io_threads=3
#通过show engine innodb status的FILE I/O选项可查看到线程分配
#设置慢查询阀值，单位为秒
long_query_time = 120
slow_query_log=1 #开启mysql慢sql的日志
log_output=table,File #日志输出会写表，也会写日志文件，为了便于程序去统计，所以最好写表
slow_query_log_file=/data/log/slow.log
##针对log_queries_not_using_indexes开启后，记录慢sql的频次、每分钟记录的条数
#log_throttle_queries_not_using_indexes = 5
##作为从库时生效,从库复制中如何有慢sql也将被记录
#log_slow_slave_statements = 1
##检查未使用到索引的sql
#log_queries_not_using_indexes = 1
#快速预热缓冲池
innodb_buffer_pool_dump_at_shutdown=1
innodb_buffer_pool_load_at_startup=1
#打印deadlock日志
innodb_print_all_deadlocks=1
```



