---
title: Linux中常用的shell脚本（二）
date: 2016-11-01 20:30:00
tags:
- shell
categories:
- Linux
---

![Linux中常用的shell脚本](http://n.sinaimg.cn/games/3ece443e/20161030/bash.png)

<!-- more -->

1、进程运行前后台切换
```
Ctrl+z		# 将进程转入后台运行
```
```
fg				# 将进程转到前台
```
示例：
```
$ ping www.baidu.com
PING www.a.shifen.com (119.75.217.109): 56 data bytes
64 bytes from 119.75.217.109: icmp_seq=0 ttl=57 time=6.829 ms
64 bytes from 119.75.217.109: icmp_seq=1 ttl=57 time=32.417 ms
64 bytes from 119.75.217.109: icmp_seq=2 ttl=57 time=5.999 ms
64 bytes from 119.75.217.109: icmp_seq=3 ttl=57 time=21.105 ms
^Z
[1]  + 1879 suspended  ping www.baidu.com
--------------------------------------------------------------
$ fg                
[1]  + 1879 continued  ping www.baidu.com
64 bytes from 119.75.217.109: icmp_seq=4 ttl=57 time=27.063 ms
64 bytes from 119.75.217.109: icmp_seq=5 ttl=57 time=5.939 ms
64 bytes from 119.75.217.109: icmp_seq=6 ttl=57 time=6.633 ms
64 bytes from 119.75.217.109: icmp_seq=7 ttl=57 time=7.462 ms
^C
--- www.a.shifen.com ping statistics ---
8 packets transmitted, 8 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 5.939/14.181/32.417/10.232 ms
```
2、截取前5个字符
```
$ var="hello world"
$ echo ${var:0:5}
hello
```
3、一次创建多个目录
```
$ mkdir -p data/{test,test1,test2}
$ tree data
data
├── test
├── test1
└── test2
```
4、获得文本的md5 hash
```
echo -n "testText" | md5sum
```
5、将tar.gz提取到新目录
```
$ tar -zxvf package.tar.gz -C /path/to/new
```
6、通过curl获取HTTP头信息
```
$ curl -I http://www.baidu.com
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Tue, 01 Nov 2016 15:00:30 GMT
Content-Type: text/html
Content-Length: 277
Last-Modified: Mon, 13 Jun 2016 02:50:08 GMT
Connection: Keep-Alive
ETag: "575e1f60-115"
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Pragma: no-cache
Accept-Ranges: bytes
```
7、快速备份一个文件
```
$ ll
-rw-r--r--   1 zhimiao  staff   200B  2 29  2016 test.txt

$ cp test.txt{,.bak}

$ ll
-rw-r--r--   1 zhimiao  staff   200B  2 29  2016 test.txt
-rw-r--r--   1 zhimiao  staff   200B 11  1 23:02 test.txt.bak
```
8、利用cat快速输入多行文字（Ctrl+d 退出）
```
$ cat > test2.txt
weizhadf
asdfads
asdf
adsf
asdf
adf
(Ctrl+d)

$ cat test2.txt
weizhadf
asdfads
asdf
adsf
asdf
adf
```
9、重复运行命令，并显示其输出（默认是2秒运行一次）
```
watch ps -ef
Every 2.0s: ps -ef                                                                     Tue Nov  1 23:10:06 2016

  UID   PID  PPID   C STIME   TTY           TIME CMD
    0     1     0   0 10:28    ??         0:05.23 /sbin/launchd
    0    47     1   0 10:28    ??         0:01.21 /usr/libexec/UserEventAgent (System)
    0    48     1   0 10:28    ??         0:00.42 /usr/sbin/syslogd
    0    50     1   0 10:28    ??         0:00.14 /System/Library/PrivateFrameworks/Uninstall.framework/Resourc
es/uninstalld
......
```
10、递归查找目录中文件中的内容
```
$ grep -r "some_text" /path/
```
11、将所有的文件名中含有"\*.txt"的文件，移入指定目录中
```
$ find -iname "*.txt*" -exec mv -v {} /home/user \;
```
12、拆分大体积tar.gz文件（拆成每个100MB），然后合并
```
$ split -b 100m /path/to/large/archive /path/to/output/files
$ cat files* > archive
```
13、Shell(Bash)中如何判断是否存在某个命令
```
# 判断foo命令是否存在
$ command -v foo >/dev/null 2>&1 || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
$ type foo >/dev/null 2>&1 || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
$ hash foo 2>/dev/null || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
```
