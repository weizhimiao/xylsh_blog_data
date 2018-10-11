---
title: Nginx服务器https配置
date: 2016-10-04 21:30:00
ctime: 2016-10-04 21:30:00
utime: 2016-10-04 21:30:00
modif_times: 0
tags:
- https
categories:
- Nginx
---

![Nginx服务器https配置](http://n.sinaimg.cn/games/3ece443e/20161004/https.png)

<!-- more -->

## 准备

Linux：Linux version 3.10.0-123.9.3.el7.x86_64
Nginx：nginx/1.6.3
openssl：1.0.1e


## 申请证书
目前网上有不少机构提供个人免费 ssl 证书，有效期几个月到几年不等。以 [StartSSL :https://www.startssl.com](https://www.startssl.com) 为例, 申请成功后有效期 3 年，到期后可免费续租。
具体申请过程也很简单。
注册登录以后选择 Certificates Wizard >> 	DV SSL Certificate 申请一个免费的 ssl 证书。

通过邮件验证域名之后，然后在自己服务器中生成 SSL 证书的 csr ，**记住生成输入的秘密**，之后要用到：
```
openssl req -newkey rsa:2048 -keyout weizhimiao.cn.key -out weizhimiao.cn.csr
```
将生成的证书，放到指定的存放证书的目录，如 `/data/secret/` 。查看证书 `weizhimiao.csr` 内容，将内容复制到页面中的 Certificate Signing Request (CSR)部分，提交页面。

下载生成好的证书,选择对应的web服务器（Nginx，1_weizhimiao.cn_bundle.crt），这样私钥和公钥我们就都有了。
- 1_weizhimiao.cn_bundle.crt（公钥）
- weizhimiao.cn.key（私钥）

## nginx配置（为指定域名增加https）
nginx.conf当前配置
```
...
http {
    ...
    include /etc/nginx/conf.d/*.conf;

    server {
        ...
    }
}
```

./conf.d/weizhimiao.cn.conf中加入
```

server{
    listen 443 ssl;
    server_name weizhimiao.cn;

    ssl_certificate /data/secret/1_weizhimiao.cn_bundle.crt;
    ssl_certificate_key /data/secret/weizhimiao.cn.key;
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    ssl_ciphers 'kEECDH+ECDSA+AES128 kEECDH+ECDSA+AES256 kEECDH+AES128 kEECDH+AES256 kEDH+AES128 kEDH+AES256 DES-CBC3-SHA +SHA !aNULL !eNULL !LOW !MD5 !EXP !DSS !PSK !SRP !kECDH !CAMELLIA !RC4 !SEED';

    add_header Strict-Transport-Security 'max-age=31536000; preload';
    add_header X-Frame-Options DENY;
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;
    keepalive_timeout 70;
    ssl_dhparam /data/secret/dhparam.pem;

    add_header X-Content-Type-Options nosniff;

    add_header X-Xss-Protection 1;

    root /data/www/weizhimiao.cn;
    index index.html;

    location / {

    }
}

```
**注：**
配置中用到一个 `/data/secret/dhparam.pem` 文件，该文件是一个PEM格式的密钥文件，用于TLS会话中。用来加强ssl的安全性。生成该文件方法，
```
cd /data/secret/
openssl dhparam 2048 -out dhparam.pem
```

将原来80端口的访问，重定向。./conf.d/weizhimiao.cn.conf中加入
```
server{
    listen 80;
    server_name  weizhimiao.cn;
    return 301 https://weizhimiao.cn$request_uri;
}

```

## 测试
检测配置文件是否有语法错误，需要输入之前生成公钥时输入的密码。
```
nginx -t
Enter PEM pass phrase:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```
重启Nginx(切记，reload不起作用)
```
nginx -s stop
Enter PEM pass phrase:
nginx
Enter PEM pass phrase:
```
浏览器访问 weizhimiao.cn ,是否生效。

另，Nginx配置了安全证书之后，nginx每次的reload、stop等操作都需要输入密码。
可以通过生成一个解密的key文件，替代原来key文件。
```
cd /data/secret/
openssl rsa -in weizhimiao.cn.key -out weizhimiao.cn.key.unsecure
```
替换 `weizhimiao.cn.conf` 中的 `weizhimiao.cn.key` 文件.
```
server {
  ...
  ssl_certificate /data/secret/1_weizhimiao.cn_bundle.crt;
  ssl_certificate_key /data/secret/weizhimiao.cn.key.unsecure;
  ...
}
```
之后每次在reload时，就不需要在输入密码了。


最后，用 [SSLLABS](https://www.ssllabs.com/ssltest/index.html) 来进行一下测试。
![ssllabs](http://n.sinaimg.cn/games/3ece443e/20161004/ssllabs.png)
结果
![ssllabs](http://n.sinaimg.cn/games/3ece443e/20161004/ssllabsres.png)




over~
