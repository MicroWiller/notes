[TOC]

# Nginx入门

## Nginx概念

### 什么是Nginx，为什么选择它

- 是一个高性能的 HTTP 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。
- Nginx 相较于 Apache 具有占有内存少，稳定性高等优势，并且依靠并发能力强，丰富的模块库以及友好灵活的配置而闻名。
- 支持热部署



### 反向代理

#### 正向代理

正向代理，意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理。

![img](https://bkimg.cdn.bcebos.com/pic/5ab5c9ea15ce36d376031ac030f33a87e950b13f?x-bce-process=image/watermark,image_d2F0ZXIvYmFpa2U5Mg==,g_7,xp_5,yp_5)

#### 反向代理

我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址

![img](https://bkimg.cdn.bcebos.com/pic/5243fbf2b2119313f8f3d19667380cd791238d67?x-bce-process=image/watermark,image_d2F0ZXIvYmFpa2U4MA==,g_7,xp_5,yp_5)



### 负载均衡

单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到各个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡。

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.bubuko.com%2Finfo%2F201912%2F20191228112503313957.png&refer=http%3A%2F%2Fimage.bubuko.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1611384604&t=7f5af7a5b780805b767c4a2639643787)



### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。



### 高可用

使用keepalive进行主从配置并定时验活，主Nginx服务器挂了就使用备用Nginx服务器。



## Nginx安装

[就这么安]: https://www.nginx.cn/install



## 运行和控制Nginx

使用nginx操作命令前提条件：进入到nginx所在目录

### nginx命令行参数

```sh
nginx -s stop       快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen     重新打开日志文件。
nginx -c filename   为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t </path/to/config> 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
nginx -v            显示 nginx 的版本。
nginx -V            显示 nginx 的版本，编译器版本和配置参数
```



### nginx控制信号

主进程可以处理以下的信号：

| TERM, INT | 快速关闭                                                 |
| --------- | -------------------------------------------------------- |
| QUIT      | 从容关闭                                                 |
| HUP       | 重载配置 用新的配置开始新的工作进程 从容关闭旧的工作进程 |
| USR1      | 重新打开日志文件                                         |
| USR2      | 平滑升级可执行程序。                                     |
| WINCH     | 从容关闭工作进程                                         |

工作进程：

| TERM, INT | 快速关闭         |
| --------- | ---------------- |
| QUIT      | 从容关闭         |
| USR1      | 重新打开日志文件 |

### nginx启动、停止、重启命令

#### 启动

```shell
sudo /usr/local/nginx/sbin/nginx
```

#### 停止

从容停止

```shell
kill -QUIT  nginx主进程号
```

快速停止

```shell
kill -TERM nginx主进程号 
```

强制停止

```shell
kill -9 nginx主进程号
```

#### 重启

1. 先关闭进程，修改配置，重启进程

   ```shell
   kill -QUIT  nginx主进程号
   sudo /usr/local/nginx/nginx
   ```

2. 重新加载配置文件，不重启进程


### 使用信号加载新的配置

```shell
nginx -t -c /etc/nginx/nginx.conf
2006/09/16 13:07:10 [info] 15686#0: the configuration file /etc/nginx/nginx.conf syntax is ok
2006/09/16 13:07:10 [info] 15686#0: the configuration file /etc/nginx/nginx.conf was tested successfully

ps aux | egrep '(PID|nginx)'
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      2213  0.0  0.0   6784  2036 ?        Ss   03:01   0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
kill -HUP 2213
```

当 nginx 接收到 `HUP` 信号，它会尝试先解析配置文件（如果指定配置文件，就使用指定的，否则使用默认的），成功的话，就应用新的配置文件。之后，nginx 运行新的工作进程并从容关闭旧的工作进程。通知工作进程关闭监听套接字但是继续为当前连接的客户提供服务。所有客户端的服务完成后，旧的工作进程被关闭。 如果新的配置文件应用失败，nginx 将继续使用旧的配置进行工作。



### 平滑升级到新的二进制代码

首先，使用新的可执行程序替换旧的（最好做好备份），然后，发送 USR2 (kill -USR2 pid)信号给主进程。主进程将重命名它的 **.pid** 文件为 **.oldbin** (比如：**/usr/local/nginx/logs/nginx.pid.oldbin**)，然后执行新的可执行程序，依次启动新的主进程和新的工作进程：



```shell
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

```shell
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

**旧的工作进程逐渐退出**

这时，因为旧的服务器还尚未关闭它监听的套接字，所以，通过下面的几步，你仍可以恢复旧的服务器：

- 发送 HUP 信号给旧的主进程 - 它将在不重载配置文件的情况下启动它的工作进程
- 发送 QUIT 信号给新的主进程，要求其从容关闭其工作进程
- 发送 TERM 信号给新的主进程，迫使其退出
- 如果因为某些原因新的工作进程不能退出，向其发送 KILL 信号

新的主进程退出后，旧的主进程会由移除 **.oldbin** 前缀，恢复为它的 **.pid** 文件，这样，一切就都恢复到升级之前了。

如果尝试升级成功，而你也希望保留新的服务器时，发送 QUIT 信号给旧的主进程使其退出而只留下新的服务器运行：

```shell
	 PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```



## Nginx配置文件

### 配置文件组成

#### 全局块

从配置文件到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令

#### events块

events块涉及的指令主要影响Nginx服务器与用户的网络连接

#### http块

Nginx服务器配置中最频繁的部分

http块也包括http全局块、server块



### 配置文件实例

#### 反向代理

##### 单服务器

由域名www.123.com 通过nginx访问tomcat主页

```properties
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass http://127.0.0.1:8080;     #就是这句话
            index  index.html index.htm;
        }
```



##### 多服务器

使用9001端口访问两台tomcat服务器

```properties

    server {
       listen       9001;
      # listen       somename:8080;
       server_name  localhost;

       location ~ /test1/ {
           proxy_pass http://127.0.0.1:8080;
        }

       location ~ /test2/ {
           proxy_pass http://127.0.0.1:8081;
        }
    }

```

http://172.24.240.246:9001/test1/a.html

http://172.24.240.246:9001/test2/a.html

**location匹配**

1. =：精确匹配
2. ~：uri包含正则表达式，区分大小写；
3. ~*：uri包含正则表达式，不区分大小写；
4. ^~：用于不含正则表达式的uri前，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配。
5. 不加任何规则时，相当于加了“~”和“^~”。/表示匹配所有uri

**匹配优先级**

= 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。
最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。

~~~properties
location  = / {
  # 只匹配"/".
  [ configuration A ] 
}
location  / {
  # 匹配任何请求，因为所有请求都是以"/"开始
  # 但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ] 
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ] 
}
location ~* .(gif|jpg|jpeg)$ {
  # 匹配以 gif, jpg, or jpeg结尾的请求. 
  # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
  [ configuration D ] 
}
~~~



