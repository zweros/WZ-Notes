---
title: 20190924 高并发
date: 2019-09-24
---

## 大数据 ##

高并发 -> 日志* -> 分析行为 ->画像 ->推荐 -> 服务*

## TCP/IP 网络模型 ##

三次握手：确认客户端和服务端的收发正常（IO 正常），可以进行连接

- C -> S : 发送 sync 包给服务端，表示客户端想与服务端建立连接

- S -> C：服务端接收到客户端的 sync 包，并将 sync +ack  包，发送给客户端

  表示服务端接收到客户端的请求，并回复客户端可以与服务端建立连接

- C -> S: 客户端接收到服务端的 ack 包，将 ack 包发送给服务端，表示客户端马上与服务端建立连接，让服务端做好准备

四次离手：TCP 是稳定的连接，对于连接断开需要经过 Client 与 Server 双方都确认后，才可关闭

- C - >S: 发送 fin 包给服务端，表示客户端想与服务端断开连接
- S -> C: 发送 ack包给客户端，表示接收到服务端的断开请求
- S -> C: 发送 fin 包给客户端，表示服务端想与客户端断开连接
- 
- C -> S: 发送 ack 包给服务端，服务端接收到后，表示与客户端断开连接。

```shell
TCP/IP协议  OSI 7L参考模型
GET / www.baidu.com/
7:应用层www.baidu.com  IP:80  1212
http，smtp，ssh
4:传输层控制：【三次握手>>（传输数据）>>四次分手】
tcp，udp
SOCKET：IP:PORT-IP:PORT
netstat -natp
3:网络层： 192.168.9.11
ip，icmp
ROUTE：下一跳
route -n
2:链路层
以太网：Ethernet：MAC
ARP：全F，两点通信，交换机学习
arp -a
```



