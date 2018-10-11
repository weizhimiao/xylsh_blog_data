---
title: Apache的安装
date: 2016-08-09 14:00:00
author: zhimiao
tags:
- Web服务器
- Apache
categories:
- Apache
---

Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。
Apache的安装无外乎两种方式：
* 二进制包安装
* 源码包安装


<!-- more -->

接下来我将会演示和记录通过源码包来安装apache到服务器。
环境：阿里云 Centos 7.2 64

## 安装前准备工作
通过源码包安装，我们需要将源码编译成计算机运行的二进制，因此我们需要编译工具。
### gcc安装
GNU编译器套件（GNU Compiler Collection）包括C、C++、Objective-C、Fortran、Java、Ada和Go语言的前端，也包括了这些语言的库（如libstdc++、libgcj等等）。GCC的初衷是为GNU操作系统专门编写的一款编译器。
```
[root@iZ28v78hcmrZ ~]# gcc
gcc: 致命错误：没有输入文件
编译中断。
[root@iZ28v78hcmrZ ~]# yum install gcc
···
作为依赖被升级:
  cpp.x86_64 0:4.8.5-4.el7            gcc-c++.x86_64 0:4.8.5-4.el7            gcc-gfortran.x86_64 0:4.8.5-4.el7    libgcc.x86_64 0:4.8.5-4.el7
  libgfortran.x86_64 0:4.8.5-4.el7    libgomp.x86_64 0:4.8.5-4.el7            libquadmath.x86_64 0:4.8.5-4.el7     libquadmath-devel.x86_64 0:4.8.5-4.el7
  libstdc++.x86_64 0:4.8.5-4.el7      libstdc++-devel.x86_64 0:4.8.5-4.el7

完毕！
[root@iZ28v78hcmrZ ~]#
```
### APR、APR-UTIL 安装

APR （全称：Apache Portable Runtime）可移植运行时库、APR-UTIL（全称：Apache Portable Runtime Utility Library）可移植运行时工具库。它们的作用是使得对平台细节的处理进行下移。对于应用程序而言，它们根本就不需要考虑具体的平台，不管是Unix、Linux还是Window，应用程序执行的接口基本都是统一一致的。
APR的目标则是希望安全合并所有的能够合并的代码而不需要牺牲性能，为大多数平台提供所有的APR特性支持，包括Win32、OS/2、BeOS、Darwin、Linux等等。



关于apr和apr-util，apache可以使用系统已经安装的版本，也可以不实用系统提供的版本。具体方法是分别下载apr和apr-util，解压到apache源码包中的srclib/apr和srclib/apr-util路径中（路径中不包含版本号等信息），在编译源码包之前的./configure 过程中使用 --with-included-apr 选项

### PRCE (Perl-Compatible Regular Expressions Library,Perl兼容的正则表达式库)
PCRE(Perl Compatible Regular Expressions)是一个用C语言编写的正则表达式函数库。
PCRE被广泛使用在许多开源软件之中，最著名的莫过于Apache HTTP服务器和PHP脚本语言、R脚本语言，此外，正如从其名字所能看到的，PCRE也是perl语言的缺省正则库。

[PCRE 下载网址][pcre_download]
[pcre_download]:https://sourceforge.net/projects/pcre/files/

