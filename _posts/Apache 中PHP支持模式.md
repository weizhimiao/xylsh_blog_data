---
title: Apache中PHP支持模式小结
date: 2016-09-07 20:10:00
tags:
- FastCGI
- CGI
- Apache&PHP
categories:
- Apache
---
Apache通过不同的方式，能够实现对PHP支持。常见的几种支持方式有：
- 模块支持（handler模式）
- CGI模块
- FastCGI模式，用Apache内置进程管理器
- FastCGI模式，用php-fpm进程管理器

<!-- more -->
## Apache安装、PHP安装、mod_fastcgi模块安装
### Apache 安装
略过···
### PHP 安装
示例：安装PHP5.6

1. 下载、解压
```sh
[root@iZ23a3ua2stZ ~]#wget http://cn2.php.net/distributions/php-5.6.25.tar.gz
[root@iZ23a3ua2stZ ~]#tar -zxvf php-5.6.25.tar.gz
```
2. ./configure
```sh
[root@iZ23a3ua2stZ ~]#cd php-5.6.25
[root@iZ23a3ua2stZ php-5.6.25]#./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql --enable-fpm --prefix=/usr/local/php5.6
```
  `--enable-fpm` 用来来激活对 FPM 的支持

  `--with-apxs2` 该参数作用是把php的解释模块编译成so 文件，并自动添加到 Apache的modules中，并自动在 `httpd.conf` 中加入对应的加载指令`LoadModule php5_module modules/libphp5.so`
> **报错1：** 在运行 `./configure` 时可能会报以下这样的错误。提示我们 运行不了 `apxs` 这个工具。

  ```sh
Sorry, I cannot run apxs.  Possible reasons follow:
1 Perl is not installed
2 apxs was not found. Try to pass the path using --with-apxs2=/path/to/apxs
3 Apache was not built using --enable-so (the apxs usage page is displayed)
The output of /usr/local/apache2/bin/apxs follows:
./configure: /usr/local/apache2/bin/apxs: /replace/with/path/to/perl/interpreter: bad interpreter: No such file or directory
configure: error: Aborting
```
> 之前我们在Apache模块化体系的小结中介绍过 `apxs` 实际是一个 perl 脚本。上面报错中 `/replace/with/path/to/perl/interpreter` 提示我们找不到这个文件，
这行实际是 perl 脚本的声明，我们需要将它改成我们服务器perl脚本的地址。

  ```sh
[root@iZ23a3ua2stZ php-5.6.25]# which perl
/usr/bin/perl
[root@iZ23a3ua2stZ php-5.6.25]# vi /usr/local/apache2/bin/apxs
#将第一行 `#!/replace/with/path/to/perl/interpreter` 改成 `#!/usr/bin/perl`
```
> **报错2：** configure: error: xml2-config not found. Please check your libxml2 installation.
提示我们缺少 `libxml2`。安装 `libxml2`.
```sh
[root@iZ23a3ua2stZ php-5.6.25]#yum install libxml2
[root@iZ23a3ua2stZ php-5.6.25]#yum install libxml2-devel -y
```

  `./configure` 成功会有如下显示。

  ```sh
