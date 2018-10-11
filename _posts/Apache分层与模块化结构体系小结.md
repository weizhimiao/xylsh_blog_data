---
title: Apache分层与模块化体系结构小结
date: 2016-08-26 10:10:00
tags:
- Apache分层
- Apache模块
categories:
- Apache
---

## Apache的分层体系结构
最新版本的Apache按照其功能一般会被划分为五层，
1. 操作系统平台功能层
> 各操作系统本身提供的底层功能，比如进程、线程管理,进程和线程之间通信、网络套接字通信、文件操作等

2. 可移植运行库层（操作系统适配层）
> 封装不同操作系统的底层细节，向上提供统一的接口。

3. Apache核心功能层
> 提供最基本的HTTP服务功能，对其他模块提供对应的API。

4. Apache可选功能层
> 这一层通常指 Apache模块，Apache核心功能层提供不了的功能交给不同的Apache模块来处理。

5. Apache第三方功能层
> Apache的一些模块中用到的一些第三方开发的类库等

![Apache的分层体系结构](http://n.sinaimg.cn/games/3ece443e/20160826/apacheFenCengYuMoKuaiHuaTiXiJieGou.png)

<!-- more -->

**Apache源码目录结构简介**

```
build/    #
docs/     #包含一些相关文档
include/  #包含一些必须的头文件
modules/  #包含Apache的各种模块
os/       #各种操作系统的依赖文件
server/   #Apache核心功能（请求处理、协议处理、多处理模块mpm等）
srclib/   #Apache开发和运行需要的基础库（Apr_util,apr,pcre）
support/  #一些辅助工具等
test/     #APR的测试函数
```

### 操作系统平台功能层
Apache实质上还是运行在操作系统上面的应用程序，因此必须使用操作系统本身提供的底层功能，比如进程和线程、进程和线程的通信，网络套接字通信和文件操作等。

### 可移植运行库（操作系统适配层，APR）
APR(Apache portable runtime) 是操作系统的适配层，通过APR也实现了Apache的跨平台。因为不同的操作系统提供的底层API不同，也就是实现同一个操作所用的函数方法不同，这时在Apache和操作系统中间设计一个APR，这样APR根据不同的操作系统分别实现一个相同的功能，这样apache可以调用APR的提供的一个API接口。


例如，如果Apache要创建一个进程，这时会调用 APR中的 apr_proc_create()函数，此时APR会自动识别操作系统的类型根据不同的类型调用操作系统通过的API，如是Unix系列则会调用unix中的fork()方法实现创建进程；如果是windows系统，则调用createProcess()创建进程。

所以，Apache在处理与操作系统有关的事物时，不用考虑是基于哪一个操作系统，直接用APR的统一API接口就可，具体的由APR来实现跨操作系统。

实际上任何应用程序都可以借助APR进行跨平台。

### Apache核心功能层
1. 核心功能层主要实现Apache的基本功能和核心功能，包括读取和响应HTTP请求，处理HTTP协议；核心功能层包括核心程序和核心模块

**核心程序** 主要是实现Apache的基本功能：
* 启动和终止apache
* 处理配置文件(config.c)
* 接受和处理HTTP连接
* 读取HTTP请求并对该请求进行处理
* 处理HTTP协议

核心功能层另一个是 **核心模块**

 2. Apache 最基本的核心功能由apache 核心完成，除此之外，核心无法提供的功能则全部由模块提供。为了允许这些模块能完成控制apache的处理，apache核心程序提供了对应的API；这些API是指每个模块中包含的一系列的函数(核心程序处理HTTP请求的时候用来将信息传递给模块)，以及一些列apr的函数。

### Apache可选功能层
Apache有很多模块，包括mod_ssl mod_proxy mod_perl ；apache的文件都是C语言开发的，如果有perl脚本写的模块，必须把mod_perl 模块加载，否则不能运行

### Apache第三方功能库
apahe的一些模块会使用到第三方的开发库，比如 mod_ssl 使用了 openssl；mod_perl 使用了perl 开发库，这些库并不属于apache，是第三方库。


## Apache模块化体系结构
Apache体系结构的模块化特点，主要体现在第三层（核心功能层）与第四层（可选功能层）。Apache采用模块化体系结构，使它作为一个HTTP服务器的大部分功能都被分割为相互独立的模块，使我们能够通过增加或者删除模块就可以扩展和修改Apache的功能。

### 核心模块&可选模块
Apache中大部分模块都是可选择的，这意味着这些模块的缺失至多影响Apache的功能完整性，而不影响起运行。但有两个模块是必须的，mod_core和mod_so。

#### 核心模块

* **mod_core:** 负责处理配置文件中的大部分配置指令，并根据这些指令运行Apache。
* **mod_so:** 负责动态加载其余的模块。没有该模块，其余的模块就无法被加载使用。
* **MPM模块** 即，多进程处理模块，虽然该模块是一个可选模块，但一般情况下，我们都会使用，所以我们将其也视为核心模块。
> 前两个模块我们必须静态编译。mpm模块的话，当我们在编译Apache是已经确定使用某一MPM模块之后通过也可将其采用静态编译的方式进行编译。但是如果我们想要在Apache安装之后动态修改MPM模式的话，那么在Apache编译安装的时候，MPM模块就需要通过动态编译的方式进行编译安装。


#### 非核心模块
一些常见的可选模块：
* mod_alias
> Provides for mapping different parts of the host filesystem in the document tree and for URL redirection
> 为不同的url地址映射到文件系统的指定位置，即『起别名』

* mod_autoindex
> 用于生成目录索引

* mod_cache
> RFC 2616标准的HTTP缓存的过滤器。是apache中基于URI键的内容动态缓冲(内存或磁盘)。
> 从Apache2.2起，mod_cache和mod_file_cache将不再是试验模块，它们已经足够稳定，可以用于实际生产中了。这些缓冲体系提供了一个强有力的途径来加速原始web服务器(origin webserver)和代理服务器(proxy)的HTTP处理速度。

* mod_cgi
>  执行cgi脚本

* mod_dir
> 目录的索引可以来自两个来源：
> * 由用户编写的文件，通常被称为 index.html。该DirectoryIndex指令设置该文件的名称。这是由控制 mod_dir。
> * 否则，由服务器生成的列表。这是通过提供mod_autoindex。
>
> 这两种功能是分开的，这样如果你想你可以完全删除（或更换）自动索引生成。
>
> 当服务器收到一个URL请求的“斜线”重定向发出 http://servername/foo/dirname那里 dirname是一个目录。目录需要一个结尾斜杠，所以mod_dir发出一个重定向 http://servername/foo/dirname/。

* mod_filter
> 该模块实现输出内容过滤器的智能，上下文相关的配置。例如，Apache可以被配置为处理不同的内容类型通过不同的过滤器，即使当内容类型是不是预先已知的（例如，在一个代理）。

* mod_include
> 服务器端包含

* mod_isapi
> ISAPI Extensions within Apache for Windows
> 本模块实现了互联网服务扩展应用程序编程接口(Internet Server extension API)。本模块使得Windows上的Apache能有限地实现互联网服务扩展(比如调用ISAPI的动态连接库)。

* mod_mime
> 关联请求的文件名的扩展名与文件的行为（处理器和过滤器）和内容（MIME类型，语言，字符集和编码)