```
#解压
[root@iZ94m2e99jtZ ~]# tar -zxvf pcre-8.38.tar.gz
[root@iZ94m2e99jtZ ~]# cd pcre-8.38
[root@iZ94m2e99jtZ pcre-8.38]# ./configure
···
pcre-8.38 configuration summary:

    Install prefix .................. : /usr/local
    C preprocessor .................. : gcc -E
    C compiler ...................... : gcc
    C++ preprocessor ................ : g++ -E
    C++ compiler .................... : g++
    Linker .......................... : /usr/bin/ld -m elf_x86_64
    C preprocessor flags ............ :
    C compiler flags ................ : -g -O2 -fvisibility=hidden
    C++ compiler flags .............. : -O2 -fvisibility=hidden -fvisibility-inlines-hidden
    Linker flags .................... :
    Extra libraries ................. :

    Build 8 bit pcre library ........ : yes
    Build 16 bit pcre library ....... : no
    Build 32 bit pcre library ....... : no
    Build C++ library ............... : yes
    Enable JIT compiling support .... : no
    Enable UTF-8/16/32 support ...... : no
    Unicode properties .............. : no
    Newline char/sequence ........... : lf
    \R matches only ANYCRLF ......... : no
    EBCDIC coding ................... : no
    EBCDIC code for NL .............. : n/a
    Rebuild char tables ............. : no
    Use stack recursion ............. : yes
    POSIX mem threshold ............. : 10
    Internal link size .............. : 2
    Nested parentheses limit ........ : 250
    Match limit ..................... : 10000000
    Match limit recursion ........... : MATCH_LIMIT
    Build shared libs ............... : yes
    Build static libs ............... : yes
    Use JIT in pcregrep ............. : no
    Buffer size for pcregrep ........ : 20480
    Link pcregrep with libz ......... : no
    Link pcregrep with libbz2 ....... : no
    Link pcretest with libedit ...... : no
    Link pcretest with libreadline .. : no
    Valgrind support ................ : no
    Code coverage ................... : no

[root@iZ94m2e99jtZ pcre-8.38]# make & make install

```

**Tips**
在编译pcre时可能会出现这样的：configure: error: You need a C++ compiler for C++ support.提示我们缺少一个C++ 的编译器，需要我们再安装一个C++的编译器。
```
[root@iZ94m2e99jtZ pcre-8.38]# yum install gcc-c++

```




### 下载

[apr、apr-uitl官网][apr_website]
[apr_website]:http://apr.apache.org/

[apr 下载地址][apr_1.5.2_download]
[apr_1.5.2_download]:http://mirrors.hust.edu.cn/apache//apr/apr-1.5.2.tar.gz
[apr-uitl 下载地址][apr-uitl_1.5.4_download]
[apr-uitl_1.5.4_download]:http://mirrors.hust.edu.cn/apache//apr/apr-util-1.5.4.tar.gz


[apache 源码包][apache_download]
[apache_download]:http://mirrors.tuna.tsinghua.edu.cn/apache//httpd/httpd-2.4.23.tar.gz

下载、解压、将apr和apr-util分别复制到httpd-2.4.23/srclib/apr和httpd-2.4.23/srclib/apr-util中

```
[root@iZ94m2e99jtZ ~]# wget http://mirrors.hust.edu.cn/apache//apr/apr-1.5.2.tar.gz
[root@iZ94m2e99jtZ ~]# wget  http://mirrors.hust.edu.cn/apache//apr/apr-util-1.5.4.tar.gz
[root@iZ94m2e99jtZ ~]# wget http://mirrors.tuna.tsinghua.edu.cn/apache//httpd/httpd-2.4.23.tar.gz

[root@iZ94m2e99jtZ ~]# tar -zxvf apr-1.5.2.tar.gz
[root@iZ94m2e99jtZ ~]# tar -zxvf apr-util-1.5.4.tar.gz
[root@iZ94m2e99jtZ ~]# tar -zxvf httpd-2.4.23.tar.gz

[root@iZ94m2e99jtZ ~]# ll
总用量 10088
drwxr-xr-x 27 1000  1000    4096 4月  25 2015 apr-1.5.2
-rw-r--r--  1 root root  1031613 4月  29 2015 apr-1.5.2.tar.gz
drwxr-xr-x 19 1000  1000    4096 9月  17 2014 apr-util-1.5.4
-rw-r--r--  1 root root   874044 9月  20 2014 apr-util-1.5.4.tar.gz
drwxr-xr-x 11  501 games    4096 7月   1 01:15 httpd-2.4.23
-rw-r--r--  1 root root  8406575 7月   5 03:50 httpd-2.4.23.tar.gz
[root@iZ94m2e99jtZ ~]#

[root@iZ94m2e99jtZ ~]# cp -r  apr-1.5.2 httpd-2.4.23/srclib/apr
[root@iZ94m2e99jtZ ~]# cp -r  apr-util-1.5.4 httpd-2.4.23/srclib/apr-util

```

## 安装

### Configuring the source tree
如果使用所有默认选项，只需键入的./configure即可，但一般我们根据自己的需求来修改一些配置。

其中最重要的选项应该是apache的安装位置的配置项 --prefix ；
此外，我们可以指定哪些功能被启用和禁用模块要包含在Apache中。apache配备了一个广泛的默认包含的模块。它们将被编译为可以装载或在运行时卸载共享对象（的DSO）。您也可以选择通过选项--enable-模块=静态编译静态模块。

