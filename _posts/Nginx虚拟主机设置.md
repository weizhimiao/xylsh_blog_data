---
title: Nginx虚拟主机配置
date: 2016-10-04 20:30:00
ctime: 2016-10-04 20:30:00
utime: 2016-10-04 20:30:00
modif_times: 0
tags:
- 虚拟主机
categories:
- Nginx
---

![virtual Host](http://n.sinaimg.cn/games/3ece443e/20161004/VirtualHost.png)

<!-- more -->

## 准备

Linux：Linux version 3.10.0-123.9.3.el7.x86_64

Nginx: nginx/1.6.3

配置文件目录结构
```
$ tree
.
├── conf.d
├── default.d
├── fastcgi.conf
├── fastcgi.conf.default
├── fastcgi_params
├── fastcgi_params.default
├── koi-utf
├── koi-win
├── mime.types
├── mime.types.default
├── nginx.conf
├── nginx.conf.default
├── scgi_params
├── scgi_params.default
├── uwsgi_params
├── uwsgi_params.default
└── win-utf
```

## 配置
将 site1.cn 和site2.cn基于域名进行配置

### 准备
分别创建两个域名的配置文件和web根目录。

./conf.d/下
```
cd conf.d/
touch site1.cn.conf
touch site2.cn.conf
```
分别创建web根目录
```
mkdri -p /data/www
cd  /data/www
mkdir site1.cn
mkdir site2.cn
```

### 修改主配置文件nginx.conf
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```
确保在http的context中的server部分前面要有
```
include /etc/nginx/conf.d/*.conf;
```
通常情况下主配置文件中的server部分，将会是默认的server（通过添加default_server来声明），当一个请求过来，目录/etc/nginx/conf.d/中的所有server部分都匹配不了，那么这个请求将会被猪配置文件中的server来进行处理.

### site1.cn
修改配置文件
```
vi ./conf.d/site1.cn.conf
```
```
server {
	listen		80;
	server_name	site1.cn;

	error_page  404  /404.html;

	error_page   500 503 504  /50x.html;
	error_log	/var/log/nginx/debug.log debug;
	index	index.html index.htm;
	root /data/www/site1.cn;

	location / {
		index index.html;
	}

  location = /favicon.ico {
  	try_files $uri $uri/favicon.ico /data/www/site1.cn/favicon.ico =404;
  }

  # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store(Mac).
  location ~ /\. {
      deny all;
  }

	location = /50x.html {
		root   /usr/share/nginx/html;
	}

	location = /404.html {
		root   /usr/share/nginx/html;
	}
}
```
添加测试页面
```
cd /data/www/site1.cn
echo "site1.cn index.html" >> index.html
```

### site2.cn
修改配置文件
```
vi ./conf.d/site2.cn.conf
```
```
server {
	listen		80;
	server_name	site2.cn;

	error_page  404  /404.html;

	error_page   500 503 504  /50x.html;
	error_log	/var/log/nginx/debug.log debug;
	index	index.html index.htm;
	root /data/www/site2.cn;

	location / {
		index index.html;
	}

  location = /favicon.ico {
  	try_files $uri $uri/favicon.ico /data/www/site2.cn/favicon.ico =404;
  }

  # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store(Mac).
  location ~ /\. {
      deny all;
  }

	location = /50x.html {
		root   /usr/share/nginx/html;
	}

	location = /404.html {
		root   /usr/share/nginx/html;
	}
}
```
添加测试页面
```
cd /data/www/site2.cn
echo "site2.cn index.html" >> index.html
```

### 重启Nginx
重启之前，需要先进行配置文件语法检测
```
nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
确定语法无问题之后，重启Nginx
```
nginx -s reload
```

### 测试
vi /etc/hosts
添加
```
site1.cn 127.0.0.1
site2.cn 127.0.0.1
```
分别访问 site1.cn和site2.cn ,查看是否输出对应内容
```
wget  site1.cn
cat index.html
#site1.cn index.html

wget site2.cn
cat index.html.2
#site2.cn index.html
```

关于nginx学习的一个网站:http://nglua.com

over~