* mod_mime_magic
> 通过读取部分文件内容自动猜测文件的MIME类型
> 本模块采取Unix系统下file(1)命令相同的方法：检查文件开始的几个字节，来判定文件的MIME类型。它被作为当mod_mime无法解析时，用来处理的"第二道防线"。

* mod_proxy
> 用于提供代理服务，能够支持的包括正向代理、反向代理、透明代理、缓存、负载均衡，HTTP代理、FTP代理、SSL代理等若干强大的功能.

* mod_rewrite
> 提供了一个基于规则的重写动态URL重写引擎。

* mod_session
> 会话支持

* mod_ssl
> 提供使用安全套接字层（SSL）和传输层安全（TLS）协议强加密。（https协议必须）

* mod_status
> 提供有关服务器活动和性能信息

* mod_vhost_alias
> 虚拟主机配置支持

### 静态模块&动态模块

#### 概念&区别
**什么是静态？**  其实就是编译的时候所有的模块自己编译进 httpd 这个文件中 ，启动的时候这些模块就已经加载进来了，也就是可以使用了。

查看当前Apache通过静态编译的模块
```
[root@MyServer ~]# httpd -l
Compiled in modules:
core.c
mod_so.c
http_core.c
event.c
```

**那么什么是动态？**  静态是直接编译进httpd中， 那么动态显然就不编译进去了，也就是你启动的时候根本不会加载这个模块， 而是给你一个module.so 文件，你一定要使用 loadmodule 这个语法来加载，这个模块才有效。

