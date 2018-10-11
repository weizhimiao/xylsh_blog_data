---
title: .htaccess 文件功能及配置介绍
date: 2016-09-05 10:00:00
tags:
- Apache
- .htaccess
categories:
- Apache
---

URL重定向是`.htaccess` 的重头戏，它可以将长地址转为短地址、将动态地址转为静态地址、重定向丢失的页面、防止盗链、实现自动语言转换等等。

**难点：** 正则表达式的运用和理解。

<!-- more -->
## 准备 mod_rewrite
要实现上述功能，首先得装上mod_rewrite 模块，并确保启用了该模块。

一般我们会这样设置。
```
<IfModule mod_rewrite.c>
  Options +FollowSymlinks
  RewriteEngine on
  ...

<IfModule>
```
***Tips:***
* FollowSymlinks:必须启用，这是rewrite引擎的安全要求。
* RewriteEngine：用于启用rewrite引擎。

## URL重写及重定向
利用`.htaccess` 文件可以实现对URL的重写（rewrite）与重定向（redirect）
1. 将`.html` 映射到`.php`
```
  Options +FollowSymlinks
  RewriteEngine on
  RewriteRule ^(.*)\.html$ $1.php [NC]
```
***注***
  - 必须保证服务器上有对应的`.php` 文件，否则会报 404
  - 浏览器和搜索引擎可同事用`.html` 和`.php` 访问到网页。
  - [NC] 表示不区分大小写

2. 临时重定向（R=302）与永久重定向（R=301）
```
RewriteEngine on
RewriteBase /
RewriteRule ^(.*)\.html $1.php [R,NC,L]
```
***注***
  - RewriteBase 定义了重写基准目录。
  > 例如，如虚拟机站点设在 /var/www目录下，删除这行这行将会导致重定向到 http://your-domain/var/www/1.php. 显然这不是我们想要的。如果RewriteBase /base/，那么将会重定向到http://your-domain/base/1.php.

  - 对于重写基准目录，我们还可以将 `$1.php` 变成 `/$1.php` 实现直接变换，这是就可以将 `RewriteBase` 省略。
  - 字母`L`，表示如果能匹配到本条规则，则本条规则是最后一条（Last），忽略之后的规则。

  永久重定向（R=301）
```
RewriteEngine on
RewriteRule ^(.*)$ http://newdomain/$1 [R=301,NC,L]
```

3. 为什么要用重定向
  1. 通过重定向，浏览器知道页面位置发生变化，从而会改变地址栏到新的地址。
  2. 通过重定向，搜索引擎能够发现位置发生变化，从而进行更新。
  3. R=302，R=301都是亲搜索引擎的，是SEO的一个重要手段。
  4. **URL重写用于将页面映射到本站的另一页面，而重写到另一个域名下，则安重定向处理。**

4. 长短地址处理   
> 利用URL重写，方便实现长短地址转换。

  ```
RewriteEngine on
RewriteRule ^grab /public/files/download/download.php
```
若访问 http://your-domain/grab?file=my.zip, 则页面会实际执行该页面 http://your-domain/public/files/download/download.php?file=my.zip
5. 去掉WWW
```
RewriteEngine on
RewriteCond %{HTTP_HOST} ^(.*)$
RewriteRule (.*) http://www\.%1/$1 [R=301,L]
```
***注:***
  - RewriteCond 定义了规则的生效条件。即，一个RewriteRule规则之前可以有一个或者多个RewriteCond指令。
  - 语法：RewriteCond TestString CondPattern [flags]

6. 加上www
```
RewriteEngine on
RewriteCond %{HTTP_HOST} ^(.*)$
RewriteRule (.*) http://www\.%1/$1 [R=301,L]
```

7. 支持多域名访问

  > 如果有些主机不支持多域名，那么`.htaccess` 或许也可以解决。

  ```
#one.com
RewriteCond %{HTTP_HOST} one.com
RewriteCond %{REQUEST_URI} !^/one
RewriteRule ^(.*)$ /one/$1 [L]
#two.com
RewriteCond %{HTTP_HOST} two.com
RewriteCond %{REQUEST_URI} !^/two
RewriteRule ^(.*)$ /two/$1 [L]
```

## 改写查询字符串 QUERY_STRING
所谓查询字符串，就是值URL问号之后的部分。
1. 利用QSA转换 QUERY_STRING
```
RewriteEngine on
RewriteRule /page/(.+) /page.php?page=$1 [QSA]
```
***注：***
 1. 将会把 `/page/123?one=two` 映射到 `/page.php?page=123&one=two`.
 2. 如果没有[QAS] 标志，则会映射到`page.php?page=123`.
 3. 如果没有用到小括号正则表达式，就不需要 QSA
 > 例，将 `/simple/flat/link ` => server-side.php?first-var=flat&second-var=link

    ```
RewriteEngine on
RewriteRule ^/simple/([^/]+)/([^/]+)/? /server-side.php?first-var=$1&second-var=$2 [QSA]
```
2. 利用RewriteCond 改写 QUERY_STRING
```
RewriteEngine On
RewriteCond %{QUERY_STRING} foo=(.*)
RewriteRule ^grab(.*) /page.php?bar=%1
```
  - 该规则将访问请求http://mysite/grab?foo=bar转换为http://mysite/page.php?bar=bar
  - RewriteCond用于捕获查询字符串（QUERY_STRING）中变量foo的值，并存储在%1中
  - QUERY_STRING是Apache定义的“变量=值”向量（数组）

