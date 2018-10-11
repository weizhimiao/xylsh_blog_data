---
title: keepalived实现Nginx双机高可用
date: 2017-02-11 22:30:00
tags:
- keepalived
categories:
- Linux
---


`Keepalived` 是一个基于`VRRP`协议来实现的`LVS`服务高可用方案，可以利用其来避免单点故障。一个`LVS`服务至少会有2台服务器运行`Keepalived`，一台为主服务器（`MASTER`），一台为备份服务器（`BACKUP`），但是对外表现为一个`虚拟IP`，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管`虚拟IP`，继续提供服务，从而保证服务的高可用性。


<!-- more -->

## `keepalived`的工作原理

`Keepalived`是 `VRRP` 的完美实现，因此在介绍 `keepalived` 之前，先介绍一下 `VRRP` 的原理。

### VRRP 简介
`VRRP（Virtual Router Redundancy Protocol）`虚拟路由冗余协议,在 `VRRP` 中有两组重要的概念：
- `VRRP` 路由器 和 虚拟路由器，
> - `VRRP`路由器 是指运行`VRRP`的路由器，是物理实体，
> - 虚拟路由器是指`VRRP`协议创建的，是逻辑概念。
>
> 一组`VRRP`路由器协同工作，共同构成一台虚拟路由器。

- 主控路由器 和 备份路由器。
> `VRRP` 中存在着一种选举机制，用以选出提供服务的路由即主控路由，其他的则成了备份路由。当主控路由失效后，备份路由中会重新选举出一个主控路由，来继续工作，来保障不间断服务。

在现实的网络环境中，两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况，通常的解决方法有以下两种：

- 在主机上使用动态路由协议(RIP、OSPF等)
- 在主机上配置静态路由

很明显，在主机上配置动态路由是非常不切实际的，因为管理、维护成本以及是否支持等诸多问题。配置静态路由就变得十分流行，但路由器(或者说默认网关`default gateway`)却经常成为单点故障。`VRRP`的目的就是为了解决静态路由单点故障问题，`VRRP`通过一竞选(`election`)协议来动态的将路由任务交给`LAN`中虚拟路由器中的某台`VRRP`路由器。


### VRRP 工作流程

#### (1).初始化：    
路由器启动时，如果路由器的优先级是255(最高优先级，路由器拥有路由器地址)，要发送`VRRP`通告信息，并发送广播ARP信息通告路由器IP地址对应的MAC地址为路由虚拟MAC，设置通告信息定时器准备定时发送`VRRP`通告信息，转为`MASTER`状态；否则进入`BACKUP`状态，设置定时器检查定时检查是否收到`MASTER`的通告信息。

#### (2).Master
- 设置定时通告定时器；
- 用`VRRP` `虚拟MAC`地址响应路由器`IP`地址的`ARP`请求；
- 转发目的`MAC`是`VRRP`虚拟`MAC`的数据包；
- 如果是虚拟路由器`IP`的拥有者，将接受目的地址是虚拟路由器`IP`的数据包，否则丢弃；
- 当收到`shutdown`的事件时删除定时通告定时器，发送优先权级为0的通告包，转初始化状态；
- 如果定时通告定时器超时时，发送`VRRP`通告信息；
- 收到`VRRP`通告信息时，如果优先权为0，发送`VRRP`通告信息；否则判断数据的优先级是否高于本机，或相等而且实际`IP`地址大于本地实际`IP`，设置定时通告定时器，复位主机超时定时器，转`BACKUP`状态；否则的话，丢弃该通告包；

#### (3).Backup
- 设置主机超时定时器；
- 不能响应针对虚拟路由器`IP`的`ARP`请求信息；
- 丢弃所有目的`MAC`地址是虚拟路由器`MAC`地址的数据包；
- 不接受目的是虚拟路由器`IP`的所有数据包；
- 当收到`shutdown`的事件时删除主机超时定时器，转初始化状态；
- 主机超时定时器超时的时候，发送`VRRP`通告信息，广播`ARP`地址信息，转`MASTER`状态；
- 收到`VRRP`通告信息时，如果优先权为`0`，表示进入`MASTER`选举；否则判断数据的优先级是否高于本机，如果高的话承认`MASTER`有效，复位主机超时定时器；否则的话，丢弃该通告包；