***配置方法：***
静态的模块通常在http.conf中用<ifmodule></ifmodule> 来配置，动态的要先loadmoule来加载，然后再<ifmodule></ifmodule>配置。
官方说静态的比动态的在性能方面多5%左右。

***比较：***
相对来说，静态的效率高些，而动态方式配置方面灵活。想想如果编译进去的C这个module你想升级或者去掉，静态方式的就只能重新编译Apache了。

下面这句在Apache源文件夹下运行，可以查看默认情况下Apache都给你装了那些module进去：
```
./configure –help | grep disable
```

####

#### 模块管理

##### 1. 模块的类型：
* 基本(B)模块默认包含，必须明确禁用；
* 扩展(E)/实验(X)模块默认不包含，必须明确启用。

那么，针对以上这些类型的模块，在编译时有以下几种操作方式：

**--disable-MODULE**
禁用MODULE模块(仅用于基本模块)

**--enable-MODULE=shared**
将MODULE编译为DSO(可用于所有模块)

**--enable-MODULE=static**
将MODULE静态连接进核心(仅用于扩展和实验模块)

**--enable-mods-shared=MODULE-LIST**
将MODULE-LIST中的所有模块都编译成DSO(可用于所有模块)

**--enable-modules=MODULE-LIST**
将MODULE-LIST静态连接进核心(可用于所有模块)

***针对--enable-modules和--enable-mods-shared有两个懒办法就是 most参数和all参数，分别表示“很多的”和“所有”。***
**例如：**
```
mod_alias是个基本模块，不想安装的话就： --disable-alias
mod_rewrite是个扩展模块，想动态加载它：--enable-rewrite=shared，想静态加载就是：--enable-rewrite=static
想静态编译mod_alias和mod_rewrite：--enable-modules='alias rewrite'
想动态编译mod_alias和mod_rewrite：--enable-mods-shared='alias rewrite'
```

##### 2. 动态模块管理
**Tips:** 让Apache日后可以动态编译和加载模块：
如果想让Apache日后可以支持动态编译(DSO)更多的module，需要在初次安装时把so这个模块编译到核心（即，静态编译）。
> 如果编译中包含任何DSO模块，则mod_so会被自动包含进核心。如果希望核心以后能够装载DSO，但不实际编译任何DSO模块，则需明确指定：
> * 针对apache1.x: --enable-module=so
> * 针对apache2.x: --enable-so=static

针对Apache2.2.x的一些例子：
```
最大化静态安装Apache:
./configure --prefix=/usr/local/apache --enable-modules=all
最大化动态安装Apache:
./configure --prefix=/usr/local/apache --enable-mods-shared=all
静态安装rewrite、动态安装deflate以及headers
./configure --prefix=/usr/local/apache --enable-rewrite=static --enable-deflate=shared --enable-headers=shared
不安装基本的alais，保留以后的扩展DSO能力：
./configure --prefix=/usr/local/apache --enable-so=static --disable-alias
```

##### 利用APXS工具动态为Apache编译新的DSO（动态共享对象）
一般如果我们需要开启或者关闭某一些模块，只需要在 `httpd.conf` 中注释相应的模块的加载指令或者去掉指令前面的注释。
但如果我们需要的模块在 Apache编译安装的时候没有编译进去，我们可以用APXS工具来动态编译，并加入到Apache中。

APXS,是一个给Apache服务器编译和安装扩展模块的工具。即将一个或者多个源代码或者目标文件编译成一个动态共享对象（DSO），然后可以通过Apache的 LoadModule 指令加载运行。
因此，要使用该工具，我们的Apache必须支持DSO特性，即已经安装有mod_so 模块，否则安装会报错。

apxs命令选项说明：
```
apxs -g [ -S name=value ] -n modname
apxs -q [ -v ] [ -S name=value ] query ...
apxs -c [ -S name=value ] [ -o dsofile ] [ -I incdir ] [ -D name=value ] [ -L libdir ] [ -l libname ] [ -Wc,compiler-flags ] [ -Wl,linker-flags ] files ...
apxs -i [ -S name=value ] [ -n modname ] [ -a ] [ -A ] dso-file ...
apxs -e [ -S name=value ] [ -n modname ] [ -a ] [ -A ] dso-file ...

```