![](http://img.zwer.xyz/blog/20190924154314.png)

注意： 三次握手和四次离手中间不可拆分，一定针对于两台特定  IP:Port - IP:Port

```
netstat antp

route -n # -n 查看 IP 地址

arp -a  
0.0.0.0 默认网关
```

### 功能分层 ###

层与层依赖
1，能够申请到端口号
2，路由表有下一跳条目
3，ARP能请求到下一跳MAC
4，三次握手
5，传输数据
6，四次分手

![](http://img.zwer.xyz/blog/20190924202409.png)

### 下一跳 ###

```
整个互联网建立在下一跳的模式下
IP是逻辑上的两个端点
MAC是物理上连接的两个节点
端点间TCP传输过程中
确认机制
状态机制
不可分割
解析数据包需要成本
交换机：二层，只关心MAC地址
学习机制：
路由器：三层，只关心IP和路由表
LVS服务器：四层，只关心PORT，状态
nginx：七层，关心socket对应关系

## 负载均衡 ##
```

## LVS  ##

> LVS技术要达到的目标是：通过LVS提供的负载均衡技术和Linux操作系统实现一个高性能、高可用的服务器群集，它具有良好可靠性、可扩展性和可操作性。从而以低廉的成本实现最优的服务性能。
>
> LVS自从1998年开始，发展到现在已经是一个比较成熟的技术项目了。可以利用LVS技术实现高可伸缩的、高可用的网络服务，例如WWW服务、Cache服务、DNS服务、FTP服务、MAIL服务、视频/音频点播服务等等，有许多比较著名网站和组织都在使用LVS架设的集群系统，例如：Linux的门户网站（[www.linux.com](http://www.linux.com/)）、向RealPlayer提供音频视频服务而闻名的Real公司（[www.real.com](http://www.real.com/)）、全球最大的开源网站（sourceforge.net）等

**LVS 参考** https://superproxy.github.io/docs/lvs/index.html

### 硬件 ###

```
昂贵，性能优越
F5 BIG-IP 
Citrix NetScaler
A10
```

### 软件 ###

> 便宜，灵活度（开源）

|      | 四层                 | 七层                                           |
| ---- | -------------------- | ---------------------------------------------- |
|      | tcp 之上的第四层协议 | LVS 只能操作IP,端口 ，在操作系统内核中。       |
|      |                      | nginx<br/>haproxy<br/>httpd  “apache”webserver |

### LVS -DR ###

> Director 接收用户的请求，然后根据负载均衡算法选取一台 realserver，将包转发过去，最后由realserver直接回复给用户。

```
DR：Director
客户端发送对VIP的请求
lvs负载到后端某一台server
后端server处理后，直接封包回送客户端
源IP地址一定是lvs上面陪的那个公网服务地址
也就后端server要配置这个ip
后端server收到的数据包是lvs没有变动过的（IP：vip）
目标ip一定是自己持有的
so：多个server，接入互联网的server持有相同的IP，是不对的
必须将后端server中的vip隐藏起来（对外隐藏）

VIP: 虚拟服务器地址
DIP: 转发的网络地址
1，和RIP通信：ARP协议，获取Real Server的RIP：MAC地址
2，转发Client的数据包到RIP上（隐藏的VIP）
RIP: 后端真实主机(后端服务器)

CIP: 客户端IP地址
```

![](http://img.zwer.xyz/blog/20190927160750.png)





### LVS 配置实验 ###

#### 环境 ####

| 主机名  | node01          | node02          | node03          |
| ------- | --------------- | --------------- | --------------- |
| IP 类型 | DIP             | RIP             | RIP             |
| IP      | 192.168.170.101 | 192.168.170.102 | 192.168.170.103 |

#### 实验图 ####

![](http://img.zwer.xyz/blog/20190928094014.png)



#### 步骤 ####

1. 配置 LVS 服务器的虚拟IP 和开启 IPv4 转发

   ```
   [root@node01 ~]# ifconfig eth0:0 192.168.170.100/24  # 临时生效
   echo “1” > /proc/sys/net/ipv4/ip_forward 
   ```

2. 修改 RP 服务器的文件

   ```shell
   echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
   echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce 
   echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
   echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
   ```

3. 配置 RP 服务器的本地环回地址

   ```shell
   ifconfig lo:0 192.168.170.100 netmask 255.255.255.255
   ```

4. 在 RP 服务器上安装 httpd 服务

   ```
   yum install -y httpd
   ```

5. 设置 主页

   ```
   cd /var/www/html
   vi index.html
   内容： from 192.168.170.102
   ```

6. 启动 httpd 服务

   ```shell
   service httpd start
   ```

7. LVS 配置

   ```shell
    yum install ipvsadm -y
   ipvsadm -A -t 192.168.170.100:80 -s rr 
   ipvsadm -a -t 192.168.170.100:80 -r 192.168.170.102 -g
   ipvsadm -a -t 192.168.170.100:80 -r 192.168.170.103 -g
   
   ipvsadm -ln
   浏览器刷新: 访问vip
   ipvsadm –lnc
   netstat -natp
   ```




## keepalived ##

> 集群管理中保证集群高可用的服务软件
>
> 用于做健康检查和主备切换，这里用在 LVS 服务器，使用 LVS 服务器达到高可用 HA(High Availability)
>
> 高可用 High Available

```
1、需要心跳机制探测后端RS是否提供服务。
	a) 探测down，需要从lvs中删除该RS
	b) 探测发送从down到up，需要从lvs中再次添加RS。
2、Lvs DR，需要主备（HA）
```

### 实验 ###

#### 环境 ####

| 主机名 | node01          | node02          | node03          | node04          |
| ------ | --------------- | --------------- | --------------- | --------------- |
| 类型   | LVS(MASTER)     | RIP1            | RIP2            | LVS(BACKUP)     |
| IP     | 192.168.170.101 | 192.168.170.102 | 192.168.170.103 | 192.168.170.104 |

#### 实验图 ####

![](http://img.zwer.xyz/blog/20190928100941.png)

#### 步骤 ####

```shell
#  node01 上LVS 服务器上关闭之前的 ipvsadm 设置
ipvsadm -C  #              Clear the virtual server table.
ifconfig eth0:0 down  

# 在 node01、node04上安装 keepalived 
yum install -y keepalived 

# 安装完成后， 进入 cd /etc/keepalived 目录下,备份 keepalived.conf 文件
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER   # 主节点
    interface eth0  # 操作的网卡
    virtual_router_id 51
    priority 100  # 优先级
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {  # VIP  虚拟IP配置
	192.168.170.100/24 dev eth0 label eth0:3 
    }
}

virtual_server 192.168.170.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR   # 指定 LVS DR模式
    nat_mask 255.255.255.0
    persistence_timeout 0 # 同一个IP地址在50秒内lvs转发给同一个后端服务器
    protocol TCP

    real_server 192.168.170.102 80 { #设置真实服务器的心跳机制 RID PORT
        weight 1
        HTTP_GET {  #心跳检测的方式
            url {
              path /   # 心跳检查的地址
	    	  status_code 200      #心跳检查返回的状态
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.170.103 80 {
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
 }
 
# 配置完成 keepalived.conf ，将文件拷贝到 node04 上，并修改
scp ./keepalived.conf root@192.168.170.104:/`pwd`

# keepalived.conf 修改 node04 部分内容
vrrp_instance VI_1 {
    state BACKUP   # 从节点
    interface eth0  # 操作的网卡
    virtual_router_id 51
    priority 50  # 优先级，从节点的优先级应低于主节点的优先级
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {  # VIP  虚拟IP配置
	192.168.170.100/24 dev eth0 label eth0:3 
    }
}

# -------------------------------------------------------------------
# 在 node01、node04 上启动 keepalived 服务
service keepalived start

# 在 ifconfig 命令查看
ifocnifg

# 查看路由
ipvsadm -ln
```

## Nginx  ##

> Nginx ("engine x") 是一个高性能的 HTTP 和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

### Apache 与 Nginx 的比较 ###

|      | Apache                         | Nginx        |
| ---- | ------------------------------ | ------------ |
|      | 一个客户端连接开辟一个进程处理 | 仅有一个进程 |
|      | 同步阻塞                       | 异步非阻塞   |

### nginx.conf ###

#### events ####

```
#工作模式与连接数上限
events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
    epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
    use epoll;
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections 65535;
}

event下的一些配置及其意义
#单个后台worker process进程的最大并发链接数    
worker_connections  1024;
# 并发总数是 worker_processes 和 worker_connections 的乘积
# 即 max_clients = worker_processes * worker_connections
# 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么
# 为什么上面反向代理要除以4，应该说是一个经验值
# 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000
# worker_connections 值的设置跟物理内存大小有关
# 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数
    
event下的一些配置及其意义
# 而系统可以打开的最大文件数和内存大小成正比，一般1GB内存的机器上可以打开的文件数大约是10万左右
# 我们来看看360M内存的VPS可以打开的文件句柄数是多少：
# $ cat /proc/sys/fs/file-max
# 输出 34336
# 32000 < 34336，即并发连接总数小于系统可以打开的文件句柄总数，这样就在操作系统可以承受的范围之内
# 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置
# 使得并发总数小于操作系统可以打开的最大文件数目
# 其实质也就是根据主机的物理CPU和内存进行配置
# 当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。
# ulimit -SHn 65535

#定义Nginx运行的用户和用户组
user www www;

#nginx进程数，建议设置为等于CPU总核心数。
worker_processes 8;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;

#进程文件
pid /var/run/nginx.pid;

#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀，所以建议与ulimit -n的值保持一致。
worker_rlimit_nofile 65535;

```

#### httpd ####

```
#设定http服务器
http
{
include mime.types; #文件扩展名与文件类型映射表
default_type application/octet-stream; #默认文件类型
#charset utf-8; #默认编码
server_names_hash_bucket_size 128; #服务器名字的hash表大小
client_header_buffer_size 32k; #上传文件大小限制
large_client_header_buffers 4 64k; #设定请求缓
client_max_body_size 8m; #设定请求缓
sendfile on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。
tcp_nopush on; #防止网络阻塞
tcp_nodelay on; #防止网络阻塞
keepalive_timeout 120; #长连接超时时间，单位是秒
```

#### gzip ####

```
gzip的一些配置及其意义
#gzip模块设置gzip on;
#开启gzip压缩输出 gzip_min_length 1k; 
#最小压缩文件大小 gzip_buffers 4 16k;
#压缩缓冲区 gzip_http_version 1.0; 
#压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
gzip_comp_level 2; 
#压缩等级
gzip_types text/plain application/x-javascript text/css application/xml;
#压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，
但是会有一个warn。
gzip_vary on;
#limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用
```

#### 虚拟主机 ####

```
虚拟主机一些配置及其意义
#虚拟主机的配置
server{
    #监听端口 listen 80;
    #域名可以有多个，用空格隔开
    server_name www.ha97.com ha97.com;
    index index.html index.htm index.jsp;
    root /data/www/ha97;
    location ~ .*\.(php|php5)?${
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.jsp;
        include fastcgi.conf;
}
通过nginx可以实现虚拟主机的配置，nginx支持三种类型的虚拟主机配置，
1、基于ip的虚拟主机， （一块主机绑定多个ip地址）
2、基于域名的虚拟主机（servername）
3、基于端口的虚拟主机（listen如果不写ip端口模式）
示例基于虚拟机ip的配置，这里需要配置多个ip
server
{
    listen 192.168.20.20:80;
    server_name www.linuxidc.com;
    root /data/www;
}
server
{
    listen 192.168.20.21:80;
    server_name www.linuxidc.com;
    root /data/www;
}

```

#### location ####

```shell
location 映射（ngx_http_core_module）
location [ = | ~ | ~* | ^~ ] uri { ... }
location URI {}:
对当前路径及子路径下的所有对象都生效；
location = URI {}: 注意URL最好为具体路径。
精确匹配指定的路径，不包括子路径，因此，只对当前资源生效；
location ~ URI {}:
location ~* URI {}:
模式匹配URI，此处的URI可使用正则表达式，~区分字符大小写，~*不区分字符大小写；
location ^~ URI {}:
不使用正则表达式
优先级：= > ^~ > ~|~* >  /|/dir/

/loghaha.html
/logheihei.html
^/log.*html$

## location 配置规则
=前缀的指令严格匹配这个查询。如果找到，停止搜索。
所有剩下的常规字符串，最长的匹配。如果这个匹配使用^〜前缀，搜索停止。
正则表达式，在配置文件中定义的顺序。
如果第3条规则产生匹配的话，结果被使用。否则，如同从第2条规则被使用

## location配置规则
location 的执行逻辑跟 location 的编辑顺序无关。
矫正：这句话不全对，“普通 location ”的匹配规则是“最大前缀”，
因此“普通 location ”的确与 location 编辑顺序无关；

但是“正则 location ”的匹配规则是“顺序匹配，且只要匹配到第一个就停止后面的匹配”；
“普通location ”与“正则 location ”之间的匹配顺序是？
先匹配普通 location ，再“考虑”匹配正则 location 。
注意这里的“考虑”是“可能”的意思，也就是说匹配完“普通 location ”后，有的时候需要继续匹配“正则 location ”，有的时候则不需要继续匹配“正则 location ”。两种情况下，不需要继续匹配正则 location ：
（ 1 ）当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普通 location 一旦匹配上，则不需要继续正则匹配；
（ 2 ）当普通location 恰好严格匹配上，不是最大前缀匹配，则不再继续匹配正则

loghaha.html
l:  logha
l:  ^~ loghah
l:  loghaha.html
l:  =loghaha.html
l:   ^logh.*html$
l:   ^logha.*html$

# 总结
nginx  收到请求头：判定ip，port，hosts决定server
nginx location匹配：用客户端的uri匹配location的uri
先普通
顺序无关
最大前缀
匹配规则简单
打断：
^~
完全匹配
再正则
不完全匹配
正则特殊性：一条URI可以和多条location匹配上
有顺序的
先匹配，先应用，即时退出匹配

```

#### 请求头 ####

```
请求头
host：决策server负责处理
uri：决策location
反向代理：proxy_pass  ip:port[uri];
```

#### IP 访问控制 ####

```shell
IP访问控制
location  {
	   deny  IP /IP段
	   deny  192.168.1.109;
	   allow 192.168.1.0/24;192.168.0.0/16;192.0.0.0/8
	}
规则：按照顺序依次检测，直到匹配到第一条规则
```

### Nginx 的 session 一致性问题 ###

​		http协议是无状态的，即你连续访问某个网页100次和访问1次对服务器来说是没有区别对待的，因为它记不住你。那么，在一些场合，确实需要服务器记住当前用户怎么办？比如用户登录邮箱后，接下来要收邮件、写邮件，总不能每次操作都让用户输入用户名和密码吧，为了解决这个问题，session的方案就被提了出来，事实上它并不是什么新技术，而且也不能脱离http协议以及任何现有的web技术

​		session的常见实现形式是会话cookie（session cookie），即未设置过期时间的cookie，这个cookie的默认生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。实现机制是当用户发起一个请求的时候，服务器会检查该请求中是否包含sessionid，如果未包含，则系统会创造一个名为JSESSIONID的输出 cookie返回给浏览器(只放入内存，并不存在硬盘中)，并将其以HashTable的形式写到服务器的内存里面；当已经包含sessionid是，服务端会检查找到与该session相匹配的信息，如果存在则直接使用该sessionid，若不存在则重新生成新的 session。这里需要注意的是session始终是有服务端创建的，并非浏览器自己生成的。　但是浏览器的cookie被禁止后session就需要用get方法的URL重写的机制或使用POST方法提交隐藏表单的形式来实现

#### Session共享 ####

​		首先我们应该明白，为什么要实现共享，如果你的网站是存放在一个机器上，那么是不存在这个问题的，因为会话数据就在这台机器，但是如果你使用了负载均衡把请求分发到不同的机器呢？这个时候会话id在客户端是没有问题的，但是如果用户的两次请求到了两台不同的机器，而它的session数据可能存在其中一台机器，这个时候就会出现取不到session数据的情况，于是session的共享就成了一个问题

#### Session一致性解决方案 ####

​	1、session复制
​		tomcat 本身带有复制session的功能。（不讲）
​	2、共享session
​		需要专门管理session的软件，
​		memcached 缓存服务，可以和tomcat整合，帮助tomcat共享管理session。

- **安装 memecache 缓存服务**

```shell
安装memcached
1、安装libevent
2、安装memcached
3、启动memcached
	memcached -d -m 128m -p 11211 -l 192.168.170.101 -u root -P /tmp/
	-d:后台启动服务
	-m:缓存大小
	-p：端口
	-l:IP
	-P:服务器启动后的系统进程ID，存储的文件
	-u:服务器启动是以哪个用户名作为管理用户
如果源配置了也可以用
	yum –y install memcached
```

- **tomcat 配置 session共享**

```xml
配置session共享如下：
3、拷贝jar到tomcat的lib下，jar包见附件
4、配置tomcat，每个tomcat里面的context.xml中加入
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager" 
	memcachedNodes="n1:192.168.170.101:11211" 
    sticky="false" 
    lockingMode="auto"
    sessionBackupAsync="false"
	requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
    sessionBackupTimeout="1000" transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory" 
/>
```

**注意： tomcat 集群一定要保持时间一致性**

![](http://img.zwer.xyz/blog/20190928155702.png)

