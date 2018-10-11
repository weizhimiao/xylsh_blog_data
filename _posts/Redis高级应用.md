---
title: Redis高级应用
date: 2016-09-27 20:30:00
tags:
- Redis
categories:
- Linux
---

- 给redis服务器设置密码
- 持久化
- 主从备份

<!-- more -->

##	给redis服务器设置密码

1、 修改redis服务器的配置文件
```
[root@localhost redis]# vi /usr/local/redis/etc/redis.conf
# requirepass foobared					（340行）
#找到这句话，requirepass后面就是登录redis的密码，改为
requirepass flzx_3QC
```
2、 重启redis
```
[root@localhost redis]# pkill  redis
[root@localhost redis]# bin/redis-server  /usr/local/redis/etc/redis.conf
```
3、 连接redis
```
[root@localhost redis]# /usr/local/redis/bin/redis-cli
127.0.0.1:6379> keys *						//可以正常连接redis
(error) NOAUTH Authentication required.		//但因为没有密码，提示操作拒绝
127.0.0.1:6379> auth flzx_3QC				//利用auth命令输入密码
OK
127.0.0.1:6379> keys *						//才可以正常使用
1) "name"
```
或
```
[root@localhost redis]# /usr/local/redis/bin/redis-cli -a flzx_3QC
```
在登录的同时指定密码

**注意历史命令中会明文保存此密码**
```
127.0.0.1:6379> keys *
1) "name"
```

## 持久化

Redis 提供了不同级别的持久化方式:
### RDB持久化方式
> RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。是默认的持久化方式。这种方式是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为dump.rdb。这种持久化方式被称为快照 snapshotting（快照）。

```
save 900 1		
#900秒内，最少有1个键被改动。则自动保存一次数据集
save 300 10
#300秒内，最少有10个键被改动。则自动保存一次数据集
save 60 10000
#60秒内，最少有10000个键被改动。则自动保存一次数据集
```
实验：验证dump.rdb数据保存文件
```
[root@localhost ~]# ls
anaconda-ks.cfg  dump.rdb  install.log  install.log.syslog
```
root目录下有dump.rdb文件
```
[root@localhost ~]# /usr/local/redis/bin/redis-server  \
/usr/local/redis/etc/redis.conf
```
在root目录中启动redis
```
[root@localhost ~]# /usr/local/redis/bin/redis-cli
127.0.0.1:6379> auth 123
OK
127.0.0.1:6379> keys *
1) "name2"
2) "name"
3) "name1"
```
0库中有键
```
[root@localhost ~]# cd /usr/local/redis/
[root@localhost redis]# pkill -9 redis
[root@localhost redis]# /usr/local/redis/bin/redis-server  \
/usr/local/redis/etc/redis.conf
```
在/usr/local/redis/库中重启redis，
```
[root@localhost redis]# ls
[root@localhost redis]# /usr/local/redis/bin/redis-cli
127.0.0.1:6379> keys *
(empty list or set)
```
0库中没有键
```
127.0.0.1:6379> save
OK
```
保存
```
127.0.0.1:6379> quit
[root@localhost redis]# ls
bin  dump.rdb  etc
```
在redis目录中也生成dump.rdb文件

**结论：**
```
[root@localhost redis]# vi /usr/local/redis/etc/redis.conf
dir ./
```
定义了dump.rdb数据库文件保存在当前位置。所以每次重启redis服务的所在位置不同，导致生成新的dump.rdb文件
```
dir /usr/local/redis/
```
将数据库保存目录写为绝对路径（注意只能是目录）

###  使用AOF
> 使用AOF会让你的Redis更加耐久: 你可以使用不同的持久化策略：无备份,每秒备份,每次写的时候备份。使用默认的每秒备份策略,Redis的性能依然很好(备份是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据。

```
appendonly no
#默认不使用AOF持久化（450行）

appendonly yes
#开启AOF持久化
# appendfsync always		#有写操作，就马上写入磁盘。效率最慢，到那时最按
appendfsync everysec		#默认，每秒钟写入磁盘一次。
# appendfsync no			#不进行AOF备份，将数据交给操作系统处理。最快，最不安全
```


## 主从备份
Redis主从复制特点：
- a.Master可以拥有多个slave
- b.多个slave可以连接同一个master外，还可以连接到其它slave
- c.主从复制不会阻塞master，在同步数据时，master可以继续处理client请求
- d.提高系统的伸缩性


Redis主从复制过程：
- a.Slave与master建立连接，发送sync同步命令
- b.Master会启动一个后台进程，将数据库快照保存到文件中，同时master主进程会开始收集新的写命令并缓存。
- c.后台完成保存后，就将此文件发送给slave
- d.Slave将此文件保存到硬盘上

###	不同服务器配置主从

1. 克隆一台linux作为从服务器
克隆机需要进行如下操作：

```
			①	vi /etc/sysconfig/network-scripts/ifcfg-eth0
				删除MAC地址行
			②	rm  -rf  /etc/udev/rules.d/70-persistent-net.rules
				删除网卡和MAC地址绑定文件
			③	注意关闭防火墙和SELinux
			④	重启动系统
```

2. 在从服务器上配置

```
[root@localhost ~]# vi /usr/local/redis/etc/redis.conf
# slaveof <masterip> <masterport>
#把此句开启，并指定主服务器ip和端口	（196行）

masterauth flzx_3QC
#设定主服务器密码
```

3. 重启从服务器上redis

### 同一台服务器实现主从配置

这里我们以本机配置 1台Master + 1台Slave 为例子,其中:
> - Master IP:127.0.0.1  PORT:6379
> - Slave1 IP:127.0.0.1  PORT:63791


1.  复制出从服务器目录

```
[root@localhost ~]# cp -r /usr/local/redis/ /usr/local/redis-slave1
```

2.  修改redis-slave1配置文件


```
[root@localhost ~]# vi /usr/local/redis-slave1/etc/redis.conf
pidfile /usr/local/redis-slave1/redis.pid
#指定pid文件
port 63791
#指定端口号
dir /usr/local/redis-slave1/
#指定服务器目录
slaveof 127.0.0.1 6379
#指定主服务器IP和端口
masterauth flzx_3QC
#指定主服务器密码
```

3. 启动服务


```
/usr/local/redis-slave1/bin/redis-server /usr/local/redis-slave1/etc/redis.conf
#启动从服务器，并调用从服务器配置文件

[root@localhost ~]# netstat -tlun
tcp     0      0 :::6379                     :::*                        LISTEN      
tcp     0      0 :::63791                    :::*                        LISTEN
#验证两个端口是否都启动
```

4. 验证

```
[root@localhost ~]# /usr/local/redis/bin/redis-cli -a flzx_3QC   
#启动主服务器，并建立一个键
127.0.0.1:6379> set bb 234
OK
127.0.0.1:6379> keys *
1) "sex"
2) "aa"
3) "name"
4) "age"
5) "bb"

[root@localhost ~]# /usr/local/redis-slave1/bin/redis-cli -a flzx_3QC -p 63791
#启动从服务器，发现键已经同步
127.0.0.1:63791> keys *
1) "aa"
2) "sex"
3) "age"
4) "name"
5) "bb"
```
