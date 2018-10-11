---
title: Xdebug之常见问题
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---

- xdebug的使用

- xdebug的编译安装

<!-- more -->

## xdebug的使用


问：phpinfo（）报告Xdebug已安装和启用，但我仍然不会得到任何堆栈跟踪时发生错误。

A1：你必须搜索所有的PHP库，并包含任何“set_error_handler”调用的文件。如果有任何，你必须将其注释掉，或者更改handler函数的主体以调用xdebug_ * api函数。

A2：您没有在php.ini中将display_errors设置为1

---
问：Xdebug不格式化输出。

A：确保php.ini中的PHP的html_errors设置为1

---
问：调试客户端没有收到任何连接，我怎么办？

A：您可能忘记设置环境变量或向URL中添加必要的信息。有关详细信息，请参阅文档。

A：您检查过防火墙设置了吗？确保Xdebug正在侦听的端口（默认为9000）未被阻止。

A：你使用FastCGI，可能是通过FPM（FastCGI过程管理器）？它默认使用相同的端口作为Xdebug（9000），因此您需要将其中一个更改为不同的数字。在Xdebug，你可以使用xdebug.remote_port。

A：如果你使用SELinux运行，你应该确保它不会阻止连接;查找关于name_connect或与Xdebug端口相关的任何警告。您可能必须明确允许访问。访问此网站或搜索“selinux name_connect apache”有关如何执行此操作的详细信息


## xdebug的编译安装

问：我没有phpize工具。

答：Debian和Ubuntu用户需要用apt安装php5-dev软件包。

---

问：如何处理：Xdebug需要Zend Engine API版本xxxxxxxx。安装的Zend Engine API版本2xxxxxxxx较新。

A：此消息意味着您正在尝试加载Xdebug的PHP版本，它不是为它构建的。如果你自己编译PHP，很可能是因为你编译Xdebug对PHP头，属于你运行的不同的PHP版本。例如，你使用PHP 5.3，你使用的头仍然是PHP 5.2。如果你使用一个预编译的二进制文件，那么你使用的是错误的。

要诊断这是否是您的问题，请执行以下步骤：
- 通过查看phpinfo（）（或“php -i”）输出，检查您正在运行的PHP版本的“Zend Extension”API号。您可以在输出的顶部找到它，与PHP徽标和PHP版本在同一个块中。例如，对于PHP 5.2，数字是“220060519”，对于PHP 5.3，它是“220090626”。

- 当您完成编译步骤时，检查“phpize”的输出。您要查找的数字是在“Zend Extension Api No”的行上。


如果上面的两个数字不匹配，你正在使用错误的PHP头文件进行编译。请参阅下一个常见问题条目以确定要使用的phpize。


---

问：Xdebug仅作为PHP扩展加载，而不是作为Zend扩展。
定制的安装指导可能已指向此条目。

为了使Xdebug正常工作，包括断点等，它需要它作为Zend扩展加载，而不只是作为一个普通的PHP扩展。有些安装工具（PEAR / PECL）有时建议您使用extension = xdebug.so加载Xdebug。这是不正确的。为了解决这个问题，请在顶部块中“加载配置文件”和“其他.ini文件解析”下列出的任何INI文件中查找一行extension = xdebug.so。删除此行，并返回到定制安装说明。

---

问：我如何找到使用哪个phpize？

A：运行：“phpize --help”。这将显示phpize的完整路径。此路径应该与您具有CLI二进制文件“php-config”和安装的“pear”和“pecl”二进制文件的路径相同。如果你运行“php-config --version”它应该显示与你运行的PHP版本相同。如果它不匹配，并且可能在路径上找到错误的“phpize”二进制，您可以运行configure如下：

```
/full/path/to/php/bin/phpize
./configure --with-php-config=/full/path/to/php/bin/php-config
```

---
问：我使用XAMPP，但我似乎不能得到打包的xdebug扩展正常工作。

A：如果你取消注释“extension = php_xdebug.dll”行，这是预期的。 Xdebug需要加载zend_extension_ts =“C：\ path \ to \ php_xdebug.dll”指令。 你也可能必须禁用加载Zend优化器，因为它默认情况下启用，并且不能很好地与Xdebug。 因此，查找php.ini中的相关条目，并将其注释掉。 从PHP 5.3起，您总是需要使用zend_extension PHP.ini设置名称，而不是zend_extension_ts。
问：在Debian，我看到以下问题与Xdebug的构建....任何修复？

```
/usr/lib/libc_nonshared.a(stat.oS)(.text.__i686.get_pc_thunk.bx+0x0):
In function `__i686.get_pc_thunk.bx':
: multiple definition of `__i686.get_pc_thunk.bx'
/usr/lib/gcc-lib/i486-linux/3.3.5/crtbeginS.o
(.gnu.linkonce.t.__i686.get_pc_thunk.bx+0x0): first defined here
collect2: ld returned 1 exit status
make: *** [xdebug.la] Error 1
```

A：这是Debian本身的问题，有关更多信息，请参阅[此处](http://www.xdebug.org/archives/xdebug-general/0825.html)和[此处](http://www.xdebug.org/archives/xdebug-general/0825.html)。
