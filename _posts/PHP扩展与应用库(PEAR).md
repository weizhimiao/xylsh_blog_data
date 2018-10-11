---
title: PHP扩展与应用库(PEAR)
date: 2016-10-12 20:30:00
tags:
- PEAR
categories:
- PHP
---

PEAR（the PHP Extension and Application Repository），PHP扩展与应用库。它是一个PHP扩展及应用的一个代码仓库。所有的扩展均以PHP代码的形式出现，功能强大，安装简单，甚至可以改改就用。使用的时候，要在代码中进行Include才能够使用。

官网：

http://pear.php.net

<!-- more -->

## PEAR安装和使用

在官网上有说明详细的安装信息，这里作简单说明。
http://pear.php.net/manual/en/about-pear.php

### 下载
#curl -o go-pear.php  http://pear.php.net/go-pear
```
$ curl -o go-pear.php  http://pear.php.net/go-pear
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 88959  100 88959    0     0  43329      0  0:00:02  0:00:02 --:--:-- 43352

```

### 运行go-pear.php
```
# /usr/local/php5/bin/php go-pear.php
```

### 利用PEAR安装PHPDOC
```
$ pear install phpdoc/phpDocumentor-alpha
Attempting to discover channel "phpdoc"...
Attempting fallback to https instead of http on channel "phpdoc"...
unknown channel "phpdoc" in "phpdoc/phpDocumentor-alpha"
invalid package name/package file "phpdoc/phpDocumentor-alpha"
install failed
```
如出现上面错误，需要我们增加一个phpDocumentor pear渠道
```
$ pear channel-discover pear.phpdoc.org
Adding Channel "pear.phpdoc.org" succeeded
Discovery of channel "pear.phpdoc.org" succeeded
```
重新安装
```
$ pear install phpdoc/phpDocumentor-alpha
downloading phpDocumentor-2.8.5.tgz ...
Starting to download phpDocumentor-2.8.5.tgz (8,184,822 bytes)
.................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................done: 8,184,822 bytes
install ok: channel://pear.phpdoc.org/phpDocumentor-2.8.5
```
查看是否安装成功
```
$ phpdoc -V
phpDocumentor version 2.8.5
```


## PEAR常用功能及命令
```
$ phpdoc --help
Usage:
 project:run [-t|--target[="..."]] [--cache-folder[="..."]] [-f|--filename[="..."]] [-d|--directory[="..."]] [--encoding[="..."]] [-e|--extensions[="..."]] [-i|--ignore[="..."]] [--ignore-tags[="..."]] [--hidden] [--ignore-symlinks] [-m|--markers[="..."]] [--title[="..."]] [--force] [--validate] [--visibility[="..."]] [--defaultpackagename[="..."]] [--sourcecode] [-p|--progressbar] [--template[="..."]] [--parseprivate] [--log[="..."]]

Aliases: run
Options:
 --target (-t)         模板文件生成路径
 --cache-folder        缓存文件路径
 --filename (-f)       要解析的文件的逗号分隔列表。 通配符？ 和*（支持多个值）
 --directory (-d)      逗号分隔的目录列表（递归）解析（允许多个值）
 --encoding            编码用于解释源文件
 --extensions (-e)     以逗号分隔的解析扩展列表，默认为php，php3和phtml（允许多个值）
 --ignore (-i)         以逗号分隔的将被忽略的文件和目录（相对于源代码目录）列表。 通配符*和？ 支持（允许多个值）
 --ignore-tags         将忽略的逗号分隔的标签列表，默认为none。 package，subpackage和ignore不能被忽略。 （允许多个值）
 --hidden              使用此选项可以告诉phpDocumentor解析以句点（。）开头的文件和目录，默认情况下这些被忽略
 --ignore-symlinks     忽略到其他文件或目录的符号链接，默认值为on
 --markers (-m)        要过滤的标记/标记的逗号分隔列表（允许多个值）
 --title               设置此项目的标题; 默认是phpDocumentor标志
 --force               强制完整构建文档，不会增加现有文档
 --validate            使用PHP Lint验证每个处理的文件，成本很高的性能
 --visibility          指定应在文档中显示的解析可见性（逗号分隔，例如“public，protected”）（允许多个值）
 --defaultpackagename  用于默认软件包的名称。(default: "Default")
 --sourcecode          是否包含语法高亮的源代码
 --progressbar (-p)    是否显示进度条; 将自动静默记录到stdout
 --template            要使用的模板的名称（可选）（允许多个值）
 --parseprivate        是否解析标记有@internal标签的DocBlocks
 --log                 要写入的日志文件
 --help (-h)           显示此帮助消息
 --quiet (-q)          不输出任何消息
 --verbose (-v|vv|vvv) 增加消息的详细程度：1用于正常输出，2用于更详细的输出，3用于调试
 --version (-V)        显示此应用程序版本
 --ansi                强制ANSI输出
 --no-ansi             禁用ANSI输出
 --no-interaction (-n) 不要问任何互动问题
 --config (-c)         自定义配置文件的位置

Help:
  phpDocumentor从PHP源文件创建文档。 最简单的方法
  使用它是：

     $ phpdoc run -d [directory to parse] -t [output directory]

 这将解析在<directory to parse>中以.php，.php3和.phtml结尾的每个文件，然后在<output directory>中输出一个包含易于阅读的文档的HTML网站。

 phpDocumentor will try to look for a phpdoc.dist.xml or phpdoc.xml file in your
 current working directory and use that to override the default settings if
 present. In the configuration file can you specify the same settings (and
 more) as the command line provides.

phpDocumentor将尝试在当前工作目录中查找phpdoc.dist.xml或phpdoc.xml文件，
并使用该文件覆盖默认设置（如果存在）。 在配置文件中，您可以指定与命令行提供的相同的设置（和更多）。



 Other commands
 除了这个命令phpDocumentor还支持附加命令：

 Available commands:
   help
   list
   parse
   run
   transform
 project
   project:parse
   project:run
   project:transform
 template
   template:generate
   template:list
   template:package

您可以使用list命令获取更详细的命令列表，并通过在命令名前添加help来获取帮助。
```