额外的模块使用--enable模块选项，其中模块与除去mod_个串并转换为破折号任何下划线模块的名称启用。
同样，我们可以禁用与--disable模块选件模块。使用这些选项的时候，因为配置无法警告你，如果你指定的模块不存在要小心;它会简单地忽略选项。 此外，有时需要提供的配置脚本与你的编译器，库和头文件的位置额外信息。这是通过两种环境变量或命令行选项来配置完成。有关详细信息，请参考配置页面。或使用--help选项调用配置。

[Apache 官方配置说明][apache_configure_doc]
[apache_configure_doc]:http://httpd.apache.org/docs/2.4/programs/configure.html#installationdirectories


```
概要

你应该叫configure从分布的根目录中的脚本。

./configure [OPTION]... [VAR=VALUE]...

要指定环境变量（例如CC， CFLAGS......），它们指定为 。请参见下面 的一些有用的变量的说明。VAR=VALUE

最佳
选项

配置选项
安装目录
系统类型
可选功能
支持程序的选项
配置选项
下列选项影响的行为 configure本身。

-C
--config-cache
这是一个别名 --cache-file=config.cache
--cache-file=FILE
测试结果将在文件中缓存文件。此选项默认为禁用 ​​。
-h
--help [short|recursive]
输出的帮助和退出。随着说法short只是具体 ​​到这个包的选项将显示出来。参数 recursive显示所有包含包的简短帮助。
-n
--no-create
该configure脚本运行正常，但不创建输出文件。这是有用的生成makefile文件编译前检查测试结果。
-q
--quiet
不打印checking ...在配置过程的消息。
--srcdir=DIR
定义目录DIR是源文件目录。默认为所在目录configure的位置，或父目录。
--silent
与...一样 --quiet
-V
- 版
显示版权信息并退出。
安装目录
这些选项定义安装目录。安装树取决于所选的布局。

--prefix=PREFIX
在安装结构无关的文件PREFIX。默认安装目录设置为 /usr/local/apache2。
--exec-prefix=EPREFIX
在安装体系相关的文件EPREFIX。默认安装目录设置为 PREFIX目录。
默认情况下，make install将安装所有文件 /usr/local/apache2/bin，/usr/local/apache2/lib 等等。您可以指定以外的安装前缀 /usr/local/apache2使用--prefix，例如--prefix=$HOME。

定义一个目录布局
--enable-layout=LAYOUT
配置源代码和编译脚本的假设基础上的布局安装树布局。这使您可以分别指定为Apache HTTP服务器安装在每个类型的文件的位置。该config.layout 文件包含几个示例配置，你还可以创建下面的例子中你自己的自定义配置。该文件中的不同布局分为<Layout FOO>...</Layout>部分，并通过名称简称为中FOO，默认布局Apache。
安装目录微调
为了更好地控制安装目录中，使用下面的选项。请注意，目录默认被设置 autoconf，并通过相应布局设置被覆盖。

--bindir=DIR
在用户安装可执行文件DIR。用户可执行文件都支持这样的程序htpasswd， dbmmanage等等。这对于网站管理员有用。默认情况下DIR设置为 EPREFIX/bin。
--datadir=DIR
在安装只读体系结构无关的数据DIR。默认datadir设置为 PREFIX/share。此选项是提供 autoconf与当前未使用。
--includedir=DIR
在安装C头文件DIR。默认 includedir设置为 EPREFIX/include。
--infodir=DIR
在安装info文档DIR。默认infodir设置为 PREFIX/info。此选项当前未使用。
--libdir=DIR
在安装目标代码库DIR。默认 libdir设置为 EPREFIX/lib。
--libexecdir=DIR
在安装程序的可执行文件（例如，共享模块） DIR。默认libexecdir设置为 EPREFIX/modules。
--localstatedir=DIR
在安装修改的单机数据DIR。默认localstatedir设置为 PREFIX/var。此选项是提供 autoconf与当前未使用。
--mandir=DIR
在安装该男子文档DIR。默认 mandir设置为 EPREFIX/man。
--oldincludedir=DIR
在非GCC安装C头文件DIR。默认oldincludedir设置为 /usr/include。此选项是提供 autoconf与当前未使用。
--sbindir=DIR
在系统中安装管理员可执行DIR。这些都是服务器程序，如httpd， apachectl，suexec等，这些都需要运行在Apache HTTP服务器。默认 sbindir设置为 EPREFIX/sbin。
--sharedstatedir=DIR
在安装修改的架构无关的数据DIR。默认sharedstatedir设置为 PREFIX/com。此选项是提供 autoconf与当前未使用。
--sysconfdir=DIR
安装只读的单机数据，如服务器配置文件httpd.conf，mime.types等在 DIR。默认sysconfdir设置为 PREFIX/conf。
系统类型
这些选项用于交叉编译Apache HTTP服务器到另一个系统上运行。在正常的情况下，建立和运行在同一系统上的服务器时，不使用这些选项。

--build=BUILD
定义系统类型上的工具正在建立该系统。它默认为脚本的结果 config.guess。
--host=HOST
定义系统类型的服务器将运行，系统的 HOST默认为BUILD。
--target=TARGET
配置构建编译器系统类型 目标。它默认为HOST。此选项被提供autoconf，而不是必要的Apache HTTP服务器。
可选功能
这些选项用于微调您的HTTP服务器将具备的功能。

一般语法
一般来说，你可以使用下面的语法来启用或禁用功能：

--disable-FEATURE
不包括特征。这是相同的 。--enable-FEATURE=no
--enable-FEATURE[=ARG]
包括特征。为默认值ARG 为yes。
--enable-MODULE=shared
相应的模块将被建设成为DSO模块。默认情况下启用的模块是动态链接的。
--enable-MODULE=static
相应的模块将被静态链接。
注意

configure不会抱怨 ，即使富不存在，所以你需要仔细类型。 --enable-foo
选择模块编译
大多数模块由默认编译并已被明确或通过使用关键字禁用few （见--enable-modules，--enable-mods-shared 并且--enable-mods-static下面进一步解释）或--enable-modules=none作为一组被删除。

其它模块默认不编译并已被明确或通过使用关键字启用all或 reallyall可用。

要了解哪些模块是默认编译，运行 ./configure -h或./configure --help 下看Optional Features。假设你有兴趣mod_example1和 mod_example2，你看这个：

可选功能：
  ...
  --disable-例1示例模块1
  --enable-例题例如模块2
  ...
然后mod_example1是默认启用的，你就可以使用--disable-example1不编译。 mod_example2默认情况下禁用，你就可以使用--enable-example2 编译它。

多道处理模块
多道处理模块，或的MPM，实现了服务器的基本行为。单个MPM必须为了使服务器的功能被激活。出现在可用MPM列表 模块索引页。

的MPM可以建成为数字存储示波器的动态加载或静态与服务器相连，并使用下列选项被启用：

--with-mpm=MPM
选择适合您的服务器的默认MPM。如果MPM的构建为DSO模块（见--enable-mpms-shared），该指令选择将默认的配置文件中加载的MPM。否则，这个指令选择唯一可用的MPM，这将静态链接到服务器。

如果省略此选项，默认的MPM为您的操作系统将被使用。

--enable-mpms-shared=MPM-LIST
使动态共享模块的MPM的列表。这些模块之一必须动态使用加载 LoadModule指令。

MPM-LIST是加了引号的MPM名称的空格分隔的列表。例如：

--enable-mpms-shared='prefork worker'
此外，您还可以使用特殊关键字all，这将选择支持在当前平台上动态加载所有的MPM和他们建立的DSO模块。例如：

--enable-mpms-shared=all
第三方模块
要添加其他第三方模块使用下列选项：

--with-module=module-type:module-file[, module-type:module-file]
添加一个或多个第三方的模块，以静态链接模块列表。该模块的源文件module-file 将在搜索 你的Apache HTTP服务器的源代码树的子目录。如果没有找到有它正在考虑模块文件是一个绝对文件路径，并尝试将源文件复制到 模块式子目录。如果子目录不存在，它将被创建并与标准的填充 。modules/module-typeconfigureMakefile.in

此选项很有用添加由一个源文件小的外部组件。对于更复杂的模块，你应该阅读供应商的文档。

注意

如果你想建立一个DSO模块，而不是一个静态链接使用apxs。
累积和其他选项
--enable-maintainer-mode
打开调试和编译时警告，并加载所有编译的模块。
--enable-mods-shared=MODULE-LIST
定义启用并建立动态共享模块模块的列表。这意味着，这些模块必须通过使用动态加载 LoadModule指令。

MODULE-LIST是加了引号的modulenames空格分隔列表。该模块名称没有给出前面mod_。例如：

--enable-mods-shared='headers rewrite dav'
此外，您还可以使用特殊的关键字reallyall， all，most和few。例如，

--enable-mods-shared=most
将编译大多数模块，并将其建设成为DSO模块，

--enable-mods-shared=few
将只编译一个非常基本的模块组。

默认设置为most。

在LoadModule对所选择的模块的指令将在主配置文件中自动生成。默认情况下，所有的指令都只是由一个配置要求或明确选择的模块被注释掉--enable-foo的说法。您可以更改设置启用或关闭加载的模块LoadModule的指令 httpd.conf。此外， LoadModule所有构建的模块的指令可通过配置选项被激活 --enable-load-all-modules。

--enable-mods-static=MODULE-LIST
此选项的行为类似--enable-mods-shared，但在给定的模块静态链接。这意味着，这些模块将始终存在，同时运行httpd。他们不必被加载LoadModule。
--enable-modules=MODULE-LIST
此选项的行为等来--enable-mods-shared，并且还将动态地链接的给定模块。特殊关键字none禁用所有模块的版本。
--enable-v4-mapped
允许IPv6的套接字来处理IPv4连接。
--with-port=PORT
这定义上的端口httpd会听。生成配置文件时，该端口号用于 httpd.conf。默认值是80。
--with-program-name
定义一个替代的可执行文件名 ​​称。默认值是 httpd。
可选包
这些选项用于定义可选包。

一般语法
一般来说，你可以使用下面的语法定义一个可选包：

--with-PACKAGE[=ARG]
使用包PACKAGE。为默认值 ARG为yes。
--without-PACKAGE
不要使用包PACKAGE。这是相同的 。此选项所提供 ，但对于Apache HTTP服务器不是非常有用。--with-PACKAGE=noautoconf
特定软件包
--with-apr=DIR|FILE
在Apache可移植运行时（APR）是httpd的源代码分发的一部分，将自动与HTTP服务器共同建立。如果您想使用已安装的年利率，而不是你要告诉configure路径的 apr-config脚本。你可以设置绝对路径和名称或目录切换到安装四月apr-config必须在该目录或子目录中存在 bin。
--with-apr-util=DIR|FILE
Apache可移植运行实用程序（APU）是httpd的源代码分发的一部分，将自动与HTTP服务器共同建立。如果您想使用已安装的APU，而不是你要告诉configure路径的 apu-config脚本。你可以设置绝对路径和名称或目录切换到安装APU，apu-config必须在该目录或子目录中存在 bin。
--with-ssl=DIR
如果mod_ssl已启用configure 已安装的OpenSSL的搜索。您可以将目录路径设置为SSL / TLS工具包来代替。
--with-z=DIR
configure已安装的自动搜索 zlib，如果你的源配置需要一个库（例如，当mod_deflate使能）。您可以设置压缩库的目录路径来代替。
Apache HTTP服务器的一些特性，如 mod_authn_dbm和mod_rewrite的DBM RewriteMap使用简单的键/值对数据库信息的快速查询。SDBM被包括在APU，所以这个数据库是始终可用。如果您想使用其他数据库类型，可以使用下面的选项，以使它们：

--with-gdbm[=path]
如果没有路径指定，configure将搜索在平时的搜索路径GNU DBM安装的包含文件和库。一个明确的 路径会导致configure在寻找 path/lib，并 path/include为相关的文件。最后，路径可以指定用冒号隔开具体包括和库路径。
--with-ndbm[=path]
像--with-gdbm，但搜索新DBM安装。
--with-berkeley-db[=path]
像--with-gdbm，但搜索一个Berkeley DB的安装。
注意

由APU提供，并通过其配置脚本传递的DBM选项。他们利用确定已安装的APU时是无用的--with-apr-util。
您可以使用自己的HTTP服务器一起使用一个以上的DBM实现。该拨款DBM类型将每次运行时配置中配置。
支持程序的选项
--enable-static-support
构建支持二进制文件的静态链接的版本。这意味着，一个独立的可执行文件将集成所有必要的库来构建。否则，支持二进制文件默认情况下，动态链接。
--enable-suexec
使用此选项可启用suexec，它允许您设置UID和GID的催生过程。除非你了解你的服务器上运行的SUID二进制的所有安全隐患，请勿使用此选项。更多选项来配置suexec介绍如下。
这可以通过使用下列选项来创建一个单一的支持程序的静态链接二进制文件：

--enable-static-ab
建立一个静态链接的版本ab。
--enable-static-checkgid
建立一个静态链接的版本checkgid。
--enable-static-htdbm
建立一个静态链接的版本htdbm。
--enable-static-htdigest
建立一个静态链接的版本htdigest。
--enable-static-htpasswd
建立一个静态链接的版本htpasswd。
--enable-static-logresolve
建立一个静态链接的版本logresolve。
--enable-static-rotatelogs
建立一个静态链接的版本rotatelogs。
suexec 配置选项
下列选项用于微调的行为suexec。参见配置和安装suEXEC的 进一步的信息。

--with-suexec-bin
这定义的路径suexec二进制文件。默认值是--sbindir（见安装目录微调）。
--with-suexec-caller
这定义允许呼叫的用户suexec。它应该是相同下，用户 httpd正常运行。
--with-suexec-docroot
这个定义下的目录树suexec允许访问的可执行文件。默认值是 --datadir/htdocs。
--with-suexec-gidmin
这个定义的最低GID成为目标用户 suexec。默认值是100。
--with-suexec-logfile
这个定义的文件名suexec​​日志文件。默认情况下，日志文件被命名为suexec_log，位于 --logfiledir。
--with-suexec-safepath
定义环境变量的值PATH由启动的进 ​​程进行设置suexec。默认值是/usr/local/bin:/usr/bin:/bin。
--with-suexec-userdir
此定义包含所有可执行文件的用户目录下的子目录suexec的访问是允许的。当你想使用此设置时必须 suexec使用特定用户目录在一起（如所提供的mod_userdir）。默认值是 public_html。
--with-suexec-uidmin
它定义为最低的UID允许为目标用户 suexec。默认值是100。
--with-suexec-umask
设置umask由启动的进 ​​程 suexec。它默认为您的系统设置。
最佳
环境变量

有一些有用的环境变量覆盖所作出的选择 configure，或帮助它找到库和程序与非标准名称或位置。

CC
定义要用于编译的C编译器的命令。
CFLAGS
要使用编译设置C编译器的标志。
CPP
定义C预处理命令使用。
CPPFLAGS
将C / C ++预处理器的标志，例如 ，如果你在一个非标准目录头了includedir。-Iincludedir
LDFLAGS
设置连接器选项，例如，如果你在一个非标准目录库LIBDIR。-Llibdir


```