+--------------------------------------------------------------------+
| License:                                                           |
| This software is subject to the PHP License, available in this     |
| distribution in the file LICENSE.  By continuing this installation |
| process, you are bound by the terms of this license agreement.     |
| If you do not agree with the terms of this license, you must abort |
| the installation process at this point.                            |
+--------------------------------------------------------------------+
Thank you for using PHP.
config.status: creating php5.spec
config.status: creating main/build-defs.h
config.status: creating scripts/phpize
config.status: creating scripts/man1/phpize.1
config.status: creating scripts/php-config
config.status: creating scripts/man1/php-config.1
config.status: creating sapi/cli/php.1
config.status: creating sapi/fpm/php-fpm.conf
config.status: creating sapi/fpm/init.d.php-fpm
config.status: creating sapi/fpm/php-fpm.service
config.status: creating sapi/fpm/php-fpm.8
config.status: creating sapi/fpm/status.html
config.status: creating sapi/cgi/php-cgi.1
config.status: creating ext/phar/phar.1
config.status: creating ext/phar/phar.phar.1
config.status: creating main/php_config.h
```

3. 编译
```sh
[root@iZ23a3ua2stZ php-5.6.25]#make && make install
```
成功安装
```
Build complete.
Don't forget to run 'make test'.
Installing PHP SAPI module:       apache2handler
/usr/local/apache2/build/instdso.sh SH_LIBTOOL='/usr/local/apache2/build/libtool' libphp5.la /usr/local/apache2/modules
/usr/local/apache2/build/libtool --mode=install install libphp5.la /usr/local/apache2/modules/
libtool: install: install .libs/libphp5.so /usr/local/apache2/modules/libphp5.so
libtool: install: install .libs/libphp5.lai /usr/local/apache2/modules/libphp5.la
libtool: install: warning: remember to run `libtool --finish /root/php-5.6.25/libs'
chmod 755 /usr/local/apache2/modules/libphp5.so
[activating module `php5' in /usr/local/apache2/conf/httpd.conf]
Installing shared extensions:     /usr/local/php5.6/lib/php/extensions/no-debug-zts-20131226/
Installing PHP CLI binary:        /usr/local/php5.6/bin/
Installing PHP CLI man page:      /usr/local/php5.6/php/man/man1/
Installing PHP FPM binary:        /usr/local/php5.6/sbin/
Installing PHP FPM config:        /usr/local/php5.6/etc/
Installing PHP FPM man page:      /usr/local/php5.6/php/man/man8/
Installing PHP FPM status page:   /usr/local/php5.6/php/php/fpm/
Installing PHP CGI binary:        /usr/local/php5.6/bin/
Installing PHP CGI man page:      /usr/local/php5.6/php/man/man1/
Installing build environment:     /usr/local/php5.6/lib/php/build/
Installing header files:           /usr/local/php5.6/include/php/
Installing helper programs:       /usr/local/php5.6/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php5.6/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php5.6/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.0
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.3.0
[PEAR] PEAR           - installed: 1.10.1
Wrote PEAR system config file at: /usr/local/php5.6/etc/pear.conf
You may want to add: /usr/local/php5.6/lib/php to your php.ini include_path
/root/php-5.6.25/build/shtool install -c ext/phar/phar.phar /usr/local/php5.6/bin
ln -s -f phar.phar /usr/local/php5.6/bin/phar
Installing PDO headers:           /usr/local/php5.6/include/php/ext/pdo/
```

4. 设置配置文件
```
cp php.ini-development /usr/local/php5.6/lib/php.ini
```

### mod_fastcgi 模块安装

1. 下载、解压
[mod_fastcgi-2.4.6.tar.gz](http://n.sinaimg.cn/games/3ece443e/20160907/mod_fastcgi-2.4.6.tar.gz)
```
[root@iZ23a3ua2stZ ~]#wget http://n.sinaimg.cn/games/3ece443e/20160907/mod_fastcgi-2.4.6.tar.gz
[root@iZ23a3ua2stZ ~]#tar -zxvf mod_fastcgi-2.4.6.tar.gz
cd mod_fastcgi-2.4.6
```
2. 编译、安装
  - 查看安装说明
```
[root@iZ23a3ua2stZ ~]#cd mod_fastcgi-2.4.6
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#
#查看 安装说明文件 (Apache 1.x请查看 INSTALL 文件)
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#vi INSTALL.AP2  
···
$ cd <mod_fastcgi_dir>
$ cp Makefile.AP2 Makefile
$ make
$ make install
If your Apache2 installation isn't in /usr/local/apache2, then
set the top_dir variable when running make (or edit the
Makefile), e.g.
  $ make top_dir=/opt/httpd/2.0.40
Add an entry to httpd.conf like this:
  LoadModule fastcgi_module modules/mod_fastcgi.so
