---
title: Apache 并行处理模块小结
date: 2016-08-22 22:36:00
tags:
- Apache Apache性能优化
categories:
- Apache
---
Apache 2.X  支持插入式并行处理模块，称为多路处理模块 MPM，（Multi-Processing Modules）它是一个用于选择和处理网络端口的绑定，接收请求并指派子进程处理来自客户端的请求的一个Apache模块。和其他Apache模块相比，它最大的区别是，在任何时间， 必须有一个，而且只有一个 MPM 加载到服务器中。
> 原理就是增加服务器服务进程或线程数量，是服务器可同时处理更多用户请求。

<!-- more -->

## 如何为Apache选择并安装一个合适的MPM模块

### 安装
MPM 必须在编译前夕，配置时指定，然后编译到服务器程序中。

即在执行 configure 时，使用参数 --with-mpm=NAME 来指定一个希望安装的MPM模块。NAME 是指定的 MPM 名称。

例：
```
./configure --prefix=/usr/local/apache2 --with-mpm=worker
make & make install
```
或者也可以编译为支持指定的几种或者全部MPM，之后通过修改配置来更换具体使用哪种MPM。
```
--enable-mpms-shared='prefork worker'
--enable-mpms-shared=all
```

例：
```
./configure --prefix=/usr/local/apache2 --enable-mpms-shared=all
···
```
安装成功之后，会在modules文件夹下，自动编译出我们指定的MPM的so模块，在httpd.conf中修改Apache的多处理模式MPM可以切换不同MPM模块：
```
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule mpm_worker_module modules/mod_mpm_worker.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
```
### 如何查看当前使用的是哪种MPM模块

1. 使用 ./httpd -l 来确定选择的 MPM。 此命令会列出编译到服务器程序中的所有模块，包括 MPM。例：
```
$ httpd -l
Compiled in modules:
  core.c
  mod_so.c
  http_core.c
  prefork.c
```
  如上显示，我们呢当前安装的是以prefork方式工作的MPM模块。

2. 使用./httpd -V 来确定当前使用的MPM模块。
```
$ httpd -V
Server version: Apache/2.4.18 (Unix)
Server built:   Feb 20 2016 20:03:19
Server's Module Magic Number: 20120211:52
Server loaded:  APR 1.4.8, APR-UTIL 1.5.2
Compiled using: APR 1.4.8, APR-UTIL 1.5.2
Architecture:   64-bit
Server MPM:     prefork
  threaded:     no
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_FLOCK_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
 -D HTTPD_ROOT="/usr"
 -D SUEXEC_BIN="/usr/bin/suexec"
 -D DEFAULT_PIDLOG="/private/var/run/httpd.pid"
 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
 -D DEFAULT_ERRORLOG="logs/error_log"
 -D AP_TYPES_CONFIG_FILE="/private/etc/apache2/mime.types"
 -D SERVER_CONFIG_FILE="/private/etc/apache2/httpd.conf"
```


## 常见的几种MPM模块以及它们之间的区别
如果我们在编译的时候没有明确选择使用哪种MPM模块，那么Apache将会根据不同的系统选择不同的MPM模块进行编译安装。

系统  | 默认MPM
-----|-------
BeOS |beos
Netware|mpm_netware
OS/2	|mpmt_os2
Unix	|prefork
Windows	|mpm_winnt

对于类UNIX系统，根据不同的场景需要我们可以选择使用不同的MPM模块。
* prefork
* worker
* event

### prefork MPM
> 非线程型的、预派生的MPM

**原理：** 启动之初，就预先fork一些子进程，然后等待请求进来。
```
# prefork MPM
# StartServers: number of server processes to start
# MinSpareServers: minimum number of server processes which are kept spare
# MaxSpareServers: maximum number of server processes which are kept spare
# MaxRequestWorkers: maximum number of server processes allowed to start
# MaxConnectionsPerChild: maximum number of connections a server process serves
#                         before terminating
<IfModule mpm_prefork_module>
    StartServers             1  #推荐 小=默认，中=20~50，大=50~100
    MinSpareServers          1  #推荐 与 StartServers 保持一致
    MaxSpareServers         10  #推荐 小=20，中=30~80，大=80~120
    MaxRequestWorkers      250  #推荐 小=500，中=500~1500，大=1500~3000
    MaxConnectionsPerChild   0  #推荐 小=10000，中或大10000~50000
    #ServerLimit           250  #推荐 与 MaxRequestWorkers 保持一致，当MaxRequestWorkers 值超过256，则需要增加该值配置
</IfModule>
```
启动时建立`StartServers`个子进程，然后按每秒创建指数级个进程数，直到达到`MinSpareServers`个进程（最多增到每秒32个）。如果空闲进程大于`MaxSpareServers`，则检查kill掉一些空闲进程。