***示例：***
这里是一个典型的例子编译Apache，安装树/ SW /包装/与特定的编译器和标志，加上两个额外的模块阿帕奇mod_ldap模块和mod_lua

```
$ CC="pgcc" CFLAGS="-O2" \
./configure --prefix=/sw/pkg/apache \
--enable-ldap=shared \
--enable-lua=shared

```

我们本次采用基本的默认设置 来进行安装
```
[root@iZ94m2e99jtZ ~]# cd httpd-2.4.23
[root@iZ94m2e99jtZ httpd-2.4.23]# ./configure  --with-included-apr

```

### Build

```
[root@iZ94m2e99jtZ httpd-2.4.23]# make
```
### Install
```
[root@iZ94m2e99jtZ httpd-2.4.23]# make install
···
Installing configuration files
mkdir /usr/local/apache2/conf
mkdir /usr/local/apache2/conf/extra
mkdir /usr/local/apache2/conf/original
mkdir /usr/local/apache2/conf/original/extra
Installing HTML documents
mkdir /usr/local/apache2/htdocs
Installing error documents
mkdir /usr/local/apache2/error
Installing icons
mkdir /usr/local/apache2/icons
mkdir /usr/local/apache2/logs
Installing CGIs
mkdir /usr/local/apache2/cgi-bin
Installing header files
Installing build system files
Installing man pages and online manual
mkdir /usr/local/apache2/man
mkdir /usr/local/apache2/man/man1
mkdir /usr/local/apache2/man/man8
mkdir /usr/local/apache2/manual
```

