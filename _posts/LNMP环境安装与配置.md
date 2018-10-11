---
title: LNMP环境安装与配置
date: 2016-09-25 23:30:00
ctime: 2016-09-25 23:30:00
utime: 2016-09-25 23:30:00
modif_times: 0
tags:
- LNMP
categories:
- PHP
---

![LNMP](http://n.sinaimg.cn/games/3ece443e/20160925/lnmp_log.png)
<!-- more -->

## 环境准备

OS：CentOS 7.2 64

MySQL：[mysql-5.7.15](http://dev.mysql.com/downloads/mysql/)

Nginx：[nginx-1.11.4](http://n.sinaimg.cn/games/3ece443e/20160925/nginx-1.11.4.tar.gz)

PHP：[php-5.6.25](http://n.sinaimg.cn/games/3ece443e/20160925/php-5.6.25.tar.gz)

## 安装
参考MySQL、Nginx、PHP的编译安装，分别安装 MySQL、Nginx、PHP到Linux。

## 整合

### 启动 php-fpm

新建用户和用户组，
```
# groupadd www-data
# useradd -g www-data www-data
```
需要修改 php-fpm.conf 配置文件，确保 php-fpm 模块使用 www-data 用户和 www-data 用户组的身份运行。

```
vi /usr/local/php56/etc/php-fpm.conf
#找到以下内容并修改：
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = www-data
group = www-data
```
修改 pid 配置，以方便我们后面根据pid管理php-fpm
```
[global]
; Pid file
; Note: the default prefix is /usr/local/php56/var
; Default Value: none
pid = run/php-fpm.pid
```
PHP-FPM运行时，将会在/usr/local/php56/var/run/下生成 php-fpm.pid 文件。

然后启动 php-fpm 服务：
```
/usr/local/php56/sbin/php-fpm
```
查看是否启动成功
```
ps aux | grep php
root     18858  0.0  0.1 148164  3208 ?        Ss   12:38   0:00 php-fpm: master process (/usr/local/php56/etc/php-fpm.conf)
www-data 18859  0.0  0.1 148164  2856 ?        S    12:38   0:00 php-fpm: pool www
www-data 18860  0.0  0.1 148164  2856 ?        S    12:38   0:00 php-fpm: pool www
root     18862  0.0  0.0 112664   972 pts/1    S+   12:38   0:00 grep --color=auto php
```
**注** 需要着重提醒的是，如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块， 以避免遭受恶意脚本注入的攻击。
将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 。

```
vi /usr/local/php56/lib/php.ini

cgi.fix_pathinfo=0
```


### PHP-FPM 重要配置
php-fpm.conf重要参数详解

- pid = run/php-fpm.pid
> pid设置，默认在安装目录中的var/run/php-fpm.pid，建议开启

- error_log = log/php-fpm.log
> 错误日志，默认在安装目录中的var/log/php-fpm.log

- log_level = notice
> 错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.

- emergency_restart_threshold = 60
- emergency_restart_interval = 60s
> 表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。

- process_control_timeout = 0
> 设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0.

- daemonize = yes
> 后台执行fpm,默认值为yes，如果为了调试可以改为no。在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置。

- listen = 127.0.0.1:9000
> fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port', '/path/to/unix/socket'. 每个进程池都需要设置.

- listen.backlog = -1
> backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。

- listen.allowed_clients = 127.0.0.1
> 允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接

- listen.owner = www
- listen.group = www
- listen.mode = 0666
> unix socket设置选项，如果使用tcp方式访问，这里注释即可。

- user = www
- group = www
> 启动进程的帐户和组

- pm = dynamic #对于专用服务器，pm可以设置为static。
> 如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定：

- pm.max_children #
> 子进程最大数

- pm.start_servers #
> 启动时的进程数

- pm.min_spare_servers #
> 保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程

- pm.max_spare_servers #
> 保证空闲进程数最大值，如果空闲进程大于此值，此进行清理

- pm.max_requests = 1000
> 设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.

- pm.status_path = /status
> FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到

- ping.path = /ping
> FPM监控页面的ping网址. 如果没有设置, 则无法访问ping页面. 该页面用于外部检测FPM是否存活并且可以响应请求. 请注意必须以斜线开头 (/)。

- ping.response = pong
> 用于定义ping请求的返回相应. 返回为 HTTP 200 的 text/plain 格式文本. 默认值: pong.

- request_terminate_timeout = 0
> 设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项。

- request_slowlog_timeout = 10s
> 当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'

- slowlog = log/$pool.log.slow
> 慢请求的记录日志,配合request_slowlog_timeout使用

- rlimit_files = 1024
> 设置文件打开描述符的rlimit限制. 默认值: 系统定义值默认可打开句柄是1024，可使用 ulimit -n查看，ulimit -n 2048修改。

- rlimit_core = 0
> 设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值.

- chroot =
> 启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用.

- chdir =
> 设置启动目录，启动时会自动Chdir到该目录. 所定义的目录需要是绝对路径. 默认值: 当前目录，或者/目录（chroot时）

- catch_workers_output = yes
> 重定向运行过程中的stdout和stderr到主要的错误日志文件中. 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 默认值: 空.


**Tips** 配置完成之后可以通过 php-fpm -t 来检测配置是否基本正确。
```
# /usr/local/php56/sbin/php-fpm -t
[25-Sep-2016 12:58:14] NOTICE: configuration file /usr/local/php56/etc/php-fpm.conf test is successful
```

### php-fpm管理

测试php-fpm配置
```
/usr/local/php/sbin/php-fpm -t
/usr/local/php/sbin/php-fpm -c /usr/local/php/etc/php.ini -y /usr/local/php/etc/php-fpm.conf -t
```
启动php-fpm
```
/usr/local/php/sbin/php-fpm
/usr/local/php/sbin/php-fpm -c /usr/local/php/etc/php.ini -y /usr/local/php/etc/php-fpm.conf
```
关闭php-fpm
```
kill -INT `cat /usr/local/php/var/run/php-fpm.pid`
```
重启php-fpm
```
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`
```

## 配置 Nginx 使其支持 PHP 应用：
```
vim /usr/local/nginx/conf/nginx.conf
```
修改默认的 location 块，使其支持 .php 文件：
```
location / {
    root   html;
    index  index.php index.html index.htm;
}
```
下一步配置来保证对于 .php 文件的请求将被传送到后端的 PHP-FPM 模块， 取消默认的 PHP 配置块的注释，并修改为下面的内容：
```
location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```
重启 Nginx。
```
# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
/usr/local/nginx/sbin/nginx -s reload
```
创建测试文件。
```
echo "<?php  phpinfo();?>" > /usr/local/nginx/html/index.php
```

打开浏览器，访问 http://ip，将会显示 phpinfo() 。

## 测试

创建测试文件
```
vi mysql_conn_test.php
```
输入
```php
<?php
$conn = mysql_connect("localhost", "root", "123456") or die("connect failed" . mysql_error());  
$sql = sprintf("SHOW DATABASES;");  
$result = mysql_query($sql, $conn);  
while ($row=mysql_fetch_array($result, MYSQL_ASSOC)){
  print_r($row);
}
?>
```
结果如下，则说明，连接测试成功。
```
Array ( [Database] => information_schema ) Array ( [Database] => mysql ) Array ( [Database] => performance_schema ) Array ( [Database] => sys )
```
至此，LNMP环境算是基本配置成功。

over~
