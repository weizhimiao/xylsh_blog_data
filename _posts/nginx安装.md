---
title: Nginx安装
date: 2016-09-25 21:30:00
ctime: 2016-09-25 21:30:00
utime: 2016-09-25 21:30:00
modif_times: 0
tags:
- Nginx安装
categories:
- Nginx
---

![nginx](http://n.sinaimg.cn/games/3ece443e/20160925/nginx_log.png)

<!-- more -->

## 准备
OS：CentOS 7.2 64

Nginx：[nginx-1.11.4](http://n.sinaimg.cn/games/3ece443e/20160925/nginx-1.11.4.tar.gz)

## 编译环境准备
```
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```
> 依赖工具说明:
> gcc 编译器
> pcre 正则表达式工具
> zlib 传输内容压缩
> openssl Https支持

## 编译安装
解压源码包
```
tar -zxvf nginx-1.11.4.tar.gz
```
进入源码包目录
```
cd nginx-1.11.4
```

执行配置命令
```
./configure --prefix=/usr/local
Configuration summary
  + using PCRE library: /usr/local/pcre
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```
执行编译安装
```
make && make install
```
## 管理
### 启动
```
/usr/local/nginx/sbin/nginx
```
### 停止
```
/usr/local/nginx/sbin/nginx -s stop
```
### 重启
```
/usr/local/nginx/sbin/nginx -s reload
```
### 查看Nginx进程状态
```
ps aux |grep nginx
```
结果形如
```
root     24367  0.0  0.0  20472   604 ?        Ss   22:52   0:00 nginx: master process /usr/local/nginx/sbin/nginx
nobody   24368  0.0  0.0  20900  1320 ?        S    22:52   0:00 nginx: worker process
root     24370  0.0  0.0 112664   976 pts/0    S+   22:52   0:00 grep --color=auto nginx
```
> master proccess为主进程 守护进程
> worker proccess为工作进程, 用于响应请求


### 设置开机自动启动
编辑文件 /etc/rc.d/rc.local
```
echo "/usr/local/nginx/sbin/nginx" >> /etc/rc.d/rc.local
```

## 测试
启动nginx，在浏览器通过IP地址访问服务器。查看是否有响应。

~over