### Customize（定制）

接下来，我们可以通过在 /usr/local/apache2/conf 目录，编辑配置文件来自定义我们的Apache HTTP服务器。

[配置指令快速参考索引][apache_conf_directive_index]
[apache_conf_directive_index]:http://httpd.apache.org/docs/2.4/zh-cn/mod/directives.html

### Testinh(测试)
启动Apache服务器
```
[root@iZ94m2e99jtZ httpd-2.4.23]# /usr/local/apache2/bin/apachectl -k start

[root@iZ94m2e99jtZ httpd-2.4.23]# ps aux | grep httpd
root     15386  0.0  0.2  70556  2188 ?        Ss   11:53   0:00 /usr/local/apache2/bin/httpd -k start
daemon   15387  0.0  0.4 359520  4260 ?        Sl   11:53   0:00 /usr/local/apache2/bin/httpd -k start
daemon   15388  0.0  0.4 359520  4260 ?        Sl   11:53   0:00 /usr/local/apache2/bin/httpd -k start
daemon   15389  0.0  0.4 359520  4256 ?        Sl   11:53   0:00 /usr/local/apache2/bin/httpd -k start
root     15472  0.0  0.0 112664   972 pts/0    S+   11:53   0:00 grep --color=auto httpd
```
浏览器访问：http://IP地址

