---
title: PHP安装
date: 2016-09-25 22:30:00
ctime: 2016-09-25 22:30:00
utime: 2016-09-25 22:30:00
modif_times: 0
tags:
- PHP-FPM
categories:
- PHP
---

![php](http://n.sinaimg.cn/games/3ece443e/20160925/php_log.png)

<!-- more -->

## 环境

OS：CentOS 7.2 64

PHP：[php-5.6.25](http://n.sinaimg.cn/games/3ece443e/20160925/php-5.6.25.tar.gz)


## 编译前准备
```
yum -y install libxml2 libxml2-devel
```
libxml2,是个C语言的XML程式库，能简单方便的提供对XML文件的各种操作，并且支持XPATH查询，及部分的支持XSLT转换等功能。

## 编译安装
解压源码包
```
tar zxvf php-5.6.25.tar.gz
```
进入源码包目录
```
cd php-5.6.25
```
编译、安装
```
./configure --prefix=/usr/local/php56 --enable-fpm --with-mysql
make && make install
Installing shared extensions:     /usr/local/php56/lib/php/extensions/no-debug-non-zts-20131226/
Installing PHP CLI binary:        /usr/local/php56/bin/
Installing PHP CLI man page:      /usr/local/php56/php/man/man1/
Installing PHP FPM binary:        /usr/local/php56/sbin/
Installing PHP FPM config:        /usr/local/php56/etc/
Installing PHP FPM man page:      /usr/local/php56/php/man/man8/
Installing PHP FPM status page:   /usr/local/php56/php/php/fpm/
Installing PHP CGI binary:        /usr/local/php56/bin/
Installing PHP CGI man page:      /usr/local/php56/php/man/man1/
Installing build environment:     /usr/local/php56/lib/php/build/
Installing header files:           /usr/local/php56/include/php/
Installing helper programs:       /usr/local/php56/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php56/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php56/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.0
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.3.0
[PEAR] PEAR           - installed: 1.10.1
Wrote PEAR system config file at: /usr/local/php56/etc/pear.conf
You may want to add: /usr/local/php56/lib/php to your php.ini include_path
/root/php-5.6.25/build/shtool install -c ext/phar/phar.phar /usr/local/php56/bin
ln -s -f phar.phar /usr/local/php56/bin/phar
Installing PDO headers:           /usr/local/php56/include/php/ext/pdo/
```
生成配置文件
```
cp php.ini-development /usr/local/php56/lib/php.ini
cp /usr/local/php56/etc/php-fpm.conf.default /usr/local/php56/etc/php-fpm.conf
```

查看配置文件是否已生效。
```
$ /usr/local/php56/bin/php -r "phpinfo();"
如果看到以下输出，则表示配置文件加载成功。
Configuration File (php.ini) Path => /usr/local/php56/lib
Loaded Configuration File => /usr/local/php56/lib/php.ini
```

将php加入到PATH中
```
vi ~/.bash_profile
#在export PATH前一行插入
PATH=$PATH:/usr/local/php56/bin:/usr/local/php56/lib
```
重新加载环境变量
```
source /root/.bash_profile
```

## 测试
```
[root@iZwz97v8o84q253plfkxvfZ php56]# php -version
PHP 5.6.25 (cli) (built: Sep 24 2016 23:30:43)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
```

**Tips:**
如何确定PHP当前使用的配置文件的位置？

php：
```
$ /usr/local/php56/bin/php -r "phpinfo();"
如果看到以下输出，则表示配置文件加载成功。
Configuration File (php.ini) Path => /usr/local/php56/lib
Loaded Configuration File => /usr/local/php56/lib/php.ini
```
php-fpm:
```
/usr/local/php56/sbin/php-fpm -t
[25-Sep-2016 18:01:40] NOTICE: configuration file /usr/local/php56/etc/php-fpm.conf test is successful
```
over~