···
```
虽然说明文件里描述的很清楚，但我们按照里面的步骤进行编译时，会产生报错。于是找了一大圈问题，发现在 `apache2.4` 下安装 `mod_fastcgi 2.4.6` 编译时汇报错，需要打个补丁。

3. 打补丁
[补丁下载地址](http://n.sinaimg.cn/games/3ece443e/20160907/byte-compile-against-apache24.diff)
```sh
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#wget http://n.sinaimg.cn/games/3ece443e/20160907/byte-compile-against-apache24.diff
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#patch -p1 < byte-compile-against-apache24.diff
```

4. 继续编译、安装
```
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#make
[root@iZ23a3ua2stZ mod_fastcgi-2.4.6]#make install
```
然后查看`/usr/local/apache2/modules/` 下是否已经编译成功 `mod_fastcgi.so`，
并在 `httpd.conf` 中添加 `LoadModule fastcgi_module modules/mod_fastcgi.so` 指令。


## 各种模式配置
### 模块模式（最简单）
- 在 `httpd.conf` 中添加或者开启
```ini
LoadModule php5_module modules/libphp5.so
```
- 然后在 `httpd.conf` 中找到 `<IfModule mime_module>` 配置段，在其中添加
```ini
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
```
- 重启 Apache ,在站点目录下添加 `index.php`
```php
<?php
    phpinfo();
 ?>
```
- 通过浏览器访问该文件，能够正确输出，则说明配置成功。
> 这时 `phpinfo();` 输出的 `Server API` 应该为 `Apache 2.0 Handler`.

### CGI模式
- 在 `httpd.conf` 中添加或者开启
```ini
LoadModule cgid_module modules/mod_cgid.so
LoadModule actions_module modules/mod_actions.so
```
> `mod_cgid` ,CGI 处理模块。在某些Unix操作系统上，在多线程服务器fork一个进程是一个代价非常大的操作，因为新进程将会复制父进程所有的线程。为了避免每次CGI调用都进行这样代价大的操作。mod_cgid 采用外部守护进程来负责分派子进程来运行CGI脚本。主服务器使用Unix套接字来与此守护进程进行通信。

  > 如果在Apache编译过程中选择了多线程MPM，那么Apache将会默认安装mod_cgid模块，而不是mod_cgi。但在用户配置使用层面，mod_cgid 和 mod_cgi 基本相同，唯一例外的是，mod_cgid 通过指令 ScriptSock 给socket命名用于与CGI守护进程通信。

  > `mod_actions` ,模块提供基于基于媒体类型或请求方法来执行CGI脚本的方法。该模块引入 `Action` 和 `Script` 两个指令。

- 然后在 `httpd.conf` 中找到 `<IfModule mime_module>` 配置段，在其中添加
```ini
AddHandler php-cgi .php
Action php-cgi "/cgi-bin/php-cgi"
```
- 然后在 `httpd.conf` 中找到 `<IfModule cgid_module>` 配置段，在其中添加
```ini
<IfModule cgid_module>
    Scriptsock /var/run/cgid.sock
</IfModule>
```

- 重启 Apache ,在站点目录下添加 `index.php`
```php
<?php
    phpinfo();
 ?>
```
- 通过浏览器访问该文件，能够正确输出，则说明配置成功。
> 这时 `phpinfo();` 输出的 `Server API` 应该为 `CGI/FastCGI`.

### FastCGI模式，用Apache内置进程管理器
- 在 `httpd.conf` 中添加或者开启
```ini
LoadModule fastcgi_module modules/mod_fastcgi.so
LoadModule actions_module modules/mod_actions.so
```
- 然后在 `httpd.conf` 中添加 `<IfModule fastcgi_module>` 配置段
```ini
<IfModule fastcgi_module>
   FastCgiServer /usr/local/apache2/cgi-bin/php-cgi -processes 20
   AddType application/x-httpd-php .php
   AddHandler php-fastcgi .php
   Action php-fastcgi /cgi-bin/php-cgi
