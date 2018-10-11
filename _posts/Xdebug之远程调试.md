---
title: Xdebug之远程调试
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---

Xdebug为与运行PHP脚本交互的调试器客户端提供了一个接口。 本节介绍如何设置PHP和Xdebug来允许这一点，并介绍一些客户端。

<!-- more -->

## 介绍

Xdebug的（远程）调试器允许您检查数据结构，交互式地浏览和调试您的代码。 正在使用的协议是打开的，称为DBGp。 此协议在Xdebug 2中实现，并替换不再支持的旧的GDB类协议。

## 客户端

Xdebug 2捆绑了一个用于DBGp协议的简单命令行客户端。 还有一些其他客户端实现（免费和商业）。 我不是任何这些的作者，所以请参考原作者的支持：

- Dev-PHP (IDE: Windows)
- Eclipse plugin (IDE).
- Emacs plugin (Editor Plugin).
- KDevelop (IDE: Linux (KDE); Open Source).
- ActiveState's Komodo (IDE: Windows, Linux, Mac; Commercial).
- MacGDBP (Standalone client for Mac OS X; Free)
- NetBeans (IDE: Windows, Linux, Mac OS X and Solaris).
- Notepad++ plugin (Editor: Windows).
- WaterProof's PHPEdit (IDE, from version 2.10: Windows; Commercial).
- PHPEclipse (Editor Plugin).
- Devsense's PHP Tools for Visual Studio (MS Visual Studio Plugin; Commercial).
- JetBrain's PhpStorm (IDE; Commercial).
- Protoeditor (Editor: Linux).
- pugdebug (Standalone client for Linux, Windows and Mac OS X; Open Source).
- VIM plugin (Editor Plugin).
- jcx software's VS.Php (MS Visual Studio Plugin; Commercial).
- Xdebug Chrome App (Chrome Application; Open Source)
- XDebugClient (Standalone client for Windows).

用于调试的简单命令行客户机与debugclient目录中的Xdebug捆绑在一起。

## 启动调试器
为了启用Xdebug的调试器，您需要在php.ini中进行一些配置设置。 这些设置为xdebug.remote_enable以启用调试器xdebug.remote_host和xdebug.remote_port来配置调试器应连接到的IP地址和端口。 还有一个xdebug.remote_connect_back设置，如果您的开发服务器与多个开发人员共享，则可以使用此设置。

如果希望调试器在发生错误情况（PHP错误或异常）时启动会话，则还需要更改xdebug.remote_mode设置。 此设置的允许值为“req”（默认值），这使得调试器在脚本启动时启动会话，或者“jit”，当会话只应在错误时启动。

完成所有这些设置后，Xdebug仍然不会在脚本运行时自动启动调试会话。 你需要激活Xdebug的调试器，你可以通过三种方式：

1、当从命令行运行脚本时，您需要设置一个环境变量，如：
```
export XDEBUG_CONFIG="idekey=session_name"
php myscript.php
```
您还可以在此相同的环境变量中配置xdebug.remote_host，xdebug.remote_port，xdebug.remote_mode和xdebug.remote_handler，只要使用空格分隔值即可：
```
export XDEBUG_CONFIG="idekey=session_name remote_host=localhost profiler_enable=1"
```
通过XDEBUG_CONFIG设置可以获得的所有设置也可以使用正常的php.ini设置进行设置。

2、如果要调试通过Web浏览器启动的脚本，只需将XDEBUG_SESSION_START = session_name作为参数添加到URL。 而不是使用GET参数，您还可以将XDEBUG_SESSION_START设置为POST参数，或通过cookie。 请参阅下一节，了解调试会话如何在浏览器窗口中工作。

3、通过Web服务器运行PHP时激活调试器的另一种方法是安装以下四个浏览器扩展之一。 它们中的每一个都允许您通过单击一个按钮来启用调试器。 当这些扩展是活动的，他们直接设置XDEBUG_SESSION cookie，而不是通过XDEBUG_SESSION_START进一步的HTTP调试会话中描述。 扩展名为：

- The easiest Xdebug
> 这个扩展的Firefox是为了使IDE的调试更容易。 您可以在https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/找到扩展程序。

- Xdebug Helper for Chrome
> Chrome的此扩展程序将帮助您通过一次点击启用/禁用调试和分析。 您可以在https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc找到扩展程序。

- Xdebug Toggler for Safari
> Safari的这个扩展允许你从Safari中自动启动Xdebug调试。 你可以从Github https://github.com/benmatselby/xdebug-toggler获取它。

- Xdebug launcher for Opera
> 这个扩展的Opera允许您从Opera启动Xdebug会话。

