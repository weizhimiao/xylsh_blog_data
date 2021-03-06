---
title: MySQL存储引擎概述
date: 2016-09-17 23:00:00
tags:
- 存储引擎
categories:
- MySQL
---

基于存储引擎在MySQL架构中的地位，在学习和使用MySQL时我们需要对MySQL的各种存储引擎有一个大概的了解。
并且知道在实际项目中如何选择适合的存储引擎，以及如何实现不同存储引擎的相互切换。

<!-- more -->

## 存储引擎分类

### InnoDB
InnoDB，是MySQL的默认事务型引擎，也是做重要、使用最广泛的存储引擎。它的性能和自动崩溃恢复特性，使得它在非事务行存储的需求中也很流行。所以除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。并且InnoDB作为事务型存储引擎，它通过一些机制和工具可以支持真正的热备份，而MySQL其他的存储引擎则不支持热备份。

### MyISAM
在MySQL 5.1 及之前的版本，MyISAM是MySQL默认的存储引擎。MyISAM提供大量的特性，包括全文索引、压缩、空间函数（GIS）等，但是MyISAM不支持事务和行级锁，而且有一个毫无疑问的缺陷就是崩溃后无法安全恢复。尽管MyISAM不支持事务、不支持崩溃后的安全恢复，但是对于一些只读的数据，或者表比较小，可以忍受repair（修复）操作，则依然可以继续使用MyISAM。**但请不要默认使用MyISAM，而是应当默认使用InnoDB**

### Archive
Archive 存储引擎只支持INSERT 和SELECT 操作，在MySQL 5.1之前还不支持索引。Archive会缓存所有的写并利用zlib对插入的行进行压缩，所以比MyISAM表的磁盘I/O更少。但是每次select操作都需要对全表进行扫描，所以Archive表适合日志和数据采集类应用，或者一些需要更快速insert操作的场合下使用。总之，MyISAM是一个针对告诉插入和压缩做了优化的简单引擎。

### Blackhole
Blackhole引擎没有实现任何的存储机制，他会丢弃所有的插入的数据，不做任何的保存。但是服务器会记录Blackhole表的日志，所以可以用于复制数据库到备库，或者只是简单地记录到日志。这种特殊的存储引擎可以在一些特殊的复制架构和日志审核时发挥所用。

### CSV
CSV引擎可以将普通的CSV文件作为MySQL的表来处理，但这种表不支持索引。CSV引擎可以在数据运行是拷入或者烤出文件。可以将Excel等电子表格中的数据存储为CSV文件，然后复制到MySQL数据目录下，然后就能在MySQL中打开使用。因此，CSV引擎可以作为一种数据交换的机制，非常有用。

### Federated
Federated引擎是访问其他MySQL服务器的一个代理，他会创建一个到远程MySQL服务器的的客户端连接，并将查询传输到远程服务器执行，然后提取或者发送需要的数据。但是尽管该引擎看起来提供了一种很好的跨服务器的灵活性，但也经常带来问题。因此默认是禁用的。

### Memory
Memory表所有的数据都保存在内存中，因此Memory表的访问速度非常的快（至少要比MyISAM快一个数量级），并且不需要磁盘的I/O操作。但是Memory表在系统重启后虽然表结构会保留，但是数据会丢失。

Memory表支持Hash索引，因此查询操作非常快。但是MySQL表采用的表级锁，因此并发写入的性能较低。

Memory表的应用场景：
- 用于查找（lookup）或者映射（mapping）表。
- 用于缓存周期性聚合数据的结果。
- 用于保存数据分析中产生的中间问题。

**Tips：**
如果MySQL在执行查询的过程中需要使用临时表来保存中间结果，内部使用的临时表就是Memory表。如果中间结果大大超出了Memory表的限制，或者有大量的BLOB或TEXT字段，则临时表会转换成MyISAM表。

### Merge
Merge引擎是MyISAM的一种变种。Merge表是由多个MyISAM表合并而来的虚拟表。

### NDB
MySQL服务器、NDB集群引擎、以及分布式的、share-nothing的、容灾的、高可用的NDB数据库组合，被称之为MySQL集群（MySQL Cluster）。

### 其他的第三方存储引擎
MySQL从07年开始提供了插件式的存储引擎API，从此出现了一系列为不同目的而设计的存储引擎。其中一些已经合并到MySQL服务器，但大多数还是第三方产品或者开源项目。比较有名的有：
- OLTP类引擎
- 面向列的存储引擎
- 社区存储引擎

## 选择合适的存储引擎的考虑因素

不同的应用可能会根据不同的需求而采用不同的存储引擎。那么我们再为应用选择存储引擎时，通常会考虑以下几点：
- 事务
引用是否需要事务支持。如果需要，那么InnoDB是目前最稳定且经过验证的选择。如果不需要，并且主要是select和insert操作，那么MyISAM是不错的选择。

- 备份
如果可以定期的通过关闭服务器来执行备份，那么备份因素就可以忽略。反之，如果需要支持在线热备份，那么选择InnoDB就是基本的要求。

- 崩溃恢复
相对而言，MyISAM崩溃后发生的数据毁坏的概率要比InnoDB高的多，并且恢复速度也要慢的多。所以，即使不需要事务支持，很多人也会选择InnoDB引擎。

- 特有的特性
有些应用可能需要一些存储引擎锁独有的特性或者优化，比如依赖聚簇索引的优化，或者需要对地理空间的搜索。

**总之，如无特殊的需求和例外，统统建议选择InnoDB存储引擎。**

## 如何转换表的存储引擎
有很多中方法可以转换，一般我们会使用以下三种方法：
### ALTER TABLE
```sql
mysql>alter table mytable engine=innodb;
```
上述语法可以适应与任何存储引擎。但是当表数据很大时，这种方法将会执行很长时间。MySQL会按行将数据从原表复制到一张新的表中。所以，在繁忙的表上执行此操作要非常小心。

### 导入/导出
使用mysqldump工具将数据到出到文件，然后修改文件中create table 语句中的存储引擎的选项，（同时注意修改表名，避免在导入的时候将原表删除，造成数据丢失）。然后导入文件到数据库，这样就得到了一个原表的一个全量复制表。

### 创建与查询
综合第一种方法的高效和第二种的安全。不需要导出整张表的数据，而是先创建一个新的存储引擎的表，然后利用 insert...select语法来导数据。
```
mysql>create table innodb_table like myisam_table;
mysql>alter table innodb_table engine=innodb;
mysql>insert into innodb_table select * from myisam_table;
```
如果数据量大的话，可以考虑分批处理，并使用事务进行提交操作；
```
mysql>start transaction;
mysql>insert into innodb_table select * from myisam_table where id between x and y;
mysql>commit;
```
如果有必要，在操作的时候可以对原表加锁，以确保新表和原表数据一致。
这样操作之后也会得到一个对原表的全量复制的表，如果需要还可以删除原表。


**Tips：**
Persona Toolkit 提供了一个 pt-online-schema-change的工具（基于Facebook的在线变更技术），可以比较简单、方便的执行上述过程，避免收工操作可能会带来的错误是繁琐。。。。。。