</IfModule>
```
> `-processes 20` ,配置启动的进程数

- 重启 Apache ,执行 `ps aux | grep php` 查看相应 php-cgi 进程是否启动
```bash
[root@iZ23a3ua2stZ apache2]# ps aux | grep php
daemon   21382  0.0  0.5  42104  5604 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21435  0.0  0.4  42104  4724 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21436  0.0  0.4  42104  4732 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21437  0.0  0.4  42104  4724 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21438  0.0  0.4  42104  4724 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21439  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21440  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21441  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21442  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21443  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21444  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21445  0.0  0.4  42104  4724 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21446  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21447  0.0  0.4  42104  4724 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21448  0.0  0.4  42104  4728 ?        S    11:30   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21449  0.0  0.4  42104  4728 ?        S    11:31   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21450  0.0  0.4  42104  4724 ?        S    11:31   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21451  0.0  0.4  42104  4724 ?        S    11:31   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21452  0.0  0.4  42104  4732 ?        S    11:31   0:00 /usr/local/apache2/cgi-bin/php-cgi
daemon   21453  0.0  0.4  42104  4728 ?        S    11:31   0:00 /usr/local/apache2/cgi-bin/php-cgi
root     21455  0.0  0.0 112668   984 pts/0    S+   11:31   0:00 grep --color=auto php
```

- 在站点目录下添加 `index.php`
```php
<?php
    phpinfo();
 ?>
```
- 通过浏览器访问该文件，能够正确输出，则说明配置成功。
> 这时 `phpinfo();` 输出的 `Server API` 应该为 `CGI/FastCGI`.

### FastCGI模式，使用php-fpm进程管理器
- 在 `httpd.conf` 中添加或者开启
```ini
LoadModule fastcgi_module modules/mod_fastcgi.so
LoadModule actions_module modules/mod_actions.so
```
- 然后在 `httpd.conf` 中添加 `<IfModule fastcgi_module>` 配置段
```ini
<IfModule fastcgi_module>
   FastCgiExternalServer /usr/local/apache2/cgi-bin/php-cgi -host 127.0.0.1:9000
   AddType application/x-httpd-php .php
   AddHandler php-fastcgi .php
   Action php-fastcgi /cgi-bin/php-cgi
</IfModule>
```
> `-host 127.0.0.1:9000` ,是php-fpm的开启端口，所以我们还需要把php-fpm打开。

- 启动 `php-fpm`
```bash
[root@iZ23a3ua2stZ apache2]# /usr/local/php5.6/sbin/php-fpm
```
> /usr/local/php/sbin/php-fpm{start|stop|quit|restart|reload|logrotate}
>
> --start 启动php的fastcgi进程
> --stop 强制终止php的fastcgi进程
> --quit 平滑终止php的fastcgi进程
> --restart 重启php的fastcgi进程
> --reload 重新平滑加载php的php.ini
> --logrotate 重新启用log文件

- 重启 Apache ,执行 `ps aux | grep php` 查看相应 php-cgi 进程是否启动
```bash
[root@iZ23a3ua2stZ apache2]# ps aux | grep php
[root@iZ23a3ua2stZ apache2]# ps aux | grep php
root     21680  0.0  0.3 147920  3660 ?        Ss   12:41   0:00 php-fpm: master process (/usr/local/php5.6/etc/php-fpm.conf)
nobody   21681  0.0  0.3 147920  3312 ?        S    12:41   0:00 php-fpm: pool www
nobody   21682  0.0  0.4 147920  4888 ?        S    12:41   0:00 php-fpm: pool www
root     21684  0.0  0.0 112664   980 pts/3    S+   12:42   0:00 grep --color=auto php
```

- 在站点目录下添加 `index.php`
```php
<?php
    phpinfo();
 ?>
```
- 通过浏览器访问该文件，能够正确输出，则说明配置成功。
> 这时 `phpinfo();` 输出的 `Server API` 应该为 `FPM/FastCGI`.

## 各种模式比较
> 有空再整理。。。
