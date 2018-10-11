---
title: Apache 虚拟主机配置
date: 2016-09-05 22:10:00
tags:
- Apache虚拟主机
categories:
- Apache
---
虚拟主机指的是在单一机器上运行多个网站。
* 常见的共有三种不同的配置方式。
  * 基于域名
  * 基于端口（需要增加相对应的 `Listen` 指令）
  * 基于IP
* 动态虚拟主机配置。

<!-- more -->

## 常规配置方式

### 在`httpd.conf` 文件中启用 `httpd-vhost.conf` 配置文件
```ini
# Virtual hosts
Include conf/extra/httpd-vhosts.conf
```

### 配置 `httpd-vhosts.conf`

1. 基于域名配置
```ini
<VirtualHost *:80>
    ServerName domain1.com
    DocumentRoot  "D:\data\domain1.com"
    <Directory />
        Options +Indexes +FollowSymLinks +ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
    DirectoryIndex index.php index.html
</VirtualHost>
```
**注：( `Require all granted` )**
> Apache 2.4 及以上版本访问控制与2.2有所变换，所以在2.4+的版本需要在目录下添加 `Require all granted` 否则会出现访问不到的情况。
> [更多详情，点击查看](http://httpd.apache.org/docs/2.4/upgrading.html)

2. 基于端口配置
  - 基于端口的配置需要先在 `httpd.conf` 中添加相应端口的 `Listen` 指令。
```ini
#httpd.conf
Listen 81
```
  - `httpd-vhosts.conf`
```ini
<VirtualHost *:81>
    #ServerName 10.235.65.14:81
    DocumentRoot  "D:\data\domain3.com"
    <Directory />
        Options +Indexes +FollowSymLinks +ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
    DirectoryIndex index.php index.html
</VirtualHost>
```

3. 基于IP地址配置
```ini
<VirtualHost *:80>
    ServerName 10.235.65.14
    DocumentRoot  "D:\data\domain2.com"
    <Directory />
        Options +Indexes +FollowSymLinks +ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
    DirectoryIndex index.php index.html
</VirtualHost>
```

### 验证
配置完成之后，需要重启 Apache 配置才会生效。在浏览器中输入相应的域名，IP或者端口进行查看，能正常输出我们分别设置的内容，则说明我们配置成功。


## 动态虚拟主机配置
上面谈到的虚拟主机配置方式，每次配置完成之后都必须重启 Apache 才能生效。并且当我们要配置多个虚拟主机时就需要增加多个 `<VirtualHost >···</VirtualHost>` ,显得繁琐。那么有没有一种更好，更有效的方式来进行配置呢？

`mod_vhost_alias` 模块为我们提供了一种动态配置虚拟主机的方式。我们只需设置一个目录，将不同的站点资源按照一定的规则放到该目录下即可。并且不用修改配置文件，且不用重新启动Apache服务器。

动态配置优缺点：
* 优点：
  * 配置文件更小，意味着Apache启动会更快，并且占用内存更少。正重要的是，更小的配置结构更易于维护，我们人为配置出错的机会更小。
  * 添加新的虚拟主机，我们只需要根据相应的规则在指定的，目录下面创建文件即可。不需要重新配置和启动Apache。
* 缺点：
  * 无法针对每个虚拟主机设置不同的日志文件。如果我们有很多的虚拟主机这确实是一个非常糟糕的事情。我们可以选择将日志记录到一个管道或者FIFO，在另一端再将日志文件按照不同的虚拟主机进行分割。比如 [`aplit-logfile`](http://httpd.apache.org/docs/current/programs/split-logfile.html) 工具。

### 配置方法
* 准备
  * 在 `httpd.conf` 中开启 `mod_vhost_alias.so` 模块,并添加一个新的子配置文件 `httpd-vhost-alias.conf` 方便我们配置。
  ```ini
  #开启 mod_vhost_alias.so 模块
  LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
  #去掉 LoadModule 前面的注释
  ···
  #添加子配置文件
  #Include /private/etc/apache2/extra/httpd-vhost-alias.conf
  ```
* 配置
  * 打开子配置文件 `httpd-vhost-alias.conf` ，添加以下配置。
  ```ini
  UseCanonicalName Off
  VirtualDocumentRoot /data1/www/%0
  <Directory "/data1/www/">
    Options None
    AllowOverride None
    Order allow,deny
    Allow from all
  </Directory>
  ```
  保存，并重启 Apache 。
  
* 测试
  * 分别绑定不同的域名到服务器，并在 `/data1/www/` 目录下按照不同域名新建目录，将个站点的资源分别放进相应的目录。
  ```ini
  127.0.0.1 domain1.com
  127.0.0.1 domain2.com
  127.0.0.1 domain3.com
  ```
  ```bash
  /data1/www/domain1.com/index.html   #this is domain1.com test text
  /data1/www/domain2.com/index.html   #this is domain2.com test text
  ```
  * 浏览器分别访问 `domain1.com` 和 `domain2.com ` ，查看是否是否能够正确显示出相应内容。
  * 在 `/data1/www/` 目录下继续添加 `/data1/www/domain3.com/index.html` ,然后继续用浏览器访问 `domain3.com` ，查看是否能将 `/data1/www/domain3.com/index.html` 的内容输出。


* [更多配置参数及方法，请查看](http://httpd.apache.org/docs/current/vhosts/mass.html)
