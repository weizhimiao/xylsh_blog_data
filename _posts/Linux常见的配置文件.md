---
title: Linux常见的配置文件
date: 2016-10-06 20:00:00
tags:
categories:
- Linux
---

在Linux中我们经常需要修改各种配置文件，例如，启动引导程序配置文件、系统启动文件核脚本、网络配置文件、文件系统配置文件、文件服务程序配置文件等等。

<!-- more -->

## /etc/passwd	用户信息配置文件

- 文件作用：保存Linux用户信息
- 文件内容：
```
root:x:0:0:root:/root:/bin/bash
```

说明：
- 第一位：用户名
- 第二位：密码位（只是一个密码标志，真实密码并不是存在这）
- 第三位：UID	用户ID
>	0	超级管理员
>	1-499	系统用户（伪用户）
>	\>500	普通用户

- 第四位：GID	初始组ID
- 第五位：用户说明
- 第六位：家目录
- 第七位：用户登录之后的权限
> 常见的两种权限：
>
>	/bin/bash
>
>	/sbin/nologin伪用户，不能进行登陆

## /etc/shadow	影子文件
- 文件作用：保存用户密码，及密码的有效期
- 文件内容：
```
root:$6$w5RBeFe7Y1bixrQR$tp0zHL1bMmAVv0SAS56LsCOyZ4KUj3V0GI0dRZ4KSlf.ggisV7dwiQ8s5xebcZghVDxwlYTjN6qKGU9zRc.En1:16247:0:99999:7:::
```

说明：
- 第一字段：用户名（也被称为登录名），在/etc/shadow中，用户名和/etc/passwd 是相同的，这样就把passwd 和shadow中用的用户记录联系在一起；这个字段是非空的

- 第二字段：密码（已被加密），如果是有些用户在这段是x，表示这个用户不能登录到系统；这个字段是非空的

- 第三字段：上次修改口令的时间；这个时间是从1970年01月01日算起到最近一次修改口令的时间间隔（天数），您可以通过passwd 来修改用户的密码，然后查看/etc/shadow中此字段的变化

- 第四字段：两次修改口令间隔最少的天数；如果设置为0,则禁用此功能；也就是说用户必须经过多少天才能修改其口令；此项功能用处不是太大；默认值是通过/etc/login.defs文件定义中获取，PASS_MIN_DAYS 中有定义

- 第五字段：两次修改口令间隔最多的天数；这个能增强管理员管理用户口令的时效性，应该说在增强了系统的安全性；如果是系统默认值，是在添加用户时由/etc/login.defs文件定义中获取，在PASS_MAX_DAYS 中定义

- 第六字段：提前多少天警告用户口令将过期；当用户登录系统后，系统登录程序提醒用户口令将要作废；如果是系统默认值，是在添加用户时由/etc/login.defs文件定义中获取，在PASS_WARN_AGE 中定义

- 第七字段：在口令过期之后多少天禁用此用户；此字段表示用户口令作废多少天后，系统会禁用此用户，也就是说系统会不能再让此用户登录，也不会提示用户过期，是完全禁用

- 第八字段：用户过期日期；此字段指定了用户作废的天数（从1970年的1月1日开始的天数），如果这个字段的值为空，帐号永久可用

- 第九字段：保留字段，目前为空，以备将来Linux发展之用

## /etc/group	用户组配置文件
- 文件作用：用户组配置
- 文件内容：
```
root:x:0:
bin:x:1:bin,daemon
```
说明：
- 组名：组密码位：组ID：组中的附加用户

- 初始组：每个用户初始组只能有一个，初始组只能有一个，一般都是和用户名相同的组作为初始组

- 附加组：每个用户可以属于多个附加组。要把用户加入组，都是加入附加组

## /etc/sysconfig/network-scripts/ifcfg-eth0	网卡信息文件
- 文件作用：保存网卡配置
- 文件内容：
```
DEVICE=eth0					网卡设备名
BOOTPROTO=none				是否自动获取IP。none：不生效					static：手动		dhcp：动态获取IP
BROADCAST=192.168.140.255			广播地址
HWADDR=00:0c:29:21:80:48			mac地址
IPADDR=192.168.140.253			IP地址
IPV6INIT=yes					IPv6开启
IPV6_AUTOCONF=yes				IPv6获取
NETMASK=255.255.255.0				掩码
NETWORK=192.168.140.0				网段
ONBOOT=yes					网卡开机启动
TYPE=Ethernet					以太网
GATEWAY=192.168.140.1				网关
```

说明：
- 如果是采用DHCP方式获取IP信息，则可以只填写上述用橙色标记的配置项即可。

- 修改配置文件后，需要重启服务（如service  network  restart）。

## /etc/udev/rules.d/70-persistent-net.rules	网卡和MAC地址绑定文件
- 文件作用：绑定网卡和MAC地址
- 文件内容：
```
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:41:c7:1f", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
```

说明：
- 当虚拟机网卡不通是，有时需要删除`/etc/sysconfig/network-scripts/ifcfg-eth0` 中的MAC地址行，并且要删掉此文件。

## /etc/inittab	系统默认运行级别配置文件
- 文件作用：设置默认系统运行级别。
- 文件内容：
```
id:3:initdefault:
```

说明：
```
运行级别可分为：
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode	命令行
#   4 - unused
#   5 - X11			图形界面
#   6 - reboot (Do NOT set initdefault to this)
```

## /etc/rc.local---->/etc/rc.d/rc.local	自启动程序配置文件
- 文件作用：设置自启动程序，系统启动时会一次执行该文件中的命令
- 文件内容
```
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
ulimit -SHn 65535
```

## /etc/services	所有系统常见端口
- 文件作用：显示系统常见的端口及绑定的协议


## /etc/sysconfig/network		主机名配置文件
- 文件作用：配置主机名，永久保存
- 文件内容：
```
NETWORKING=yes
HOSTNAME=localhost.localdomain
```

说明：
- 建议不要更改。

## /etc/resolv.conf		DNS配置文件
- 文件作用：设置ＤＮＳ服务器
- 文件内容：
```
nameserver  202.106.0.20
```

说明：
- 如有多个DNS服务器地址，可在IP地址后面直接加，并以“，”分割，或者再起下一行写入“nameserver  xx.xx.xx.xx”

## /etc/vimrc vim配置文件
别名配置文件
```
~.bashrc
```

## /etc/selinux/config seLinux的配置文件
- 文件内容
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

关闭seLinux
- 将配置文件中seLinux项改为SELINUX=disabled，然后重启Linux。