`MaxRequestWorkers`指定Apache最多可以同时同时处理的请求数，即进程数，当请求数多余这个值之后，多余的请求就会进入请求队列等待处理（默认不能大于256）。但可以通过设定ServerLimit来增大限制数，serverlimit最大为20000。apache2.3.1之前的版本该参数叫 MaxClients 。当我们的服务器资源很多，但访问却很慢时，我们就可以试一下增大该值，来提高服务器的请求处理能力。
`MaxConnectionsPerChild`每个子进程可处理的请求数。处理完之后子进程就会自动销毁。`0`表示无限，永不销毁。

* 优点：成熟稳定，兼容所有新老模块。同时，不需要担心线程安全的问题。
* 缺点：相对于线程，进程相对占用更多的系统资源，消耗更多的内存。所以不擅长处理高并发请求。


### worker MPM
> 支持混合的多线程、多进程的MPM。相比于prefork，worker采用了多进程和多线程混合模式，所以在使用中它占据更少的内存，在高并发情况下表现更优秀。

```
# worker MPM
# StartServers: initial number of server processes to start
# MinSpareThreads: minimum number of worker threads which are kept spare
# MaxSpareThreads: maximum number of worker threads which are kept spare
# ThreadsPerChild: constant umber of worker threads in each server process
# MaxRequestWorkers: maximum number of worker threads
# MaxConnectionsPerChild: maximum number of connections a server process serves
#                         before terminating
<IfModule mpm_worker_module>
    StartServers             3  #推荐 小=默认，中=3~5，大=5~10
    MinSpareThreads         75  #推荐 小=默认，中=50~100，大=100~200
    MaxSpareThreads        250  #推荐 小=默认，中=80~160，大=200~400
    ThreadsPerChild         25  #推荐 小=默认，中=50~100，大=100~200
    MaxRequestWorkers      400  #推荐 小=500，中=500~1500，大=1500~3000
    MaxConnectionsPerChild   0  #推荐 小=10000，中或大=10000~50000
    #ServerLimit           250  #推荐 当 MaxRequestWorkers/ThreadsPerChild 大于16时，则需要增加该值配置。并且该值必须大于等于MaxRequestWorkers/ThreadsPerChild 的值
</IfModule>

```
`ThreadsPerChild` 每个进程包含线程数
`MaxSpareThreads` 定义最大空闲线程数，超过则清理

* 优点：占用更少系统资源，高并发情况下表现更优秀。
* 缺点：必须考虑线程安全的问题。

### event MPM
> worker方式的升级版，也采用多进程和多线程混合模式，并且解决了在 `keep-alive` 情况下，长期被占用的线程的资源浪费问题。

```
# event MPM
# StartServers: initial number of server processes to start
# MinSpareThreads: minimum number of worker threads which are kept spare
# MaxSpareThreads: maximum number of worker threads which are kept spare
# ThreadsPerChild: constant number of worker threads in each server process
# MaxRequestWorkers: maximum number of worker threads
# MaxConnectionsPerChild: maximum number of connections a server process serves
#                         before terminating
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>

```


* 优点：更好的高并发请求处理能力。
* 缺点：兼容性问题可能不是很好（最新的官方自带的模块，已经全部支持event MPM了），需要Linux系统（Linux 2.6+）对EPoll的支持，才能启用


**Tips：**
* ***空闲子进程：*** 即没有正在处理请求的子进程。
* ***请求等待队列：*** 任何超过 MaxClients 或 MaxRequestWorkers 限制的请求都将进入到等待队列，直到收到 ListenBacklog 指令限制的最大值为止（默认 ListenBacklog 511）
* ***ServerLimit：*** 该值表示Apache允许创建的最大进程数。值得注意的是 Apache在编译的时候会有一个硬限制 ServerLimit 20000 ，你不能超过该值。该值如果设置过高，将会有过多的内存被分配，可能会导致Apache 无法启动或者不稳定的情况。

## 简单测试对比

对上面三种模式，我们做简单的测试进行对比。

### 静态页面
```
./ab -k -c 200 -n 200000 192.168.1.234/index.html
```

结果：
```
prefork：9556QPS
worker ：11038QPS
event ：10224QPS
```
### PHP页面
```
./ab -k -c 200 -n 200000 192.168.1.234/index.php  #echo "hello world";
```
结果：
```
prefork：6094QPS
worker ：7411QPS
event ：7089QPS
```
