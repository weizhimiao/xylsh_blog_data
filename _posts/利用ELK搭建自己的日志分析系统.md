---
title: 利用ELK搭建自己的日志分析系统
date: 2017-04-23 20:30:00
tags:
- ELK
- Elasticsearch
- Logstash
- Kibana
categories:
- ELK
---

不管是用于记录，监控或者程序的Debug，日志，对于任何系统来说都是一个及其重要的部分。但一般日志的数据量会比较大，并且分散在各个地方。如果管理的服务器或者程序比较少的情况我们还可以逐一登录到各个服务器去查看，分析。但如果服务器或者程序的数量比较多了之后这种方法就显得力不从心。基于此，一些集中式的日志系统也就应用而生。
目前比较有名成熟的有，Splunk(商业)、FaceBook 的Scribe、Apache 的 Chukwa
Cloudera 的 Fluentd、还有ELK 等等。

<!-- more -->

## ELK简介
ELK不是一款软件，是三个软件产品的首字母缩写，Elasticsearch，Logstash 和 Kibana。这三款软件都是开源的，现在归于 Elastic.co 公司。

<center>
![ELK协议栈](https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/img001.png)
</center>

### Elasticsearch
Elasticsearch 是一个实时的分布式搜索和分析引擎，它可以用于全文搜索，结构化搜索以及分析。它是一个建立在全文搜索引擎 Apache Lucene 基础上的搜索引擎，使用 Java 语言编写。作为ELK协议栈的核心，它用于集中存储数据。目前，最新最新的版本是 5.3。

特点：
- 实时（准实时）
- 分布式
- 面向文档
- 高可用性，易扩展，支持集群（Cluster）、分片和复制（Shards 和 Replicas）。
- 接口友好（RESTful）


### Logstash
Logstash 是一个具有实时渠道能力的数据收集引擎。使用 JRuby 语言编写。其作者是世界著名的运维工程师乔丹西塞 (JordanSissel)。目前最新的版本是 5.3。

特点
- 几乎可以访问任何数据
- 可以和多种外部应用结合
- 支持弹性扩展

