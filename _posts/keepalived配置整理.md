---
title: keepalived配置整理
date: 2017-02-11 21:30:00
tags:
- keepalived
categories:
- Linux
---

keepalived只有一个配置文件 keepalived.conf ，里面主要包括以下几个配置区域，分别是
- **global_defs**
> 主要是配置故障发生时的通知对象以及机器标识

- static_ipaddress
> 配置的是是本节点的IP信息。（如果当前服务器上已经配置了IP，那么这这个区域可以不用配置）

- static_routes
> 配置的是是本节点的路由信息。（如果当前服务器上已经配置了路由，那么这这个区域可以不用配置）

- vrrp_script
> 用来做健康检查的，当时检查失败时会将`vrrp_instance`的`priority`减少相应的值。

- **vrrp_instance**
> 用来定义对外提供服务的VIP区域及其相关属性。

- vrrp_sync_group
> 用来定义`vrrp_intance`组，使得这个组内成员动作一致。

- **virtual_server**
> 虚拟服务器，来源`vrrp_instance` 中配置的 的虚拟IP地址，后面加空格加端口号

- virtual_server_group
> 用来定义`virtual_server` 组，一般在超大型的LVS中用到，一般LVS用不过这东西。



<!-- more -->

## 配置说明：

- 注释以“＃”或“！”开头到行尾，可以从一行的任何位置开始。
- 关键字 `include` 允许在主配置文件中包含其他配置文件。
- keepalived 还支持条件配置。(根据在启动 keepalived 时的命令行选项 -i 后面的参数，来确定指定命令是否生效)
> 这样做的目的是允许单个配置文件用于多个系统，keepalived 中多个系统上的配置文件有区别的配置，可能只有以下几个配置，
> - router_id，
> - vrrp实例优先级，
> - 以及可能的接口名称。


条件配置具体方法，示例
```
global_defs
{
  @main router_id main_router
  @backup router_id backup_router
}}
```
以“@”开头的任何配置行都是条件配置行。 '@'字符紧接着的（即没有任何空格）字符串，会与使用 `-i` 命令行选项指定的字符串进行比较，如果不匹配匹配，则这条配置行被会被忽略。如，上面的配置，当使用 `/usr/local/keepalived/sbin/keepalived -f /etc/keepalived/keepalived.conf -i main` 启动 `keepalived` 时，则 `router_id` 将设置为 `main_router`。当使用 `-i backup` 参数启动时，则 `router_id` 将设置为 `backup_router`。

**注意**：如果不使用 `-i` 或者 `-i` 后面是其他不匹配的参数启动时，则 `router_id` 将不会被设置。

## 全局配置
包含的子块有，
- Global definitions，
- static_ipaddress，
- static_routes

### Global definitions

#### global_defs 区块
示例：
```
global_defs {
   # 故障发生时给谁发邮件通知。
   notification_email {
     acassen@firewall.loc
     # ....
   }
   # 通知邮件从哪个地址发出。
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30

   router_id LVS_DEVEL

   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
```
- notification_email 故障发生时给谁发邮件通知。
- notification_email_from 通知邮件从哪个地址发出。
- smtp_server 通知邮件的smtp地址。
> 格式：smtp_server 127.0.0.1 [<PORT>] # 带可选端口号的IP地址或域名（默认为25）

- smtp_helo_name
> 格式：smtp_helo_name <HOST_NAME> # 在HELO消息中使用的名称（默认为本地主机名）

- smtp_connect_timeout 连接smtp服务器的超时时间。
- router_id 标识本节点的字条串，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到。


- vrrp_mcast_group4 224.0.0.18 # 可选, 默认为 224.0.0.18
- vrrp_mcast_group6 ff02::12   # 可选, 默认为 ff02::12
- default_interface p33p1.3    # 设置静态地址的默认接口，默认为eth0

-  lvs_sync_daemon <INTERFACE> <VRRP_INSTANCE> [id <SYNC_ID>] [maxlen <LEN>] [port <PORT>] [ttl <TTL>] [group <IP ADDR>]
> - ＃绑定接口，vrrp实例  
> - ＃syncid for lvs syncd
> - ＃syncid（0 to 255）for lvs syncd
> - ＃maxlen（1..65507）最大包长度
> - ＃port（1..65535）要使用的UDP端口号
> - ＃ttl（1..255）
> - ＃group - 组播组地址（IPv4或IPv6）
> - ＃注意：maxlen，port，ttl和group仅在Linux 4.3或更高版本上可用。