[正则表达式详解]: https://www.jb51.net/article/143420.htm
[nginx location匹配规则]: https://www.nginx.cn/115.html



#### 负载均衡

由域名www.123.com发送请求http://www.123.com/mabh/a.html，nginx将负载分配给两台tomcat服务器

```properties
    #放在http块中
    upstream myserver{
       server 127.0.0.1:8080;
       server 127.0.0.1:8081;
    }  
```

```properties
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass http://myserver;    #就是这个
            index  index.html index.htm;
        }

```



##### nginx分配服务器策略

1. 轮询（默认）

   每个请求按时间顺序逐一分配到不同的后端服务器。

2. 权重weight

   默认为1，权重越高被分配的客户端越多。

   ```properties
   #放在http块中
   upstream myserver{
      server 127.0.0.1:8080 weight=5;
      server 127.0.0.1:8081 weight=10;
   }  
   ```

3. ip_hash

   每个请求按访问ip的hash结果分配，每位访客固定访问一个后端服务器。

   ```properties
   #放在http块中
   upstream myserver{
      ip_hash;
      server 127.0.0.1:8080;
      server 127.0.0.1:8081;
   }  
   ```

4. least_conn

   把请求转发给连接数较少的后端服务器

   ```properties
   #放在http块中
   upstream myserver{
      least_conn;
      server 127.0.0.1:8080;
      server 127.0.0.1:8081;
   }  
   ```

5. fair（第三方）

   按后端服务器的响应时间来分配请求，响应时间短的优先分配。

   ```properties
   #放在http块中
   upstream myserver{
      server 127.0.0.1:8080;
      server 127.0.0.1:8081;
      fair;
   }  
   ```



#### 动静分离

通过location指定不同的后缀名实现不同的请求转发

```shell
[root@localhost /]# ll data
总用量 0
drwxr-xr-x. 2 root root 26 12月 28 10:29 image
drwxr-xr-x. 2 root root 20 12月 28 10:29 www
```

