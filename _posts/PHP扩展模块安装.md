---
title: PHP扩展模块安装
date: 2016-09-26 21:30:00
ctime: 2016-09-26 21:48:00
utime: 2016-09-26 21:48:00
modif_times: 1
tags:
- PHP扩展
- phpize
categories:
- PHP
---

PHP编译安装之后，通常我们还需要根据我们的业务需求去安装各种扩展。通常我们可以通过php提供的phpize这个工具来为PHP动态地添加我们需要的模块。

<!-- more -->

## 工具介绍
- phpize，是用来扩展php扩展模块的，通过phpize可以建立php的外挂模块。一般在我们安装PHP时已经一起安装了。位置一般在/path/to/php/bin/phpize。

- autoconf，是用来生成自动配置软件源代码脚本（configure）的 工具.configure脚本能独立于autoconf运行,且在 运行的 过程中,不需要用户的干预.

- m4，是 一个宏处理器.将输入拷贝到输出,同时将宏展开.宏可以是 内嵌的 ,也可以是 用户定义的 .除了可以展开宏,m4还有一些内建的 函数,用来引用文件,执行命令,整数运算,文本操作,循环等.m4既可以作为编译器的 前端,也可以单独作为一个宏处理器.

## 示例（为PHP添加mysqli扩展）
进入PHP源码包的ext/mysqli扩展目录
```
cd  ext/mysqli
./configure --with-php-config=/usr/local/php56/bin/php-config --with-mysqli=/usr/local/mysql/bin/mysql_config
make && make install
#
#Installing shared extensions:     /usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/
#Installing header files:           /usr/local/php56/include/php/
```

查看模块是否编译成功
```
cd /usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/
ll
#-rwxr-xr-x 1 root root  756714 Sep 26 17:32 mysqli.so
#-rwxr-xr-x 1 root root 1333912 Sep 24 23:31 opcache.a
#-rwxr-xr-x 1 root root  618435 Sep 24 23:31 opcache.so
```

将模块加载到php
```
vi /usr/local/php56/lib/php.ini
#将下面这行写入到php.ini中
extension=/usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/mysqli.so
```
重启php-fpm
```
kill -USR2 `cat /usr/local/php56/var/run/php-fpm.pid`
```

查看是否加载成功
```
php -m
```
或浏览器访问index.php (包含phpinfo()函数)