- lvs_flush＃在启动时刷新任何现有的LVS配置

- vrrp_garp_master_delay 10    # 秒，默认值5 ,0表示没有设置秒数
> 在转换到MASTER之后的第二组ARP的延迟

- vrrp_garp_master_repeat 1    # default 5
> 在转换到MASTER之后一次发送的ARP消息的数量

- vrrp_garp_lower_prio_delay 10
> 当 MASTER 接收的较低优先级的 advert 延迟第二组ARP发送

- vrrp_garp_lower_prio_repeat 1
> 当MASTER接收到较低优先级的广播之后，一次发送的RP消息的数量

- vrrp_garp_master_refresh 60  # secs, default 0 (no refreshing)
> MASTER 刷新`gratuitous ARPs`的最小时间间隔

- vrrp_garp_master_refresh_repeat 2 # default 1
> 在 MASTER 状态时发送的`gratuitous ARPs`消息的数量

- vrrp_garp_interval 0.001          # decimal, seconds (resolution usecs). Default 0.
> 接口发送的`gratuitous ARPs`的延迟时间（毫秒）

- vrrp_gna_interval 0.000001        # decimal, seconds (resolution usecs). Default 0.
> 接口发送的 主动NA消息 之间的延迟（ms）

- vrrp_lower_prio_no_advert [<BOOL>]
> 如果收到较低优先级的广告，请不要发送另一个广告。 默认为false，除非设置了strict_mode。

- vrrp_version <2 or 3>        # default version 2
> 设置要使用的默认VRRP版本

- vrrp_iptables keepalived     # default INPUT
> - ＃指定 iptables链 以确保版本3实例不对其不拥有的地址做出响应。
> - ＃注意：指定的 链 必须存在于 `iptables`/`ip6tables`配置中，并且链从`iptables`配置中的适当点调用。
> - ＃可能需要在接受任何`ESTABLISHED`，`RELATED`数据包之后进行此过滤，因为`IPv4`可能选择`VIP`作为传出连接的源地址。

- vrrp_iptables keepalived_in keepalived_out
> - ＃或用于出站过滤
> - ＃注意，出站过滤将不适用于IPv4，因为可以选择VIP作为出站连接的源地址。 对于IPv6，由于地址已过时，因此不太可能。

- vrrp_iptables
> 或者不添加任何iptables规则：

- vrrp_ipset [keepalived [keepalived6 [keepalived_if6]]]
> - ＃Keepalived可以选择使用iptables结合使用ipetsets。
> - ＃如果是这样，那么可以指定 ipset 名称，默认如下。
> - ＃如果未指定名称，则不使用 ipsets，否则将通过向先前指定的名称添加“`_if`”和/或“`6`”来构造任何省略的名称。



- vrrp_check_unicast_src
> ＃以下启用检查在单播模式下，VRRP数据包的源地址是我们的单播对等体之一。


- vrrp_skip_check_adv_addr     # Default - don’t skip
> - ＃查看接收到的VRRP报文中的所有地址可能比较耗时。
> - ＃设置此标志表示如果广告来自与接收的上一个广告相同的主路由器，则不执行检查。


- vrrp_strict
> ＃严格遵守VRRP协议。 这将禁止：
> - 0 VIPs
> - unicast peers (单播对等体)
> - IPv6 addresses in VRRP version 2(VRRP版本2中的IPv6地址)

----

如果vrrp或检查程序超时，则可以使用以下4个选项。 这可以通过备份vrrp实例成为主，即使是因为主或备份系统太忙，无法处理vrrp数据包，但主服务器仍然运行的情况。

- vrrp_priority <-20 to 19>    # 设置vrrp子进程优先级 。 （负值增加优先级）
- checker_priority <-20 to 19> # 设置 子进程检查器优先级
- vrrp_no_swap                 # 设置vrrp子进程不可交换
- checker_no_swap              # 设置 子进程检查器不可交换

----