```properties
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /www/ {
            root   /data/;
            index  index.html index.htm;
        }

        location /image/ {
            root  /data/;
            autoindex on;
        }

```



### 配置文件

[nginx中文文档]: https://www.nginx.cn/doc/
[nginx配置文件详解]: https://blog.csdn.net/tjcyjd/article/details/50695922
[epoll模型与nginx]: https://zhuanlan.zhihu.com/p/77887952
[proxy_set_header]: https://blog.csdn.net/bao19901210/article/details/52537279



```properties
#nginx服务的用户及用户组
user agentuser agentuser;

#指定处理请求的进程数，一般设置为CPU核数
worker_processes  1; 

#指定错误日志存放路径
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#指定pid进程文件存放路径
#pid        logs/nginx.pid;

events {
	#使用epoll I/O模型
	use epoll;
	
	#每一个worker进程能并发处理（发起）的最大连接数
    worker_connections  1024;
}


http {
    #文件扩展名与类型映射表
    include       mime.types;
    
    #默认文件类型
    default_type  application/octet-stream;

	#日志模板
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

	#使用log_format后，指定日志文件的存放路径
    #access_log  logs/access.log  main;

	#开启高效传输模式
    sendfile on;
   
    #keepalive超时时间
    keepalive_timeout  180;
    
	#开启gzip压缩输出
    gzip on;
       
	#设定通过nginx上传文件的大小
    client_max_body_size 8M;

	#客户端请求头部的缓冲区大小。
    client_header_buffer_size 16k;
    
    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 16k;
        
    #后端服务器连接的超时时间_发起握手等候响应超时时间
    proxy_connect_timeout 90; 
    
    #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
    proxy_read_timeout 180;
    
    #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
    proxy_send_timeout 180;
    
    #该指令设置缓冲区大小,从被代理的后端服务器取得的响应内容,会先读取放置到这里.小的响应header通常位于这部分响应内容里边。默认来说，proxy_buffers等于该大小，但是可以设置的更小。
    proxy_buffer_size 256k;
    
    #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小
    proxy_buffers 4 256k;
    
    #设置在写入proxy_temp_path时数据的大小，超过该大小，就不再缓冲到磁盘
    proxy_temp_file_write_size 256k;
    
    #设置内存缓存空间大小为200MB，7天没有被访问的内容自动清除，硬盘缓存空间大小为200GB。
    proxy_cache_path   /dragonball/dirmap/data/nginx_docker/shm/cache/one  levels=1:2  keys_zone=cache_one:2000m inactive=7d max_size=200g;
    proxy_temp_path    /dragonball/dirmap/data/nginx_docker/shm/temp;
    
    #定义实现负载均衡的服务器组
    upstream balance { 

      server 127.0.0.1:8080;  
      server 127.0.0.1:8090;

    }
    
    #server 虚拟主机配置
    server {
        listen       80;
        #服务名称
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #对 / 所有做负载均衡+反向代理
        location / {
            #将请求代理到 8080/8090端口的tomcat服务器上的ssm项目
            proxy_pass http://balance/ssm/;  

            #当后端Web服务器上配置多个虚拟主机时，用该Header来区分反向代理哪个主机名
            proxy_set_header Host $host;  
            proxy_set_header X-Real-IP $remote_addr; 
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; client, proxy1, proxy2 ...
			#以上三行，目的是将代理服务器收到的用户的信息传到真实服务器上
        }
        
        #静态文件，nginx自己处理，不去balance请求后端的服务
        location ~ .*\.(gif|jpg|jpeg|png|ico|js|css)$ {   

            #静态文件存放目录  
            root /usr/local/soft/nginx/images;

            #设置缓存时间：一天 
            expires      1d;
            
            #缓存名 需和proxy_cache_path的cache名对应
            proxy_cache cache_one;
        }

		#定义错误页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        
    }

	#支持https请求
    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      cert.pem;
        ssl_certificate_key  cert.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }

}

```



## Nginx原理简介

### master和worker

Nginx 采用的是多进程（单线程） & 多路IO复用模型。

