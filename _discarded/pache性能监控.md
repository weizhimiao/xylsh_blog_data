title: Apache性能监控
tags:
  - Apache 监控
categories:
  - Apache
author: zhimiao
date: 2016-08-23 16:10:00
---
谈到服务器性能监控，目前市面上有很多成熟的关于性能监控的产品可供我们使用。比如 [Cloud Insight](http://www.oneapm.com/ci/feature.html)
。但通过 Apache本身提供的监控模块或者通过一些简单的bash命令也能实现简单的监控。

## Linux下通过Server-status来监控Apache

1. 加载 mod_status.so 模块
> mod_status, Apache状态管理模块

```
#在httpd.conf中加入下面这句或将其前面注释去掉
LoadModule status_module modules/server_status.so
```

2. 修改配置文件
  * 方式一：直接在 httpd.conf 底部添加以下配置
```
<location /c-server-status>
    setHandle Server-status
    Order Deny,Allow
    Deny from nothing
    Allow from all
</location>
ExtendedStatus on
```
  * 方式二：添加到子配置文件中
  在 httpd.conf 中找到 `Include conf/extra/httpd-info.conf`,去掉`#`,
  在 httpd-info.conf 文件中加入 方式一 中的内容。

  **Tips：**
  * `<location /c-server-status>`: 这个名字可以任意取，最好不要让别人猜到。
  * `ExtendedStatus on`: 启用扩展状态。该设置仅能用于全局设置，不能在特定的虚拟主机中打开或者关闭。并且，启用该扩展会使服务器运行效率降低。

3. 重启Apache
```
/usr/local/apache2/bin/httpd -k start|restart|stop
```

4. 访问页面

  http://your-domain/c-server-status

  http://your-domain/c-server-status?refresh=5

  [官网示例](http://www.apache.org/server-status)

5. 监控参数

参数名称|	参数描述
-------|---------
Total Accesses|	服务器自启动来接收到的请求连接数
Total kBytes|	传输的总数据量，单位是KB
CPULoad	|NCPU负荷
Uptime	|运行时间，单位秒
ReqPerSec	|每秒请求数
BytesPerSec	|每秒传输数据量，单位B/s
BytesPerReq	|平均每个请求的数据传输量（事实上就是BytesPerSec/BytesPerSec）
BusyWorkers	|在跑的进程数
IdleWorkers	|空闲的进程数

## Linux下通过命令来实现监控

1. ps 查看httpd进程数
```
$ps -ef | grep httpd | wc -l
```
当连接数多了，他就会生出更多进程来处理请求。当然这结果包含 grep httpd 的进程输出，所以一般来说实际进程数比输出结果少1
2. 用netstat来查看当前连接数
```
$netstat -ant | grep ":80" | wc -l
```
连接数目并不等于httpd线程数目，当然连接数目越多，httpd进程数就有可能数会增多。上面的返回结果数目，有可能包括多种连接状态，比如 LISTEN、ESTABLISHED、TIME_WAIT等等，可以加入状态关键字进一步过滤，得到想要的结果。
3. 实时检测httpd连接数
```
#watch -n 1 -d "pgrep httpd|wc -l"
```
4. 计算httpd进程占用内在的平均数
```
#ps aux|grep -v grep|awk '/httpd/{sum+=$6}; END{print sum/n}'
```
5. 查看Apache的并发请求数及期TCP连接状态
```
#netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a, S[a]}'
返回结果示例：
LAST_ACK 5
SYN_RECV 30       #表示正在等待处理的请求数；
ESTABLISHED 1597  #表示正常数据传输状态；
FIN_WAIT1 51
FIN_WAIT2 504
TIME_WAIT 1057    #表示处理完毕,等待超时结束的请求数
```

常见的连接状态

状态|描述
---|----
CLOSED|无连接是活动的或正在进行
LISTEN|服务器在等待进入呼叫
SYN_RECV|一个连接请求已经到达,等待确认
SYN_SENT|应用已经开始,打开一个连接
ESTABLISHED|正常数据传输状态
FIN_WAIT1|应用说它已经完成
FIN_WAIT2|另一边已同意释放
ITMED_WAIT|等待所有分组死掉
CLOSING|两边同时尝试关闭
TIME_WAIT|另一边已初始化一个释放
LAST_ACK|等待所有分组死掉