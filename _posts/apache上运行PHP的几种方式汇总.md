---
title: apache上运行PHP的几种方式汇总
date: 2016-10-21 22:40:00
categories:
- Apache
---

之前也有整理过一篇 「apache中PHP的几种支持模式」的blog，但是感觉还是比较混乱，很多问题没有理清楚，一些方式也没有整理到。所以今天通过这篇blog再整理一下。

- Using proxy_fcgi and php-fpm (for apache 2.4)

- Using php with fastcgi (for 2.2 and older)

- Using php with fcgid (for 2.2 and older)

- Using mod_php as a DSO with a threaded mpm (2.0 and newer)

- Using mod_php as a DSO (deprecated)

<!-- more -->

## Using proxy_fcgi and php-fpm (for apache 2.4)

此方法优先于所有其他方案，适用于Apache 2.4及更高版本。 它还允许线程mpm，如event或worker，这将显著减少对服务器的RAM需求。

详情见[httpd 2.4.x上使用mod_proxy_fcgi和php-fpm实现高性能PHP](https://weizhimiao.github.io/2016/10/20/High-performance%20PHP%20on%20apache%20httpd%202.4.x%20using%20mod_proxy_fcgi%20and%20php-fpm/)

## Using php with fastcgi (for 2.2 and older)

此方法适用于2.2及更高版本。 它还允许线程mpm，如event或worker，这将显著减少对服务器的RAM需求。

本文的范围是讨论使用Apache httpd 2.2和php的可能配置。在大多数情况下使用mod_php不是一个可行的解决方案，因为它引入了对每个httpd进程增加的RAM需求的可扩展性问题。另外，此外，它排除了使用线程MPM，因为php扩展在许多情况下不是线程安全的。

理想的配置是轻线程httpd进程，与外部fastcgi服务器（如PHP-FPM）通信。

第一个使用mod_fastcgi的方案
```
Alias /php5.fcgi /var/www/fcgi/php5.fcgi
FastCGIExternalServer /var/www/fcgi/php5.fcgi -flush -host 127.0.0.1:9000
# 创建一个新的处理程序名称并将其用于PHP文件
AddHandler my-fastcgi .php
#  *.php的请求实际上作为参数馈送到php-fastcgi
Action my-fastcgi /php5.fcgi
<Directory "/var/www/fcgi/">
  Order deny,allow
  Deny from all
  <Files "php5.fcgi">
    Order allow,deny
    Allow from all
  </Files>
</Directory>
```

## Using php with fcgid (for 2.2 and older)

此方法适用于2.0或2.2版本。 它允许线程mpm，如worker，这将显着减少您的服务器上的RAM要求。 而mod_fcgid是一个官方的Apache模块。


### Why ?

- 因为mod_php迫使你加载prefork MPM，这是低效的。
- 因为mod_php将被加载到httpd内存中，即使在服务静态页面
- 2013年发布的大多数发行版提供了预编译的包，让您可以使用fcgi运行php。 这只是一个配置问题。
- mod_fcgid是一个官方Apache模块，可在 http://httpd.apache.org/mod_fcgid/ 查看

### 优点

- 巨大的性能提升，在CPU和内存消耗
- PHP运行在一个单独的进程

### 不在fcgid上运行php的情况

如果你运行httpd 2.4，你应该考虑[PHP-FPM](http://wiki.apache.org/httpd/PHP-FPM)


### 快速体验

按照所有步骤，或许最终会丢失一些东西。

1、摆脱mod_php。 你需要从你的配置中注释掉“LoadModule php5_module”。
在debian上，只要运行“apt-get remove libapache2-mod-php5”


2、Install mod_fcgid
在 debian上, "apt-get install libapache2-mod-fcgid"

3、Install PHP as CGI
在 debian上, "apt-get install php5-cgi"

编写一个小封装包，例如：/usr/local/bin/php-wrapper
```
#!/bin/sh

# Set desired PHP_FCGI_* environment variables.

# Example:

# PHP FastCGI processes exit after 1000 requests by default.

PHP_FCGI_MAX_REQUESTS=1000

export PHP_FCGI_MAX_REQUESTS

# Replace with the path to your FastCGI-enabled PHP executable

exec /usr/lib/cgi-bin/php5
```
确保它是可读的和可执行的apache user/group.

5、修改配置 httpd.conf
```
LoadModule fcgid_module /usr/lib/apache2/modules/mod_fcgid.so

AddHandler fcgid-script .php

FcgidWrapper /usr/local/bin/php-wrapper .php
```

6、用多线程MPM替换prefork MPM，例如worker。

在debian上，只需运行“apt-get install apache2-mpm-worker”

### 常见问题/It doesn't work

不要惊慌。 检查apache错误日志。

PHP文件被下载，不解释
PHP files are downloaded, not interpreted

If you have a handler already set for PHP, it may be conflicting. So you can try something like : "grep -ri handler /etc/httpd | grep php" depending on the result, you may need to comment out some config you are having.

如果您已经为PHP设置了处理程序，则可能会发生冲突。 所以你可以尝试类似：“grep -ri handler /etc/httpd | grep php”根据结果，你可能需要注释掉一些存在配置。


### 更多选项

请参考： http://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html#upgrade

## Using mod_php as a DSO with a threaded mpm (2.0 and newer)

这种方法与下一个配方相同，只是可以使用event或worker等线程化的mpm。主要的要求是php系统库和DSO必须用线程安全标志（重新）编译。

如果使用apache httpd 2.0或更早版本，必须重新编译才能更改mpm。 对于2.4，加载适当的mpm模块后缀。

必须特别注意确保工作程序经常重启（MaxConnectionsPerChild> 0），因为子进程仍然容易出现php内存泄漏，并且进程可能消耗大量RAM并耗尽可用的系统资源。

这可能是所有最少使用的方法，由于维护一个线程安全的php库是一件非常头痛的事，并且因为大多数linux发行版不发运这些包。


## Using mod_php as a DSO (deprecated)

此方法是最早和可能是最慢的配置。 它适合2.2版本和更旧，并要求使用prefork mpm。

### 为什么你不应该使用mod_php与prefork mpm了

- mod_php始终加载到每个httpd进程中。 即使当httpd服务静态/非php内容。
- mod_php不是线程安全的，并且迫使你坚持使用prefork mpm（多进程，没有线程），这可能是最慢的配置


### 如何使用
首先，必须加载模块：
```
LoadModule php5_module lib/httpd/modules/libphp5.so
```
然后，添加dso的处理程序：

```
# Then, configure the handler for all files that end with .php
# A regexp such as \.(php|php4|php5)$ can also be used to support more extensions
<FilesMatch \.php$>
  SetHandler application/x-httpd-php
</FilesMatch>
```

参考:

[官方php安装和配置说明](http://www.php.net/manual/en/install.unix.apache2.php)