常用选项：
- -n modname
  > 明确的设置模块名称, -i(安装)和-g（模板生成）选项

执行选项：
- -q
  > 设置编译httpd时的变量和环境

配置选项：
- -S name=value
  > 此选项更改apxs的上述设置。

模板生成选项：
- -g
  > 该选项将生成一个子目录（名称将取决 -n设置），并会生成两个文件，一个要编译模块的源文件，用来创建模块或作为一个快速启动的apxs机制。另一个，用于编译和安装此模块，如mod_name.cMakefile

DSO 编译选项
- -c
 > 表明将进行编译操作。它首先编译C源文件(.c)，到对应的目标文件（.o），然后通过连接这些目标文件以及其余的目标文件（.a和.a）构建一个动态的共享对象dsofile

- -o dsofile
  > 明确规定创建动态共享对象文件名。如果没有指定，并且不能从文件名猜测到，则会生成 mod_unknow.so

- -D name=value
 > 直接传给编译命令自定义参数

- -L libdir
 > 设置编译时将要用到的自定义类库路径

- -l libname
 > 设置编译时用到的自定义类库名称

- -Wc，compiler-flags
 > 设置或添加本地编译器特定的选项

- -Wl，linker-flags
 > 设置或添加本地特定连接的选项。

- -p
 > 该选项将会使apxs 连接和引用apr/apr-util类库，使用apr/apr-util将会对编译非常有用。

DSO的安装和配置选项

- -i
  > 表明安装操作，安装一个或多个动态共享对象到服务器的模块目录

- -a
 > 自动添加 LoadModule 指令到 httpd.conf 配置文件，或者开启该指令。

- -A
 > 同 -a 选项，但创建的 LoadModule指令是被注释的状态，也就是说该模块已经准备就绪，但没开启。

- -e
 > 类似于 -a 和 -A 用来编辑 httpd.conf 而不安装该模块。


##### 示例
示例：我们有一个可用的Apache 模块 mod_foo.c 想要编译进Apache的DSO。
```
$ apxs -c mod_foo.c
/path/to/libtool --mode=compile gcc ... -c mod_foo.c
/path/to/libtool --mode=link gcc ... -o mod_foo.la mod_foo.slo
$ _
```

然后，在Apache的配置文件中加入 loadModule 指令加载此共享对象。为了简化该步骤 apxs 提供了自动更新配置文件的的功能选项(-a,-A)。

```
$ apxs -i -a mod_foo.la
/path/to/instdso.sh mod_foo.la /path/to/apache/modules
/path/to/libtool --mode=install cp mod_foo.la /path/to/apache/modules ... chmod 755 /path/to/apache/modules/mod_foo.so
[activating module `foo' in /path/to/apache/conf/httpd.conf]
$ _
```
这时，我们在httpd.conf 中就能看到这条指令。
```
LoadModule foo_module modules/mod_foo.so
```
如果没有使用 -a 选项自动添加，则需要手动添加进去。

如果想默认不开启该模块，可以使用 -A 选项。即
```
$ apxs -i -A mod_foo.c
```

**apxs快速测试**

我们可以通过创建一个Apache的测试模块，通过对应的Makefile
```
$ apxs -g -n foo
Creating [DIR] foo
Creating [FILE] foo/Makefile
Creating [FILE] foo/modules.mk
Creating [FILE] foo/mod_foo.c
Creating [FILE] foo/.deps
$ _
```
然后可以立即编译该测试模块到DSO，并加载到Apache。

```
$ cd foo
$ make all reload
apxs -c mod_foo.c
/path/to/libtool --mode=compile gcc ... -c mod_foo.c
/path/to/libtool --mode=link gcc ... -o mod_foo.la mod_foo.slo
apxs -i -a -n "foo" mod_foo.la
/path/to/instdso.sh mod_foo.la /path/to/apache/modules
/path/to/libtool --mode=install cp mod_foo.la /path/to/apache/modules ... chmod 755 /path/to/apache/modules/mod_foo.so
[activating module `foo' in /path/to/apache/conf/httpd.conf]
apachectl restart
/path/to/apache/sbin/apachectl restart: httpd not running, trying to start
[Tue Mar 31 11:27:55 1998] [debug] mod_so.c(303): loaded module foo_module
/path/to/apache/sbin/apachectl restart: httpd started
$ _
```

over~