3. RewriteCond与QSA双剑齐发
```
RewriteEngine On
RewriteCond %{QUERY_STRING} foo=(.+)
RewriteRule ^grab/(.*) /%1/index.php?file=$1 [QSA]
```
 - 会把/grab/foobar.zip?level=5&foo=bar 映射到 /bar/index.php?file=foobar.zip&level=5&foo=bar
 - 转换后根目录是bar目录
 - foobar.zip?level=5中的“问号”变成了foobar.zip&level=5中的“与”符号

4. 剥离查询字符串
> 只需在要开始剥离的链接后面加个“问号”，并且不要启用QSA标志，就可剥离查询字符串

  ```
RewriteEngine On
# Whatever QS is
RewriteCond %{QUERY_STRING} .
# I don't want it with Question mark
RewriteRule foo.php(.*) /foo.php? [L]
```



## 访问控制
1. 文件访问控制
> 利用Order、Files及FilesMatch命令实现的访问控制可以满足大部分要求，但是当用户被拒绝时，他们看到的是硕大的“403 Forbidden”，如果你不想伤害用户的感情，就需要显示一些别的东西，通过Rewrite就可以实现这个特性：

  ```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !^(.+)\.css$
RewriteCond %{REQUEST_FILENAME} !^(.+)\.js$
RewriteCond %{REQUEST_FILENAME} !special.zip$
RewriteRule ^(.+)$ /chat/ [NC]
```
 - 该规则将仅允许用户请求.css, .js类型的文件，还有special.zip文件
 - RewriteRule 后面指定了限制规则：映射到/char/目录下处理
 - RewriteCond 后面的“感叹号”(!)起到了“否定”作用，它表明，对不满足后面正则表达式者应用RewriteRule规则，也就是对当前类型的文件将不应用规则
 - RewriteCond 之间是以逻辑“与”连接的，也就是只有当三个条件都不满足时才执行RewriteRule
 - 该规则也会限制访问.htm, .jpg等格式
 - 该规则不可以放在虚拟站点根目录（/）下，否则会死循环
 - 如果是二级目录，如/test/，那么传入RewriteCond的参数是以/test/开始的，因此从(.+)获得的文件名也含有/test/，读者必须对此多加小心
 - 要想仅获得文件名，可以将(.+)替换成([^/]+)，并且去掉符号^，如下所示：
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !([^/]+)\.css$
RewriteCond %{REQUEST_FILENAME} !([^/]+)\.js$
RewriteRule ^(.+)$ /chat/ [NC]
```

2. 用.htaccess阻止User-agent
> User-agent用于浏览器向服务器“自报家门”，更确切的说是所有HTTP客户端都得用User-agent向服务器“自报家门”，以便服务器对不同的客户端作出不同响应。比如，某站点可能需要对浏览器、搜索引擎crawl还有各类下载工具作出不同的响应。服务器就是通过所谓的User-agent进行区分的。
如果你的服务器提供某些资源的下载，那么你就必须多加小心诸如“迅雷”等下载软件，因为它们可能把你网站资源吸干，并且影响你的正常访客访问。

  为此，我们可以利用Rewrite限制某些UA的访问：
  ```
RewriteEngine on
RewriteCond %{HTTP_USER_AGENT} 2.0.50727 [NC]
RewriteRule . abuse.txt [L]
```
 - 该规则限制“迅雷”客户端下载资源，并将下载文件重置到abuse.txt
 - HTTP_USER_AGENT是Apache的内置变量
 - 2.0.50727是迅雷User-agent的特征字符串
 - RewriteRule后面的“点”表示“任意URI”，也就是不管请求的是什么，都输出abuse.txt

  通常，我们不会仅限制一个UA。利用[OR]即可实现对多个UA作出统一处理：
  ```
RewriteEngine on
RewriteCond %{HTTP_USER_AGENT} 2.0.50727 [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^BlackWidow [NC,OR]
# etc..
RewriteCond %{HTTP_USER_AGENT} ^Net\ Vampire [NC]
RewriteRule . abuse.txt [L]
```

3. 用.htaccess阻止盗链(hot-linking)
> 盗链，特别是图片，是非常可耻的！哪怕将图片复制到自己服务器上，也比盗用他人的图片链接来得光彩！

  .htaccess的Rewrite功能可以提供非常简单、有效的方法阻止这种可耻行为：
  ```
RewriteEngine On
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^http://(www\.)?lesca\.me/ [NC]
RewriteCond %{REQUEST_URI} !hotlink\.png [NC]
RewriteRule .*\.(gif|jpg|png)$ /hotlink.png [NC]
```
简单解释一下该规则的功能：
 - 除本站以外其他网站都不得引用本站图片，具体可以理解为
 - 如果引用站点为“空”或者是“本站”，或者，所引用对象是“hotlink.png”，那么就允许访问
 - 再次提醒，RewriteCond之间默认的逻辑连接词是逻辑“与”
 - 这里的难点是理解逻辑转换，即德·摩根定律


## htaccess 缺点
说了这么多`.htaccess` 可以实现这么多的功能，那么它有没有什么缺点呢?

答案是，当然有。由于开启了`.htaccess` 文件，当我们访问了一个地址对应到服务器端某个目录之后`Apache` 会遍历并解析每层目录下的`.htaccess` 文件，所有对服务器性能会有一定的影响。

但是具体影响有多大，要不要使用还需要看具体的环境。或者需要进行前后对比在进行选择。

## htaccess 生成工具
说了这么多，貌似真正配置起来还是非常麻烦。不用着急目前网上有非常多`htaccess` 在线生成工具，可大大方便我们进行完成各种配置。

如：[smarty .htaccess :http://htaccess.uuz.cc/](http://htaccess.uuz.cc/)

[转自:Lesca技术宅](http://lesca.me/archives/htaccess-rewrite.html)