如果 `Keepalived` 已构建与SNMP支持，以下关键字可用
> 注意：Keepalived, checker 和 RFC支持 可以单独启用/禁用

- snmp_socket udp:1.2.3.4:705  # 指定用于连接到SNMP主代理的套接字（默认`unix:/var/agentx/master`）
                               # 除非使用网络命名空间，默认为udp：`udp:localhost:705`
- enable_snmp_keepalived       # 启用SNMP处理KEEPALIVED MIB的vrrp元素
- enable_snmp_checker          # 启用SNMP处理KEEPALIVED MIB的checker元素
- enable_snmp_rfc              # 启用SNMP处理 RFC2787 和 RFC6527 VRRP MIB
- enable_snmp_rfcv2            # 启用SNMP处理 RFC2787 VRRP MIB
- enable_snmp_rfcv3            # 启用SNMP处理 RFC6527 VRRP MIB
- enable_traps                 # 启用SNMP陷阱

---

如果Keepalived已构建与DBus支持，以下关键字可用
- enable_dbus                  # 启用DBus接口

- script_user username [groupname] # 如果未指定groupname，则其默认为用户的组
> - ＃指定要在其下运行脚本的默认`用户名`/`组名`。
> - ＃如果未指定此选项，则用户默认为 `keepalived_script`（如果该用户存在），否则为`root`。

- enable_script_security       # 如果非必要，请不要配置为以`root`身份运行的脚本。


#### 其它全局配置

- net_namespace NAME         # 用于在单独的网络命名空间中运行keepalived
> - ＃设置运行的网络命名空间`/var/run/keepalived `目录将创建为非共享挂载点，例如pid文件。
> - syslog条目将`_NAME`附加到 `ident`。
> **注意：** 不能在配置重新加载时更改命名空间


- namespace_with_ipsets        
> ＃`Linux 3.13`之前，网络命名空间找不到 `ipetsets` ，因此如果运行在早期版本的内核，默认情况下，如果使用命名空间和`vrrp_ipsets`没有指定， `ipsets` 是被禁用的。 此选项覆盖默认值，并允许将 `ipsets` 与 `3.11` 之前的内核上的命名空间一起使用。

- instance NAME               
> ＃如果 `keepalived` 的多个实例在同一命名空间中运行，这将创建 pid 文件，其中 NAME 是文件名的一部分，在`/var/run/keepalived`中。
>
> 注意：不能在配置重新加载时更改实例名称


- use_pid_dir              # 在 `/var/run/keepalived` 创建 `pid` 文件

- linkbeat_use_polling         # 轮询检测媒体链路故障，否则尝试使用 `ETHTOOL` 或 `MII` 接口

### Static routes/addresses/rules

Keepalived可以配置静态地址，路由和规则。 如果我们的计算机上已有IP和路由，并且可以互相ping通，则不需要此部分。
规则和路由的语法与`ip rule add/ip route add`相同。

```
static_ipaddress
{
        192.168.1.1/24 dev eth0 scope global
        ...
}
```
```
static_routes
{
  192.168.2.0/24 via 192.168.1.100 dev eth0
  192.168.100.0/24 table 6909 nexthop via 192.168.101.1 dev wlan0 onlink weight 1  nexthop  via  192.168.101.2  dev  wlan0 onlink weight 2
  192.168.200.0/24  dev  p33p1.2 table 6909 tos 0x04 protocol bird scope link priority 12 mtu 1000 hoplimit 100 advmss 101 rtt 102 rttvar 103 reordering 104 window 105 cwnd 106 ssthresh lock 107 realms PQA/0x14 rto_min 108 initcwnd 109 initrwnd 110 features ecn
  2001:470:69e9:1:2::4  dev  p33p1.2 table 6909 tos 0x04 protocol bird scope link priority 12 mtu 1000 hoplimit 100 advmss 101 rtt 102 rttvar 103 reordering 104 window 105 cwnd 106 ssthresh lock 107 rto_min 108 initcwnd 109  initrwnd  110  features ecn
  ...
}
```
```
static_rules
{
  from 192.168.2.0/24 table 1
  to 192.168.2.0/24 table 1
  from  192.168.28.0/24  to 192.168.29.0/26 table small iif p33p1 oif wlan0 tos 22 fwmark 24/12 preference 39 realms 30/20 goto 40
  to 1:2:3:4:5:6:7:0/112 from 7:6:5:4:3:2::/96 table 6908
  ...
}
```
## VRRPD配置

