---
title: Linux目录结构介绍
date: 2016-10-06 22:30:00
categories:
- Linux
---

![Linux目录结构](http://www.centoscn.com/uploads/allimg58981378314368.jpg)

<!-- more -->

## Linux常见目录

```
/		根目录
/bin		命令保存目录（普通用户就可以读取的命令）
/boot		启动目录，启动相关文件
/dev		设备文件保存目录
/etc		配置文件保存目录
/etc/rc.d		主要在记录一些开关机过程中的 scripts 档案， scripts 有点像是 DOS 下的批次档（.bat檔名）
/etc/rc.d/init.d	所以服务预设的启动 scripts 都是放在这里的，例如要启动与关闭 iptables 的话，可以：
/etc/rc.d/init.d/iptables start
/etc/rc.d/init.d/iptables stop
/etc/xinetd.d		这个路径在较新的 Linux distribution 当中才有，由于早期的版本用来开启服务的档案是 inetd.conf ，但是在较新的版本中，开启服务的项目已经变成使用 xinetd.conf 这个档案，因此，你若需要启动一些额外的服务的话，在 Mandrake 9.0 或者是 Red Hat 7.0 以后就要到 /etc/xinetd.d 这个目录下了。
/etc/X11	这是与 X windows 有关的设定文件所在的目录，尤其里面的 XF86Config-4 更是重要呢！
/etc/opt/		/opt/的配置文件
/etc/sgml/		SGML的配置文件
/etc/xml/		XML的配置文件
/home		普通用户的家目录
/lib		系统库保存目录
/mnt		系统挂载目录（空目录）（一般建议将光盘、U盘等都关在在此目录下）
/media		挂载目录（空目录）
/root		超级用户的家目录
/tmp		临时目录
/sbin		命令保存目录（超级用户才能使用的目录）
/proc		直接写入内存的
/proc/cpuinfo		关于处理器的信息，如类型、厂家、型号和性能等
/proc/devices		当前运行内核所配置的所有设备清单
/proc/dma		当前正在使用的DMA通道。/proc/filesystems 当前运行内核所配置的文件系统
/proc/interrupts		正在使用的中断，和曾经有多少个中断
/proc/ioports		当前正在使用的I/O端口
/sys		同上
/usr		系统软件资源目录
/usr/bin/		系统命令（普通用户）
/usr/sbin/		系统命令（超级用户）
/usr/locate/		源码包安装的软件存放目录
/usr/X11		同/usr/X11R6 （/usr/X11R6的符号连接）
/usr/X11R6		X Window System存放相关档案的目录
/usr/X11R6/bin		大量的小X-WINDOWS应用程序（也可能是一些在其它子目录下大执行文件的符号连接）
/usr/include		一些套件的header檔。基本上，当我们在以 tarball 方式（ *.tar.gz 的方式安装软件）安装某些数据时，会使用到的一些函式库都在这个目录底下喔！
/usr/lib		内含许多程序与子程序所需的函式库。
/usr/local		在你安装完了 Linux 之后，基本上所有的配备你都有了，但是软件总是可以升级的，例如你要升级你的 proxy 服务，则通常软件预设的安装地方就是在 /usr/local 中（ local 是『当地』的意思），同时，安装完毕之后所得到的执行文件，为了与系统原先的执行文件有分别，因此升级后的执行档通常摆在 /usr/local/bin 这个地方。
/usr/local/bin		可能是用户安装的小的应用程序，和一些在/usr/local目录下大应用程序的符号连接
/usr/share/doc		放置一些系统说明文件的地方，例如你安装了 lilo 了，那么在该目录底下找一找，就可以查到 lilo 的说明文件了！很是便利！
/usr/share/man		放置一些程序的说明文件的地方，那是什么？呵呵！就是你使用 man 的时候，会去查询的路径呀！例如你使用 man ls 这个指令时，就会查出 /usr/share/man/man1/ls.1.bz2 这个说明档的内容啰！
/usr/src		这是放置核心原始码的预设目录，未来我们要编译核心的时候，就必须到这个目录底下呦！
/var		系统相关文档内容
/var/log/	系统日志位置
/var/spool/mail/		系统默认邮箱位置
/var/lib/mysql/		默认安装的mysql的库文件目录
/ost+found /		分区的复目录（当系统非正常关机时，可以进行恢复，每个分区都会有这样一个lost+found目录）系统不正常产生错误时，会将一些遗失的片段放置于此目录下，通常这个目录会自动出现在装置目录下。例如你加装一棵硬盘于 /disk 中，那在这个目录下就会自动产生一个这样的目录 /disk/lost+found
```

## /etc/目录
特定主机系统范围内的配置文件：

```
/etc/rc /etc/rc.d		启动、或改变运行级时运行的scripts或scripts的目录.
/etc/rc*.d
/etc/hosts			本地域名解析文件
/etc/sysconfig/network	IP、掩码、网关、主机名配置
/etc/resolv.conf		DNS服务器配置
/etc/fstab			开机自动挂载系统，所有分区开机都会自动挂载
/etc/inittab	设定系统启动时Init进程将把系统设置成什么样的runlevel及加载相关的启动文件配置
/etc/exports			设置NFS系统用的配置文件路径
/etc/init.d			这个目录来存放系统启动脚本
/etc/profile,
/etc/csh.login,
/etc/csh.cshrc		全局系统环境配置变量
/etc/issue			认证前的输出信息，默认输出版本内核信息
/etc/motd			设置认证后的输出信息，
/etc/mtab		当前安装的文件系统列表.由scripts初始化，并由mount 命令自动更新.需要一个当前安装的文件系统的列表时使用，例如df 命令
/etc/group			类似/etc/passwd ，但说明的不是用户而是组.
/etc/passwd		用户数据库，其中的域给出了用户名、真实姓名、家目录、加密的口令和用户的其他信息.
/etc/shadow		在安装了影子口令软件的系统上的影子口令文件.影子口令文件将
/etc/passwd 		文件中的加密口令移动到/etc/shadow 中，而后者只对root可读.这使破译口令更困难.
/etc/sudoers			可以sudo命令的配置文件
/etc/syslog.conf		系统日志参数配置
/etc/login.defs		设置用户帐号限制的文件
/etc/securetty		确认安全终端，即哪个终端允许root登录.一般只列出虚拟控制台，这样就不可能(至少很困难)通过modem或网络闯入系统并得到超级用户特权.
/etc/printcap			类似/etc/termcap ，但针对打印机.语法不同.
/etc/shells		列出可信任的shell.chsh 命令允许用户在本文件指定范围内改变登录shell.提供一台机器FTP服务的服务进程ftpd 检查用户shell是否列在 /etc/shells 文件中，如果不是将不允许该用户登录.
/etc/xinetd.d		如果服务器是通过xinetd模式运行的，它的脚本要放在这个目录下。有些系统没有这个目录，比如Slackware，有些老的版本也没有。在Redhat Fedora中比较新的版本中存在。
/etc/opt/			/opt/的配置文件
/etc/X11/			X_Window系统(版本11)的配置文件
/etc/sgml/			SGML的配置文件
/etc/xml/			XML的配置文件
/etc/skel/			默认创建用户时，把该目录拷贝到家目录下
```

## /usr/目录
默认软件都会存于该目录下。用于存储只读用户数据的第二层次；包含绝大多数的用户工具和应用程序。
## /var/目录
/var 包括系统一般运行时要改变的数据.每个系统是特定的，即不通过网络与其他计算机共享.
## /proc/目录
虚拟文件系统，将内核与进程状态归档为文本文件（系统信息都存放这目录下）。

例如：uptime、 network。在Linux中，对应Procfs格式挂载。该目录下文件只能看不能改（包括root）
## /dev/目录
设备文件分为两种：
- 块设备文件(b)
- 字符设备文件(c)