在你开始你的脚本之前，你需要告诉你的客户端它可以接收调试连接，请参考具体客户端的文档如何做到这一点。 要使用捆绑的客户端，只需在编译和安装后启动它。 您可以通过运行“debugclient”启动它。

当debugclient启动时，它将显示以下信息，然后等待，直到调试服务器启动连接：
```
Xdebug Simple DBGp client (0.10.0)
Copyright 2002-2007 by Derick Rethans.
- libedit support: enabled

Waiting for debug server to connect.
```
在建立连接后，将显示调试服务器的输出：
```
Connect
<?xml version="1.0" encoding="iso-8859-1"?>
<init xmlns="urn:debugger_protocol_v1"
      xmlns:xdebug="http://xdebug.org/dbgp/xdebug"
      fileuri="file:///home/httpd/www.xdebug.org/html/docs/index.php"
      language="PHP"
      protocol_version="1.0"
      appid="13202"
      idekey="derick">
  <engine version="2.0.0RC4-dev"><![CDATA[Xdebug]]></engine>
  <author><![CDATA[Derick Rethans]]></author>
  <url><![CDATA[http://xdebug.org]]></url>
  <copyright><![CDATA[Copyright (c) 2002-2007 by Derick Rethans]]></copyright>
</init>
(cmd)
```
现在，您可以使用DBGp文档页面中说明的命令集。 当脚本结束时，调试服务器与客户端断开连接，并且调试客户端在等待新连接时恢复。


## 通讯建立

### 使用静态IP /单个开发者
通过远程调试，嵌入在PHP中的Xdebug就像客户端，而IDE作为服务器。 以下动画显示了通信通道的设置：