当出现『It works!』字样说明我们已经安装成功

***注意*** 如果通过浏览器访问不到，可能是请求服务器防火墙给拦截了，所以我们需要在防火墙里将我们用到的80端口给放行。
我以我当前Centos7 的系统为例，Centos7现在默认的防火墙是firewalld，Centos6以及6以前的版本则使用的是iptables，所以具体设置方法还请自己去搜索。
```
[root@iZ94m2e99jtZ ~]# firewall-cmd  --permanent --zone=public --add-port=80/tcp  #将80端口开放
success
[root@iZ94m2e99jtZ ~]# firewall-cmd  --reload #重新加载防火墙配置
success
```
## 日常管理
一般常见的管理方式有两种
* 直接通过httpd命令来管理
* 通过apachectl来管理
***Tips*** apachetl其实一个是对httpd命令进行了封装sh脚本

### 常用命令

```
#通过apachectl来管理
[root@iZ94m2e99jtZ ~]# /usr/local/apache2/bin/apachectl -k start  #启动
[root@iZ94m2e99jtZ ~]# /usr/local/apache2/bin/apachectl -k restart  #重启
[root@iZ94m2e99jtZ ~]# /usr/local/apache2/bin/apachectl -k stop  #停止

#直接通过httpd命令来管理
[root@iZ94m2e99jtZ ~]# /usr/local/apache2/bin/httpd -k start|restart|graceful|graceful-stop|stop

```

