---
title: ElasticSearch整理
date: 2016-10-05 21:00:00
tags:
- ElasticSearch
categories:
- Linux
---

ElasticSearch是由Shay Banon发起的一个开源搜索服务器项目。由于其分布式特性和实时搜索能力，成为当前搜索和数据分析解决方案领域的重要成员。


![ElasticSearch](http://n.sinaimg.cn/games/3ece443e/20161006/ElasticSearch.png)

<!-- more -->
## 基本概念
索引
> 索引（index）是ElasticSearch存放数据的一种逻辑结构。类比关系型数据库中的数据表。

文档
> 文档（document）是ElasticSearch中存储的主要实体。每个ElasticSearch文档类比关系型数据库中数据表的没一行数据。
文档由字段（行数据的列）组成，一个文档由多个字段组成，并且ElasticSearch允许一个字段重复出现多次，该类型字段被称为多只字段。每个字段对应一种类型（字符串型、数值型、日期型等），并且ElasticSearch可以自动确定字段类型。不同于关系型数据库，ElasticSearch的文档结构可以是不固定的。即不同的文档可以有不同的字段集合。

文档类型
> ElasticSearch中，一个索引可以存储许多不同用途的对象。按照不同的用途我们可以将文档划分成不同的类型加以区分。

节点和集群
> ElasticSearch既可以作为一个独立搜索服务器工作，也支持多台一起协作进行运行，构成一个集群（cluster），其中的每个服务器被称为节点（node）。ElasticSearch可以通过索引分片,将海量的数据进行分割并分布到不同的节点，来实现更强的可用性和更高的性能。

分片
> 对于存储大规模的文档，ElasticSearch会将数据进行切分，每部分切成一个单独的Apache Lucene索引，称为分片（shared）。每个分片可以存储在集群的不同的节点上，当一个查询需要用到多个分片时，ElasticSearch会将请求发送至多个分片，之后结果进行合并。

副本
> 分片副本是对原始分片的一个拷贝，每个主分片可以有零个或者多个副分片。当主分片丢失时，副分片就会被提升为主分片。启用分片副本功能，可以提高查询的吞吐或实现系统的高可用性。

## 安装
### 环境准备

JDK6+

JDK的安装方式比较简单，只需将下载回来的程序包解压到相应的目录即可。
```
wget http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz
mkdir /usr/local/java
tar -zxf jdk-8u101-linux-x64.tar.gz -C /usr/local/java/
```
设置JDK的环境变量，如下：
```
# tail -3 ~/.bash_profile
export JAVA_HOME=/usr/local/java/jdk1.8.0_101
export PATH=$PATH:$JAVA_HOME/bin
exportCLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
```
重新加载环境变量
```
# source .bash_profile
```
在Shell提示符中执行java –version命令，显示如下结果，说明安装成功：
```
# java -version
java version "1.8.0_45"
Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02,mixed mode)
```

### 安装Elasticsearch

下载Elasticsearch后，解压到对应的目录就完成Elasticsearch的安装。
```
# wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.7.0.zip
# unzip elasticsearch-1.7.0.zip
# mv elasticsearch-1.7.0 /usr/local/elasticsearch
```




### 目录结构
```
elasticsearch/
├── bin
│   ...#运行elasticsearch和进行插件管理所需的脚本
├── config
│   ...#elasticsearch配置文件所在目录
├── data
│   ...#存储elasticsearch用到的数据
├── lib
│   ...#elasticsearch运行中用到的库
├── LICENSE.txt
├── logs
│   ...#存储elasticsearch运行中产生的事件信息和错误信息
├── nohup
├── NOTICE.txt
└── README.textile

10 directories, 60 files
```


### 配置
所有的配置文件都位于config目录下。该目录下包含两个文件。
```
├── config
│   ├── elasticsearch.yml
│   └── logging.yml
```

elasticsearch.yml
> 负责设置服务器的默认配置。
> 比较重要的两个值cluster.name 和 node.name.
> - cluster.name,保存的是集群名称。通过集群名称可以区分不同的集群。配置具有相同名称的节点将尝试组成一个集群。
> - node.name，节点名称。我们也可以不指定该名称，elasticsearch会自动为节点选择一个唯一名称。但每次启动时这个唯一名称会发生改变。

其他配置
```
##################### Elasticsearch Configuration Example #####################

＃此文件包含各种配置设置的概述，
＃针对操作人员。应用程序开发人员应该
＃在咨询<http://elasticsearch.org/guide>指南。
#
＃安装过程被覆盖在
＃<http://elasticsearch.org/guide/en/elasticsearch/reference/current/setup.html>。
#
＃Elasticsearch附带了大多数设置合理的默认值，
＃所以你可以尝试使用它而不用修改。
#
＃大多数时候，这些默认值是蛮好的运行生产
＃集群。如果你正在微调群集，或者想知道的某些配置选项＃效果，请_DO ask_上
＃邮件列表或IRC频道[http://elasticsearch.org/community。
#
＃配置中的任何元素都可以用环境变量替换
＃通过将它们放置在$ {...}符号。 例如：
#
#node.rack: ${RACK_ENV_VAR}

＃有关支持的格式和配置文件信息语法，请参阅
＃<http://elasticsearch.org/guide/en/elasticsearch/reference/current/setup-configuration.html>

################################### Cluster 集群###################################

＃集群名称标识群集的自动发现。如果你在同一个网络上的正在运行多个群集，请确保您使用的是唯一的名称。
＃
＃cluster.name：elasticsearch

#################################### Node 节点#####################################


＃节点名称是在启动时动态生成的，但你也可以手动配置它们。你可以给这个节点起一个特定的名称：
＃
#node.name: "Franz Kafka"

＃每个节点可以被配置为允许或拒绝称为集群主节点，
＃和允许或拒绝来存储数据。
＃
＃允许此节点可以作为主节点（默认启用）：
＃
#node.master: true
#
# 允许此节点可以用来存储数据（默认启用）：
#
#node.data: true


＃你可以利用这些设置，以设计高级集群拓扑。
＃
＃1，你想这个节点永远不会成为一个主节点，只保存数据。这将是群集的“主力”。
＃
＃
#node.master: false
#node.data: true
#
＃2，你要这个节点只能作为主：不存储任何数据和有免费的资源。这将是群集的“协调员”。
＃
＃
#node.master: true
#node.data: false
#
＃3，你希望这个节点是不是主数据节点，但可以充当“搜索负载平衡器”（取从节点的数据，聚集结果等）
＃
#node.master: false
#node.data: false


＃使用群集健康状况API[http://localhost:9200/_cluster/health]，
＃节点信息API[http://localhost:9200/_nodes]或GUI工具
＃如<http://www.elasticsearch.org/overview/marvel/>
＃<http://github.com/karmi/elasticsearch-paramedic>
＃<http://github.com/lukas-vlcek/bigdesk>和
＃<http://mobz.github.com/elasticsearch-head>检查群集状态。



＃一个节点可以有与之关联的通用属性，可用于定制分片分配过滤，或分配意识。
＃
＃属性是一个简单的键值对，类似node.key：值，下面是一个例子：
＃
#node.rack: rack314

＃默认情况下，多个节点被允许从相同的安装位置
# 如果要禁用的话，设置如下：
#node.max_local_storage_nodes: 1


#################################### Index 索引####################################

#
＃您可以设置一些选项（如分片/副本选项，映射或分析器的定义，事务日志设置...）对全局参数，在这个文件中。
#
# Note, that it makes more sense to configure index settings specifically for
# a certain index, either when creating it or by using the index templates API.
#
＃请注意，它更有意义专门配置索引设置
＃某个具体的索引，创造它或使用该索引模板API。
# See <http://elasticsearch.org/guide/en/elasticsearch/reference/current/index-modules.html> and
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/indices-create-index.html>
# for more information.


# 设置一个索引的分片（副本）数量（默认 5）
#
#index.number_of_shards: 5

＃设置索引（默认值为1）的副本（额外副本）数量：
＃
#index.number_of_replicas: 1

# Note, that for development on a local machine, with small indices, it usually
# makes sense to "disable" the distributed features:
#
＃注意，对于在本地机器，它通常比较小的值。禁用分布式特点是很有必要的
＃
#index.number_of_shards: 1
#index.number_of_replicas: 0


＃这些设置直接影响索引和搜索操作的性能
＃在集群中。假设你有足够多的机器来保存分片及
＃副本，经验法则是：
#
# 1. 更多的分片能提高索引效率，一个大的索引允许存储在不同的服务器上
# 2. 更多的副本能够提高搜索效率，并且能够增强系统的可用性
#
#  "number_of_shards" 对于一个索引不能动态修改设置一次.
#
# T "number_of_replicas" 可以增加或者减少在任何时候，通过索引更新或者api操作
#
# Elasticsearch takes care about load balancing, relocating, gathering the
# results from nodes, etc. Experiment with different settings to fine-tune
# your setup.

＃Elasticsearch需要关心负载均衡，搬迁，从节点收集结果，等。
# 你可以通过不断的微调进行设置

＃使用索引状态API（<http://localhost:9200/A/_status>）检查
＃索引状态。


#################################### Paths ####################################

# 路径包含目录配置（此文件并logging.yml）：
#path.conf: /path/to/conf

# 索引数据存储路径配置
#path.data: /path/to/data
#
＃可以任选地包括一个以上的位置，方便扩展和使用。 例如：
#
#path.data: /path/to/data1,/path/to/data2

# 临时文件路径：
#
#path.work: /path/to/work

# 日志文件路径:
#
#path.logs: /path/to/logs

# 插件安装目录
#
#path.plugins: /path/to/plugins


#################################### Plugin ###################################

# 如果这里列出的插件没有安装用于当前节点，该节点将无法启动。
#
#plugin.mandatory: mapper-attachments,lang-groovy


################################### Memory ####################################

＃Elasticsearch表现不佳时，JVM启动交换：你应该确保它永远不会_交换。
#
# 将此属性设置为true锁定内存：
#
#bootstrap.mlockall: true


# 确保ES_MIN_MEM和ES_MAX_MEM环境变量设置
＃为相同的值，并且该机器有足够的内存来分配
＃为Elasticsearch，留出足够的内存为操作系统本身。
＃
#
# You should also make sure that the Elasticsearch process is allowed to lock
# the memory, eg. by using `ulimit -l unlimited`.
# 你还应该确保该Elasticsearch允许进程锁定内存
＃例如。通过使用`ulimit -l unlimited`。

############################## Network And HTTP ###############################


＃Elasticsearch，默认情况下，本身绑定到0.0.0.0地址，并监听
＃端口[9200-9300] HTTP流量和端口[9300-9400]节点到节点
＃沟通。（范围意味着，如果端口忙，它会自动将
＃尝试下口）。

# 设置绑定专用地址（IPv4或IPv6）：
#network.bind_host: 192.168.0.1

＃设置其他节点将使用与该节点通信的地址。否则
＃，它会自动的产生。它必须指向一个实际的IP地址。
#
#network.publish_host: 192.168.0.1

# 同时设置“bind_host'和'publish_host”：
#
#network.host: 192.168.0.1

# 设置一个自定义的节点间通讯端口，（默认为 9300）
#transport.tcp.port: 9300

# 启用节点间通讯压缩
#
#transport.tcp.compress: true

# 设置自定义端口侦听HTTP流量：
#
#http.port: 9200

# 设置自定义允许的内容长度：
#
#http.max_content_length: 100mb

# 完全禁用HTTP：
#
#http.enabled: false


################################### Gateway ###################################


＃网关允许持收集集群之间的群集状态
＃集群中每个状态改变（例如添加一个索引），都将会存储
＃在网关，并且当集群首次启动时，
＃它会从网关读出其状态。


＃有几种类型的网关实现的。欲了解更多信息，请参阅
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-gateway.html>.

＃默认网关类型为“本地”网关（推荐）：
#
#gateway.type: local

# 设置当集群崩溃时，如何重新启动并恢复进程（使用共享网关时，尽可能多的使用本地数据）

# 允许恢复后，在一个集群中有N个节点：
#
#gateway.recover_after_nodes: 1

＃设置超时时间，在启动恢复过程中，前面设置的N个节点全部重启（接受时间值）：
#

#gateway.recover_after_time: 5m

＃设置在一个句群众预计的节点数
#一旦这些N个节点全部启用（满足recover_after_nodes），立即开始恢复过程（无需等待恢复时间后到期）：
#
#gateway.expected_nodes: 2


############################# Recovery Throttling 恢复节流 #############################


＃这些设置可以控制碎片分配的过程，在节点恢复期间、副本定位、重新平衡，或添加和删除节点时


#
# 设置一个节点上同时恢复数量
# 1. 初步恢复期间
#
#cluster.routing.allocation.node_initial_primaries_recoveries: 4
#
# 2. 在添加/删除节点，再平衡期间
#
#cluster.routing.allocation.node_concurrent_recoveries: 2

# 设置一个恢复时的吞吐量值（如100MB，默认20MB）
#
#indices.recovery.max_bytes_per_sec: 20mb


# 设置一个并发流数量限制，在同级别的分片恢复时
#
#indices.recovery.concurrent_streams: 5


################################## Discovery 发现##################################


＃发现基础设施，确保节点可以在群集内找到和主节点的选举。是默认是通过多播方式来进行发现。

# 设置一个节点确保能在集群内发现N个其他合格的节点能成为主节点。

#discovery.zen.minimum_master_nodes: 1


# 设置一个过期时间，在通过ping发现其他节点时
# 在一个较差的网络环境中设置一个较长的值，可以最大限度的减少报错

#discovery.zen.ping.timeout: 3s

# 更多的信息，请看
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-discovery-zen.html>


＃单播发现允许明确地控制哪些节点将被用于发现群集。它可以在多播不存在，或限制集群通信时使用。
#
# 1. 多播发现（默认启用）
#
#discovery.zen.ping.multicast.enabled: false
#
# 2. 配置一个初始清单在集群的主节点上，为了发现一个刚启用的节点
#
#discovery.zen.ping.unicast.hosts: ["host1", "host2:port"]

# 为了发现 ，EC2允许使用AWS EC2 API
# 你必须安装云AWS插件启用EC2发现。
#
# 更多信息请查看
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-discovery-ec2.html>
#
# See <http://elasticsearch.org/tutorials/elasticsearch-on-ec2/>
# 一步一步的教程

# 为了发现 ，GCE发现允许使用谷歌Compute Engine的API
#
# 你必须安装云GCE GCE插件启用的发现。
#
# For more information, see <https://github.com/elasticsearch/elasticsearch-cloud-gce>.

# Azure的发现允许以执行发现使用Azure的API。
#
# 你必须安装云cloud-azure插件启用的发现。
# For more information, see <https://github.com/elasticsearch/elasticsearch-cloud-azure>.

################################## Slow Log ##################################

# 分片级查询并会的对应等级的日志

#index.search.slowlog.threshold.query.warn: 10s
#index.search.slowlog.threshold.query.info: 5s
#index.search.slowlog.threshold.query.debug: 2s
#index.search.slowlog.threshold.query.trace: 500ms

#index.search.slowlog.threshold.fetch.warn: 1s
#index.search.slowlog.threshold.fetch.info: 800ms
#index.search.slowlog.threshold.fetch.debug: 500ms
#index.search.slowlog.threshold.fetch.trace: 200ms

#index.indexing.slowlog.threshold.index.warn: 10s
#index.indexing.slowlog.threshold.index.info: 5s
#index.indexing.slowlog.threshold.index.debug: 2s
#index.indexing.slowlog.threshold.index.trace: 500ms

################################## GC Logging ################################

#monitor.jvm.gc.young.warn: 1000ms
#monitor.jvm.gc.young.info: 700ms
#monitor.jvm.gc.young.debug: 400ms

#monitor.jvm.gc.old.warn: 10s
#monitor.jvm.gc.old.info: 5s
#monitor.jvm.gc.old.debug: 2s

################################## Security ################################

# Uncomment if you want to enable JSONP as a valid return transport on the
# http server. With this enabled, it may pose a security risk, so disabling
# it unless you need it is recommended (it is disabled by default).

＃不推荐，如果要启用JSONP作为HTTP服务器上的返回。
#启用此功能，它可能会带来安全风险，因此禁用它，除非你需要的建议（它被默认禁用）。

#http.jsonp.enable: true

```



logging.yml
> 定义多少信息写入系统日志、定义日志文件，并定期创建新文件



### 管理
启动Elasticsearch
```
# /usr/local/elasticsearch/bin/elasticsearch  -d
```
> -d 表示将进程放入后台运行

或者通过nohup命令
```
# nohup /usr/local/elasticsearch/bin/elasticsearch > nohup
```

将elasticsearch设置成开机自启动
```
# echo "nohup /usr/local/elasticsearch/bin/elasticsearch > nohup" > /etc/rc.local
```

确认elasticsearch的9200端口已监听，说明elasticsearch已成功运行

```
# netstat -anp |grep :9200
tcp        0      0 :::9200                     :::*                        LISTEN      3362/java
```
如何关闭elasticsearch
- 方法一：如果节点与控制台相连并且当前elasticsearch是使用-f选项运行，则只需要按下Ctrl+C组合键即可
- 方法二：通过发送TERM信号来终止服务器进程 kill -9 进程ID
- 方法三：使用REST API

关闭整个集群
```
# curl -XPOST http://localhost:9200/_cluster/nodes/_shutdown
```
关闭单个节点
```
# curl -XPOST  http://127.0.0.1:9200/_cluster/nodes/2ens0yuEQ12G6ct1UDpihQ/_shutdown
```
2ens0yuEQ12G6ct1UDpihQ ，为要关闭的节点标志符

查看节点标志符，可以从elasticsearch日志中或者通过REST API中获得
```
# curl http://localhost:9200/_nodes/?pretty
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
13QfvIdATEurGAhVAlO6tQ,即为节点标志符


### 测试
```
# curl http://localhost:9200
{
  "status" : 200,
  "name" : "Destroyer of Demons",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.7.0",
    "build_hash" : "929b9739cae115e73c346cb5f9a6f24ba735a743",
    "build_timestamp" : "2015-07-16T14:31:07Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```
查看elasticsearch服务器当前运行状况
```
# curl http://localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0
}
```

## 使用（基于REST API数据操作）

### REST

REST 定义了一组体系架构原则，您可以根据这些原则设计以系统资源为中心的 Web 服务，包括使用不同语言编写的客户端如何通过 HTTP 处理和传输资源状态。 它用简洁易行的方式向HTTP中的条目暴露CRUD（Create、Replace、Update、Delete，创建、替换、更新、删除）的应用功能。

### 创建文档

示例：创建一个文档用来存储一篇blog，内容如下：
```
{
  "title":"my first article title",
  "content":"this is article content",
  "date":"2016-10-05"
}
```
操作：
```
# curl -XPUT  http://localhost:9200/blog/article/1 -d '{"title":"my first article title","content":"this is article content","date":"2016-10-05"}'
```
返回结果如下：
```
{"_index":"blog","_type":"article","_id":"1","_version":1,"created":true}
```
返回了操作结果信息，并显示新文档的存储位置。并且包含文档的唯一标识符以及当前版本信息。

### 检索文档
按照REST风格，我们想要查看刚才创建的文档。
```
# curl -XGET  http://localhost:9200/blog/article/1
```
结果
```
curl -XGET  http://localhost:9200/blog/article/1?pretty
{
  "_index" : "blog",
  "_type" : "article",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source":{"title":"my first article title","content":"this is article content","date":"2016-10-05"}
}
```
### 更新文档
Elasticsearch中更新索引中的文档是非常复杂的工作。必须先提取文档、从_source字段获得数据、移除旧文档、应用变更，作为新文档创建索引。

示例，更改之前创建的blog，并增加author字段
```
# curl -XPOST  http://localhost:9200/blog/article/1 -d '{"title":"my first article title","content":"this is article content","date":"2016-10-05","author":"zhimiao"}'
```
结果：
```
# {"_index":"blog","_type":"article","_id":"1","_version":3,"created":false}
```

查看是否更新成功
```
# curl -XGET  http://localhost:9200/blog/article/1?pretty
{
  "_index" : "blog",
  "_type" : "article",
  "_id" : "1",
  "_version" : 4,
  "found" : true,
  "_source":{"title":"my first article title","content":"this is article content","date":"2016-10-05","author":"zhimiao"}
}
```

### 删除文档

```
# curl -XDELETE  http://localhost:9200/blog/article/1?pretty
{
  "found" : true,
  "_index" : "blog",
  "_type" : "article",
  "_id" : "1",
  "_version" : 7
}
```
查看是否已删除
```
# curl -XGET  http://localhost:9200/blog/article/1?pretty
{
  "_index" : "blog",
  "_type" : "article",
  "_id" : "1",
  "found" : false
}
```