<center>
![logstash](https://static-www.elastic.co/assets/blt8a9ac25aedbd9ca7/logstash-img1.png?q=414)
</center>

组成部分
- INPUTS, 从各种地方获取各种数据数据
- FILTERS, 过滤处理获取到的数据
- OUTPUTS, 将处理完的数据输出到指定地方（消息队列，或者 ElasticSearch中）

### Kibana

Kibana 是一款基于 Apache 开源协议，使用 JavaScript 语言编写，为 Elasticsearch 提供分析和可视化的 Web 平台。它可以在 Elasticsearch 的索引中查找，交互数据，并生成各种维度的表图。目前最新的版本是 5.3。

## ELK 日志分析系统的搭建

### 准备
#### 服务器配置：
- 系统：CentOS 7.3 64
- 配置：CPU：2核 内存：4G
- 实例
  - 10.47.45.66(内)
  - 10.30.70.120(内)
  - 10.47.45.32(内)

#### 用到的软件或文件
- JDK 8 （jdk-8u131-linux-x64.tar.gz）
- elasticsearch(https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip)
- kibana(https://artifacts.elastic.co/downloads/kibana/kibana-5.3.0-linux-x86_64.tar.gz)
- x-pack (https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.3.0ll.zip)
- logstash(https://artifacts.elastic.co/downloads/logstash/logstash-5.3.0.zip)
- Nginx
- metricbeat(https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.3.1-x86_64.rpm)

#### 规划

<center>
![ELK日志系统架构图](http://wx4.sinaimg.cn/mw690/8e242e12ly1fevtki0lwkj20xc0gota6.jpg)
</center>

> 这里将会用三台服务器来模拟一个简单的集群环境

ElasticSearch集群
- 服务器节点
  - 【主节点】10.47.45.66(内)
  - 【数据节点】10.30.70.120(内)
  - 【数据节点】10.47.45.32(内)
- 将要安装的软件或插件
  - ElasticSearch
  - ElasticSearch x-pack插件

Kibana
- 服务器节点
  - 10.47.45.66(内)
- 将要安装的软件或插件
  - Nginx（提供前端代理）
  - Kibana（提供Kibana UI）
  - Kibana x-pack插件

Logstash
- 服务器节点
  - 10.47.45.32(内)
- 将要安装的软件或插件
  - Apache(模拟产生访问日志)
  - Logstash
  - beats

### 具体安装与配置过程

1. JDK（所有节点）的安装与配置
2. ElasticSearch 的安装与配置
3. Kibana 的安装与配置
4. Nginx 的安装与配置
5. Logstash 的安装与配置
6. Apache、beats 的安装与配置
7. 验证

#### JDK 8 的安装与配置
> 安装范围：所有节点

Elasticsearch 5.3对Java的版本至少是 Java 8.
查看当前的Java 版本,及 JAVA_HOME 环境变量的设置：
```sh
java -version
echo $JAVA_HOME
```
如果没有安装，或版本不满足，则需要安装并配置

下载解压
```sh
wget http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz?AuthParam=1492857236_f012353e60ed757db930682d071feefc -O jdk-8u131-linux-x64.tar.gz
tar -zxf jdk-8u131-linux-x64.tar.gz
mv jdk1.8.0_131 /usr/local/
```

配置环境变量
```sh
vi /etc/bashrc
```
在文件底部追加下面两行
```sh
export JAVA_HOME=/usr/local/jdk1.8.0_131
export PATH=$PATH:$JAVA_HOME/bin
```
加载，使之生效
```sh
source /etc/bashrc
```
测试
```sh
java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

echo $JAVA_HOME
/usr/local/jdk1.8.0_131
```

#### ElasticSearch 的安装与配置
> 在三台ElasticSearch节点上分别安装，并按照节点的类型，分别修改相应配置

##### 下载并解压 Elasticsearch
```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip
unzip elasticsearch-5.3.0.zip
mv elasticsearch-5.3.0 /usr/local/
```

Elasticsearch 的目录结构如下：
```sh
.
├── bin
│   ├── elasticsearch
│   ├── elasticsearch.bat
│   ├── elasticsearch.in.bat
│   ├── elasticsearch.in.sh
│   ├── elasticsearch-keystore
│   ├── elasticsearch-keystore.bat
│   ├── elasticsearch-plugin
│   ├── elasticsearch-plugin.bat
│   ├── elasticsearch-service.bat
│   ├── elasticsearch-service-mgr.exe
│   ├── elasticsearch-service-x64.exe
│   ├── elasticsearch-service-x86.exe
│   ├── elasticsearch-systemd-pre-exec
│   ├── elasticsearch-translog
│   └── elasticsearch-translog.bat
├── config
│   ├── elasticsearch.yml
│   ├── jvm.options
│   └── log4j2.properties
├── lib
│   ├── elasticsearch-5.3.0.jar
│   ├── HdrHistogram-2.1.6.jar
│   ├── hppc-0.7.1.jar
│   ├── jackson-core-2.8.6.jar
│   ├── jackson-dataformat-cbor-2.8.6.jar
│   ├── jackson-dataformat-smile-2.8.6.jar
│   ├── jackson-dataformat-yaml-2.8.6.jar
│   ├── java-version-checker-5.3.0.jar
│   ├── jna-4.2.2.jar
│   ├── joda-time-2.9.5.jar
│   ├── jopt-simple-5.0.2.jar
│   ├── jts-1.13.jar
│   ├── log4j-1.2-api-2.7.jar
│   ├── log4j-api-2.7.jar
│   ├── log4j-core-2.7.jar
│   ├── lucene-analyzers-common-6.4.1.jar
│   ├── lucene-backward-codecs-6.4.1.jar
│   ├── lucene-core-6.4.1.jar
│   ├── lucene-grouping-6.4.1.jar
│   ├── lucene-highlighter-6.4.1.jar
│   ├── lucene-join-6.4.1.jar
│   ├── lucene-memory-6.4.1.jar
│   ├── lucene-misc-6.4.1.jar
│   ├── lucene-queries-6.4.1.jar
│   ├── lucene-queryparser-6.4.1.jar
│   ├── lucene-sandbox-6.4.1.jar
│   ├── lucene-spatial3d-6.4.1.jar
│   ├── lucene-spatial-6.4.1.jar
│   ├── lucene-spatial-extras-6.4.1.jar
│   ├── lucene-suggest-6.4.1.jar
│   ├── securesm-1.1.jar
│   ├── snakeyaml-1.15.jar
│   ├── spatial4j-0.6.jar
│   └── t-digest-3.0.jar
├── LICENSE.txt
├── modules
│   ├── aggs-matrix-stats
│   ├── ingest-common
│   ├── lang-expression
│   ├── lang-groovy
│   ├── lang-mustache
│   ├── lang-painless
│   ├── percolator
│   ├── reindex
│   ├── transport-netty3
│   └── transport-netty4
├── NOTICE.txt
├── plugins
└── README.textile
```

##### 配置

ElasticSearch默认有三个配置文件：
```sh
config/
├── elasticsearch.yml # 基本配置
├── jvm.options       # java虚拟机的相关配置
└── log4j2.properties # 日志相关配置
```

elasticsearch了配置文件示例
```yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
#cluster.name: my-application
cluster.name: zhimiao-app
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
#node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#

node.master: false

node.data: true

# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
path.data: /data/elasticData
#
# Path to log files:
#
#path.logs: /path/to/logs
path.logs: /data/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.zen.ping.unicast.hosts: ["host1", "host2"]
discovery.zen.ping.unicast.hosts: ["10.47.45.66", "10.30.70.120","10.47.45.32"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
#discovery.zen.minimum_master_nodes: 3
#
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
```

关于配置的几点简单说明
- cluster.name ，集群的名称标识将会被用于群集的自动发现。如果在同一个网络上的正在运行多个群集，则需要确保该集群的所有节点上使用的是相同的名称。
- node.master ，允许此节点可以作为主节点（默认启用）。
- node.data ， 允许此节点可以用来存储数据（默认启用）。
- path.data ， 索引数据存储的路径，可以配置多个位置
- path.logs ， 日志文件路径
- discovery.zen.ping.unicast.hosts ， 集群的初始清单


ElasticSearch 集群角色规划

- 如果想让这个节点只能作为ElasticSearch集群的主节点，而不存储数据，则应该这样配置。
```yaml
...
node.master: true
node.data: false
...
```

- 如果希望这个节点只作为数据节点，而不会成为主节点，则应该这样配置。
```yaml
...
node.master: false
node.data: true
...
```
- 如果希望这个节点既不是不是主节点，又不是数据节点，而是充当“搜索负载平衡器”（取从节点的数据，聚集结果等），则可以这样配置。
```yaml
...
node.master: false
node.data: false
...
```

##### 安装x-pack插件
```sh
$ sudo /usr/local/elasticsearch-5.3.0/bin/elasticsearch-plugin  install x-pack
 [=================================================] 100%  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission accessClassInPackage.com.sun.activation.registries
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.RuntimePermission setFactory
* java.security.SecurityPermission createPolicy.JavaPolicy
* java.security.SecurityPermission getPolicy
* java.security.SecurityPermission putProviderProperty.BC
* java.security.SecurityPermission setPolicy
* java.util.PropertyPermission * read,write
* java.util.PropertyPermission sun.nio.ch.bugLevel write
* javax.net.ssl.SSLPermission setHostnameVerifier
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed x-pack
```

x-pack插件的安装需要在线下载安装包，所以对于网络环境比较差或者一些离线的服务器，可以离线下载 x-pack 安装包，来手动指定。

```sh
$ wget  https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.3.0ll.zip

$ sudo /usr/local/elasticsearch-5.3.0/bin/elasticsearch-plugin  install file:///home/datanode/x-pack-5.3.0.zip
-> Downloading file:///root/x-pack-5.3.0.zip
[=================================================] 100%  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission accessClassInPackage.com.sun.activation.registries
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.RuntimePermission setFactory
* java.security.SecurityPermission createPolicy.JavaPolicy
* java.security.SecurityPermission getPolicy
* java.security.SecurityPermission putProviderProperty.BC
* java.security.SecurityPermission setPolicy
* java.util.PropertyPermission * read,write
* java.util.PropertyPermission sun.nio.ch.bugLevel write
* javax.net.ssl.SSLPermission setHostnameVerifier
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed x-pack
```

##### 启动
```sh
$ /usr/local/elasticsearch-5.3.0/bin/elasticsearch -d
```

##### 验证

1. 查看端口
```sh
$ netstat -tlun
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
udp        0      0 47.90.15.84:123         0.0.0.0:*                          
udp        0      0 10.27.46.180:123        0.0.0.0:*                          
udp        0      0 127.0.0.1:123           0.0.0.0:*                          
udp        0      0 0.0.0.0:123             0.0.0.0:*                          
udp6       0      0 :::123                  :::*      
```

2. 使用REST API
> 由于我们已经安装x-pack，所以在使用 REST API 进行请求的时候需要加上认证信息。默认：
>
> - 用户名为：`elastic`
>
>
> - 密码为：`changeme`

```sh
$ curl http://localhost:9200 -u elastic
{
  "name" : "E0QjUN9",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "oSsjMVpETViMkzUCf110Cw",
  "version" : {
    "number" : "5.3.0",
    "build_hash" : "3adb13b",
    "build_date" : "2017-03-23T03:31:50.652Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
```

##### ElasticSearch的简单管理

启动
```sh
$ /usr/local/elasticsearch-5.3.0/bin/elasticsearch  -d
```
设置开机自启动
```sh
echo "/usr/local/elasticsearch-5.3.0/bin/elasticsearch -d" > /etc/rc.local
```

如何关闭elasticsearch
- 方法一：如果节点与控制台相连并且当前elasticsearch是使用-f选项运行，则只需要按下Ctrl+C组合键即可
- 方法二：通过发送TERM信号来终止服务器进程 kill -9 进程ID
- 方法三：使用REST API

前两种就不说了，简单说一下第三种，
- 关闭整个集群
```sh
curl -XPOST http://localhost:9200/_cluster/nodes/_shutdown -u elastic
```
- 关闭单个节点
```sh
curl -XPOST  http://127.0.0.1:9200/_cluster/nodes/2ens0yuEQ12G6ct1UDpihQ/_shutdown -u elastic
```
> 2ens0yuEQ12G6ct1UDpihQ ，为要关闭的节点标志符
> -
> 查看节点标志符，可以从elasticsearch日志中或者通过REST API中获得
```sh
curl http://localhost:9200/_nodes/?pretty -u elastic
{
  "cluster_name" : "elasticsearch",
  "nodes" : {
    "13QfvIdATEurGAhVAlO6tQ" : {
      "name" : "Edwin Jarvis",
      ...
    }
  }
}
```

##### 安装或者启动时可能会遇到的问题列表：

1.  can not run elasticsearch as root

解决：新建一个专门运行和管理ElasticSearch 的用户。切换到该用户下重新运行。

2.  max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]

解决：切换到root用户，编辑limits.conf 添加类似如下内容
```sh
vi /etc/security/limits.conf
```
修改文件末尾如下内容:
```
* soft nofile 655350
* hard nofile 655350
```
然后切换回用户，使用`ulimit -a` 查看是否已经更改(对应的是`open file` 选项)


3.  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决：切换到root用户修改配置sysctl.conf
```sh
vi /etc/sysctl.conf
```
添加下面配置：
```
vm.max_map_count=655300
```
并执行命令：
```sh
sysctl -p
```
然后，切换回用户，重新运行 ElasticSearch

#### Kibana 的安装与配置

##### 下载并解压 Kibana
```sh
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-5.3.0-linux-x86_64.tar.gz
$ tar -zxf kibana-5.3.0-linux-x86_64.tar.gz
sudo mv kibana-5.3.0-linux-x86_64 /usr/local/
```
##### 安装 x-pack 插件

```sh
$ bin/kibana-plugin  install x-pack
```

离线安装

同ElasticSearch 的x-pack插件一样，在安装时需要在线下载安装包，所以对于网络环境比较差或者一些离线的服务器，也需要离线下载 x-pack 安装包，来手动指定。

> Elasticsearch，Kibana和Logstash的插件都包含在同一个zip文件中,即 之前下载过的 x-pack-5.3.0ll.zip

```sh
$ sudo /usr/local/kibana-5.3.0-linux-x86_64/bin/kibana-plugin  install file:///home/datanode/x-pack-5.3.0.zip
```

> 安装过程可能会比较长（请耐心等待）...

##### 配置
```sh
$ cd /usr/local/kibana-5.3.0-linux-x86_64
$ vi config/kibana.yml
```

kibana.yml 示例
```yaml
# Kibana is served by a back end server. This setting specifies the port to use.
#server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
#server.host: "127.0.0.1"

# Enables you to specify a path to mount Kibana at if you are running behind a proxy. This only affects
# the URLs generated by Kibana, your proxy is expected to remove the basePath value before forwarding requests
# to Kibana. This setting cannot end in a slash.
#server.basePath: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayloadBytes: 1048576

# The Kibana server's name.  This is used for display purposes.
#server.name: "your-hostname"

# The URL of the Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://10.30.70.120:9200"

# When this setting's value is true Kibana uses the hostname specified in the server.host
# setting. When the value of this setting is false, Kibana uses the hostname of the host
# that connects to this Kibana instance.
#elasticsearch.preserveHost: true

# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"

# The default application to load.
#kibana.defaultAppId: "discover"

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
#elasticsearch.username: "user"
#elasticsearch.password: "pass"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# Optional settings that provide the paths to the PEM-format SSL certificate and key files.
# These files validate that your Elasticsearch backend uses the same key files.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
#elasticsearch.ssl.verificationMode: full

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 0

# Time in milliseconds to wait for Elasticsearch at Kibana startup before retrying.
#elasticsearch.startupTimeout: 5000

# Specifies the path where Kibana creates the process ID file.
#pid.file: /var/run/kibana.pid

# Enables you specify a file where Kibana stores log output.
#logging.dest: stdout

# Set the value of this setting to true to suppress all logging output.
#logging.silent: false

# Set the value of this setting to true to suppress all logging output other than error messages.
#logging.quiet: false

# Set the value of this setting to true to log all events, including system usage information
# and all requests.
#logging.verbose: false

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000.
#ops.interval: 5000
```

配置文件简单说明
- `server.host` , 配置Kibana服务器绑定地址。IP地址或者域名
> 默认是 “localhost”。远程用户无法连接。如果需要通过网络进行直接访问，则需要将该参数设置为非环回地址。
>
> 这里我将它设置为 “127.0.0.1”，即表示远程用户无法直接访问，需要通过之后安装配置的Nginx进行代理访问。

- `elasticsearch.url` , 指向 Elasticsearch 实例


##### 运行
```sh
$ nohup /usr/local/kibana-5.3.0-linux-x86_64/bin/kibana &
```
查看是否已经运行
```sh
$ ps aux | grep kiba
```
或
```sh
$ netstat -tlun | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN
```

####  Nginx 的安装与配置

安装
```sh
# yum install nginx -y
```
验证
```sh
# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

添加配置
```sh
# vi /etc/nginx/conf.d/kibana.conf
```

kibana.conf 示例
```ini
server {
    listen 80;
    server_name 47.90.15.84;

    location / {
        proxy_pass http://127.0.0.1:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
启动 Nginx
```
# nginx
```
验证

浏览器打开 `http://47.90.15.84`，能看到登录界面，则说明配置成功

<center>
![登录页面](http://wx1.sinaimg.cn/mw690/8e242e12ly1fevxyyb4rej21kw0zk0v4.jpg)
</center>

> 默认账号
>
> - username: elastic
> - password: changeme
#### Logstash 的安装与配置

##### 下载并解压 Logstash
```
# wget https://artifacts.elastic.co/downloads/logstash/logstash-5.3.0.zip
# unzip logstash-5.3.0.zip
# mv logstash-5.3.0 /usr/local/
```
##### 测试 Logstash
```
cd /usr/local/logstash-5.3.0
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
> -e 选项表示直接从命令行指定配置

待 Logstash 启动， 看到 “Pipeline main started” ，在命令行中输入 “hello logstash”，将会看到
```
hello logstash
2017-04-22T17:19:31.051Z iZj6c1rlufgkgsimhzuumaZ hello logstash
```
> Logstash 将时间戳和主机名添加到了消息中

##### 添加一个 Logstash 配置文件，将我们上面安装的Nginx 的访问日志作为input，输出到 ElasticSearch 中

Nginx 访问日志的位置：`/var/log/nginx/access.log`

```
vi first-pipeline.conf
```
first-pipeline.conf 示例
```
input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx"
  }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
output {
    #stdout { codec => rubydebug }
    elasticsearch {
        hosts => ["10.30.70.120:9200","10.47.45.32:9200"]
        user => "elastic"
        password => "changeme"
  }
}
```

配置文件简析
- input 区块
> 使用了 input 的file 插件， 该插件跟踪 path 路径下的文件变化，并将数据读取到 Logstash。

- filter 区块
> - 该区块应用了 grok 插件，会将非结构化的日志数据转化为可查询的结构化的数据
> - geoip插件将IP地址转换为实际地址坐标信息

- output区块
> - 该区块使用了 ElasticSearch 插件，直接将数据输出到 ElasticSearch集群中
> - stdout插件则会直接将结果输出到控制台。（开发时将其打开，方便调试）



##### 测试&运行
测试配置文件
```
 /usr/local/logstash-5.3.0/bin/logstash -f first-pipeline.conf --config.test_and_exit
```
- --config.test_and_exit , 测试我们编写的配置文件

如果测试通过，用下面的命令启动 Logstash
```
bin/logstash -f first-pipeline.conf --config.reload.automatic
```
- --config.reload.automatic ,启用自动重新加载配置

> 启用自动重新加载配置，之后每次修改配置文件时不必停止再重新启动Logstash。

##### 验证

浏览器打开 `http://47.90.15.84` ，登录 Kibana

添加“`logstash-*`” 索引

<center>
![添加索引](http://wx3.sinaimg.cn/mw690/8e242e12ly1fewnxhoiqyj21hc0u0tdk.jpg)
</center>

然后再 Discovery 中查看是否有数据被写入

<center>
![验证效果](http://wx1.sinaimg.cn/mw690/8e242e12ly1few1y2tl0uj21h30prtfm.jpg)
</center>

#### beats 的安装与配置
Beats是作为代理在服务器上安装的开源的 data shippers，能将各种不同类型的操作数据（如， wireData、LogFiles、Metrics、WinEvent）直接发送到 Elasticsearch，或者通过Logstash将其发送到Elasticsearch。我们可以使用它来解析和转换我们需要收集的各种数据。

这里以 Metricbeat 为例，来监控我们想要监控的服务器。

##### 步骤1：安装Metricbeat
```
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.3.1-x86_64.rpm
sudo  rpm -vi metricbeat-5.3.1-x86_64.rpm
```
> yum 包管理工具安装：https://www.elastic.co/guide/en/beats/metricbeat/5.3/setup-repositories.html

测试
```
metricbeat -version
```

##### 步骤2：配置Metricbeat

```
cat /etc/metricbeat/metricbeat.yml
```
内容如下：
```
metricbeat.modules:
- module: system
 metricsets:
   - cpu
   - load
   - core
   - diskio
   - filesystem
   - fsstat
   - memory
   - network
   - process
   - socket
 enabled: true
 period: 10s
 processes: ['.*']
- module: nginx
 metricsets: ["stubstatus"]
 enabled: true
 period: 10s
 hosts: ["http://127.0.0.1"]
 server_status_path: "NginxStatus"

output.elasticsearch:
 hosts: ["10.30.70.120:9200","10.47.45.32:9200"]
 username: "elastic"
 password: "changeme"

logging.level: debug
```
配置文件检查
```
# metricbeat.sh -c metricbeat.yml  -configtest
```
显示 `Config OK` 则说明配置文件语法正确


##### 第3步：加载索引模板Elasticsearch
```
curl -H 'Content-Type: application/json' -XPUT 'http://10.30.70.120:9200/_template/metricbeat' -d@/etc/metricbeat/metricbeat.template.json -u elastic:changeme
```
有如下显示则表示加载成功
```
{"acknowledged":true}
```

##### 第4步：启动Metricbeat

```
/etc/init.d/metricbeat start
```

验证服务器的统计信息是否被正确的已经发送在Elasticsearch中：
```
curl -XGET 'http://10.30.70.120:9200/metricbeat-*/_search?pretty' -u elastic:changeme
```

##### 第5步：导入Kibana仪表板模板

`Metricbeat`自带了 `scripts/import_dashboards` 导入工具，我们可以使用它来导入例如dashboards, visualizations。该脚本还会为Metricbeat创建` metricbeat-*` 索引。

> 我们可以通过 scripts/import_dashboards 后跟相应的选项，来控制是否只导入 dashboards 或者 index。
> - `-only-index` , 只创建索引
> - `-only-dashboards` , 只导入 仪表盘

由于我们采用的是 rpm包 安装的 Metricbeat，所以该脚本是在`/usr/share/metricbeat/` 下。

```
# cd /usr/share/metricbeat/
# ./scripts/import_dashboards -es http://10.30.70.120:9200 -user elastic -pass changeme
Create temporary directory /tmp/tmp252960117
Downloading https://artifacts.elastic.co/downloads/beats/beats-dashboards/beats-dashboards-5.3.0.zip
Unzip archive /tmp/tmp252960117
Importing Kibana from /tmp/tmp252960117/beats-dashboards-5.3.0/filebeat
...
Importing Kibana from /tmp/tmp252960117/beats-dashboards-5.3.0/packetbeat
Importing Kibana from /tmp/tmp252960117/beats-dashboards-5.3.0/winlogbeat
```
导入之后，我们登录 Kibana 查看

<center>

![img](http://wx2.sinaimg.cn/mw690/8e242e12ly1fewv6vsm41j21kw0fggr5.jpg)

![img](http://wx1.sinaimg.cn/mw690/8e242e12ly1fewv89jhhdj21kw0w6wlv.jpg)

</center>

点击其中相应的 dashboard 我们就能查看到相应的参数。

#### over~

到此，这个简单的日志分析系统的架构就算基本搭建出来了。

对于搭建过程中遇到的其它问题和上述相关软件的使用，留待以后慢慢整理~~~~
