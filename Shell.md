# Shell

[Linux文件系统](https://labuladong.gitee.io/algo/%E6%8A%80%E6%9C%AF/linux%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F.html?q=)：`目录结构` 

[进程、线程 和文件描述符](https://labuladong.gitee.io/algo/%E6%8A%80%E6%9C%AF/linux%E8%BF%9B%E7%A8%8B.html): `task_struct` 

[关于Linux必须知道的知识](https://labuladong.gitee.io/algo/%E6%8A%80%E6%9C%AF/linuxshell.html)：`标准输入和参数` / `shell和子程序` / `单引号和双引号的区别` / `sudo找不到命令` 







## 定义

> 作用：可以**更方便的** 使用 **操作系统** 的 `接口` 

> 工作原理： 负责把用户输入的字符串转换到需要执行的程序，并把结果以某个形式画出来的 
>
> 列如： 在命令行里输入 “ls -Rl” 这种字符串。
>
> - 这个字符串被翻译成“ls”，“-R”，“-l”
> - “ls”帮我们找到那个之前写好的程序，并启动它；“-R”和“-l”被作为参数传给这个程序，告诉程序走“递归所有子目录”+“输出长格式”这部分代码 



<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/ShellOfLinux.png" style="zoom:67%;" />

>  【bash】 + 【terminal】大概可以理解为一个以`字符`为**交互**方式的 **Shell** 



bash 负责按照某种格式把用户的输出的字符串翻译：

- 比如对于普通非空字符翻译为程序和参数，并尝试去PATH里找对应的程序；
- 对“空格”翻译成分隔符；对“$XXX”尝试进行环境变量的替换；
- 对“｜”翻译为管道；
- 对 “>”翻译为输出重定向；
- 对一个指令末尾的“&”翻译为将程序转到后台执行



参考： [Shell 是用来解决什么问题的](https://www.zhihu.com/question/35382632) 





## hello world

```shell
# 新建shell文件
vim hello.sh
# 创建初，文件权限为：-rw-r--r--
# 并且文件后缀是否为 .sh 都可以，甚至不写后缀名都可以；建议最好加上，方便维护
```

```sh
# 编写内容
#!/bin/bash
echo 'Hello World'
```

`#!` 告诉系统这个脚本需要什么解释器来执行 

`/bin/bash`  当我们没有指定解释器的时候默认的解释器 

```shell
# 查看本机支持的解释器:
cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin

# 如果没有指定 #!/bin/bash
./hello.sh
# 将会默认使用 $SHELL 指定的解释器
echo $SHELL
/bin/bash
```



```shell
# 执行该文件
sh hello.sh

./hello.sh
# 若使用 ./ 方法执行文件，该文件需要有可执行文件的权限
chmod +x hello.sh

# 在Kali2020虚拟机上，将 #!/bin/bash 去掉，使用 ./ 方法也可以执行 ？？？
```



## 注释模板

```shell
# shebang
# 脚本的参数
# 脚本的用途
# 脚本的注意事项
# 脚本的写作时间，作者，版权等
# 各个函数前的说明注释
# 一些较复杂的单行命令注释
```





## 常用命令

```shell
# 配置环境变量
vim /etc/profile

# 最后行添加路径
export PATH=$PATH:/home/mysql/bin

# 生效
source /etc/profile

# 显示当前系统定义的所有环境变量
export

# 输出当前的PATH环境变量的值
echo $PATH 
```





```shell
# 查看当前的进程的网络
# 可以使用 "ss" (socket statistics) 来替代 netstat
netstat -an | grep 进程号

# 查看当前进程号对应的 PID
lsof -i :进程号

kill -9 PID
```



```shell
# sha1 文件校验
sha1sum 文件名
```



```shell
# 在 “文件夹范围内” 两天内修改的文件/文件夹
find 范围 -mtime -2
```

[Linux-文件搜索命令find的使用](https://jingyan.baidu.com/article/636f38bb6e0bdad6b846103e.html)	





```shell
# 开机自启 
/sbin/chkconfig --add server_name

# 自定义开机文件
cd /etc/systemd/system
```



```shell
# 打印出CPU占用过高进程的线程栈
jstack PID

# 查看某个进程中的 "线程状态"
top -H -p pid

# 或者也可以采用ps命令，来查看繁忙的线程信息
ps -mp pid -o THREAD,tid,time 
ps -Lfp pid 
```



```shell
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service 

# 关闭 selinux
vim /etc/selinux/config
```



```shell
# 修改网络
vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
# centos6的网卡重启方法：
service network restart
# centos7的网卡重启方法：
systemctl restart network
```



```shell
# 查看 "内存" 包括物理内存和虚拟内存swap
free -h

# 查看 "磁盘" 的使用量
df -h

# fdisk创建和维护分区表的程序
fdisk -l 

# 设置分区
fdisk /dev/sdb

# 查看sdb的分区情况
lsblk /dev/sdb

# make file system, 在特定的分区上建立 linux 文件系统,
mkfs -t ext4 /dev/sdb1

# 挂载新磁盘
mount /dev/sdb1  /mnt/newdisk

# 永久挂载
vim /etc/fstab
/dev/sda1	/dragonball		ext4	defaults	0 0


hdparm -i /dev/sda1
hdparm -i /dev/sdb1 |grep -i serialno
# SerialNo=Z1Z0MKWS 硬盘序列号
```

[mount命令](https://www.linuxprobe.com/mount-detail-parameters.html)	

[How to grow file system](https://snapshooter.io/blog/how-to-grow-an-ext234-file-system-with-resize2fs-): 除了主分区外

[Resize Root Partition in Linux](https://linuxconfig.org/how-to-resize-ext4-root-partition-live-without-umount)	

[使用fdisk扩容分区](https://www.cnblogs.com/chenmh/p/5096592.html)	





```shell
# "xargs" 读取输入源，然后逐行处理
# 删除目录中的所有class文件: 
find . | grep .class$ | xargs rm -rvf

#把所有的rmvb文件拷贝到目录
ls *.rmvb | xargs -n1 -i cp {} /mount/xiaodianying
```



```shell
// 修改sshd
vim /etc/ssh/sshd_config
# 打开权限：
# PermitRootLogin yes
# PubkeyAuthentication yes
# PasswordAuthentication yes

systemctl restart sshd
```



```shell
#修改防火墙： /etc/sysconfig/iptables文件

# 查看自己目前的防火墙端口情况：
iptables -L -n

1、停止并屏蔽firewalld服务
systemctl stop firewalld
systemctl mask firewalld

2、安装iptables-services软件包
yum install iptables-services

3、在引导时启用iptables服务
systemctl enable iptables

4、启动iptables服务
systemctl start iptables

5、保存防火墙规则
service iptables save
# or
/usr/libexec/iptables/iptables.init save

6、附加：停止、启动，重启防火墙
systemctl [stop|start|restart] iptables


# 不出意外执行完这些操作之后，你应该就能轻松的发现这个文件了
# 现在接着修改/etc/sysconfig/iptables文件
vim /etc/sysconfig/iptables
```





## 查看系统命令

整机：

- top 
- uptime，系统性能命令的精简版



CPU：

- vmstat -n 2 3    每两秒采样一次，共计采样三次

  - procs：

    - r：`运行/等待CPU时间片的进程数`，原则上1核的CPU运行队列不要超过2，整个系统的运行队列不能超过 2*总核数
  - b：`等待资源的进程数`，比如正在等待磁盘I/O 、网络I/O等
    
    
  
- 





## 高效命令



- 如果你想要有语法高亮的 `cat`，可以试试 **[ccat](https://github.com/jingweno/ccat)** 命令。
- **[exa](https://github.com/ogham/exa)** 增强了 `ls` 命令，如果你需要在很多目录上浏览各种文件 ，**[ranger](https://github.com/ranger/ranger)** 命令可以比 `cd` 和 `cat` 更有效率，甚至可以在你的终端预览图片。
- **[fd](https://github.com/sharkdp/fd)** 是一个比 `find` 更简单更快的命令，他还会自动地忽略掉一些你配置在 `.gitignore` 中的文件，以及 `.git` 下的文件。
- **[fzf](https://github.com/junegunn/fzf)** 会是一个很好用的文件搜索神器，其主要是搜索当前目录以下的文件，还可以使用 `fzf --preview 'cat {}'`边搜索文件边浏览内容。
- `grep` 是一个上古神器，然而，**[ack](https://beyondgrep.com/)**、**[ag](https://github.com/ggreer/the_silver_searcher)** 和 **[rg](https://github.com/BurntSushi/ripgrep)** 是更好的grep，和上面的 `fd`一样，在递归目录匹配的时候，会使用你配置在 `.gitignore` 中的规则。
- `rm` 是一个危险的命令，尤其是各种 `rm -rf …`，所以，**[trash](https://github.com/andreafrancia/trash-cli/)** 是一个更好的删除命令。
- `man` 命令是好读文档的命令，但是man的文档有时候太长了，所以，你可以试试 **[tldr](https://github.com/tldr-pages/tldr)** 命令，把文档上的一些示例整出来给你看。
- 如果你想要一个图示化的`ping`，你可以试试 **[prettyping](https://github.com/denilsonsa/prettyping)** 。
- 如果你想搜索以前打过的命令，不要再用 Ctrl +R 了，你可以使用加强版的 **[hstr](https://github.com/dvorka/hstr)** 。
- **[htop](https://hisham.hm/htop/)** 是 top 的一个加强版。然而，还有很多的各式各样的top，比如：用于看IO负载的 **[iotop](http://guichaz.free.fr/iotop/)**，网络负载的 **[iftop](http://www.ex-parrot.com/~pdw/iftop/)**, 以及把这些top都集成在一起的 **[atop](https://github.com/Atoptool/atop)**。
- **[ncdu](https://dev.yorhel.nl/ncdu)** 比 du 好用多了。另一个选择是 [nnn](https://github.com/jarun/nnn)。
- 如果你想把你的命令行操作录制成一个 SVG 动图，那么你可以尝试使用 **[asciinema](https://asciinema.org/)** 和 **[svg-trem](https://github.com/marionebl/svg-term-cli)** 。
- **[httpie](https://github.com/jakubroztocil/httpie)** 是一个可以用来替代 `curl` 和 `wget` 的 http 客户端，`httpie` 支持 json 和语法高亮，可以使用简单的语法进行 http 访问: `http -v github.com`。
- **[tmux](https://github.com/tmux/tmux)** 在需要经常登录远程服务器工作的时候会很有用，可以保持远程登录的会话，还可以在一个窗口中查看多个 shell 的状态。
- **[sshrc](https://github.com/Russell91/sshrc)** 是个神器，在你登录远程服务器的时候也能使用本机的 shell 的 rc 文件中的配置。
- **[goaccess](https://github.com/allinurl/goaccess)** 这个是一个轻量级的分析统计日志文件的工具，主要是分析各种各样的 access log。





参考：

[shell基本语法](https://zhuanlan.zhihu.com/p/102176365?utm_oi=1129729846456283136) 







## vim



- `u` 撤销当前操作

- `.` 重复上一次的命令

- `N+命令`重复某个命令 N 次

  

参考：[vim高级使用](https://juejin.cn/post/6934580280627822599#heading-2) 







## read

请求输入， `read` 命令读取到的文本会立即被存储在一个变量里。

`read.sh` 

```bash
#!/bin/bash

read name

echo "hello $name !"
```

执行 `./read.sh` 时，会发现光标处于接收输入的状态，此时输入一个字符串 `will` 按下回车键后，控制台会打印出 `hello will` 



### 同时给几个变量赋值

可以用 `read` 命令一次性给多个变量赋值， `read` 命令一个单词一个单词（空格分开）地读取你输入的参数，并且把每个参数赋值给对应的变量。

```bash
#!/bin/bash

read oneName towName

echo "hello $oneName $towName !"
```



### 显示提示信息

`read` 命令的 `-p` 参数， `p` 是 `prompt` 的首字母，表示“提示”。

```bash
#!/bin/bash

read -p "请输入您的姓名：" name

echo "hello $name !"
```

```shell
# 换行
read -p $'please input file.csv (need format is CSV UTF-8) \x0a' file_name
```



### 限制字符数目

用 `-n` 参数可以限制用户输入的字符串的最大长度（字符数）

```bash
read -p "请输入您的姓名：" -n 5 name
```



### 限制输入时间

用 `-t` 参数可以限定用户的输入时间（单位：秒）超过这个时间，就不读取输入了。

```bash
read -p "请输入您的姓名：" -n 5 -t 10 name
```



### 隐藏输入内容

用 `-s` 参数可以隐藏输入内容，在用户输入密码时使用。

```bash
read -p "请输入密码：" -s password
```





## 变量



>  Shell 变量 分为**系统变量**和**自定义变量** 
>
> - 系统变量有$HOME、$PWD、$USER等 

```shell
set 
# 显示当前shell所有变量
```



### 设置普通变量

```shell
# 等号两侧不能有空格，变量名一般习惯用大写
# 变量名=变量值
WILL=14

# 删除变量
unset WILL

# 声明静态变量，静态变量不能unset
readonly WILL

# 使用变量
echo $WILL
```



[还有declare命令](https://www.runoob.com/linux/linux-comm-declare.html) 



### 设置环境变量



```shell
vim /etc/profile

# export 变量名=变量值，将 Shell 变量输出为环境变量
export JAVA_HOME=/usr/jdk1.8

# source 配置文件路径，让修改后的配置信息立即生效
source /etc/profile

# echo $变量名，检查环境变量是否生效
echo $JAVA_HOME
```



###  **位置参数变量** 



- $n ：$0 代表命令本身、$1-$9 代表第1到9个参数，10以上参数用花括号，如 ${10}。
- $* ：命令行中所有参数，且把所有参数**看成一个整体**。
- $@ ：命令行中所有参数，且把每个参数**区分**对待，for循环中常用。
- $# ：所有参数个数。

```shell
vim positionPara.sh

#!/bin/bash     
# 输出各个参数 
echo $0 $1 $2 
echo $* 
echo $@ 
echo 参数个数=$#
```

```shell
sh positionPara.sh left right

# 执行后的结果：
positionPara.sh left right
left right
left right
2
```



### 错误号

`$?` ： 在 Linux 下，所有的程序在结束时，都会有一个返回值，或者称为错误号 ( Error Number )

```shell
ls *.sh
# ......
echo $?
# 0: 只要返回值是 0，就代表程序执行成功了
```

 

**应用：** 

1. ```shell
   echo "database backup result: $? (0:successfully ;1:unsuccessfully)"
   ```

2. 给其中所有 markdown 文件最下方添加上一篇、下一篇、目录三个页脚链接，有的文章已经有了页脚，大部分都没有

   ```shell
   #!/bin/bash
   filename=$1
   
   # 查看文件尾部是否包含关键词
   tail | grep '下一篇' $filename
   
   # grep 查找到匹配会返回 0，找不到则返回非 0 值
   [ $? -ne 0 ] && { 添加页脚; }
   ```

3. ```shell
   # 将上次命令执行是否成功的返回值放到提示符里面去
   will@localhost ~ $ export PS1="[\$?]${PS1}"
   [0]will@localhost ~ $
   ```





**用perror查看错误提示** 

```shell
perror 2
# OS error code   2:  No such file or directory
```







###  **预定义变量** 

>  在赋值定义之前，事先在 Shell 脚本中直接引用的变量 

基本语法：

- $$ ：当前进程的 PID 进程号。
- $! ：后台运行的最后一个进程的 PID 进程号。 `!$`, 替换成上一次命令最后的路径
- $? ：最后一次执行的命令的返回状态，0为执行正确，非0执行失败。



```shell
#!/bin/bash

echo 当前的进程号=$$

# &：后台的方式运行进程
./ hello.sh &

echo 最后一个进程的进程号=$!
echo 最后执行的命令结果=$?
```

```shell
[root@izbp17osf716ivk4ilma92z test]# sh prePara.sh 
当前的进程号=4781
最后一个进程的进程号=4782
最后执行的命令结果=0
```



```shell
# 没有加可执行权限
$ /usr/bin/script.sh
zsh: permission denied: /usr/bin/script.sh

$ chmod +x !$
chmod +x /usr/bin/script.sh
```



### CDPATH

**在环境变量 `CDPATH` 中加入你常用的工作目录**，当 `cd` 命令在当前目录中找不到你指定的文件/目录时，会自动到 `CDPATH` 中的目录中寻找。

```shell
$ export CDPATH='~:/var/log'
# cd 命令将会在 ～ 目录和 /var/log 目录扩展搜索

$ pwd
/home/will/musics

$ cd mysql
cd /var/log/mysql

$ pwd
/var/log/mysql

$ cd my_pictures
cd /home/will/my_pictures
```











## 双引号“

==使用”$”来获取变量的时候最好加上双引号== 

 举一个例子： 

```shell
#!/bin/sh
#已知当前文件夹有一个a.sh的文件
var="*.sh"
echo $var
echo "$var"
```

```shell
# 运行结果如下：
a.sh
*.sh
```

 为啥会这样呢？其实可以解释为他执行了下面的命令： 

```shell
echo *.sh
echo "*.sh"
```



## 大括号{}

```shell
$ cp /very/long/path/file{,.bak}
# 给 file 复制一个叫做 file.bak 的副本

$ rm file{1,3,5}.txt
# 删除 file1.txt file3.txt file5.txt

$ mv *.{c,cpp} src/
# 将所有 .c 和 .cpp 为后缀的文件移入 src 文件夹
```

​	花括号中的每个字符都可以和之后（或之前）的字符串进行组合拼接，**注意花括号和其中的逗号不可以用空格分隔，否则会被认为是普通的字符串对待**。





## 参数规范



> 当脚本需要接受参数的时候，一定要先判断参数是否合乎规范，并给出合适的回显，方便使用者了解参数的使用 



```shell
# 判断参数的个数
#!/bin/bash

if [ $# != 2 ]
then
    echo "Parameter incorrect."
    exit 1
fi
```





## 命令替换

获取某一段命令的执行结果，它的方式有两种：

- `command`
- `$(command)` 

```bash
all_files=`ls` # 获取ls命令的执行结果
all_files=$(ls) # 效果同上
```

这两种命令替换的方式都是等价的，可以任选其一使用。



### 实际案例

获取系统的所有用户并输出：

```bash
#!/bin/bash

index=1

for user in `cat /etc/passwd | cut -d ":" -f 1`
do
	# cat /etc/passwd 获取系统中的所有用户和密码
	# cut -d ":" -f 1 根据冒号切每一行的字符串，获取切好后的第一部分
	# 使用for循环进行遍历，并输出用户
	echo "This is $index user: $user"
	index=$(($index + 1))
done
```



获取今年已经过了多少天和周：

```bash
echo "This year have passed $(date +%j) days"
echo "This year have passed $(($(date +%j)/7)) weeks"
```



注意： `$(())` 用于数学运算， `$()` 用于命令替换，这两个是平时使用中特别容易混淆的语法。










## 运算

`bash` 中的运算方式有以下几种：

- `$((...))` 或 `$[...]` 
- `expr` 
- `let` 
- `bc` 
- `declare -i` 

### $((...))

```bash
echo $((2 + 2)) # 加法
echo $((5 / 2)) # 除法

i=0
echo $((i++)) # 先返回值后运算
echo $((++i)) # 先运算后返回值

echo $(( (2 + 3) * 4 )) # 圆括号改变运算顺序

echo $((0xff)) # 16进制转成10进制的运算

echo $((16>>2)) # 位运算

echo $((3 > 2)) # 逻辑运算，如果逻辑表达式为真，返回1，否则返回0


a=0
echo $((a<1 ? 1 : 0)) # 三元运算符

echo $((a=1)) # 变量赋值
```

注意:  这个语法只能计算整数。

### expr

语法格式： `expr $num1 operator $num2`

操作符概览：

- `num1 | num2` -- `num1` 不为空且非0，返回 `num1` ; 否则返回 `num2` 
- `num1 & num2` -- `num1` 不为空且非0，返回 `num1` ；否则返回0
- `num1 < num2` -- `num1` 小于 `num2` ，返回1；否则返回0
- `num1 <= num2` -- `num1` 小于等于 num2 ，返回1；否则返回0
- `num1 = num2` -- `num1` 等于 `num2` ，返回1；否则返回0
- `num1 != num2` -- `num1` 不等于 `num2` ，返回1；否则返回0
- `num1 > num2` -- `num1` 大于 `num2` ，返回1；否则返回0
- `num1 >= num2` -- `num1`  大于等于 `num2` ，返回1；否则返回0
- `num1 + num2` -- 求和
- `num1 - num2` -- 求差
- `num1 * num2` -- 求积
- `num1 / num2` -- 求商
- `num1 % num2` -- 求余

```bash
$num1=30; $num2=50
expr $num1 \> $num2 # 大于不成立，返回0
expr $num1 \< $num2 # 小于成立，返回1
expr $num1 \| $num2 # 返回 num1
expr $num1 \& $num2 # 返回 num1
expr $num1 + $num2 # 计算加法
num3=`expr $num1 + $num2` # 计算结果赋值给num3
expr $num1 - $num2 # 计算减法
```

[注意] `>`、`<` 等操作符是 `Shell` 中的保留关键字，因此需要进行转义。否则就变成输出和输入重定向了。



#### 练习案例

提示用户输入一个正整数 `num` ，然后计算从 `1+2+3` 加到给定正整数。 必须对给定的 `num` 值进行是否为正整数判断，如果不是正整数则重新输入。

代码实现：

```bash
#!/bin/bash

while true # 无限循环接收用户输入
do
	read -p "pls input a positive number: " num # 接收到用户的输入，存为num值

	expr $num + 1 2>&1 /dev/null # 对num进行加1的运算，运算结果重定向到/dev/null
	
      # 由于 expr 只能运算整数，如果运算浮点数的话会报错，$?获取的是表达式执行结果，并非运算结果
      # 执行结果如果是正常的返回0
	if [ $? -eq 0 ];then # $? 获取到上一次运算结果
		if [ `expr $num \> 0` -eq 1 ];then # 上面判断了是否为整数，这里判断是否为正整数
    	# 类似JavaScript循环的写法遍历一个数值
			for((i=1;i<=$num;i++))
			do
				sum=`expr $sum + $i` # 获取运算结果总和
			done	
			echo "1+2+3+....+$num = $sum" # 输出运算结果
			exit # 退出循环
		fi
	fi
	echo "error,input enlegal" # 表示输入的值有误
	continue # 如果输入有误继续循环让用户输入
done
```

### let 命令

`let` 命令声明变量时，可以直接执行算术表达式

```bash
let "foo = 1 + 2"
echo $foo # 3
```

### bc

支持浮点数运算

```bash
echo "scale=2;23/5" | bc # scale表示浮点位数
num1=`echo "scale=2;23/5" | bc` # 运算结果保存为变量
```



### declare -i 命令

`-i` 参数声明整数变量以后，可以直接进行数学运算

```bash
declare -i val1=12 val2=5
declare -i result
result=val1*val2
echo $result # 60
```





## declare

`declare` 命令可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。



### 语法格式

```bash
declare OPTION VARIABLE=value
```



### 常用参数

- `-r` 将变量设为只读；
- `-i` 将变量设为整数；
- `-a` 将变量定义为数组；
- `-f` 显示此脚本前定义过的所有函数及内容；
- `-F` 仅显示此脚本前定义过的函数名；
- `-x` 将变量声明为环境变量。

 

`declare`命令如果用在函数中，声明的变量只在函数内部有效，等同于`local`命令。

不带任何参数时，`declare`命令输出当前环境的所有变量，包括函数在内，等同于不带有任何参数的`set`命令。



##### `-i`

`-i` 参数声明整数变量以后，可以直接进行数学运算。

```bash
declare -i val1=12 val2=5
declare -i result
result=val1*val2
echo $result # 60
```

##### `-x`

`-x` 参数等同于 `export` 命令，可以输出一个变量为子 `Shell` 的环境变量。

```bash
declare -x foo=3 

# 等同于
export foo=3 
```

##### `-r`

`-r` 参数可以声明只读变量，无法改变变量值，也不能 `unset` 变量。

```bash
declare -r bar=1

# 如果此时更改bar
bar=2 # -bash: bar：只读变量
```

##### `-u` 

`-u` 参数声明变量为大写字母，可以自动把变量值转成大写字母。

```bash
declare -u foo
foo=upper
echo $foo # UPPER
```

##### `-l` 

`-l` 参数声明变量为小写字母，可以自动把变量值转成小写字母。

```bash
declare -l bar
bar=LOWER
echo $bar # lower
```

##### `-p` 

`-p` 参数输出变量信息。

```bash
foo=hello
declare -p foo #控制台输出：declare -x foo="hello"
```







## 数组



### 定义数组

```bash
array=('v1' 'v2' 'v3') 
```



### 输出数组内容

```bash
${array[@]} # 输出全部内容
${array[*]} # 也是输出全部内容
${array[1]} # 输出下标索引为1的内容
```



### 获取数组长度

```bash
${#array} # 数组内元素个数
${#array[2]} # 数组内下标为2的元素长度
```



### 数组赋值

```bash
array[0]="frank" # 给数组下标为1的元素赋值为frank
array[20]="lion" # 在数组尾部添加一个新元素
```



### 删除元素

```bash
unset array[2] # 清除元素
unset array # 清空整个数组
```



### 数组遍历

```bash
for v in ${array[@]}
do
done
```







## 条件



```shell
vim guessGame.sh

#!/bin/bash

echo '你猜的数字为：'$1

if [ $1 -gt 7 ]
then
        echo '大于7'
elif [ $1 -lt 4 ]
then
        echo '小于4'
fi
```

注意：

```shell
[ condition ] : 前后都要有空格，非空返回0，0为true，否则为false

if [ : `if`距离条件方括号之间，需要有空格
```

```shell
# 是否存在文件/root/shell/isExist.txt 
if [ -e /root/shell/isExist.txt ] 
then
     echo '存在' 
fi  
```

```shell
if [ $1 = 7 ] && echo 'hello' || echo 'world' 
then
     echo '条件满足，执行后面的语句' 
fi
```



## case

基本语法：

```shell
case $变量名 in
"值1")
如果变量值等于值1，则执行此处程序1
;;
"值2")
如果变量值等于值2，则执行此处程序2
;;
...省略其它分支...
*)
如果变量值不等于以上列出的值，则执行此处程序
;;
esac
```



```shell
case $1 in
"1")
echo 周一
;;
"2")
echo 周二
;;
*)
echo 其它
;;
esac
```



## for

基本语法：

```shell
# 语法1
for 变量名 in 值1 值2 值3...
do
    程序
done

# 语法2
for ((初始值; 循环控制条件; 变量变化))
do
    程序
done
```



### 语法一

==$*== ：

```shell
#!/bin/bash  

# 使用$* 
for i in "$*" 
do     
    echo "the arg is $i" 
done 

# 输出：
thearg is 1 2 3 
```

 ==$@== ：

```shell
#!/bin/bash  

# 使用$@ 
for j in "$@" 
do     
    echo "the arg is $j" 
done

# 输出：
the arg is 1 
the arg is 2 
the arg is 3
```



### 语法二

```shell
#!/bin/bash 
SUM=0  
for ((i=1;i<=100;i++)) 
do     
    SUM=$[$SUM+$i] 
done 

echo $SUM
# 输出从1加到100的值
```





## while

基本语法：

```shell
while [ 条件判断式 ]
do
    程序
done 
```



```shell
#!/bin/bash
SUM=0
i=0

while [ $i -le $1 ]
do
    SUM=$[$SUM+$i]
    i=$[$i+1]
done       
echo $SUM
# 输出从1加到100的值
```











## 读取控制台输入

基本语法：

```shell
read(选项)(参数)
```

- 选项
  -  -p：指定读取值时的提示符 
  - -t：指定读取值时等待的时间（秒），如果没有在指定时间内输入，就不再等待
- 参数
  - 变量名：读取值的变量名



```shell
vim guessGame.sh


```





## 函数





### 自定义函数

基本语法：

```shell
[ function ] funname[()]
{
    Action;
    [return int;]
}

# 调用
funname 参数1 参数2...
```

- 函数名后面的圆括号中`不加任何参数`，这点与主流编程语言不相同
- 函数的完整定义必须`置于函数的调用之前` 



```shell
#!/bin/bash

function getSum(){
    SUM=$[$n1+$n2]
}   

# 函数中的变量 均为全局变量，没有局部变量
echo "sum=$SUM"


read -p "请输入第一个参数n1：" n1
read -p "请输入第二个参数n2：" n2

# 调用 getSum 函数
getSum $n1 $n2
```



### 传递参数

```bash
#!/bin/bash

print_something(){
    echo "hello $1" # $1 获取第一个参数
}
print_something Lion # Lion 为参数
print_something Frank # Frank 为参数
```




### 函数返回值

 shell中函数的返回值`只能是整数`  

```shell
#!/bin/bash

print_something(){
    echo "Hello $1"
    return 1
}

print_something will

echo "函数的返回值是 $?" 
# $? 获取到的是上一个函数的返回值
```



 也可以通过下面变通的方法:   通过echo或者print之类的就可以做到`传一些额外参数`的目的 

```shell
#!/bin/bash

func(){
    echo "2333"
}
res=$(func)

echo "This is from function: $res."
```



### 函数的局部变量

```bash
#!/bin/bash

local_global(){
# 通过 local 关键字定义局部变量
  local var1='local 1' 
  echo "var1 is $var1"
}

local_global
```





### 巧用main函数

```shell
#!/bin/bash

func1(){
    #do sth
}
func2(){
    #do sth
}
main(){
    func1
    func2
}
main "$@"
```

> func1 / func2 必须定义在**调用**main函数之前，可以定义在main函数之后



### 作用域



```shell
#!/bin/bash

var=1

func(){
   var=2
}
func

echo $var
# 2 : 输出结果就是2而不是1
```



### 间接引用









### basename

-  basename，删掉路径最后一个 / 前的所有部分（包括/），常用于获取文件名 
   - basename [pathname] [suffx]
   - basename [string] [suffx]
   - 如果指定 suffx，也会删掉pathname或string的后缀部分

```shell
basename /usr/bin/sort  
# sort  

basename include/stdio.h  
# stdio.h  

basename include/stdio.h .h 
# stdio
```



### dirname

-  dirname，删掉路径最后一个 / 后的所有部分（包括/），常用于获取文件路径 
   - dirname pathname
   - 如果路径中不含 / ，则返回 '.' （当前路径）

```shell
dirname /usr/bin/  
# /usr  

dirname dir1/str dir2/str 
# dir1 
# dir2  

dirname stdio.h 
# .
# 表示当前路径
```





## 字符串



### 字符串替换

替换规则：

- `${变量名#匹配规则}` 从变量开头进行规则匹配，将符合最短的数据删除。
- `${变量名##匹配规则}` 从变量开头进行规则匹配，将符合最长的数据删除。
- `${变量名%匹配规则}` 从变量尾部进行规则匹配，将符合最短的数据删除。
- `${变量名%%匹配规则}` 从变量尾部进行规则匹配，将符合最长的数据删除。
- `${变量名/旧字符串/新字符串}` 变量内容符合旧字符串，则第一个旧字符串会被新字符串取代。
- `${变量名//旧字符串/新字符串}` 变量内容符合旧字符串，则全部旧字符串会被新字符串取代。

```bash
var_1="I love you, Do you love me"

var=${var_1#*ov} # e you, Do you love me

var=${var_1##*ov} # e me

var=${var_1%ov*} # I love you, Do you l

var=${var_1%%ov*} # I l

var_2="/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"

# 第一个小写bin被替换为大写的BIN
var=${var_2/bin/BIN} # /usr/local/BIN:/usr/bin:/bin:/usr/sbin:/sbin 

# 所有小写bin会被替换为大写的BIN
var=${var_2//bin/BIN} # /usr/local/BIN:/usr/BIN:/BIN:/usr/sBIN:/sBIN
```



### 计算字符串的长度

- `${#string}` 
- `expr length "$string"` 如果 `string` 有空格则必须加双引号。

```bash
var_1="Hello world"

len=${#var_1} # 11

len=`expr length "$var_1"` # 11
```

### 获取子串在字符串中的索引位置

- `expr index $string $substring` 从1开始计算索引位置。

```bash
var_1="quickstart is an app"

ind=`expr index "$var_1" start` # 6
ind=`expr index "$var_1" uniq` # 1
ind=`expr index "$var_1" f` # 0
```

它其实是按照子串的每个字符每个字符去进行匹配

- 第一个例子匹配到的是 `s` 位置6。
- 第二个匹配到的是 `q` 位置1。
- 第三个例子什么都没有匹配到所以位置是0。



### 计算子串长度

`expr match $string substr`从头开始匹配子串长度，如果没有匹配到则返回0，匹配到了则返回配的子串长度。

```bash
var_1="quickstart is an app"

sub_len=`expr match "$var_1" app` # 0
sub_len=`expr match "$var_1" quic` # 4
sub_len=`expr match "$var_1" quic.*` # 18
```

### 抽取子串

- `${string:position}` 从 `string` 的 `position` 开始。
- `${string:position:length}` 从 `position` 开始，匹配长度为 `length` 。
- `${string: -position}` 从右边开始匹配。
- `${string:(position)}` 从左边开始匹配。
- `expr substr $string $postion $length` 从 `position` 开始，匹配长度为 `length` 。

```bash
var_1="quickstartisanapp"

# 从索引10到最后提取子串，这种方式的索引是从0开始计算的
substr=${var_1:10} # isanapp

substr=${var_1:10:2} # is

# -5 前面需要添加空格，如果不添加空格也可以使用括号
substr=${var_1: -5} # anapp
substr=${var_1:(-5)} # anapp

# expr 这种方式的索引是从1开始
substr=`expr substr "$var_1" 10 5` # tisan
```



### 实战练习

变量 `string="Bigdata process framework is Hadoop,Hadoop is an open source project"` 执行脚本后，打印输出 `string` 字符串变量，并给用户以下选项：

- (1)、打印 `string` 长度
- (2)、删除字符串中所有的 `Hadoop` 
- (3)、替换第一个 `Hadoop` 为 `Mapreduce` 
- (4)、替换全部 `Hadoop` 为 `Mapreduce` 

用户输入数字 `1|2|3|4` ，可以执行对应项的功能；输入 `q|Q` 则退出交互模式。

#### 思路分析

1. 打印字符串很简单，直接 `echo` 打印即可。
2. 删除字符串也不难，使用 `${变量名//旧字符串/新字符串}` ，把 `Hadoop` 替换为空就从原字符串中删除了。
3. 至于替换也是使用 `${变量名/旧字符串//新字符串}`
4. 用户输入，则使用的是上一篇文章讲到的 `read` 命令。



#### 代码实现

```bash
#!/bin/bash

string="Bigdata process framework is Hadoop,Hadoop is an open source project"

# 打印提示函数
print_tips(){
  echo "(1)、打印string长度"
  echo "(2)、删除字符串中所有的Hadoop"
  echo "(3)、替换第一个Hadoop为Mapreduce"
  echo "(4)、替换全部Hadoop为Mapreduce"
}

# 输出字符串长度函数
len_of_string(){
  echo "${#string}"
}

# 替换Hadoop子串为空，相当于删除
delete_Hadoop(){
  echo "${string//Hadoop/}"
}

# 替换一个Hadoop为Mapreduce
rep_hadoop_mapreduce_first(){
  echo "${string/Hadoop/Mapreduce}"
}

# 替换全部Hadoop为Mapreduce
rep_hadoop_mapreduce_all(){
  echo "${string//Hadoop/Mapreduce}"
}

# 无限循环的接收用户输入，直到用户输入了q|Q则退出进程
while true
do
  echo "$string" # 打印字符串
  echo # 打印一个空
  print_tips # 输出提示信息
  read -p "请输入你的选择(1|2|3|4|q|Q)：" choice # 接收用户输入
  # 根据用户的输入，进入不同的分支，调用不同的函数
  case $choice in
    1)
    	len_of_string
      ;;
    2)
    	delete_Hadoop
      ;;
    3)
    	rep_hadoop_mapreduce_first
      ;;
    4)
    	rep_hadoop_mapreduce_all
      ;;
    q|Q)
    	exit
      ;;
    *)
    	echo "错误的输入"
      ;;
    esac  
done
```





## 文本三剑客

[参考：文本处理](https://juejin.cn/post/6935365727205457928#heading-56) 



### grep

全局搜索一个正则表达式，并且打印到屏幕。简单来说就是，在文件中查找关键字，并显示关键字所在行。

#### 基础语法

```bash
grep text file # text代表要搜索的文本，file代表供搜索的文件

# 实例
grep path /etc/profile
# pathmunge () {
#     pathmunge /usr/sbin
#     pathmunge /usr/local/sbin
#     pathmunge /usr/local/sbin after
#     pathmunge /usr/sbin after
# unset -f pathmunge
```



#### 常用参数

- `-i` 忽略大小写， `grep -i path /etc/profile`
- `-n` 显示行号，`grep -n path /etc/profile`
- `-v` 只显示搜索文本不在的那些行，`grep -v path /etc/profile`
- `-r` 递归查找， `grep -r hello /etc` ， `Linux` 中还有一个 `rgrep` 命令，作用相当于 `grep -r`



#### 高级用法

`grep` 可以配合正则表达式使用。

```bash
grep -E path /etc/profile --> 完全匹配path
grep -E ^path /etc/profile --> 匹配path开头的字符串
grep -E [Pp]ath /etc/profile --> 匹配path或Path
```



### sed

`stream Editor` 的缩写，**<u>流编辑器</u>**，对***标准输出或文件*** `逐行` 进行处理。



#### 语法格式

```bash
sed [option] "pattern/command" file

# 例如：
sed '/python/p' name.txt 
# 省略了option
# /python/ 为pattern正则
# p 为command命令打印的意思
```

注意：匹配模式中存在<u>变量要使用双引号</u>。



#### 选项 option

##### `-n` 只打印模式匹配行

```bash
sed 'p' name.txt 
# 会对每一行字符串输出两遍，第一遍是原文本，第二遍是模式匹配到的文本

sed -n 'p' name.txt 
# 加了-n参数后就只打印模式匹配到的行
```



##### -e 默认选项

支持多个 `pattern command` 的形式

```bash
sed -e "pattern command" -e "pattern command" file
```



##### -f 指定动作文件

```bash
# 简单命令
sed -n '/python/p' name.txt 
# 但是一旦正则比较复杂，我们就可以把它保存在一个单独文件中，使用-f进行指定


sed -n -f edit.sed name.txt 
# '/python/p'这个命令保存在edit.sed文件中，使用-f指定
# edit.sed 内容为 /python/p
```



##### -E 扩展表达式

```bash
sed -n -E '/python|PYTHON/p' name.txt
```



##### -i 直接修改文件内容

```bash
sed -i 's/love/like/g' name.txt
# -i修改原文件
# 全部love替换为like
```



#### 模式匹配 pattern

`pattern` 用法表

| 匹配模式                       | 含义                                                     | 用法                                                         |
| ------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| `10command`                    | 匹配第10行                                               | `sed -n "17p" name.txt` 打印 name 文件的第17行               |
| `10,20command`                 | 匹配从第10行开始，到第20行结束                           | `sed -n "10,20p" name.txt` 打印 `name` 文件的10到20行        |
| `10,+5command`                 | 匹配从第10行开始，到第15行结束                           | `sed -n "10,+5p" name.txt`                                   |
| `/pattern1/command`            | 匹配到 `pattern1` 的行                                   | `sed -n "/^root/p" name.txt` 打印 `root` 开头的行            |
| `/pattern1/,/pattern2/command` | 匹配到 `pattern1` 的行开始，至匹配到 `pattern2` 的行结束 | `sed -n "/^ftp/,/^mail/p" name.txt`                          |
| `10,/pattern1/command`         | 匹配从第10行开始，到匹配到 `pattern1` 的行结束           | `sed -n "4,/^ok/p" name.txt` 打印 `name` 文件从第4行开始匹配，直到以 `ok` 开头的行结束 |
| `/pattern1/,10command`         | 匹配到 `pattern1` 的行开始，到第10行匹配结束             | `sed -n "/root/,10p" name.txt`                               |

#### 命令 command

- 查询
  - `p` -- 打印
- 增加
  - `a` -- 行后追加
  - `i` -- 行前追加
  - `r` -- 外部文件读入，行后追加
  - `w` -- 匹配行写入外部文件
- 删除
  - `d` -- 删除
- 修改
  - `s/old/new` -- 将行内第一个 `old` 替换为 `new`
  - `s/old/new/g` -- 将行内全部的 `old` 替换为 `new`
  - `s/old/new/2g` --同一行，只替换从第二个开始到剩下所有的
  - `s/old/new/ig` -- 将行内 `old` 全部替换为 `new` ，忽略大小写

实例：

```bash
sed '/python/p' name.txt # 打印

sed '1d' name.txt # 删除第一行

sed -i '/frank/a hello' name.txt # 匹配到字符串frank就在这一行后面插入“hello”
sed -i '/frank/r list' name.txt # 把list文件内容追加到匹配frank字符串后面的行
sed -n '/frank/w tmp.txt' name.txt # 将匹配到frank的行全部写入tmp.txt文件中

sed -i 's/love/like/g' name.txt # 全局替换love为like，并修改源文件
```



#### 反向应用

引用前面的匹配到的字符。

假设有一个文件 `test.txt` 内容为：

```bash
a heAAo vbv
c heBBo sdsad
k heCCo mnh
```

现在需要匹配到 `heAAo heBBo heCCo` 并在他们后面添加 `s` 。

使用反向引用可以这样做：

```bash
sed -i 's/he..o/&s/g' test.txt # 其中&就是引用前面匹配到的字符
sed -i 's/\(he..o\)/\1s/g' test.txt # 与上面的写法等价

# 输出结果
a heAAos vbv
c heBBos sdsad
k heCCos mnh
```



### awk

`awk` 是一个文本处理工具，通常用于处理数据并生成结果报告。



#### 语法格式

```bash
awk 'BEGIN{}pattern{commands}END{}' file_name
```

- `BEGIN{}` 正式处理数据之前执行；
- `pattern` 匹配模式；
- `{commands}` 处理命令，可能多行；
- `END{}` 处理完所有匹配数据后执行。







#### 内置变量

- `$0` -- 整行内容。
- `$1-$n` -- 当前行的第 `1-n` 个字段。
- `NF` -- `Number Field` 的缩写，表示当前行的字段个数，也就是有多少列。
- `NR` -- `Number Row` 的缩写，表示当前行的行号，从1开始计数。
- `FNR` -- `File Number Row` 的缩写，表示多文本处理时，每个文件行号单独计数，都是从1开始。
- `FS` -- `Field Separator` 的缩写，表示输入字段分隔符，不指定默认以空格或 `tab` 键分割。
- `RS` -- `Row Separator` 的缩写，表示输入行分隔符，默认回车换行 `\n` 。
- `OFS` -- `Output Field Separator` ，表示输出字段分隔符，默认为空格。
- `ORS` -- `Output Row Separator` 的缩写，表示输出行分隔符，默认为回车换行。
- `FILENAME` -- 当前输入的文件名字。
- `ARGC` -- 命令行参数个数。
- `ARGV` -- 命令行参数数组。

注意：字段个数，非字符个数，例如 `"frank lion alan"` 这一行有3个字段。

```bash
awk '{print $0}' /etc/passwd 
# 只有处理命令，输出每一行的内容

awk 'BEGIN{FS=":"}{print $1}' /etc/passwd 
# 处理文本之前先定义好分隔符为冒号，然后打印每一行被冒号分割后的文本的第一个字段

awk 'BEGIN{FS=":"}{print $NF}' /etc/passwd 
# 输出每一行的最后一列，NF中存的总列数，因此$NF表示最后一列的字段

awk '{print $NF}' /etc/passwd 
# 输出每一行的字段个数（默认以空格分割后的字段个数）

awk '{print NR}' /etc/passwd 
# 输出文件的行号

awk '{print FNR}' /etc/passwd name.txt 
# 输出两个文件的行号，每个文件单独计数

awk 'BEGIN{RS="---"}{print $0}' /etc/passwd 
# 行分隔符设置为---

awk 'BEGIN{ORS="---"}{print $0}' /etc/passwd 
# 定义输出的每一行的分隔符为 --- 

awk 'BEGIN{ORS="---";RS="---"}{print $0}' /etc/passwd 
# BEGIN 中定义多个变量需要;分割

awk '{print FILENAME}' name.txt 
# 输出name.txt
```

备注：`/etc/passwd` 就是 `Linux` 系统中的密码文本，后续会一直使用，大概是如下格式：

```bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
```



实例：

```shell
# 切割字符串，并生成数组
cat excel-utf8.csv | while read line
do
        #echo ${line} | awk 'BEGIN {FS=",";OFS=" "} {print $1, $2}'
        
        # 读取每行，以","号分割，生成数组
        arr=(`echo ${line} | awk '{len=split($0,a,","); for(i=1; i<=len; i++) print a[i]}'`)
        
        # 输出数组元素
        echo "coustomer: ${arr[0]}, path: ${arr[1]}"
done
```

[cut 可以从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出](https://www.runoob.com/linux/linux-comm-cut.html) 



```shell
awk -F : 'BEGIN {print "start1, start7"} {print $1 "," $7} END {print "end1, end7"}' /etc/passwd


# NF,浏览记录的字段个数（Field) ; NR,已读的记录数, 每行依次递增; 
awk -F : '{ print NR  " "  NF  " "  $NF}' /etc/passwd  
1 7 /bin/bash
2 7 /sbin/nologin
3 7 /sbin/nologin
4 7 /sbin/nologin
```









#### printf

`awk` 格式化输出。

格式符：

- `%s` -- 打印字符串；
- `%d` -- 打印10进制数；
- `%f` -- 打印浮点数；
- `%x` -- 打印16进制数；
- `%o` -- 打印8进制数；
- `%e` -- 打印数字的科学计数法格式；
- `%c` -- 打印单个字符的 `ASCII` 码。

修饰符：

- `-` -- 左对齐；
- `+` -- 右对齐；
- `#` -- 显示8进制在前面加0，显示16进制在前面加 `0x` 。

```bash
awk 'BEGIN{FS=":"}{printf $1}' /etc/passwd
```

输入上面 `awk` 命令打印后得到这样的结果： ![](https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/AWKPrintf.png) 不再像使用 `print` 命令会默认以换行符作为默认的输出了，而是全部集中在一行，因此使用 `printf` 需要自定义输出格式：

```bash
awk 'BEGIN{FS=":"}{printf "%s\n",$1}' /etc/passwd # %s表示打印字符串 \n表示换行
```

<img src="C:%5CUsers%5Cwille%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210307140705556.png" alt="image-20210307140705556" style="zoom:57%;" />



 `printf` 在处理多个字段的分隔符上是非常方便的：

```bash
awk 'BEGIN{FS=":"}{printf "%20s %20s\n",$1,$7}' /etc/passwd
```

- `%20s` 表示 `20` 个字符，如果实际字符不够会默认使用空格补全
- `$1` 表示第一部分， `$7` 表示第 `7` 部分

<img src="https://cdn.jsdelivr.net/gh/MicroWiller/photobed@master/PrintfWithFormat.png" style="zoom:50%;" />

默认是右对齐，也可以添加修饰符，使其左对齐：

```bash
awk 'BEGIN{FS=":"}{printf "%-20s %-20s\n",$1,$7}' /etc/passwd
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d88954a4d18c4b62bff3bcbdbb8f71f0~tplv-k3u1fbpfcp-zoom-1.image)

其它示例：

```bash
awk 'BEGIN{FS=":"} {printf "%d\n", $3}' /etc/passwd 
# 打印十进制数字，默认是左对齐的

awk 'BEGIN{FS=":"} {printf "%20d\n", $3}' /etc/passwd 
# %20d 不够的位数默认补全空格，因此它是右对齐

awk 'BEGIN{FS=":"} {printf "%f\n", $3}' /etc/passwd 
# 打印浮点数，默认保留小数点后6位置

awk 'BEGIN{FS=":"} {printf "%0.2f\n", $3}' /etc/passwd 
# 打印浮点数，保留小数点后2位置
```



#### 模式匹配 pattern

它与 `sed` 的 `pattern` 非常类似，都是`按行`进行匹配。

模式匹配的两种用法：

1. 按正则表达式匹配
2. 运算符匹配

正则匹配：

```bash
# 匹配/etc/passwd文件行中包含root字符串的所有行
awk '/root/{print $0}' /etc/passwd

# 匹配/etc/passwd文件行中以lion开头的所有行
awk '/^lion/{print $0}' /etc/passwd
```



运算符匹配：

- `<` 小于
- `>` 大于
- `<=` 小于等于
- `>=` 大于等于
- `==` 等于
- `!=` 不等于
- `~` 匹配正则表达式
- `!~` 不匹配正则表达式
- `||` 或
- `&&` 与
- `!` 非

```bash
# 以:为分隔符，匹配/etc/passwd文件中第三个字段小于50的所有信息
awk 'BEGIN{FS=":"}$3<50{print $0}' /etc/passwd

# 输出第三个字段值等于1的行
awk 'BEGIN{FS=":"}$3==1{print $0}' /etc/passwd

# 输出第七个字段值等于 /bin/bash 的行
awk 'BEGIN{FS=":"}$7=="/bin/bash"{print $0}' /etc/passwd

# 输出第三个字段值的位数大于3的行
awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{print $0}' /etc/passwd

# 以:为分隔符，匹配/etc/passwd文件中第一个字段包含hdfs或yarn的所有行信息
awk 'BEGIN{FS=":"}$1=="lion" || $1=="frank" {print $0}' /etc/passwd

# 以:为分隔符，匹配/etc/passwd文件中第三个字段小于50并且第四个字段大于50的所有行信息
awk 'BEGIN{FS=":"}$3<50 && $4>50{print $0}' /etc/passwd
```



#### awk 中的表达式

```bash
awk 'BEGIN{var=20;var1="aaa";print var,var1}' 
# 输出 20 aaa

awk 'BEGIN{n1=20;n2+=n1;print n1,n2}' 
# 20 20

awk 'BEGIN{n1=20;n2=30;printf "%0.2f\n",n1/n2}' 
# 输出浮点数，精确到小数点后两位
```

注意：在 `BEGIN` 和 `END` 中都可以正常执行表达式。



##### 练习：使用 awk 计算 `/etc/services` 中的空白行数量

```bash
# BEGIN 定义变量，通过正则/^$/匹配到空行后，执行sum++，全面匹配结束后在打印sum的值
awk 'BEGIN{sum=0}/^$/{sum++}END{print sum}' /etc/services 
```



#### awk 中的条件语句

语法格式：

```bash
if(条件表达式)
	{动作1} 
else if(条件表达式)
	{动作2}
else
	{动作3}
```

练习：以冒号为分隔符，只打印 `/etc/passwd` 中第3个字段的数值在 `50-100` 范围内的行信息

```bash
awk 'BEGIN{FS=":"}{ if($3>50 && $3<100){print $0} }' /etc/passwd
```

#### awk 中的循环语句

语法格式：

```bash
# while 循环
while(条件表达式)
  动作
    
# do while 循环
do
  动作
while(条件表达式)
  
# for循环
for(初始化计数器;计数器测试;计数器变更)
  动作
```



##### 计算 1+2+3+...+100 的和

`while` 循环：

```bash
创建文件：while.awk
BEGIN{
  while(i<=100){
    sum+=i
    i++
  }
  print sum
}
执行命令：awk -f while.awk # 以脚本的方式执行awk动作
```

`for` 循环：

```bash
创建文件：for.awk
BEGIN{
  for(i=0;i<=100;i++)
  {
  	sum+=i
  }
  print sum
}
执行命令：awk -f for.awk # 以脚本的方式执行awk动作
```

`do...while` 循环：

```bash
创建文件：do-while.awk
BEGIN{
  do
  {
    sum+=i
    i++
  }while(i<=100)
  print sum
}
执行命令：awk -f do-while.awk # 以脚本的方式执行awk动作
```

#### awk 字符串函数

- `length(str)` 计算长度；
- `index(str1,str2)` 返回在 `str1` 中查询到的 `str2` 的位置；
- `tolower(str)` 小写转换；
- `toupper(str)` 大写转换；
- `split(str,arr,fs)` 分隔符字符串，并保存到数组中；
- `match(str,RE)` 返回正则表达式匹配到的子串位置；
- `substr(str,m,n)` 截取子串，从 `m` 个字符开始，截取 `n` 位。若 `n` 不指定，则默认截取到字符串末尾；
- `sub(RE,ReqStr,str)` 替换查找到的第一个子串；
- `gsub(RE,ReqStr,str)` 替换查找到的所有子串。

##### length(str)

计算字符长度

```bash
awk 'BEGIN{FS=":"}{print length($3)}' /etc/passwd # 计算第三个字段的长度
```

##### index(str1,str2)

返回在 `str1` 中查询到的 `str2` 的位置

```bash
awk 'BEGIN{print index("I have a dream","ea")}' # 查找ea在字符串中的位置
```

##### tolower(str)

转换为小写字符

```bash
awk 'BEGIN{print tolower("I have a dream")}' # i have a dream
```

##### toupper(str)

转换为大写字符

```bash
awk 'BEGIN{print toupper("I have a dream")}' # I HAVE A DREAM
```

##### split(str,arr,fs)

分隔符字符串，并保存到数组中

```bash
# 将字符串"I have a dream"按照空格进行分割

awk 'BEGIN{str="I have a dream";split(str,arr," ");for(a in arr) print arr[a]}'

# 输出
# dream
# I
# have
# a
```

通过 `for…in` 得到是无序的数组。如果需要得到有序数组，需要通过下标获得：

```bash
awk 'BEGIN{str="I have a dream";len = split(str,arr," ");for(i=1;i<=len;i++) print arr[i]}'
```

##### match(str,RE)

返回正则表达式匹配到的子串位置

```bash
# 输出字符串第一个数字出现所在的位置
awk 'BEGIN{str="I have 1 dream";print match(str,/[0-9]/)}' # 8
```

##### substr(str,m,n)

截取子串，从 `m` 个字符开始，截取 `n` 位。若 `n` 不指定，则默认截取到字符串末尾。

```bash
# 截取字符串"I have a dream"的子串，从第四个位置开始，一直到最后
awk 'BEGIN{str="I have a dream";print substr(str,4)}' # ave a dream
```

##### sub(RE,ReqStr,str)

替换查找到的第一个子串。

- `RE` 为正则表达式
- `ReqStr` 为要替换的字符串
- `str` 为源字符串

```bash
# 替换字符串"I have 123 dream"中第一个数字为$符号
awk 'BEGIN{str="I have 123 dream";sub(/[0-9]+/,"$",str);print str}' # I have $ dream
```

##### gsub(RE,ReqStr,str)

替换查找到的所有子串

- `RE` 为正则表达式
- `ReqStr` 为要替换的字符串
- `str` 为源字符串

```bash
# 替换字符串"I have 123 dream 456"中第一个数字为$符号
awk 'BEGIN{str="I have 123 dream 456";gsub(/[0-9]+/,"$",str);print str}' # I have $ dream $
```

#### 选项 option

- `-v` 参数传递
- `-f` 指定脚本文件
- `-F` 指定分隔符
- `-V` 查看 `awk` 的版本号

##### 参数传递

`awk` 的 `BEGIN` 中不能直接使用 `shell` 中定义的变量，需要通过 `-v` 参数进行传递

```bash
[root@lion ~]# var=20
[root@lion ~]# num=30
[root@lion ~]# awk -v var1="$var" -v var2="$num" 'BEGIN{print var1,var2}'
20 30
```

##### 指定脚本文件

`'BEGIN{}pattern{commands}END{}'` 这里的内容可以写在一个独立的文件中，通过 `-f` 参数来引入。

```bash
# test.awk
BEGIN{
  var=30
  print var
}

awk -f test.awk # 输出30
```

这种写法更易于编写和维护，适合复杂 `awk` 语句。

##### 指定分隔符

```bash
awk 'BEGIN{FS=":"}{print length($3)}' /etc/passwd

等同于：

awk -F : '{print length($3)}' /etc/passwd
```

##### 查看 awk 的版本号

```bash
[root@lion ~]# awk -V
GNU Awk 4.0.2
Copyright (C) 1989, 1991-2012 Free Software Foundation.
...
```



#### awk 处理生产数据实例

假设有一堆以下格式数据保存在文件 `data.txt` 中：

```bash
2020-01-01 00:01:01 1000 Batches: user frank insert 2010 records databases:product table:detail, insert 1000 records successfully,failed 1010 records
2020-01-01 00:01:02 1001 Batches: user lion insert 1030 records databases:product table:detail, insert 989 records successfully,failed 41 records
2020-01-01 00:01:03 1002 Batches: user mike insert 1290 records databases:product table:detail, insert 235 records successfully,failed 1055 records
2020-01-01 00:01:04 1003 Batches: user alan insert 3452 records databases:product table:detail, insert 1257 records successfully,failed 2195 records
2020-01-01 00:01:05 1004 Batches: user ben insert 4353 records databases:product table:detail, insert 2245 records successfully,failed 2108 records
2020-01-01 00:01:06 1005 Batches: user bily insert 5633 records databases:product table:detail, insert 3456 records successfully,failed 2177 records
2020-01-01 00:01:07 1006 Batches: user frank insert 2010 records databases:product table:detail, insert 1000 records successfully,failed 1010 records
```

统计每个人员分别插入了多少条 record 进数据库，预期结果：

```bash
USER	Total Records
frank	4020
lion  1030
...
```

代码演示：

```bash
# data.awk
BEGIN{
	printf "%-10s %-10s\n", "User", "Total Records"
}

{
	USER[$6]+=$8 # USER为数组，空格分割后$6这个变量是用户名，$8是records，每次都去数组中取出$6的值进行相加以获取重复用户的总Records
}

END{
    for(u in USER) # 遍历USER数组，打印数组每一项
    {
    	printf "%-10s %-10d\n", u , USER[u]
    }
}

# awk 语句
awk -f data.awk data.txt

# 输出结果
User       Total Records
frank      4020      
mike       1290      
bily       5633      
alan       3452      
lion       1030      
ben        4353 
```










## **heredocs**



 使用heredocs，可以非常方便的生成一些模板文件： 

```shell
cat>>/etc/rsyncd.conf << EOF
log file = /usr/local/logs/rsyncd.log
transfer logging = yes
log format = %t %a %m %f %b
syslog facility = local3
EOF
```







## 脚本执行路径

```shell
pwd
# pwd获得的是当前shell的执行路径，而不是当前脚本的执行路径

# /home/will/test
```

```shell
# 正确的做法应该是下面这两种：
script_dir=$(cd $(dirname $0) && pwd)
script_dir=$(dirname $(readlink -f $0 ))
```

>  先cd进 当前脚本的目录然后再pwd，或者直接读取当前脚本的所在路径。 





## **并行化 执行**

 shell中最简单的并行化是通过”&”以及”wait”命令来做 

```shell
#!/bin/bash

func(){
    #do sth
｝
for((i=0;i<10;i++))do
    func &
done
wait
```

这里并行的次数不能太多，否则机器会卡死。

如果图省事可以使用**parallel**命令来做，或者是用 **xargs** 来处理。



## -a到-z的意思

```shell
[ -a FILE ] 如果 FILE 存在则为真。
[ -b FILE ] 如果 FILE 存在且是一个块特殊文件则为真。
[ -c FILE ] 如果 FILE 存在且是一个字特殊文件则为真。
[ -d FILE ] 如果 FILE 存在且是一个目录则为真。
[ -e FILE ] 如果 FILE 存在则为真。
[ -f FILE ] 如果 FILE 存在且是一个普通文件则为真。
[ -g FILE ] 如果 FILE 存在且已经设置了SGID则为真。
[ -h FILE ] 如果 FILE 存在且是一个符号连接则为真。
[ -k FILE ] 如果 FILE 存在且已经设置了粘制位则为真。
[ -p FILE ] 如果 FILE 存在且是一个名字管道(F如果O)则为真。
[ -r FILE ] 如果 FILE 存在且是可读的则为真。
[ -s FILE ] 如果 FILE 存在且大小不为o则为真。
[ -t FD ] 如果文件描述符 FD 打开且指向一个终端则为真。
[ -u FILE ] 如果 FILE 存在且设置了SUID (set user ID)则为真。
[ -w FILE ] 如果 FILE 如果 FILE 存在且是可写的则为真。
[ -x FILE ] 如果 FILE 存在且是可执行的则为真。
[ -O FILE ] 如果 FILE 存在且属有效用户ID则为真。
[ -G FILE ] 如果 FILE 存在且属有效用户组则为真。
[ -L FILE ] 如果 FILE 存在且是一个符号连接则为真。
[ -N FILE ] 如果 FILE 存在 and has been mod如果ied since it was last read则为真。
[ -S FILE ] 如果 FILE 存在且是一个套接字则为真。
[ FILE1 -nt FILE2 ] 如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。
[ FILE1 -ot FILE2 ] 如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。
[ FILE1 -ef FILE2 ] 如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。
[ -o OPTIONNAME ] 如果 shell选项 “OPTIONNAME” 开启则为真。
[ -z STRING ] “STRING” 的长度为零则为真。
[ -n STRING ] or [ STRING ] “STRING” 的长度为非零 non-zero则为真。
[ STRING1 == STRING2 ] 如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。
[ STRING1 != STRING2 ] 如果字符串不相等则为真。
[ STRING1 < STRING2 ] 如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。
[ STRING1 > STRING2 ] 如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。

[ -z “echo 111s|sed 's/[0-9]//g'” ] && echo 1 || echo 0 #把字符串中的数字都替换掉
```





## 正则表达式



```shell
[[ $test =~ ^[0-9]+ ]] && echo 1 || echo 0
```

 ` ~` 其实是对后面的正则表达式表示匹配的意思，如果匹配就输出1， 不匹配就输出0 







## 比较符

​    ***\*-eq(equal)\****   : 测试两个整数是否相等；比如 $A -eq $B

​    ***\*-ne\*\*(\*\*inequality\*******\*)\**** : 测试两个整数是否不等；不等，为真；相等，为假；

​    ***\*-gt(greter than)\**** : 测试一个数是否大于另一个数；大于，为真；否则，为假；

​    ***\*-lt\*******\*(less than)\**** : 测试一个数是否小于另一个数；小于，为真；否则，为假；

​    ***\*-ge\*\*(greter equal)\*\**\***: 大于或等于

​     ***\*-le\*******\*(less equal)\**** ：小于或等于



## iptables

[iptables详解](http://www.zsythink.net/archives/1199)	



## 进程结构体

```c
struct task_struct {
    // 进程状态
    long              state;
    // 虚拟内存结构体
    struct mm_struct  *mm;
    // 进程号
    pid_t              pid;
    // 指向父进程的指针
    struct task_struct __rcu  *parent;
    // 子进程列表
    struct list_head        children;
    // 存放文件系统信息的指针
    struct fs_struct        *fs;
    // 一个数组，包含该进程打开的文件指针
    struct files_struct        *files;
};
```









