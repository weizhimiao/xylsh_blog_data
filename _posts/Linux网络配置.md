---
title: Linux网络配置
date: 2016-10-06 22:30:00
categories:
- Linux
---

![Linux网络配置](http://n.sinaimg.cn/games/3ece443e/20161007/ifconfig.png)


<!-- more -->

##虚拟机网卡的配置（IP地址的配置）
### IP地址的配置
给Linux配置IP
```
setup  进行IP地址的设置
service network restart
```
手工启动Linux网卡
```
vi  /etc/sysconfig/network-scripts/ifcfg-eth0		编辑网卡配置文件
```
配置文件内容如下：
```
DEVICE=eth0
IPADDR=192.168.140.158
GATEWAY=192.168.140.1
NETMASK=255.255.255.0
HWADDR=00:0c:29:41:c7:1f
UUID="60bfdea1-c598-4dc8-bf93-da162ea4fb41"
TYPE=Ethernet
BOOTPROTO=none
IPV6INIT=no
ONBOOT=yes
USERCTL=no
```
修改如下配置项：
```
ONBOOT=no		改为
ONBOOT=yes		（是否开机启动）
```
修改UUID
> 解决虚拟机网卡不通：（需要进行如下修改）

```
vi  /etc/sysconfig/network-scripts/ifcfg-eth0
```
删除MAC地址行
```
rm –rf /etc/udev/rules.d/70-persistent-net.rules
```
删除网卡和MAC地址绑定文件
重启系统
修改虚拟机配置
> 网卡连接方式改为桥接模式
> 确定桥接到有线网卡上
（桥接方式是虚拟机和真实机共享使用本机的真实物理网卡，故采用这种方式vMare0和8的这两块虚拟网卡可以同时禁掉）

## Linux网络配置
### IP地址配置（3种方法，推介使用地3种）
1.	setup
```
  	service   network  restart
```
2.	ifconfig  eth0  ip    netmask   	临时生效
3.	修改网卡配置文件

**网卡信息文件：`/etc/sysconfig/network-scripts/ifcfg-eth0` 文件内容如下：**
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
如果是采用DHCP方式获取IP信息，则可以只填写上述用橙色标记的配置项即可。
修改配置文件后，需要重启服务（如service  network  restart）

**主机名配置文件**
```
/etc/sysconfig/network		永久生效，但是要重启
```
文件内容如下：
```
NETWORKING=yes
HOSTNAME=localhost.localdomain
```
另外，还可以用命令进行修改，不过只能临时生效
```
	hostname  sc		将用户名修改为sc
	hostname		查看主机名
```

**DNS配置文件**
```
		/etc/resolv.conf
```
文件内容如下：
```
nameserver  202.106.0.20
如有多个DNS服务器地址，可在IP地址后面直接加，并以“，”分割，或者再起下一行写入“nameserver  xx.xx.xx.xx”
```

###	网络命令
1.	ifconfig		查看网卡信息
>	-a 	全部（包括没有启动的）

2.	ifup  eth0		快速开启
>	ifdown    eth0		快速关闭

3.	netstat   -an		查看所有网络连接
> netstat  -tlun		查看TCP和UDP协议监听的端口
>
>	netstat   -rn		查看路由default：默认路由（网关）

4.	route		查看路由
>	route  add   default   gw    192.168.140.1 	手工设定网关，临时生效
>	route  del    default   gw  192.168.190.6   	删除网关

5.	ping	IP	探测网络畅通

6.	traceroute    ip或域名		探测到目的地址的路径（Linux命令）
>	tracert   ip   	（windows命令）

7.	tcpdump
>	tcpdump -i eth0 -nnX port 21		抓取eth0网卡  21端口的传输信息