## 常用的PEAR模块简介



- Benchmark/Timer 测试你的一段php代码的运行效率  

- Benchmark/Benchmark_Iterate 测试你某个函数循环执行时的性能  

- Cache/Output 可以将你的php脚本的输出进行缓存，可以使用多种方式缓存（存在文件，数据库或者是共享内存中）,如果使用这个模块有可能增大服务器的负载，所以，如果你想通过动态脚本的缓存来提供效率，不妨使用Zend optimize,这个模块未必适合  

- Cache/Graphics 可以将你需要动态输出的图片进行缓存  

- Console/Getopt 命令行参数的处理模块  

- CMD 一个虚拟的shell，可以用它来运行一些系统的命令  

- Crypt/CBC 实现Perl Crypt::CBC 模块的仿真  

- Crypt/HCEMD5 实现Perl Crypt::HCE_MD5 模块的功能  

- Date/Calc 实现日期的相关操作  

- Date/Human Human历法的转换  

- DB 提供统一的、抽象的数据库操作层，后端支持多种数据库  

- File/Find 文件查找  

- File/Passwd 操纵password类的文件，如password,httppass,cvspassword  

- File/SearchReplace 在文件中查找替换字符串  

- HTML/Form 可以在html中快速地创建form  

- HTML/IT 实现模板定制，动态生成页面的功能，类似phplib中的模板功能，但是要简单易用  

- HTML/ITX 实现对IT的扩展功能，可以更加灵活地定制你的模板，实现更复杂的操作  

- HTML/Processor XML_Parser的扩展，使之可以应用于html文件的操作  

- HTTP/Compress 用于Php 输出缓冲机制的一个包装类，同时可以对缓冲的内容进行压缩存储  

- Image/Remote 无需把整个图片都下载到本地就可以获取远端系统的图片的信息，  

- Log/composite Horde对log抽象类做的一个扩展，可以使多个日志处理对象能够获得同一个日志事件。注意，Log目录下面的模块都是Horde项目的一部分，大部分都是抽象的超类  

- Log/file 将日志信息写入文件  

- Log/mcal 将信息发送到本地或远端的日程管理软件-mcal的数据库中  

- Log/observer Horder中Observer的一个超类  

- Log/sql 将日志信息发送到sql数据库中  

- Log/syslog 将信息发送到syslog中  

- Mail/RFC822 检查一个email地址是否是合法的rf822 email地址  

- Mail/sendmail 使用sendmail来发送信件  

- Mail/smtp 使用smtp服务器来发送信件  

- Math/Fraction 处理分形的数学计算  

- Math/Util 计算最大公约数  

- NET/Curl 对php的Curl扩展所作的面向对象的包装  

- NET/Dig 操纵dig，进行dns相关的查询操作  

- NET/SMTP 使用NET/Socket实现SMTP协议  

- NET/Socket 通用的Socket类，实现了常用的socket操作的包装  

- Numbers/Roman 阿拉伯数字和罗马数字的相互转换  

- Payment/Verisign 实现和Verisign支付网关的交互  

- Pear 提供Pear模块的2个基本类，PEAR 和PEARError类  

- PEAR/Installer pear的安装类，提供Perl中的CPAN模块类似的功能  

- PHPDoc 从php代码中自动生成API文档  

- Schedule/at 和Unix 上的AT守护进程进行交互  

- XML/Parser 基于php的xml扩展所作的xml的解析器  

- XML/Render 将xml文档生成其它的格式（html,pdf),这只是一个抽象类，在最新的pear cvs代码中已经有了html的实现  

- XML/RPC 用php实现xml-rpc的一个抽象类，在最新的pear cvs代码中已经有了xml/RPC/Server的实现  


## PEAR与PECL关系


PEAR（the PHP Extension and Application Repository），PHP扩展与应用库。它是一个PHP扩展及应用的一个代码仓库。

PECL（PHP Extension Community Library），PHP的扩展库。它提供了一系列已知的扩展库，由C、C++等其他语言编写而成，以.so形式出现，.so 为共享库,是shared object,用于动态连接的,和dll差不多，为比PEAR更快，但是与PEAR不同的是，PECL需要在服务器上配置并被注册到主机中。

基于他们的实现方式不同，在使用时候也有不同。

Pear：是PHP的扩展代码包，所有的扩展均以PHP代码的形式出现，功能强大，安装简单，甚至可以改改就用。使用的时候，要在代码中进行Include才能够使用。

Pecl：是PHP的标准扩展，可以补充实际开发中所需的功能，所有的扩展都需要安装，在Windows下面以Dll的形式出现，在linux下面，需要单独进行编译，它的表现形式为根据PHP官方的标准用C语言写成，尽管源码开放但是一般人无法随意更改源码。

**最直接的表述：Pear是PHP的上层扩展，Pecl是PHP的底层扩展。**
