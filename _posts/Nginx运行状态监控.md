---
title: Nginx运行状态监控
date: 2016-09-27 22:30:00
tags:
- 监控
categories:
- Nginx
---

Nginx可以通过stub_status模块来查看服务器的状态信息。

<!-- more -->
## 安装stub_status模块
查看服务器当前是否已经编译安装过stub_status模块
```
/usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.11.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC)
configure arguments: --prefix=/usr/local/nginx
```
安装 stub_status 模块

解压相应版本的nginx源码包，
```
cd nginx-1.11.4
```
配置
```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module
```
编译（不执行make install操作）
```
make
```
手动替换 nginx 执行文件
```
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
cp ./objs/nginx /usr/local/nginx/sbin/
```
查看是否安装成功
```
/usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.11.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC)
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module
```

## 启用nginx status配置

修改配置
```
vi /usr/local/nginx/conf/nginx.conf
```
加入配置
```
location /ngx_status
{
    stub_status on;
    access_log off;
    allow all;
    #deny all;
}
```
重启nginx
```
# /usr/local/nginx/sbin/nginx -s reload 可能或有问题，所以先停止当前nginx进程，然后在重启。
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx
```

## 测试

浏览器或者curl访问

http://120.76.250.101/ngx_status
或
```
curl 127.0.0.1/ngx_status
Active connections: 1
server accepts handled requests
 2 2 2
Reading: 0 Writing: 1 Waiting: 0
```

参数说明

参数 | 说明
----|-----
Active connections | 活跃的连接数量
server accepts handled requests | 2 2 2 表示总共处理了2个连接 , 成功创建2次握手, 总共处理了2个请求
reading | 读取客户端的连接数.
writing | 响应数据到客户端的数量
waiting | 开启 keep-alive 的情况下,这个值等于 active – (reading+writing), 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接.

over~