包含的子块有，
- VRRP script(s),VRRP脚本
- VRRP synchronization group(s), VRRP同步组
- VRRP gratuitous ARP ,
- unsolicited neighbour advert delay group(s), 主动邻居广播延迟组
- VRRP instance(s),VRRP实例

### VRRP script(s)
添加要定期执行的脚本。 VRRP instances 会根据 脚本的退出码来调整优先级
```
vrrp_script <SCRIPT_NAME> {
  script <STRING>|<QUOTED-STRING> # 脚本的路径执行
  interval <INTEGER>  # 脚本调用之间的间隔，默认1秒
  timeout <INTEGER>   # 脚本运行超时时间
  weight <INTEGER:-254..254>  # 按此权重调整优先级，默认为2
  rise <INTEGER>              # 转换为OK状态，所需的成功数量
  fall <INTEGER>              # 转换为KO状态，所需的成功数量
  user USERNAME [GROUPNAME]   # 运行脚本的 用户/组名 （组默认为用户组）
  init_fail                   # 假设脚本最初处于失败状态
}
```

### VRRP synchronization group(s)，VRRP同步组
```
#VG_1, 一起故障转移的IP组的名称
vrrp_sync_group VG_1 {
   group {
     inside_network   # vrrp_instance的名称（见下文）
     outside_network  # One for each movable IP
     ...
   }

   # 通知脚本和警报（可选）
   #
   ＃filenames的脚本在转换时运行，可以不加引号（如果只是文件名）或加引号（如果它有参数）用户名和组名指定脚本应该运行的用户和组。
   ＃如果指定了username，则组默认为用户的组。
   ＃如果未指定username，则它们默认为全局 script_user 和 script_group 到 MASTER 转换

   notify_master /path/to_master.sh [username [groupname]]
   # 转换成 BACKUP 状态
   notify_backup /path/to_backup.sh [username [groupname]]
   # 转换成 FAULT
   notify_fault "/path/fault.sh VG_1" [username [groupname]]

   # for ANY state transition.
   # "notify" script is called AFTER the
   # notify_* script(s) and is executed
   # with 3 arguments provided by Keepalived
   # (so don’t include parameters in the notify line).
   # arguments

   ＃ 对于任何状态的转换。
   ＃ “notify”脚本在 notify_* 脚本之后调用，并使用 Keepalived 提供的3个参数执行（不包括通知行中的参数）。
   ＃ 参数：
   #    $1 = "GROUP"|"INSTANCE"
   #    $2 = 组或实例的名称
   #    $3 = 转型目标状态 ("MASTER"|"BACKUP"|"FAULT")
   notify /path/notify.sh [username [groupname]]

   # 在状态转换期间使用global_defs中的地址发送电子邮件通知。
   smtp_alert

   global_tracking     # 所有VRRP共享相同的跟踪配置
}
```
### VRRP gratuitous ARP and unsolicited neighbour advert delay group(s)

指定发送 gratuitous ARPs 和未经请求的邻居 advert 的延迟的设置。 这是为了防止上游交换机无法处理 ARPs/NAs 泛滥。
- 当限制适用于单个物理接口时使用接口。
- 当一组接口链接到同一交换机时使用接口，则限制适用于整个交换机。

如果设置了全局`vrrp_garp_interval`或`vrrp_gna_interval`，那么在`garp_group`中未指定的任何接口时，将继承全局设置。
```
garp_group {
  # 设置免费ARP之间的间隔（以秒，微秒为单位）
  garp_interval <DECIMAL>
  # 设置非请求 NA 之间的默认间隔（以秒，微秒为单位）
  gna_interval <DECIMAL>
  # 应用间隔的物理接口
  interface <STRING>
  # 聚合延迟的接口列表。
  interfaces {
     <STRING>
     <STRING>
     ...
     }
}
```
### VRRP instance(s)
描述了`vrrp_sync_group`中组的每个实例的IP。 这里描述两个IP（在`inside_network`和在`outer_network`），在机器“`my_hostname`”，属于组`VG_1`并且将在任何状态更改时一起转换。

