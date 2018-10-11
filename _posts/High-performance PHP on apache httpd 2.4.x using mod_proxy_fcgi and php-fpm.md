---
title: httpd 2.4.x上使用mod_proxy_fcgi和php-fpm实现高性能PHP
date: 2016-10-20 22:40:00
tags:
- PHP-FPM
- mod_proxy_fcgi
categories:
- PHP
---


内容
- apache httpd 2.4.x上使用mod_proxy_fcgi和php-fpm实现高性能PHP
  - php-fpm
  - apache httpd 2.4
    - TCP套接字（IP和端口）方法
    - unix域套接字（UDS）方法
    - 通过代理程序处理
    - 先睹为快
      - 一个简单的例子
      - 一个更灵活的例子
    - 性能和陷阱
    - 警告



<!-- more -->

apache httpd 2.4.x上使用mod_proxy_fcgi和php-fpm实现高性能PHP

随着apache httpd 2.4的发布，我们已经获得了一些非常整洁的功能关于apache和php：运行PHP作为一个fastCGI进程服务器的能力，并且这个fastCGI服务器可以直接从apache中通过一个专用的模块代理来实现 （mod_proxy_fcgi.）

> 从2010年初的版本5.3.3开始，PHP已经将php-fpm fastCGI进程管理器合并到其代码库中，现在（从5.4.1开始）相当稳定。
> php-fpm ， http://php-fpm.org/

这意味着我们现在可以运行安全，快速和可靠的PHP代码，只给apache httpd和php.net版本使用; 没有更多的混乱像suphp、suexec 或者 mod_php。


## php-fpm
准备：

- 安装软件包

- 编辑配置文件

- 控制服务守护程序。

从5.3.3版本开始，PHP现在在源代码中包括fastCGI进程管理器（php-fpm）。
您的发行版或操作系统将其包含在库存PHP包中，或将其作为附加包提供;
我们可以通过向./configure选项添加“--enable-fpm”从源代码构建它。

这为我们提供了一个新的二进制文件，称为php-fpm，一个名为php-fpm.conf的默认配置文件安装在/ etc中。

此文件中的默认值是可以启动的，但请注意，你在本地安装的时候可能会有修改，其位置也可能会变。

在这个配置文件中，您可以创建任意数量的fastcgi“池”，这些池由它们侦听的IP和端口定义，就像apache虚拟主机一样。

每个池中最重要的设置是TCP套接字（IP和端口）或unix域套接字（UDS）php-fpm将监听接收fastCGI请求; 这是使用listen选项配置的。

默认池[www]，配置为listen 127.0.0.1:9000：它将只响应本地环回网络接口（localhost）上的请求，在TCP端口9000上。

另外，有趣的是 per-pool 的user和group选项，他们允许在指定的uid和gid下运行特定的fpm池。再见 suphp！

让我们使用默认值作为运行并启动php-fpm守护进程; 如果您的发行版使用提供的init脚本，请运行

```
/etc/init.d/php-fpm start
```

如果没有，请手动启动
```
php-fpm -y /path/to/php-fpm.conf -c /path/to/custom/php.ini
```


如果你不提供php-fpm自己的php.ini文件，将使用全局php.ini。记住这一点，当你想要包括更多或更少的扩展比如CLI或CGI二进制使用，或需要改变一些其他值。

你可以使用php [admin]（flag | value）以同样的方式包含每个池的php.ini值，方法与以前在apache中为mod_php定义的一样。