### ARP查询处理
ARP（Address Resolution Protocol），即地址解析协议。是根据IP地址获取物理地址的一个TCP/IP协议。主机发送信息时将包含目标IP地址的ARP请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址.

所以当内部主机通过`ARP`查询虚拟路由器`IP`地址对应的`MAC`地址时，`MASTER`路由器回复的`MAC`地址为虚拟的`VRRP`的`MAC`地址，而不是实际网卡的 `MAC`地址，这样在路由器切换时让内网机器觉察不到；

而在路由器重新启动时，也不能主动发送本机网卡的实际`MAC`地址。如果虚拟路由器开启的ARP代理 (`proxy_arp`)功能，代理的`ARP`回应也回应`VRRP`虚拟MAC地址；

## 示例

![示例架构示意图](http://n.sinaimg.cn/games/3ece443e/20170207/ShiLiJiaGouShiYiTu.png)


- `Master` 和 `BackUp` 两台服务器作为负载均衡器，`BackUp` 为 `Master` 的热备份。
- `Node_x` 作为我们的业务机


### 准备：

- `CentOS 6.8 X86_64`
- `keepalived`、`ipvsadm`,实现 `Master` 和 `BackUp` 双机高可用。
- `Nginx`,在 `Master` 和 `BackUp` 上作为 7层负载均衡器，在 `Node_x` 节点作为 web 服务器来使用。

### 安装

- 1、同步各服务器的时间：
```
ntpdate 202.120.2.101
```

- 2、网络设置
> - **Master**：192.168.1.201
> - **BackUp**：192.168.1.202
> - **Node_1**：192.168.1.210


- 3、安装 `Nginx`

分别在 `Master`、`BackUp`、`Node_1`节点上安装 `Nginx`。

先安装`nginx`的`yum`源
```
rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
```
查看：
```
yum info nginx
```
安装：
```
yum install nginx
```

- 4、安装 `keepalived`

分别在 `Master`、`BackUp` 上安装 `keepalived`

```
yum install -y keepalived
```

### 配置

- `Master`、`BackUp` 节点上 `Nginx` 配置(负载均衡)：
```
cat /etc/nginx/conf.d/default.conf
upstream app {
	server 192.168.1.210:80;
  # server 192.168.1.xxx:80;
}
server {
    listen       80;
    server_name  localhost;

    location / {
				proxy_pass http://app;
    }
}
```
`Master`、`BackUp` 节点上分别启动
```
service nginx start
```

- `Node_1` 节点上 `Nginx` 配置（web服务器）：
```
cat /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
启动 `Nginx`
```
service nginx start
```
- `Master` 节点 `keepalived` 配置
```
cat /etc/keepalived/keepalived.conf
global_defs {  
    router_id NodeA  
}  
vrrp_instance VI_1 {  
    state MASTER    #设置为主服务器  
    interface eth0  #监测网络接口  
    virtual_router_id 51  #主、备必须一样  
    priority 100   #(主、备机取不同的优先级，主机值较大，备份机值较小,值越大优先级越高)  
    advert_int 1   #VRRP Multicast广播周期秒数  
    authentication {  
    auth_type PASS  #VRRP认证方式，主备必须一致  
    auth_pass 1111   #(密码)  

    virtual_ipaddress {  
        192.168.1.200/24  #VRRP HA虚拟地址  
    }
}  
```
- `BackUp` 节点 `keepalived` 配置
```
cat /etc/keepalived/keepalived.conf
global_defs {  
    router_id NodeB  
}  
vrrp_instance VI_1 {  
    state BACKUP    #设置为主服务器  
    interface eth0  #监测网络接口  
    virtual_router_id 51  #主、备必须一样  
    priority 90   #(主、备机取不同的优先级，主机值较大，备份机值较小,值越大优先级越高)  
    advert_int 1   #VRRP Multicast广播周期秒数  
    authentication {  
    auth_type PASS  #VRRP认证方式，主备必须一致  
    auth_pass 1111   #(密码)  

    virtual_ipaddress {  
        192.168.1.200/24  #VRRP HA虚拟地址  
    }
}  
```
分别启动主节点和备用节点的 `keepalived`
```
service keepalived start
```

### 验证

- 1、分别查看 `keepalived` 、`Nginx` 是否启动

**`Nginx`:**
```
[root@localhost ~]# service nginx status
nginx (pid  1185) 正在运行...
```
或：
```
[root@localhost ~]# ps aux | grep nginx
root      1185  0.0  0.0   8596   740 ?        Ss   Feb06   0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx     1188  0.0  0.1   8600  1584 ?        S    Feb06   0:00 nginx: worker process                   
root      2115  0.0  0.0   5724   548 pts/0    S+   Feb06   0:00 tail -f /var/log/nginx/access.log
root      2635  0.0  0.0   6052   780 pts/2    S+   00:05   0:00 grep nginx
```

**`keepalived`:**
```
[root@localhost ~]# service keepalived status
keepalived (pid  1925) 正在运行...
```
或:
```
[root@localhost ~]# ps aux | grep keepalived
root      1925  0.0  0.1  17200  1092 ?        Ss   Feb06   0:00 /usr/sbin/keepalived -D
root      1927  0.0  0.2  17256  2584 ?        S    Feb06   0:00 /usr/sbin/keepalived -D
root      1928  0.0  0.1  17256  1896 ?        S    Feb06   0:01 /usr/sbin/keepalived -D
root      3553  0.0  0.0   6052   780 pts/2    S+   00:09   0:00 grep keepalived
```

> 为了确保稳定性，`keepalived` 守护程序分为3个不同的进程。父进程，负责监控两个子进程，两个子进程，一个负责`VRRP`框架，另一个负责健康检查。


开启 `Master` 上 `keepalived` ,`Master` 会广播ARP消息

```
[root@localhost ~]# tail -f /var/log/messages

Feb  7 04:50:06 localhost Keepalived[9864]: Starting Keepalived v1.2.13 (03/19,2015)
Feb  7 04:50:06 localhost Keepalived[9865]: Starting Healthcheck child process, pid=9867
Feb  7 04:50:06 localhost Keepalived[9865]: Starting VRRP child process, pid=9868
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Netlink reflector reports IP 192.168.1.201 added
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Netlink reflector reports IP fe80::a00:27ff:fe89:8c8 added
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Registering Kernel netlink reflector
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Registering Kernel netlink command channel
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Registering gratuitous ARP shared channel
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Netlink reflector reports IP 192.168.1.201 added
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Opening file '/etc/keepalived/keepalived.conf'.
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Configuration is using : 35102 Bytes
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Netlink reflector reports IP fe80::a00:27ff:fe89:8c8 added
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Registering Kernel netlink reflector
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Registering Kernel netlink command channel
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: Using LinkWatch kernel netlink reflector...
Feb  7 04:50:06 localhost Keepalived_vrrp[9868]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Opening file '/etc/keepalived/keepalived.conf'.
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Configuration is using : 5225 Bytes
Feb  7 04:50:06 localhost Keepalived_healthcheckers[9867]: Using LinkWatch kernel netlink reflector...
Feb  7 04:50:07 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Transition to MASTER STATE
Feb  7 04:50:08 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Entering MASTER STATE
Feb  7 04:50:08 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) setting protocol VIPs.
Feb  7 04:50:08 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.1.200
Feb  7 04:50:08 localhost avahi-daemon[1013]: Registering new address record for 192.168.1.200 on eth0.IPv4.
Feb  7 04:50:08 localhost Keepalived_healthcheckers[9867]: Netlink reflector reports IP 192.168.1.200 added
Feb  7 04:50:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.1.200
```

同样，开启 `BackUp` 上 `keepalived` 时的日志信息
```
[root@localhost ~]# tail -f /var/log/messages

Feb  7 04:40:45 localhost Keepalived[3771]: Starting Keepalived v1.2.13 (03/19,2015)
Feb  7 04:40:45 localhost Keepalived[3772]: Starting Healthcheck child process, pid=3774
Feb  7 04:40:45 localhost Keepalived[3772]: Starting VRRP child process, pid=3775
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Netlink reflector reports IP 192.168.1.202 added
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Netlink reflector reports IP fe80::a00:27ff:fec9:2d9 added
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Registering Kernel netlink reflector
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Registering Kernel netlink command channel
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Registering gratuitous ARP shared channel
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Opening file '/etc/keepalived/keepalived.conf'.
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Netlink reflector reports IP 192.168.1.202 added
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Configuration is using : 35100 Bytes
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: Using LinkWatch kernel netlink reflector...
Feb  7 04:40:45 localhost Keepalived_vrrp[3775]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Netlink reflector reports IP fe80::a00:27ff:fec9:2d9 added
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Registering Kernel netlink reflector
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Registering Kernel netlink command channel
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Opening file '/etc/keepalived/keepalived.conf'.
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Configuration is using : 5223 Bytes
Feb  7 04:40:45 localhost Keepalived_healthcheckers[3774]: Using LinkWatch kernel netlink reflector...
Feb  7 04:40:46 localhost Keepalived_vrrp[3775]: VRRP_Instance(VI_1) Transition to MASTER STATE
Feb  7 04:40:46 localhost Keepalived_vrrp[3775]: VRRP_Instance(VI_1) Received higher prio advert
Feb  7 04:40:46 localhost Keepalived_vrrp[3775]: VRRP_Instance(VI_1) Entering BACKUP STATE
```

> 在日志的最后三行，是 `keepalived` 选举 `Master状态` 的记录
> 在启动 BackUp 节点的 `keepalived` 时，Master 的日志中会新增如下内容：
```
Feb  7 04:50:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.1.200
Feb  7 04:51:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Received lower prio advert, forcing new election
Feb  7 04:51:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.1.200
Feb  7 04:51:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Received lower prio advert, forcing new election
Feb  7 04:51:13 localhost Keepalived_vrrp[9868]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.1.200
Feb  7 04:54:26 localhost dhclient[9723]: parse_option_buffer: malformed option dhcp.x-display-manager (code 49): option length exceeds option buffer length.
```

- 2、通过浏览器（`192.168.1.100`） 访问 `http://192.168.1.200`

我们通过查看 `Node_1` 节点的 访问日志
```
[root@localhost ~]# tail -f /var/log/nginx/access.log
192.168.1.201 - - [06/Feb/2017:22:51:27 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
```

发现 访问路径为 ： `浏览器（192.168.1.100）` --> `Master(192.168.1.201)` --> `Node_1(192.168.1.210)`

这时我们 关闭 `Master`(`192.168.1.201`) 的网卡，
```
service network stop
```

之后，再次通过 `浏览器（192.168.1.100）` 访问 `http://192.168.1.200`，再次查看 `Node_1` 节点的 访问日志
```
[root@localhost ~]# tail -f /var/log/nginx/access.log
192.168.1.201 - - [06/Feb/2017:22:51:27 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
192.168.1.202 - - [06/Feb/2017:23:02:49 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
```
我们发现 访问路径已经切换成： `浏览器（192.168.1.100）` --> `BackUp(192.168.1.202)` --> `Node_1`(`192.168.1.210`)


再次开启，`Master(192.168.1.201)` 的网卡，
```
service network restart
```
使用`浏览器（192.168.1.100）` 访问 `http://192.168.1.200`，`Node_1` 节点的 访问日志
```
[root@localhost ~]# tail -f /var/log/nginx/access.log
192.168.1.201 - - [06/Feb/2017:23:02:27 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
192.168.1.202 - - [06/Feb/2017:23:02:49 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
192.168.1.201 - - [06/Feb/2017:23:03:46 +0800] "GET / HTTP/1.1" 200 14 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/602.3.12 (KHTML, like Gecko) Version/10.0.2 Safari/602.3.12" "-"
```

访问路径有变回到， `浏览器（192.168.1.100）` --> `BackUp(192.168.1.202)` --> `Node_1(192.168.1.210)`

至此，使用 `keepalived` 实现一个双机高可用的基本架构就算搭建完成。