```
# 除inside_network外，还需要为outer_network编写另一个块。
vrrp_instance inside_network {
  # 初始状态，MASTER | BACKUP
  # 一旦其他机器启动，将进行选举，具有最高优先级的机器将变为MASTER。
  # 所以这里的配置并不重要。
  state MASTER

  # 为 inside_network 配置网络接口, 由 vrrp 绑定
  interface eth0

  # 使用VRRP虚拟MAC。
  use_vmac [<VMAC_INTERFACE>]

  # 从基本接口（而不是VMAC接口）发送/恢复VRRP消息
  vmac_xmit_base

  native_ipv6         # 强制实例使用IPv6（混合IPv4和IPv6配置时）。

  # 忽略VRRP接口故障（默认未设置）
  dont_track_primary

  # 可选，监视
  # 如果任何一个下降，进入FAULT状态。
  track_interface {
    eth0
    eth1
    eth2 weight <-254..254>
    ...
  }

  # 向接口添加跟踪脚本（<SCRIPT_NAME>是vrrp_script条目的名称）
  track_script {
      <SCRIPT_NAME>
      <SCRIPT_NAME> weight <-254..254>
  }

  # 默认 vrrpd 绑定的 IP 是接口上的主 IP 。 （可选的）
  # 如果要隐藏 vrrpd 的位置，请将此 IP 当做 src_addr 用于组播或单播 vrrp 数据包。 （
  # 因为它是多播，vrrpd 将获得答复数据包，无论使用什么 src_addr ）。
  mcast_src_ip <IPADDR>
  unicast_src_ip <IPADDR>

  version <2 or 3>            # VRRP版本在接口上运行
                              # default是全局参数vrrp_version。

  # 不要通过 VRRP 组播组发送 VRRP adverts
  # 相反，它使用单播将 adverts 发送到以下IP地址列表。 在不支持多播的网络环境中使用VRRP FSM和功能可能很酷！
  # 指定的IP地址可以是IPv4和IPv6。
  unicast_peer {
    <IPADDR>
    ...
  }
  # 接口特定设置，与全局参数相同; 默认为全局参数
  garp_master_delay 10
  garp_master_repeat 1
  garp_lower_prio_delay 10
  garp_lower_prio_repeat 1
  garp_master_refresh 60
  garp_master_refresh_repeat 2
  garp_interval 100
  gna_interval 100

  lower_prio_no_advert [<BOOL>]

  # 从0到255的任意唯一编号，用于区分在同一NIC（和相同的套接字）上运行的vrrpd的多个实例。
  virtual_router_id 51

  # 用于选择MASTER，最高优先级的会被选举出来。
  # 超过50个以上的其它机器的设置，会被选为 MASTER
  priority 100

  # VRRP广告间隔（以秒为单位）（例如0.92）（使用默认值）
  advert_int 1

  # 注意：在2004年，RFC3768 从 VRRPv2 规范中删除了身份验证。
  # 此选项的使用不符合规定，可能会导致问题;
  # 所以尽可能避免使用，除非使用单播。
  authentication {     # 验证块
      # PASS||AH
      #   PASS - 简单密码（建议）
      #   AH - IPSEC（不推荐）
      auth_type PASS
      # 访问vrrpd的密码。
      # 应在所有机器上相同。
      # 只使用前八（8）个字符。
      auth_pass 1234
  }

  # 设置虚拟IP地址，所有的机器上应该都使用相同的配置
  virtual_ipaddress {
      <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
      192.168.200.17/24 dev eth1
      192.168.200.18/24 dev eth2 label eth2:1
  }

  # 从可选的VRRP中排除VRRP IP。
  # 对于在同一接口上有大量（例如200）的IP的情况。 要减少广告中发送的数据包数量，我们可以从广告中排除大多数IP。

  # 为virtual_ipaddress添加或者删除。
  # 因为virtual_ipaddress中的所有地址必须是同一系列，所以如果我们希望能够添加IPv4和IPv6地址的混合，也可以使用，virtual_ipaddress_excluded 来进行配置。
  virtual_ipaddress_excluded {
   <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE>
   <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE>
      ...
  }


  # 设置接口上的promote_secondaries标志，以便在删除其中一个时，停止删除同一CIDR中的其他地址。例如，如果在接口上同时配置了10.1.1.2/24和10.1.1.3/24，一旦删除了一个，除非接口上设置了 promote_secondaries 标志，否则其他地址也将会被删除。
  prompte_secondaries

  # 当转换成 MASTER 时添加下面的routes，当转换成 BACKUP 时，会删除下面的routes 配置
  # 有关详细信息，请参阅static_routes
  virtual_routes {
      # src <IPADDR> [to] <IPADDR>/<MASK> via|gw <IPADDR> [or <IPADDR>] dev <STRING> scope <SCOPE> table <TABLE>
      src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
      192.168.110.0/24 via 192.168.200.254 dev eth1
      192.168.111.0/24 dev eth2
      192.168.112.0/24 via 192.168.100.254
      192.168.113.0/24 via 192.168.200.254 or 192.168.100.254 dev eth1
      blackhole 192.168.114.0/24
      0.0.0.0/0 gw 192.168.0.1 table 100  # To set a default gateway into table 100.
  }

  # 当转换成 MASTER 时添加下面的 rules ，当转换成 BACKUP 时，会删除下面的 rules 配置
  # 有关详细信息，请参阅static_rules
  virtual_rules {
      from 192.168.2.0/24 table 1
      to 192.168.2.0/24 table 1
  }

  # VRRPv3有一个 接受模式，以允许虚拟路由器在没有地址所有者时接收发往VIP的数据包。 这是默认设置，除非设置了strict模式。
  # 作为扩展，这也适用于VRRPv2（RFC 3768没有定义接受模式）。
  accept          # 接受数据包到非地址所有者
  no_accept       # 丢弃数据包到非地址所有者。

  # 当较高优先级机器联机时，VRRP通常抢占较低优先级机器。 “nopreempt”允许较低优先级机器维护主机角色，即使较高优先级机器恢复在线时也是如此。
  # 注意：要使其工作，此条目的初始状态必须为BACKUP。
  nopreempt
  preempt             # 用于向后兼容


   # 请参见全局vrrp_skip_check_adv_addr的描述，它设置默认值。 默认为vrrp_skip_check_adv_addr
   skip_check_adv_addr [on|off|true|false|yes|no]      # 如果没有指定，默认开启

   # 请参见全局vrrp_strict的描述
   # 如果未指定vrrp_strict，则它使用vrrp_strict的值
   # 如果指定了不带参数的strict_mode，则其默认为on
   strict_mode [on|off|true|false|yes|no]

   # 启动后的秒数或看到较低优先级主机直到抢占（如果未由“nopreempt”禁用）。
   # Range: 0 (default) to 1000
   # 注意：要使其工作，此条目的初始状态必须为BACKUP。
   preempt_delay 300    # 等待5分钟

   # 调试级别，尚未实现。
   debug <LEVEL>        # LEVEL是0到4范围内的数字

   # 通知脚本，警报如上
   notify_master <STRING>|<QUOTED-STRING> [username [groupname]]
   notify_backup <STRING>|<QUOTED-STRING> [username [groupname]]
   notify_fault <STRING>|<QUOTED-STRING> [username [groupname]]
   notify_stop <STRING>|<QUOTED-STRING> [username [groupname]]      # 在停止vrrp时执行
   notify <STRING>|<QUOTED-STRING> [username [groupname]]
   smtp_alert
}

# 用于SSL_GET检查的参数。
# 如果未指定任何参数，则将自动生成SSL上下文。
SSL {
   password <STRING>   # 密码
   ca <STRING>         # ca文件
   certificate <STRING>  # 证书文件
   key <STRING>        # 密钥文件
}          

```