有关所有可能的配置选项，请[参阅fpm的官方PHP文档](http://www.php.net/manual/en/install.fpm.configuration.php)。

更改php-fpm.conf的loging选项，方便我们查看错误信息
```
error_log /var/log/php-fpm.log
```

如果不设置php-fpm日志文件，将按照php.ini中定义的方式记录错误。

> note:我们可以强制 php-fpm 重新加载它的配置文件，通过想php-fpm发送 SIGUSR2 信号。SIGUSR1将循环日志文件（完美的logrotate脚本！）一些实验很漫长

注意：如果php-fpm在启动的时候没有出现报错，那么它就已经在监听端口等待连接了。

## apache httpd 2.4

准备：

- 编辑httpd.conf

- 理解vhost上下文

- 理解URL到文件系统命名空间映射

- 控制apache httpd守护进程


这个版本的apache httpd引入了两个值得注意的特性：一个新的代理模块专门用于fastCGI（mod_proxy_fcgi），并将 event MPM作为默认的apache进程管理器。



与以前版本的worker MPM一样，当使用非线程安全的第三方PHP扩展时，此MPM的线程模型会导致问题。



自从apache 2.2发布以来，这已经成为mod_php用户的一个祸根，实际上迫使他们将fastcgi解决方案拼凑在一起，或者使用更慢和内存饥饿的prefork MPM。


要使用PHP fastCGI进程管理器工作，我们将使用一个新模块mod_proxy_fcgi，该模块专用于与（可能是外部）fastCGI服务器通信。

确保您在httpd.conf中包含proxy_fcgi模块，以便我们可以使用其功能; 因为这需要基本代理模块，请确保两者都加载（取消注释）：

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

现在，有不同的方法实际转发请求的.php文件到这个模块，从转发所有的请求（使用ProxyPass），到只转发非常特定，或者通过重写文件或模式（使用带有[P]标志的mod_rewrite）的方法。


我们选择介于复杂性和灵活性之间的方法（使用ProxyPassMatch），因为它允许您为特定vhost的所有PHP内容设置一个规则，但只会代理.php文件（或包含.php的urls）。


### TCP套接字（IP和端口）方法

编辑所选主机的配置，并向其中添加以下行：
```
ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/path/to/your/documentroot/$1
DirectoryIndex /index.php index.php
```
说明：
ProxyPassMatch
> 只有与指定的正则表达式模式匹配的代理内容; 在这种情况下：

^/(.\*\.php(/.\*)?)$
> 从文档根开始，匹配以.php结尾的所有内容（使用点转义），可选地后跟一个斜杠和您喜欢的任何继续路径（一些应用程序使用这个所谓的PathInfo将参数传递给php脚本）。

> ^和$符号用于锚定URL的绝对开始和结束，以确保请求中的任何字符都不会转义我们的模式匹配。

> 嵌套括号使我们能够将整个请求URI（减去前导斜杠）引用为$ 1，同时仍然保持尾随pathinfo可选。


fcgi://127.0.0.1:9000
> 通过mod_proxy_fcgi，使用fastCGI协议，转发到我们的php-fpm守护程序正在侦听的端口。
> 这确定哪个fastcgi池将服务由此规则代理的请求。

/path/to/your/documentroot/

> **重要！**  这必须与您的php文件的真实文件系统位置完全匹配，因为这是php-fpm守护程序将查找它们的位置。
> php-fpm只是解释传递给它的php文件; 它不是一个Web服务器，也不了解您的Web服务器的命名空间，虚拟主机布局或别名。
>
> **重要！**  请再看一遍以上内容

$1
> 从原始请求扩展到整个请求URI，减去前导斜杠（因为我们已经添加了上面的。）

DirectoryIndex /index.php index.php

> 对根目录/的请求，需要用默认索引文件映射到fcgi上。

> 没有解决这个问题可能导致一个空白响应，通常被称为WSOD（死亡白屏），特别是如果仅代理包含php扩展名的请求URI，如本示例。

> 处理流程将首先将针对/的请求映射到/index.php或相对于当前请求uri的任何其他index.php文件，然后正确地代理到PHP-FPM后端。

### unix域套接字（UDS）方法

编辑所选主机的配置，并向其中添加以下行：

```
ProxyPassMatch ^/(.*\.php(/.*)?)$ unix:/path/to/socket.sock|fcgi://127.0.0.1:9000/path/to/your/documentroot/
```


unix:/path/to/socket.sock
> 您的fpm套接字的路径
>
> **请注意**，使用此方法，捕获的请求URI（$ 1）不会在路径之后传递

### Proxy via handler(通过代理程序处理)

使用这种方法，您可以在代理到php-fpm后端之前检查资源的存在。
```
＃定义工作器将提高性能
＃在这种情况下，重新使用worker（依赖于fcgi应用程序的支持）
＃如果你有足够的空闲工作，这只会略微提高性能
<Proxy "fcgi://localhost:9000/" enablereuse=on max=10>
</Proxy>
<FilesMatch "\.php$">
    # 选择以下方法之一
    # 1、使用标准的TCP套接字
    # SetHandler "proxy:fcgi://localhost:9000"
    # 2、如果您的版本的httpd是2.4.9或更新版本（或具有后端功能），您可以使用unix域套接字
    # SetHandler "proxy:unix:/path/to/app.sock|fcgi://localhost:9000"
</FilesMatch>
```

### For the impatient

#### Very simple example

首先, 创建一个文件/var/www/info.php 内容如下:
```
<?php phpinfo() ?>
```
假设/ var / www是现有vhost的DocumentRoot。

在此vhost内，添加以下行：
```
ProxyPassMatch ^/info$ fcgi://127.0.0.1:9000/var/www/info.php
```

使用apachectl优雅重新加载apache，您现在可以使用http://example.com/info调用phpinfo页面

这是一个非常简单的示例，将一个唯一的URL映射到单个PHP文件。

#### A more flexible example

要使用其真实的php文件位置将vhost中的所有.php文件代理到fcgi服务器，您可以使用更灵活的匹配：
```
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/var/www/$1
```
同样，假设/ var / www是所讨论的vhost的DocumentRoot.

Reload apache with apachectl graceful and you can now call up the phpinfo page using http://example.com/yourscript.php
使用apachectl优雅重新加载apache，您现在可以使用http://example.com/yourscript.php调用phpinfo页面

### 性能和陷阱

mod_proxy_fcgi现在支持unix域套接字自2.4.9（ [Unix域套接字支持mod_proxy_fcgi](https://issues.apache.org/bugzilla/show_bug.cgi?id=54101））

这是很容易占满你的系统的可用套接字，通过ulimits等等。一些提示，以避免这一点：


使用太多的套接字将导致apache给出一个（（(99)Cannot assign requested address:）的错误。 这意味着您的操作系统不允许创建新的套接字。


在linux上，可以使用/ proc / sys / net / ipv4 / tcp_tw_reuse 建立尽可能多的套接字，但是在NAT之后将会出现很多有关使用这些套接字的警告。

确保修改ulimit并允许apache用户和php-fpm用户都有足够的打开文件和进程。 ulimit -n 和 ulimit -u（nofile，最大文件打开数＆nproc，最大进程数）


如果php-fpm没有足够大的nproc（最大进程数），它将退出（代码255，没有php 5.3的附加信息），没有附加消息。


如果php-fpm没有足够大的nofile（最大文件打开数），你可能无法获得每个子进程的日志记录，如上所示。 它会在一般的错误日志中给出。


如果apache和php-fpm作为同一用户运行（不必要或不推荐），且nproc太小，apache将无法启动，并显示以下消息（11）Resource temporarily unavailable：AH02162：setuid: unable to change to uid: 600


**警告：** 当ProxyPass向另一个服务器（在这种情况下，php-fpm守护程序）的请求，身份验证限制和放置在目录块或.htaccess文件中的其他配置可能被绕过。


### Caveats（警告）


有人可能会指出，贪婪的ProxyPassMatch伪指令可能允许由HTTP客户端上传的某些恶意内容。


这不是一个全面的安全文件，而是将指出一个可能的注入向量，可以从本文档中的指令生成。

例如，
```
/uploads/malicious.jpg/lalalaalala.php
```

将导致php-fpm处理该文件（/uploads/malicious.jpg），并且没有某些健全性检查，可能导致被攻击的服务器。

这当然不推荐。 使用php上传的内容应该安全地保存在DocumentRoot之外，并且应该仔细检查pathinfo。

此外，php-fpm应检查是否允许调用脚本。

如果这样的限制不能容易地实现，则可以在用RewriteCond或FallbackResource在代理之前执行检查，以确保URI不被HTTP客户端改变。


[【原文】](http://wiki.apache.org/httpd/PHP-FPM)

over~
