---
title: memcache高速缓存工作原理及应用
date: 2016-09-29 18:30:00
ctime: 2016-09-19 18:48:00
utime: 2016-09-29 18:48:00
modif_times: 0
tags:
- 缓存
- memcache
- memcached
categories:
- Linux
---

Memcached,是高性能的分布式内存缓存服务器，主要功能就是通过缓存数据库的查询，减少对数据库的访问次数，来提高动态web应用的速度和可扩展性。

memcached 是以守护程序方法运行于一个或多个服务器中，随时接受客户端的连接操作，客户端可以由各种语言编写，目前一致的客户端api包括 Perl / PHP / Python / Java / C# / C 等等。客户端在与 memcached 服务建立连接之后，接下来的事情就是存取对象了， 每个被存取的对象都有一个唯一的标识符 key，存取操作均通过这个 key进行，保存到 memcached 中的对象实际上是放置内存中的，并不是保存在平时的 cache 文件中的，这也是为什么 memcached 能够如此高效快速的原因。

注意，这些对象并不是持久的， 服务停止之后， 里边的数据就会丢失。

![Memcached应用模型](http://n.sinaimg.cn/games/3ece443e/20160929/MemcacheYingYongMoXing.png?1)

<!-- more -->

## Memcached的安装
### 准备
libevent
> Libevent 是一个用C语言编写的、轻量级的开源高性能网络库。
> 主要有以下几个亮点：事件驱动（ event-driven），高性能;轻量级，专注于网络，不如 ACE 那么臃肿庞大；源代码相当精炼、易读；跨平台，支持 Windows、 Linux、 BSD 和 Mac Os；支持多种 I/O 多路复用技术， epoll、 poll、 dev/poll、 select 和 kqueue 等；支持 I/O，定时器和信号等事件；注册事件优先级。
> Libevent 已经被广泛的应用，作为底层的网络库；比如 memcached、 Vomit、 Nylon、 Netchat等等。

```
wget http://n.sinaimg.cn/games/3ece443e/20160929/libevent-2.0.22-stable.tar
tar -vxf libevent-2.0.22-stable.tar
cd libevent-2.0.22-stable
./configure --prefix=/usr/local/libevent
make && make install
```

### 安装
```
wget http://n.sinaimg.cn/games/3ece443e/20160930/memcached-1.4.31.tar.gz
cd memcached-1.4.31
 ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local/libevent
make && make install
```
查看是否已经安装成功
```
cd /usr/local/memcached/
ll
drwxr-xr-x 2 root root 4096 9月  28 10:56 bin
drwxr-xr-x 3 root root 4096 9月  28 10:56 include
drwxr-xr-x 3 root root 4096 9月  28 10:56 share
```
## Memcached管理

### 启动

启动Memcached服务器
```
/usr/local/memcached/bin/memcached -d -m 128 -u root -p 11211
```
查看是否启动成功
```
ps aux | grep memcache
root     24428  0.0  0.0 323120   864 ?        Ssl  11:00   0:00 /usr/local/memcached/bin/memcached -d -m 128 -u root -p 11211
root     24436  0.0  0.0 112664   984 pts/0    D+   11:00   0:00 grep --color=auto memcache
```
或
```
netstat -tlun | grep 11211
tcp        0      0 0.0.0.0:11211           0.0.0.0:*               LISTEN
tcp6       0      0 :::11211                :::*                    LISTEN
udp        0      0 0.0.0.0:11211           0.0.0.0:*
udp6       0      0 :::11211                :::*
```

设置开机自启动
```
echo "/usr/local/memcached/bin/memcached -d -m 128 -u root -p 11211" >> /etc/rc.d/rc.local
```

Memcached启动选项及说明

选项           |描述
--------------|----
-p <num>      | Memcached监听的TCP端口，要保证该端口号未被占用
-U <num>      | 指定监听UDP的端口，默认11211，0表示关闭
-s <file>     | 指定Memcached用于监听的UNIX socket文件
-A            | enable ascii "shutdown" command
-a <mask>     | 设置-s选项指定的UNIX socket文件的权限(默认权限: 0700)
-l <addr>     | 监听的服务器IP地址，如果有多个地址的话，使用逗号分隔，格式可以为“IP地址:端口号”，例如：-l 指定192.168.0.184:19830,192.168.0.195:13542；端口号也可以通过-p选项指定
-d            | 指定memcached进程作为一个守护进程启动
-r            | 设置产生core文件大小
-u <username> | 运行memcached的用户 (only when run as root)
-m <num>      | 指定分配给memcached使用的内存，单位是MB(默认: 64 MB)
-M            | 当内存使用超出配置值时，禁止自动清除缓存中的数据项，此时Memcached不可用，直到内存被释放
-c <num>      | 设置最大运行的并发连接数，默认是1024
-k            | 设置锁定所有分页的内存，对于大缓存应用场景，谨慎使用该选项
-v            | 输出警告和错误信息
-vv           | 打印信息比-v更详细：不仅输出警告和错误信息，也输出客户端请求和响应信息
-vvv          | extremely verbose (also print internal state transitions)
-h            | 显示Memcached版本和摘要信息
-i            | 打印libevent和Memcached的licenses信息
-V            | 输出Memcached版本号
-P <file>     | 保存memcached进程的pid文件，（与 -d 一起搭配使用）
-f <factor>   | 用于计算缓存数据项的内存块大小的乘数因子，默认是1.25
-n <bytes>    | 为缓存数据项的key、value、flag设置最小分配字节数，默认是48
-L            | 尝试使用大内存分页（pages）
-D <char>     | 用于统计报告中Key前缀和ID之间的分隔符，默认是冒号“:”  
-t <num>      | 指定用来处理请求的线程数，默认为4
-R            | 为避免客户端饿死（starvation），对连续达到的客户端请求数设置一个限额，如果超过该设置，会选择另一个连接来处理请求，默认为20
-C            | 禁用CAS
-b <num>      | Set the backlog queue limit (default: 1024)
-B            | 指定使用的协议，默认行为是自动协商（autonegotiate），可能使用的选项有auto、ascii、binary。
-I            | Override the size of each slab page. Adjusts max item size(default: 1mb, min: 1k, max: 128m)
-F            | 禁用flush_all命令
-o            | 指定逗号分隔的选项，一般用于用于扩展或实验性质的选项

### 通过telnet连接使用Memcache
连接
```
telnet 127.0.0.1 11211
```

命令格式：<command name> <key> <flags> <exptime> <bytes>\r\n <data block>\r\n
> <command name> 可以是”set”, “add”, “replace”
> <key> 客户端需要保存数据的key。
> <flags> 是一个16位的无符号的整数(以十进制的方式表示)。
> <exptime> 过期的时间。
> 最后客户端需要加上”\r\n”作为”命令头”的结束标志。即回车

示例：

保存一个数据（保存一个『cache_key1=>12345』的键值对到memcached 60s）
```
set cache_key1 0 60 5
12345
STORED
```
获取刚保存的值
```
get cache_key1
VALUE cache_key1 0 5
12345
END
```
其他命令：

Command | Description | Example
--------|-------------|-----------------
get     | 获取值 | get mykey
set     | 设置值（可以存在可以不存在） | set mykey 0 60 5
add     | 添加新值 | add newkey 0 60 5
replace | 替换值（必须已存在） | replace key 0 60 5
append  | 在原有值之后添加数据 | append key 0 60 15
prepend | 在原有值之前添加数据| prepend key 0 60 15
incr    | Increments numerical key value by given number | incr mykey 2
decr    | Decrements numerical key value by given number | decr mykey 5
delete  | 删除一条数据 | delete mykey
flush_all | 清除所有数据 | flush_all
        |清除900秒之内的数据 | flush_all 900
stats   | 查看所有状态| stats
        | Prints memory statistics | stats slabs
        | Prints memory statistics | stats malloc
        | Print higher level allocation statistics | stats items
        | | stats detail
        | 已使用大小 | stats sizes
        | 重置状态 | stats reset
version | 查看版本 | version
verbosity | Increases log level | verbosity
quit    | 退出telnet连接 | quit


### 通过客户端（PHP）连接和使用Memcached
php扩展Memcached安装

依赖
> libmemcached, 是一个 memcached 的库，客户端库，C 和 C++ 语言实现的客户端库，具有低内存占用率、线程安全、并提供对memcached功能的全面支持。它还采用 多种命令行工具： memcat ， memflush ， memrm ， memstat ，并memslap （负载代）。程序库一直在设计，让不同的散列方法对密钥，分割的钥匙，并使用统一的散列分配。

```
wget https://launchpadlibrarian.net/165454254/libmemcached-1.0.18.tar.gz
tar -zxvf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18
./configure --prefix=/usr/local/libmemcached
make && make install
```

安装扩展
```
wget http://n.sinaimg.cn/games/3ece443e/20160929/memcached-2.2.0.tar
tar -xvf memcached-2.2.0.tar
cd memcached-2.2.0
/usr/local/php56/bin/phpize
./configure --with-php-config=/usr/local/php56/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached  --enable-memcached --disable-memcached-sasl
make && make install
Installing shared extensions:     /usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/
```

查看是否安装成功
```
cd /usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/
ll
-rwxr-xr-x 1 root root  380475 9月  28 22:25 memcached.so
-rwxr-xr-x 1 root root  756714 9月  26 17:32 mysqli.so
-rwxr-xr-x 1 root root 1333912 9月  24 23:31 opcache.a
-rwxr-xr-x 1 root root  618435 9月  24 23:31 opcache.so
```
修改配置
```
vi /usr/local/php56/lib/php.ini
extension=/usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/memcached.so
```
重启php-fpm
```
kill -USR2 `cat /usr/local/php56/var/run/php-fpm.pid`
```
查看是否已经加载成功
```
/usr/local/php56/bin/php -m
或通过phpinfo();查看
```
测试
```
vi memcache_test.php
```
```php
<?php
        $mc = new Memcached();
        var_dump($mc);
        $mc->addServer('127.0.0.1', 11211);
        $mc->set('cache_key','mem_value',30);
        $val = $mc->get('cache_key');
        var_dump($val);
        var_dump($mc->delete('cache_key'));
        $mc->quit();
```
访问结果：
```
object(Memcached)#1 (0) { } string(9) "mem_value" bool(true)
```

php关于memcached 的两种扩展memcache 和 memcached 介绍

1. 目前大多数php环境里使用的都是不带d的memcache版本，这个版本出的比较早，是一个原生版本，完全在php框架内开发的。与之对应的带d的memcached是建立在libmemcached的基础上，所以相对来说，memcached版本的功能更全一些。
> [memcache:](http://cn2.php.net/manual/en/book.memcache.php)
>
> [memcached:](http://cn2.php.net/manual/en/book.memcached.php)

2. Memcache是原生实现的，支持OO和非OO两套接口并存。而memcached是使用libmemcached，只支持OO接口。
3. memcached有了一个统一的setOption()来设置设置，而不用在操作的时候设置了。Memcached实现了更多的memcached协议。
4. memcached支持Binary Protocol，而memcache不支持。这意味着memcached会有更高的性能。不过memcached目前还不支持长连接。
5. 另外一点也是大家比较关心的，就是所使用的算法。“一致性hash算法”是当添加或删除存储节点时，对存储在memcached上的数据影响较小的一种算法。在php的两个扩展库中，都可以使用该算法，只是设置方法有所不同。


  - Memcache

修改php.ini添加：
```
[Memcache]
Memcache.allow_failover = 1
Memcache.hash_strategy =consistent
Memcache.hash_function =crc32
```
或在php中使用ini_set方法：
```
ini_set(‘memcache.hash_strategy','standard');
ini_set(‘memcache.hash_function','crc32');
```
  - Memcached

```
$mem = new memcached();
$mem->setOption(Memcached::OPT_DISTRIBUTION,Memcached::DISTRIBUTION_CONSISTENT);
$mem->setOption(Memcached::OPT_LIBKETAMA_COMPATIBLE,true);
```

## Memcached监控

### 利用phpmemcache.php图形监控工具
下载 phpmemcache.php
```
wget http://n.sinaimg.cn/games/3ece443e/20160929/phpmemcache.php
```
将phpmemcache.php放入web目录
```
mv phpmemcache.php /usr/local/nginx/html
```
修改phpmemcache.php中配置
```
define('ADMIN_USERNAME','xxxx');    // 用户名修改，在访问 phpmemcache.php 需要进行认证
define('ADMIN_PASSWORD','xxxx');    // 密码
define('DATE_FORMAT','Y/m/d H:i:s');
define('GRAPH_SIZE',200);
define('MAX_ITEM_DUMP',50);

$MEMCACHE_SERVERS[] = '127.0.0.1:11211'; // 加入需要监控的memcached服务器
//$MEMCACHE_SERVERS[] = '192.168.200.104:11212'; // add more as an array
```

浏览器访问
![phpmemcache浏览器访问效果](http://n.sinaimg.cn/games/3ece443e/20160929/phpmemcache.png?1)


### 利用Stats命令查看
利用stats命令可以查看当前memcached的各种状态
```
telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
stats
STAT pid 24732
STAT uptime 66597
STAT time 1475115983
STAT version 1.4.31
STAT libevent 2.0.22-stable
STAT pointer_size 64
STAT rusage_user 6.194421
STAT rusage_system 2.419890
STAT curr_connections 10
STAT total_connections 16
STAT connection_structures 11
STAT reserved_fds 20
STAT cmd_get 5
STAT cmd_set 8
STAT cmd_flush 0
STAT cmd_touch 0
STAT get_hits 4
STAT get_misses 1
STAT get_expired 0
STAT get_flushed 0
STAT delete_misses 0
STAT delete_hits 2
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 3850
STAT bytes_written 293
STAT limit_maxbytes 134217728
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT time_in_listen_disabled_us 0
STAT threads 4
STAT conn_yields 0
STAT hash_power_level 16
STAT hash_bytes 524288
STAT hash_is_expanding 0
STAT malloc_fails 0
STAT log_worker_dropped 0
STAT log_worker_written 0
STAT log_watcher_skipped 0
STAT log_watcher_sent 0
STAT bytes 0
STAT curr_items 0
STAT total_items 6
STAT expired_unfetched 0
STAT evicted_unfetched 0
STAT evictions 0
STAT reclaimed 2
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
END
```

Stats详解

选项| 说明
---|-----
pid |memcache服务器的进程ID
uptime |服务器已经运行的秒数
time |服务器当前的unix时间戳
version |memcache版本
pointer_size |当前操作系统的指针大小（32位系统一般是32bit）
rusage_user |进程的累计用户时间
rusage_system |进程的累计系统时间
curr_items |服务器当前存储的items数量
total_items |从服务器启动以后存储的items总数量
bytes |当前服务器存储items占用的字节数
curr_connections|当前打开着的连接数
total_connections |从服务器启动以后曾经打开过的连接数
connection_structures |服务器分配的连接构造数
cmd_get |get命令（获取）总请求次数
cmd_set |set命令（保存）总请求次数
get_hits |总命中次数
get_misses |总未命中次数
evictions |为获取空闲内存而删除的items数（分配给memcache的空间用满后需要删除旧的items来得到空间分配给新的items）
bytes_read|总读取字节数（请求字节数）
bytes_written|总发送字节数（结果字节数）
limit_maxbytes|分配给memcache的内存大小（字节）
threads|当前线程数


### 利用各种监控软件查看（例如：nagios监控memcache的插件）
> 只以命中率大于和小于为例两种状态。

```
vim check_memcache
```
```bash
#!/bin/sh
if [ $# -ne 1 ]
then
echo "Usage:$0 -c num2"
exit 0
fi
cmd_get=`/usr/local/nagios/libexec/check_tcp -H localhost -p 11211 -E -s 'stats\r\nquit\r\n' -e 'uptime' |grep cmd_get | awk '{print $3+0}'`
get_hits=`/usr/local/nagios/libexec/check_tcp -H localhost -p 11211 -E -s 'stats\r\nquit\r\n' -e 'uptime' |grep get_hits | awk '{print $3+0}'`
hit_rate=`echo "$get_hits*100/$cmd_get"|bc`
if [ $hit_rate -gt $1 ];then
echo "OK - hit rate is $hit_rate | hit_rate=$hit_rate; cmd_get=$cmd_get; get_hits=$get_hits"
exit 0
else
echo "CRITICAL - hit rate is $hit_rate | hit_rate=$hit_rate; cmd_get=$cmd_get; get_hits=$get_hits"
exit 2
fi
```

测试命中率大于80%为正常为例;
```
eg：
sh check_memcache 80
root@ip-10-250-114-95:/liang# sh check_memcache 80
OK - hit rate is 99 | hit_rate=99; cmd_get=142547; get_hits=141880
```
以上证明命中率99%，即状态为OK.