## LVS 配置

包含的子块有，
- Virtual server group(s) ,虚拟服务器组
- Virtual server(s) ,虚拟服务器

子块包含了 ipvsadm（8）的参数。  了解 ipvsadm（8）将会有助于我们的配置。

### Virtual server group(s) ,虚拟服务器组
```
# 可选配置项
# 此组允许real_server上的服务属于多个虚拟服务，并且只进行一次健康检查。
# 仅适用于非常大的LVS。
virtual_server_group <STRING> {
       #VIP port
       <IPADDR> <PORT>
       <IPADDR> <PORT>
       ...
       # <IPADDR RANGE> 的格式为 XXX.YYY.ZZZ.WWW-VVV，例如 192.168.200.1-10 范围包括.1到.10地址
       <IPADDR RANGE> <PORT># VIP  VPORT 范围
       <IPADDR RANGE> <PORT>
       ...
       fwmark <INT>  # fwmark
       fwmark <INT>
       ...  
}
```
### Virtual server(s) ,虚拟服务器
virtual_server是一个`vip vport (IPADDR PORT pair)` 或  `fwmark <INT>` 的声明。
```
(virtual server) group <STRING>

# 设置服务
virtual_server IP port |
virtual_server fwmark int |
virtual_server group string
{
# 延迟定时器用于服务轮询
delay_loop <INT>

# LVS调度器
lb_algo rr|wrr|lc|wlc|lblc|sh|dh

# 启用散列条目
hashed
# 启用 flag-1 调度器 (-b flag-1 in ipvsadm)
flag-1
# 启用 flag-2 调度器(-b flag-2 in ipvsadm)
flag-2
# 启用 flag-3 调度器 (-b flag-3 in ipvsadm)
flag-3
# 启用 sh-port 调度器 (-b sh-port in ipvsadm)
sh-port
# 启用 sh-fallback 调度器  (-b sh-fallback in ipvsadm)
sh-fallback

# 为UDP启用单分组调度（在ipvsadm中为-O）
ops
# LVS转发方法
lb_kind NAT|DR|TUN
# LVS持久超时，秒
persistence_timeout <INT>
# LVS粒度掩码（ipvsadm中的-M）
persistence_granularity <NETMASK>
# 仅实现TCP
protocol TCP
# 如果没有设置VS IP地址，则暂停healthchecker的活动
ha_suspend

# 用于HTTP_GET或SSL_GET的VirtualHost字符串
# 例如 virtualhost www.firewall.loc
virtualhost <STRING>

# 假定所有RS关闭,并且在启动时检查运行状况失败。
# 这有助于防止启动时的假的积极操作。 默认情况下禁用Alpha模式。
alpha

# 在守护程序关闭时，在适当的情况下，考虑降低仲裁和RS通知程序执行。
# 默认情况下禁用Omega模式。
omega

# Minimum total weight of all live servers in
# the pool necessary to operate VS with no
# quality regression. Defaults to 1.
quorum <INT>

# Tolerate this much weight units compared to the
# nominal quorum, when considering quorum gain
# or loss. A flap dampener. Defaults to 0.
hysteresis <INT>

# 获得 quorum 时启动的脚本。
quorum_up <STRING>|<QUOTED-STRING>

# 丢失quorum时启动的脚本。
quorum_down <STRING>|<QUOTED-STRING>

# 设置 realserver(s)

# 在所有 realserver 都关闭时生效
sorry_server <IPADDR> <PORT>
# 将 inhibit_on_failure 行为应用于前面的 sorry_server 指令
sorry_server_inhibit

# 每个realserver,一个条目
real_server <IPADDR> <PORT>
  {
      # 相对权重，默认：1
      weight <INT>
      # 健康检查器检测到故障时，将权重设置为0
      inhibit_on_failure

      # 当运行状况检查器将服务视为启动时启动的脚本。
      notify_up <STRING>|<QUOTED-STRING>
      # 当运行状况检查器将服务视为停机时启动的脚本。
      notify_down <STRING>|<QUOTED-STRING>

      # 选择一个健康检查
      # HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK

      # HTTP和SSL健康检查程序
      HTTP_GET|SSL_GET
      {
          # 一个url测试
          # 可以有多个条目
          url {
            # 例如 path / , or path /mrtg2/
            path <STRING>
            # healthcheck需要status_code或status_code和digest
            # 摘要用genhash计算
            # 例如 digest 9b3a0c85a887a256d6939da88aabd8cd
            digest <STRING>
            # HTTP标头中返回的状态代码
            # 例如 status_code 200
            status_code <INT>
          }
          # 获取重试次数
          nb_get_retry <INT>
          # 延迟后重试
          delay_before_retry <INT>

          # ======== 通用连接选项
          # 要连接的可选IP地址。
          # 默认值为真实服务器的IP
          connect_ip <IP ADDRESS>
          # 可选，端口，如果没有连接
          # 默认值为真实服务器的端口
          connect_port <PORT>
          # 用于发起连接的可选接口
          bindto <IP ADDRESS>
          # 用于发起连接的可选源端口
          bind_port <PORT>
          # 可选，连接超时（秒）。
          # 默认值为5秒
          connect_timeout <INTEGER>
          # 可选，用于标记所有传出检查程序包的fwmark
          fwmark <INTEGER>

          # 可选的，随机延迟最大N秒后开始初始检查。
          # 用于将多个同时检查分散到同一个RS。 默认情况下启用，最大值为delay_loop。 指定 0 为禁用
          warmup <INT>
      } # HTTP_GET|SSL_GET

      # TCP健康检查器（绑定到IP端口）
      TCP_CHECK
      {
          # ======== 通用连接选项
          # 可选，要连接的IP地址。
          # 默认值为真实服务器的IP
          connect_ip <IP ADDRESS>
          # 可选，连接端口
          # 默认值为真实服务器的端口
          connect_port <PORT>
          # 可选，发起连接使用的接口
          bindto <IP ADDRESS>
          # 可选，发起连接源端口
          bind_port <PORT>
          # 可选，连接超时（以秒为单位）。
          # 默认值为5秒
          connect_timeout <INTEGER>
          # 可选， 用于标记所有传出检查程序包的fwmark
          fwmark <INTEGER>

          # 可选， 随机延迟最大N秒后开始初始检查。
          # 用于将多个同时检查分散到同一个RS。 默认情况下启用，最大值为delay_loop。 指定 0 为禁用
          warmup <INT>
      } #TCP_CHECK

      # SMTP健康检查器
      SMTP_CHECK
      {
          # 可选，主机接口检查。
          # 如果没有主机指令，则只检查真实服务器的IP地址。
          host {
            # ======== 通用连接选项
            # 可选，要连接的IP地址。
            # 默认值为真实服务器的IP
            connect_ip <IP ADDRESS>
            # 可选，连接端口
            # 默认值为25
            connect_port <PORT>
            # 可选，发起连接使用的接口
            bindto <IP ADDRESS>
            # 可选，发起连接源端口
            bind_port <PORT>
            # 可选，每个主机连接超时。
            # Default is outer-scope connect_timeout
            connect_timeout <INTEGER>
            # 可选， 用于标记所有传出检查程序包的fwmark
            fwmark <INTEGER>
         }
         # 连接超时时间
         connect_timeout <INTEGER>
         # 重试失败检查的次数
         retry <INTEGER>
         # 重试前延迟秒
         delay_before_retry <INTEGER>
         # 用于smtp HELO请求的可选字符串
         helo_name <STRING>|<QUOTED-STRING>

         # 可选， 随机延迟最大N秒后开始初始检查。
         # 用于将多个同时检查分散到同一个RS。 默认情况下启用，最大值为delay_loop。 指定 0 为禁用
         warmup <INT>
      } #SMTP_CHECK

      # 运行外部程序，进行MISC健康检查
      MISC_CHECK
      {
          # 外部系统脚本或程序
          misc_path <STRING>|<QUOTED-STRING>
          # 脚本执行超时
          misc_timeout <INT>

          # 可选， 随机延迟最大N秒后开始初始检查。
          # 用于将多个同时检查分散到同一个RS。 默认情况下启用，最大值为delay_loop。 指定 0 为禁用
          warmup <INT>

          # 如果设置，将会根据  healthchecker 退出状态码动态调整权重如下：
          #   exit status 0: svc检查成功，权重不变。
          #   exit status 1: svc检查失败.
          #   exit status 2-255: svc检查成功，权重更改为小于退出状态2的值。
          #   (示例：退出状态为 255 会将权重设置为 253 )
          misc_dynamic
          # 指定脚本应在其下运行的用户名/组名。 如果未指定GROUPNAME，则使用用户的组
          user USERNAME [GROUPNAME]
      }
  } # realserver 定义
} # virtual service
```
