---
title: keepalived安装与配置
date: 2017-02-11 20:30:00
tags:
- keepalived
categories:
- Linux
---


[keepalived 源码包下载地址:http://www.keepalived.org/download.html](http://www.keepalived.org/download.html)

## 服务器内核要求

需要服务器内核支持：

- Kernel/User netlink socket
> netlink是基于socket的通信机制，由于socket本身的双工性、突发性、不阻塞性等特点，能够很好地满足内核空间与用户空间小量数据的及时交互，因此在Linux 2.6内核开始被广泛使用，例如内核态的netfilter与用户态的iptables的数据交换就是通过netlink机制完成的。

- LinuxVirtualServer
> 在大部分 linux 发行版中，ipvs 被默认安装。如果没有安装则需要安装。
> 检查当前加载的内核模块，看是否存在 ip_vs 模块。

```
[root@hd-4 ipvsadm-1.24]# lsmod|grep ip_vs
ip_vs 77569 0
```


<!-- more -->

### 安装 ipvs

> Ipvs 具体实现是由 ipvsadm 这个程序来完成，因此判断一个系统是否具备 ipvs 功能，也可以查看 ipvsadm 程序是否被安装。查看 ipvsadm 程序最简单的办法就是在任意路径执行命令 ipvsadm。

ipvs 版本的选择：
> 由于IPVS与用户空间的接口在不同的Linux内核版本中不同，因此在不同的Linux内核版本中有不同版本的 ipvsadm 用于IPVS。
> - 对于Linux内核2.6中的IPVS，需要使用ipvsadm 1.24版或更高版本。
> - 对于Linux内核2.4中的IPVS，需要使用ipvsadm 1.21系列。
> - 对于Linux内核2.2的IPVS补丁，需要使用ipvsadm版本1.15。
>
> 当前我使用的Linux内核是 `2.6.32-642.6.2.el6.i686` ,所以选择了 `ipvsadm-1.26`.

[ipvsadm 下载地址：http://www.linuxvirtualserver.org/software/ipvs.html](http://www.linuxvirtualserver.org/software/ipvs.html)

```
[root@localhost ~]# wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz
[root@localhost ~]# tar zxvf ipvsadm-1.26.tar.gz
[root@localhost ~]# cd ipvsadm-1.26
[root@localhost ipvsadm-1.26]# make
[root@localhost ipvsadm-1.26]# make install
```
装完之后，ipvsadm 产生的文件列表：
```
/sbin/ipvsadm
/sbin/ipvsadm-save
/sbin/ipvsadm-restore
/usr/man/man8/ipvsadm.8
/usr/man/man8/ipvsadm-save.8
/usr/man/man8/ipvsadm-restore.8
/etc/rc.d/init.d/ipvsadm
```
执行 ipvsadm
```
[root@localhost ipvsadm-1.26]# ipvsadm
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
```

查看当前加载的内核模块，看是否已经存在 ip_vs 模块。
```
[root@localhost ipvsadm-1.26]# lsmod | grep ip_vs
ip_vs                 103551  0
libcrc32c                841  1 ip_vs
ipv6                  271777  16 ip_vs,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6
```
至此，ipvs 就已经安装成功。


**安装 ipvs 时，可能会遇到的问题：**

- 1、`can't find file 'netlink/netlink.h'`
```
make -C libipvs
make[1]: Entering directory `/root/ipvsadm-1.26/libipvs'
gcc -Wall -Wunused -Wstrict-prototypes -g -fPIC -DLIBIPVS_USE_NL  -DHAVE_NET_IP_VS_H -c -o libipvs.o libipvs.c
在包含自 libipvs.h：13 的文件中，
                 从 libipvs.c：23:
ip_vs.h:15:29: 错误：netlink/netlink.h：没有那个文件或目录
ip_vs.h:16:31: 错误：netlink/genl/genl.h：没有那个文件或目录
ip_vs.h:17:31: 错误：netlink/genl/ctrl.h：没有那个文件或目录
In file included from libipvs.h:13,
                 from libipvs.c:23:
...
```
解决方法：安装 `libnl` 和 `popt` 相关的包
```
# yum install libnl* popt*
```

- 2、

```
# make
make -C libipvs
make[1]: Entering directory `/var/tmp/ipvsadm-1.24/libipvs'
gcc -Wall -Wunused -Wstrict-prototypes -g -O2 -I/usr/src/linux/include  -DHAVE_NET_IP_VS_H -c -o libipvs.o libipvs.c
In file included from libipvs.c:23:
libipvs.h:14:23: net/ip_vs.h: No such file or directory
In file included from libipvs.c:23:
libipvs.h:119: error: syntax error before "fwmark"
libipvs.h:119: warning: function declaration isn't a prototype
libipvs.c:27: error: field `svc' has incomplete type
libipvs.c:28: error: field `dest' has incomplete type
libipvs.c: In function `ipvs_init':
libipvs.c:40: error: invalid application of `sizeof' to incomplete type `ip_vs_getinfo'
libipvs.c:44: error: `IP_VS_SO_GET_INFO' undeclared (first use in this function)
libipvs.c:44: error: (Each undeclared identifier is reported only once
libipvs.c:44: error: for each function it appears in.)
libipvs.c: In function `ipvs_getinfo':
libipvs.c:56: error: invalid application of `sizeof' to incomplete type `ip_vs_getinfo'
libipvs.c:57: error: `IP_VS_SO_GET_INFO' undeclared (first use in this function)
libipvs.c: In function `ipvs_version':
libipvs.c:64: error: invalid use of undefined type `struct ip_vs_getinfo'
libipvs.c: In function `ipvs_flush':
libipvs.c:70: error: `IP_VS_SO_SET_FLUSH' undeclared (first use in this function)
libipvs.c: In function `ipvs_add_service':
libipvs.c:79: error: `IP_VS_SO_SET_ADD' undeclared (first use in this function)
libipvs.c:79: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_update_service':
libipvs.c:87: error: `IP_VS_SO_SET_EDIT' undeclared (first use in this function)
libipvs.c:87: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_del_service':
libipvs.c:95: error: `IP_VS_SO_SET_DEL' undeclared (first use in this function)
libipvs.c:95: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_zero_service':
libipvs.c:103: error: `IP_VS_SO_SET_ZERO' undeclared (first use in this function)
libipvs.c:103: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_add_dest':
libipvs.c:109: error: dereferencing pointer to incomplete type
libipvs.c:109: error: dereferencing pointer to incomplete type
libipvs.c:112: error: `IP_VS_SO_SET_ADDDEST' undeclared (first use in this function)
libipvs.c: In function `ipvs_update_dest':
libipvs.c:119: error: dereferencing pointer to incomplete type
libipvs.c:119: error: dereferencing pointer to incomplete type
libipvs.c:122: error: `IP_VS_SO_SET_EDITDEST' undeclared (first use in this function)
libipvs.c: In function `ipvs_del_dest':
libipvs.c:129: error: dereferencing pointer to incomplete type
libipvs.c:129: error: dereferencing pointer to incomplete type
libipvs.c:132: error: `IP_VS_SO_SET_DELDEST' undeclared (first use in this function)
libipvs.c: In function `ipvs_set_timeout':
libipvs.c:140: error: `IP_VS_SO_SET_TIMEOUT' undeclared (first use in this function)
libipvs.c:141: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_start_daemon':
libipvs.c:148: error: `IP_VS_SO_SET_STARTDAEMON' undeclared (first use in this function)
libipvs.c:149: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_stop_daemon':
libipvs.c:156: error: `IP_VS_SO_SET_STOPDAEMON' undeclared (first use in this function)
libipvs.c:157: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_get_services':
libipvs.c:166: error: dereferencing pointer to incomplete type
libipvs.c:167: error: invalid application of `sizeof' to incomplete type `libipvs.h'
libipvs.c:167: error: invalid use of undefined type `struct ip_vs_getinfo'
libipvs.c:172: error: dereferencing pointer to incomplete type
libipvs.c:172: error: invalid use of undefined type `struct ip_vs_getinfo'
libipvs.c:174: error: `IP_VS_SO_GET_SERVICES' undeclared (first use in this function)
libipvs.c: In function `ipvs_cmp_services':
libipvs.c:189: error: dereferencing pointer to incomplete type
libipvs.c:189: error: dereferencing pointer to incomplete type
libipvs.c:193: error: dereferencing pointer to incomplete type
libipvs.c:193: error: dereferencing pointer to incomplete type
libipvs.c:197: error: dereferencing pointer to incomplete type
libipvs.c:197: error: dereferencing pointer to incomplete type
libipvs.c:201: error: dereferencing pointer to incomplete type
libipvs.c:201: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_sort_services':
libipvs.c:208: error: dereferencing pointer to incomplete type
libipvs.c:208: error: dereferencing pointer to incomplete type
libipvs.c:209: error: invalid application of `sizeof' to incomplete type `libipvs.h'
libipvs.c: In function `ipvs_get_dests':
libipvs.c:218: error: dereferencing pointer to incomplete type
libipvs.c:218: error: invalid application of `sizeof' to incomplete type `libipvs.h'
libipvs.c:218: error: dereferencing pointer to incomplete type
libipvs.c:224: error: dereferencing pointer to incomplete type
libipvs.c:224: error: dereferencing pointer to incomplete type
libipvs.c:225: error: dereferencing pointer to incomplete type
libipvs.c:225: error: dereferencing pointer to incomplete type
libipvs.c:226: error: dereferencing pointer to incomplete type
libipvs.c:226: error: dereferencing pointer to incomplete type
libipvs.c:227: error: dereferencing pointer to incomplete type
libipvs.c:227: error: dereferencing pointer to incomplete type
libipvs.c:228: error: dereferencing pointer to incomplete type
libipvs.c:228: error: dereferencing pointer to incomplete type
libipvs.c:231: error: `IP_VS_SO_GET_DESTS' undeclared (first use in this function)
libipvs.c: In function `ipvs_cmp_dests':
libipvs.c:243: error: dereferencing pointer to incomplete type
libipvs.c:243: error: dereferencing pointer to incomplete type
libipvs.c:247: error: dereferencing pointer to incomplete type
libipvs.c:247: error: dereferencing pointer to incomplete type
libipvs.c: In function `ipvs_sort_dests':
libipvs.c:253: error: dereferencing pointer to incomplete type
libipvs.c:253: error: dereferencing pointer to incomplete type
libipvs.c:254: error: invalid application of `sizeof' to incomplete type `libipvs.h'
libipvs.c: At top level:
libipvs.c:259: error: syntax error before "fwmark"
libipvs.c:260: warning: function declaration isn't a prototype
libipvs.c: In function `ipvs_get_service':
libipvs.c:264: error: dereferencing pointer to incomplete type
libipvs.c:270: error: dereferencing pointer to incomplete type
libipvs.c:270: error: `fwmark' undeclared (first use in this function)
libipvs.c:271: error: dereferencing pointer to incomplete type
libipvs.c:271: error: `protocol' undeclared (first use in this function)
libipvs.c:272: error: dereferencing pointer to incomplete type
libipvs.c:272: error: `addr' undeclared (first use in this function)
libipvs.c:273: error: dereferencing pointer to incomplete type
libipvs.c:273: error: `port' undeclared (first use in this function)
libipvs.c:274: error: `IP_VS_SO_GET_SERVICE' undeclared (first use in this function)
libipvs.c: In function `ipvs_get_timeout':
libipvs.c:288: error: dereferencing pointer to incomplete type
libipvs.c:293: error: `IP_VS_SO_GET_TIMEOUT' undeclared (first use in this function)
libipvs.c: In function `ipvs_get_daemon':
libipvs.c:309: error: dereferencing pointer to incomplete type
libipvs.c:315: error: `IP_VS_SO_GET_DAEMON' undeclared (first use in this function)
libipvs.c: At top level:
libipvs.c:33: error: storage size of `ipvs_info' isn't known
make[1]: *** [libipvs.o] Error 1
make[1]: Leaving directory `/var/tmp/ipvsadm-1.24/libipvs'
make: *** [libs] Error 2
```

这个问题一般是编译器找不到 服务器 kernel 的源码路径。解决方法是，创建一个当前系统的 kernel 源码路径的 连接到 `/usr/src/linux`。如，
```
ln –sv /usr/src/kernels/2.6.32-642.6.2.el6.i686 /usr/src/linux
```
> 注意一定要与当前的运行的内核相一致，因为 `/usr/src/kernels` 目录下可多个目录。
>
> 若`/usr/src/kernel`目录下没有内核目录，则需要安装内核开发包：
> - 用命令`uname -a`查看内核版本，
> - 然后可以在这里查找对应的 `kernel` 开发包(https://www.kernel.org/pub/linux/kernel/v2.6/)
>
> 不推荐使用 `yum install -y kernel-devel` 安装

## 依赖：
- gcc、gcc-c++
- openssl-dev

报错：
```
configure: error:
  !!! OpenSSL is not properly installed on your system. !!!
  !!! Can not include OpenSSL headers files.            !!!
```
安装：
```
yum install openssl-devel -y
```

- libnfnetlink-devel

报错：
```
configure: error:
    !!! Please install libnfnetlink headers.              !!!
```
安装：
```
yum install -y libnfnetlink-devel
```

## keepalived 安装


```
[root@localhost ~]# wget http://www.keepalived.org/software/keepalived-1.3.2.tar.gz
[root@localhost ~]# tar zxvf keepalived-1.3.2.tar.gz
[root@localhost ~]# cd keepalived-1.3.2
[root@localhost keepalived-1.3.2]# ./configure --prefix=/usr/local/keepalived
...
Keepalived configuration
------------------------
Keepalived version       : 1.3.2
Compiler                 : gcc
Preprocessor flags       :  -I/usr/include/libnl3  
Compiler flags           : -Wall -Wunused -Wstrict-prototypes -Wextra -g -O2   
Linker flags             :
Extra Lib                : -ldl -lssl -lcrypto  -lnl-3 -lnl-genl-3 -lnl-route-3
Use IPVS Framework       : Yes
IPVS use libnl           : Yes
IPVS syncd attributes    : No
IPVS 64 bit stats        : No
fwmark socket support    : Yes
Use VRRP Framework       : Yes
Use VRRP VMAC            : Yes
Use VRRP authentication  : Yes
With ip rules/routes     : Yes
SNMP vrrp support        : No
SNMP checker support     : No
SNMP RFCv2 support       : No
SNMP RFCv3 support       : No
DBUS support             : No
SHA1 support             : No
Use Debug flags          : No
Stacktrace support       : No
Memory alloc check       : No
libnl version            : 3
Use IPv4 devconf         : No
Use libiptc              : No
Use libipset             : No
init type                : upstart
Build genhash            : Yes
Build documentation      : No
[root@localhost keepalived-1.3.2]# make && make install

[root@localhost keepalived-1.3.2]# cd /usr/local/keepalived/
[root@localhost keepalived]# tree .
.
├── bin
│   └── genhash
├── etc
│   ├── init
│   │   └── keepalived.conf
│   ├── keepalived
│   │   ├── keepalived.conf
│   │   └── samples
│   │       ├── client.pem
│   │       ├── dh1024.pem
│   │       ├── keepalived.conf.fwmark
│   │       ├── keepalived.conf.HTTP_GET.port
│   │       ├── keepalived.conf.inhibit
│   │       ├── keepalived.conf.IPv6
│   │       ├── keepalived.conf.misc_check
│   │       ├── keepalived.conf.misc_check_arg
│   │       ├── keepalived.conf.quorum
│   │       ├── keepalived.conf.sample
│   │       ├── keepalived.conf.SMTP_CHECK
│   │       ├── keepalived.conf.SSL_GET
│   │       ├── keepalived.conf.status_code
│   │       ├── keepalived.conf.track_interface
│   │       ├── keepalived.conf.virtualhost
│   │       ├── keepalived.conf.virtual_server_group
│   │       ├── keepalived.conf.vrrp
│   │       ├── keepalived.conf.vrrp.localcheck
│   │       ├── keepalived.conf.vrrp.lvs_syncd
│   │       ├── keepalived.conf.vrrp.routes
│   │       ├── keepalived.conf.vrrp.rules
│   │       ├── keepalived.conf.vrrp.scripts
│   │       ├── keepalived.conf.vrrp.static_ipaddress
│   │       ├── keepalived.conf.vrrp.sync
│   │       ├── root.pem
│   │       └── sample.misccheck.smbcheck.sh
│   └── sysconfig
│       └── keepalived
├── sbin
│   └── keepalived
└── share
    ├── man
    │   ├── man1
    │   │   └── genhash.1
    │   ├── man5
    │   │   └── keepalived.conf.5
    │   └── man8
    │       └── keepalived.8
    └── snmp
        └── mibs

14 directories, 34 files
```


## keepalived 配置与管理

简单的配置示例
```
# cat keepalived.conf
! Configuration File for keepalived
# 全局配置
global_defs {
   #指定keepalived在发生切换时需要发送email到的对象，一行一个
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc #指定发件人
   smtp_server 192.168.200.1  #指定smtp服务器地址
   smtp_connect_timeout 30  #指定smtp连接超时时间
   router_id LVS_DEVEL  #运行keepalived机器的一个标识
}

vrrp_instance VI_1 {
    state MASTER  #指定那个为master，那个为backup，如果设置了nopreempt这个值不起作用，主备考priority决定
    interface eth0  #设置实例绑定的网卡
    virtual_router_id 51  #VPID标记
    priority 100  #优先级，高优先级竞选为master
    advert_int 1  #检查间隔，默认1秒
    authentication {  #设置认证
        auth_type PASS   #认证方式
        auth_pass 1111   #认证密码
    }
    virtual_ipaddress {  #设置vip
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}

virtual_server 192.168.200.100 443 {
    delay_loop 6   #健康检查时间间隔
    lb_algo rr  #lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh
    lb_kind NAT  #负载均衡转发规则NAT|DR|RUN
    nat_mask 255.255.255.0
    persistence_timeout 50   #会话保持时间
    protocol TCP  #使用的协议

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url { #检查url，可以指定多个
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3 #连接超时时间
            nb_get_retry 3  #重连次数
            delay_before_retry 3   #重连间隔时间
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

### 管理
```
# keepalived -h
Usage: keepalived [OPTION...]
  -f, --use-file=FILE          使用指定的配置文件
  -P, --vrrp                   只能使用VRRP子系统运行
  -C, --check                  只能使用Health-checker子系统运行
  -l, --log-console            将消息记录到本地控制台
  -D, --log-detail             详细日志消息 /var/log/messages
  -S, --log-facility=[0-7]     将syslog设置设置为LOG_LOCAL [0-7]
  -V, --dont-release-vrrp      不要在守护程序停止时删除VRRP VIP和VROUTE
  -I, --dont-release-ipvs      不要在守护程序停止时删除IPVS拓扑
  -R, --dont-respawn           不要重新生成子进程
  -n, --dont-fork              D不要fork守护进程
  -d, --dump-conf              转储配置数据
  -p, --pid=FILE               对父进程使用指定的pidfile
  -r, --vrrp_pid=FILE          对VRRP子进程使用指定的pidfile
  -c, --checkers_pid=FILE      使用指定的pidfile进行checkers子进程
  -x, --snmp                   启用SNMP子系统
  -v, --version                显示版本号
  -h, --help                   显示此帮助消息
```
示例，启动 keepalived
```
# keepalived -D -f /xxx/keepalived.conf
```
查看进程列表，keepalived 是否已经启动
```
]# ps aux | grep keepalived
root      1371  0.0  0.1  17200  1092 ?        Ss   18:00   0:00 keepalived -D -f /etc/keepalived/keepalived.conf
root      1372  0.0  0.2  17256  2632 ?        S    18:00   0:00 keepalived -D -f /etc/keepalived/keepalived.conf
root      1373  0.0  0.1  17256  1840 ?        S    18:00   0:00 keepalived -D -f /etc/keepalived/keepalived.conf
root      1390  0.0  0.0   6052   776 pts/0    S+   18:02   0:00 grep keepalived
```

查看网络设置，虚拟IP是否已经设置
> 使用 ifconfig 是看不到 虚拟IP 配置的

```
# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:bf:25:88 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.103/24 brd 192.168.1.255 scope global eth0
    inet 192.168.200.16/32 scope global eth0
    inet 192.168.200.17/32 scope global eth0
    inet 192.168.200.18/32 scope global eth0
    inet6 fe80::a00:27ff:febf:2588/64 scope link
       valid_lft forever preferred_lft forever
```