![img](https://xdebug.org/images/docs/dbgp-setup.gif)

- 服务器的IP是10.0.1.2，HTTP在端口80上
- IDE在IP 10.0.1.42上，因此xdebug.remote_host设置为10.0.1.42
- IDE侦听端口9000，因此xdebug.remote_port设置为9000
- HTTP请求在运行IDE的计算机上启动
- Xdebug连接到10.0.1.42:9000
- 调试运行，提供HTTP响应


### 与未知IP /多开发者

如果使用xdebug.remote_connect_back，设置稍有不同：

![img](https://xdebug.org/images/docs/dbgp-setup2.gif)

- 服务器的IP是10.0.1.2，HTTP在端口80上
- IDE处于未知IP，因此xdebug.remote_connect_back设置为1
- IDE侦听端口9000，因此xdebug.remote_port设置为9000
- 发出HTTP请求后，Xdebug将从HTTP头中检测IP地址
- Xdebug连接到端口9000上检测到的IP（10.0.1.42）
- 调试运行，提供HTTP响应


## HTTP调试会话
Xdebug包含通过浏览器启动时跟踪调试会话的功能：Cookie。 这是这样工作：

- 当URL变量XDEBUG_SESSION_START = name附加到URL或通过具有相同名称的POST变量时，-Xdebug会发出名为“XDEBUG_SESSION”的cookie，并将值设置为XDEBUG_SESSION_START URL参数的值。 cookie的到期时间为1小时。 当连接到“idekey”属性中的debugclient时，DBGp协议也将此相同的值传递给init包。

- 当有一个GET（或POST）变量XDEBUG_SESSION_START或XDEBUG_SESSION cookie被设置时，Xdebug将尝试连接到一个调试客户端。

- 要停止调试会话（并销毁cookie），只需添加URL参数XDEBUG_SESSION_STOP即可。 Xdebug将不再尝试连接到调试客户端。

## 多用户调试
Xdebug仅允许您在执行远程调试时指定一个IP地址与xdebug.remote_host连接。它不会自动连接回浏览器运行的计算机的IP地址，除非您使用xdebug.remote_connect_back。

如果所有开发人员在同一（开发）服务器上的不同项目上工作，则可以通过Apache的.htaccess功能为每个目录设置xdebug.remote_host设置，方法是使用php_value xdebug.remote_host = 10.0.0.5。但是，对于多个开发人员在同一代码上工作的情况，.htaccess技巧不工作，因为代码所在的目录是相同的。

有两个解决方案。首先，你可以使用DBGp代理。有关如何使用此代理的概述，请参阅多个用户调试中的文章。您可以在ActiveState的网站上下载代理作为python远程调试包的一部分。 Komodo FAQ中还有一些文档。

其次，您可以使用Xdebug 2.1中引入的xdebug.remote_connect_back设置。

## 实施细则
Xdebug的DBGp协议的context_names命令的实现不依赖于栈级别。 在每个调试器会话期间返回的值总是相同的，因此，可以安全地缓存。

## 相关的设置

### xdebug.extended_info
类型：整数，默认值：1

控制Xdebug是否应该为PHP解析器强制执行'extended_info'模式;这允许Xdebug使用远程调试器执行文件/行断点。当跟踪或剖析脚本时，通常希望关闭此选项，因为PHP生成的oparrays将增加大约三分之一的大小减慢脚本。此设置不能在带有ini_set（）的脚本中设置，但只能在php.ini中设置。

### xdebug.idekey
类型：字符串，默认值：* complex *

控制哪个IDE密钥Xdebug应该传递到DBGp调试器处理程序。默认值基于环境设置。首先查询环境设置DBGP_IDEKEY，然后查询USER和最后一个USERNAME。默认值设置为找到的第一个环境变量。如果找不到，则设置为默认的“”。如果设置此设置，它始终覆盖环境变量。

### xdebug.remote_addr_header
类型：字符串，默认值：“”，在Xdebug> 2.4中引入

如果xdebug.remote_addr_header配置为非空字符串，那么该值将用作$ SERVER超全局数组中的键，以确定用于查找用于“连接回”的IP地址或主机名的头。此设置仅与xdebug.remote_connect_back结合使用，否则将被忽略。

### xdebug.remote_autostart
类型：布尔值，默认值：0

通常，您需要使用特定的HTTP GET / POST变量来启动远程调试（请参阅远程调试）。当此设置设置为1时，Xdebug将始终尝试启动远程调试会话并尝试连接到客户端，即使GET / POST / COOKIE变量不存在。

### xdebug.remote_connect_back
类型：boolean，默认值：0，在Xdebug> 2.1中引入

如果启用，xdebug.remote_host设置将被忽略，Xdebug将尝试连接到发出HTTP请求的客户端。它检查$\_SERVER ['HTTP_X_FORWARDED_FOR']和$\_SERVER ['REMOTE_ADDR']变量以找出要使用的IP地址。

如果配置了xdebug.remote_addr_header，那么将在$\_SERVER ['HTTP_X_FORWARDED_FOR']和$\_SERVER ['REMOTE_ADDR']变量之前检查具有配置名称的$ SERVER变量。

此设置不适用于通过CLI进行调试，因为$ SERVER标题变量在那里不可用。

请注意，没有可用的过滤器，任何可以连接到Web服务器的人都能够启动调试会话，即使他们的地址与xdebug.remote_host不匹配。

### xdebug.remote_cookie_expire_time
类型：整数，默认值：3600，在Xdebug> 2.1中引入

此设置可用于增加（或减少）远程调试会话通过会话cookie保持活动的时间。

### xdebug.remote_enable
类型：布尔值，默认值：0

此开关控制Xdebug是否应尝试联系正在侦听主机上的调试客户端，并使用设置xdebug.remote_host和xdebug.remote_port设置的端口。如果无法建立连接，脚本将继续，就像此设置为0。

### xdebug.remote_handler
类型：字符串，默认值：dbgp

可以是“php3”选择旧的PHP 3式调试器输出，“gdb”启用GDB像调试器接口或'dbgp' - 调试器协议。 DBGp协议是唯一支持的协议。

注意：Xdebug 2.1和更高版本只支持'db​​gp'作为协议。

### xdebug.remote_host
类型：字符串，默认值：localhost
选择运行调试客户端的主机，您可以使用主机名或IP地址。如果启用xdebug.remote_connect_back，将忽略此设置。

### xdebug.remote_log
类型：字符串，默认值：

如果设置为某个值，则将其用作所有远程调试器通信记录到的文件的文件名。 该文件始终以附加模式打开，因此默认情况下不会被覆盖。 没有可用的并发保护。 该文件的格式如下：
```
Log opened at 2007-05-27 14:28:15
-> <init xmlns="urn:debugger_protocol_v1" xmlns:xdebug="http://xdebug.org/dbgp/x ... ight></init>

<- step_into -i 1
-> <response xmlns="urn:debugger_protocol_v1" xmlns:xdebug="http://xdebug.org/db ... ></response>
```

### xdebug.remote_mode
类型：字符串，默认值：req

选择启动调试连接的时间。 此设置可以有两个不同的值：

- req
> 一旦脚本启动，Xdebug将尝试连接到调试客户端。

- jit
> Xdebug只会在出现错误情况时尝试连接到调试客户端。

### xdebug.remote_port
类型：整数，默认值：9000

Xdebug尝试在远程主机上连接的端口。 端口9000是客户端和捆绑的debugclient的默认值。 由于许多客户端使用此端口号，最好保持此设置不变。

## 相关函数

### bool xdebug_break（）
向调试客户端发出断点。

此函数使调试器在特定行上断开，就好像在此行上设置了正常的文件/行断点。



over~