### 具体参数
```
Usage: /usr/local/apache2/bin/httpd [-D name] [-d directory] [-f file]
                                    [-C "directive"] [-c "directive"]
                                    [-k start|restart|graceful|graceful-stop|stop]
                                    [-v] [-V] [-h] [-l] [-L] [-t] [-T] [-S] [-X]
Options:
  -D name            : define a name for use in <IfDefine name> directives
  -d directory       : specify an alternate initial ServerRoot
  -f file            : specify an alternate ServerConfigFile
  -C "directive"     : process directive before reading config files
  -c "directive"     : process directive after reading config files
  -e level           : show startup errors of level (see LogLevel)
  -E file            : log startup errors to file
  -v                 : show version number
  -V                 : show compile settings
  -h                 : list available command line options (this page)
  -l                 : list compiled in modules
  -L                 : list available configuration directives
  -t -D DUMP_VHOSTS  : show parsed vhost settings
  -t -D DUMP_RUN_CFG : show parsed run settings
  -S                 : a synonym for -t -D DUMP_VHOSTS -D DUMP_RUN_CFG
  -t -D DUMP_MODULES : show all loaded modules
  -M                 : a synonym for -t -D DUMP_MODULES
  -t -D DUMP_INCLUDES: show all included configuration files
  -t                 : run syntax check for config files
  -T                 : start without DocumentRoot(s) check
  -X                 : debug mode (only one worker, do not detach)

```
