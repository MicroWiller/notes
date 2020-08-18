# Linux

==在Linux世界里，一切皆文件==

- /bin：Binary，存放最常用的命令
- /proc：Processor，存放正在运行的程序？？
- /etc：Editable Text Configuration，系统配置文件目录
  - /etc/fstab：是用来存放 文件系统(如硬盘) 的 静态信息(指定目录) 的文件；系统启动时，会自动将此文件中指定的文件系统挂载到指定的目录
- /tmp： Temporary，存放临时文件，会自动清除，也可以配置清除策略
- /dev： Device，类似于Windows的设备管理器，把所有的**硬件**用文件的形式存储
- /media：自动识别的一些设备，列如U盘、光驱等等，识别后，将挂载到这个目录下
- /mnt：让用户临时挂载别的文件系统，列如可以将外部的存储挂载到该目录上，进入该目录就可以查看存储的内容
- /opt：Option，额外安装软件的安装目录
- /usr：另一个给主机额外安装软件的安装目录
- /var：Variable，变量，存放不断扩充的东西，存放经常被修改的目录，列如各种日志文件














## 整机
- top

- uptime，系统性能命令的精简版




## CPU
- vmstat -n 2 3    （每两秒采样一次，共计采样三次）
    ![](Linux.assets/20200123213355384_6841.png )
    ![](Linux.assets/20200123213736436_25428.png )
    ![](Linux.assets/20200123214423016_20485.png )

    
    
- 额外查看CPU的命令
    ![](Linux.assets/20200123214136645_886.png )

## 内存

- free
    ![](Linux.assets/20200123214730016_31588.png )

- ![](Linux.assets/20200123214917147_16852.png )


## 硬盘
- df：查看磁盘剩余空间数
    ![](Linux.assets/20200123215159985_25033.png)


## 磁盘IO
- iostat
    ![](Linux.assets/20200123215509747_334.png )
    ![](Linux.assets/20200123215622305_22791.png )

- ![](Linux.assets/20200123215835278_20913.png )


## 网络IO
- ifstat
    下载：
    ![](Linux.assets/20200123220118135_16419.png )

## 网络服务(端口)

netstat -anp 查看系统网络服务(端口)
![](Linux.assets/20200218161131544_8346.png )

## CPU占用过高的定位分析思路

结合Linux和JDK命令一块分析

![](Linux.assets/20200123224508327_15466.png )

![](Linux.assets/20200123224711828_23083.png )

![](Linux.assets/20200123224751769_29247.png )
![](Linux.assets/20200123221021232_999.png )

![](Linux.assets/20200123224825494_16492.png )
![](Linux.assets/20200123224843018_4955.png )
![](Linux.assets/20200123225544073_8217.png )
![](Linux.assets/20200123225717078_28525.png )

![](Linux.assets/20200123225841227_20026.png )
![](Linux.assets/20200123225936764_805.png )

![](Linux.assets/20200123230047886_2399.png )

## JDK性能分析工具

![](Linux.assets/20200124082304657_23337.png )
![](Linux.assets/20200124082243019_16471.png )
![](Linux.assets/20200124082338256_32190.png )

![](Linux.assets/20200124081551618_22122.png)

![](Linux.assets/20200124081737518_21039.png )
还有javac