![img](https://img-blog.csdn.net/20180804123742622?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NreTZldmVu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 多进程的工作模式

**master**  

​	首先nginx 创建一个master 进程，通过socket() 创建一个sock文件描述符用来监听（sockfd） 绑定端口(bind) 开启监听（listen）。 

**worker** 

​	然后 创建(fork)多个 worker子进程（复制master 进程的数据）， 此时所有的worker进程 继承了sockfd(socket文件描述符)， 当有连接进来之后 worker进程就可以accpet()创建已连接描述符， 然后通过已连接描述符与客户端通讯

**这样做带来的好处：**

- 节省锁带来的开销。每个 worker 进程都是独立的进程，不共享资源，不需要加锁。同时在编程以及问题排查时，也会方便很多。
- 独立进程，减少风险。采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快重新启动新的 worker 进程。当然，worker 进程的也能发生意外退出。



### 惊群现象及处理

**惊群现象**

​	由于所有子进程都继承了父进程的 sockfd，那么当连接进来时，所有子进程都将收到通知并“争着”与它建立连接，这就叫“惊群现象”。大量的进程被激活又挂起，只有一个进程可以accept() 到这个连接，这当然会消耗系统资源。

**处理**

- 原因：多个进程监听同一个端口引发的。
- 如果可以同一时刻只能有一个进程监听端口，这样就不会发生“惊群”了。因此Nginx 提供了accept_mutex锁 ，只有进程成功调用了ngx_trylock_accept_mutex方法获取锁后才可以监听端口，这样只会有一个进程被唤醒。



### worker工作流程

**worker进程做了什么**

1. accept() 与客户端建立连接 
2. recv()接收客户端发过来的数据 
3. send() 向客户端发送数据 
4. close() 关闭客户端连接

**会产生的问题**

首先 等待客户端有连接进来 

accpet()  与客户端建立连接后 recv()  一直等待客户的发送过来数据（此时处于io阻塞状态）

如果此时又有客户端过来建立连接，那么只能等待，需要一直等待close() 之后才可以建立连接 

也就是说这个worker进程会因为recv() 而处于阻塞状态，而不能处理与其他客户端建立连接， 这段时间不能做任何事，这是对性能了浪费。 



### IO多路复用

**nginx的IO多路复用是什么**

多个客户端请求处理 的io 操作都能在一个worker进程内并发交替顺序完成，这就叫io多路复用， 这里的复用指的是复用同一个worker进程



**为什么选择IO多路复用**

模拟一个worker处理30个客户socket -> 一个老师让30个学生解答一道题目

1. 按顺序检查，一个一个处理，如果一个学生出现问题，全班会耽误。(IO同步阻塞)
2. 创建30个老师分身，每个分身检查学生的答案。这种类似于为每一个socket创建一个线程。(多线程代价很高，需要CPU上下文切换，处理操作句柄)
3. 站在讲台上，谁解答完谁举手，老师监听举手的动作。

肯定是第三种更好~



**Nginx采用的epoll模型**

将用户socket对应的sockfd注册进epoll，epoll监听哪些socket上有消息到达，socket使用非阻塞状态。

这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是**事件驱动**，所谓的reactor模式。



**epoll demo**

~~~C
struct epoll_event events[5];
// epfd初始化
int epfd = epoll_create(10);
...
...
for (i=0;i<5;i++) {
    static struct epoll_event ev;
    addrlen = sizeof(client);
    ev.data.fd = accept(sockfd, (struct sockaddr*)&client, &addrlen);
    ev.events = EPOLLIN;
    // epfd配置 +fd-events
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev);
}

while(1){
    nfds = epoll_wait(epfd, events, 5, 10000);
    
    for(i=0;i<nfds;i++) {
        memset(buffer, 0, MAXBUF);
        read(events[i].data.fd, buffer, MAXBUF);
        puts(buffer);
    }
}
~~~



### Nginx中sendfile的作用

![wKiom1OC_0Oih4A7AADfiB_7GRE907.jpg](https://s3.51cto.com/wyfs02/M02/2A/42/wKiom1OC_0Oih4A7AADfiB_7GRE907.jpg)

![wKiom1ODB92wC3m3AADAG83M0Mg738.jpg](https://s3.51cto.com/wyfs02/M02/2A/49/wKiom1ODB92wC3m3AADAG83M0Mg738.jpg)



[参考博客]: https://www.cnblogs.com/yblackd/p/12194143.html

[参考博客]: https://www.cnblogs.com/xiaobaiskill/p/10969180.html
[IO多路复用技术是什么]: https://www.zhihu.com/question/28594409
[IO多路复用epoll介绍]: https://www.bilibili.com/video/BV1qJ411w7du?t=1442

