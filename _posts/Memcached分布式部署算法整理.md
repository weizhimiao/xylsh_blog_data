---
title: Memcached分布式部署算法整理
date: 2016-10-01 20:00:00
ctime: 2016-10-01 20:00:00
utime: 2016-10-01 20:00:00
modif_times: 0
tags:
- 分布式部署算法
categories:
- Linux
---
虽然memcached是一个高性能的缓存服务器，但是单台服务器对于目前大型的web应用来说还是满足不了需求的。那么就需要布置多台的memcached服务器进行分布式部署。那么一个数据应该存储到哪一台服务器就需要一个分布算法来确定。

常见的分布方案有两种，
> - 普通Hash分布
> - 一致性Hash分布

<!-- more -->

## 普通Hash分布
> 俗称哈希取模法，优点是实现简单，但缺点是缺乏灵活性（比如增加一台服务器，那么之前存储的数据的键值和具体的物理机之间的关系就被完全打乱了，即之前存储的数据就完全失效了）。

### 原理：
H(key) = hash(key)mod K;
> 假设有K台物理机，对于key为键值的记录，H(key)值即为存储数据的物理机编号。通过这种方式，就将全部数据分配到K台物理机上，而查找某条记录时，使用同样的Hash函数就可以找到对应的物理机。

### 实现(PHP)：
```php
<?php
  $servers = array(
    array("host" => "192.168.1.1","port" => "11211"),
    array("host" => "192.168.1.2","port" => "11211"),
  );
  $key = "userDatakey";
  $value = "userDataValue";
  $mc_ser = new $servers[mhash($key)%2];
  $mc = new Memcache($servers);
  $mc->set($key, $value);
  var_dump($mc->get($key));

  /**
   * [mhash 首先通过md5把key处理成一个32位的字符串，取其前8个字符。在经过hash算法处理成一个整数并返回。]
   * @param  {[type]} $key [description]
   * @return {[type]}      [description]
   */
  function mhash($key){
    $md5 = substr(md5($key), 0, 8);
    $seed = 31;
    $hash = 0;
    for($i=0;$i<8;$i++){
      $hash = $hash*$seed + ord($md5{$i});
    }
    return $hash & 0x7FFFFFFF;
  }
```
通过Hash函数把key转化成整数后，利用这个整数与服务器数量取模，得到其中一台的服务器配置。然后利用这个配置完成memcached服务器的连接操作。这样就完成了对数据的分布式存储。

## 一致性Hash分布
> 一致性hash分布是算法，是P2P网络和分布式存储中常用到一项技术。是一种当服务器有增删时，对数据影响最小的一种部署方案。

一致性Hash算法实现：
### 将一个32位整数（0~2^32-1）想象成一个闭环。
如图，
![将32位整数想象成一个闭环](http://n.sinaimg.cn/games/3ece443e/20161001/hash_1.png)
### 通过Hash函数将key处理成整数
```
$key1 = mhash("key1");
$key2 = mhash("key2");
$key3 = mhash("key3");
$key4 = mhash("key4");
```
将key通过mhash函数处理成整数，让后就可以将之对应到闭环上。如图，
![4个key通过mhash处理成整数对应到闭环上](http://n.sinaimg.cn/games/3ece443e/20161001/hash_2.png)
### 将memcached服务器，通过hash服务器所使用的IP地址，也对应到闭环上。
例如，有三台服务器，IP分别是192.168.1.1、192.168.1.2、192.168.1.3
```
$server1 = mhash("192.168.1.1");
$server2 = mhash("192.168.1.2");
$server3 = mhash("192.168.1.3");
```
如图，
![将服务器也映射到环上](http://n.sinaimg.cn/games/3ece443e/20161001/hash_server.png)

### 把数据映射到服务器上
**映射方法：**
沿着圆环的顺时针方向的key出发，直到遇上一个服务器，将key对应的数据存储到这个服务器上。

如图，
![把数据映射到服务器上](http://n.sinaimg.cn/games/3ece443e/20161001/hash_3.png)

### 移除服务器
假设服务器 server2 崩溃了，那么受影响的仅是哪些hash映射值在server1到server2的值。

如图，受影响的只有key2，它将会重新映射到server3服务器上。
![移除服务器](http://n.sinaimg.cn/games/3ece443e/20161001/hash_4.png)

### 添加服务器
如果现在需要新添加一台服务器，其IP地址为（192.168.1.4）
```
$server4 = mhash("192.168.1.4");
```
其在闭环上的映射位置位于如图的位置，那么受影响是hash映射值在其当前的位置和server2的位置之间的key。

如图，受影响的仅为key3，其将会重新映射到server4上。
![添加服务器](http://n.sinaimg.cn/games/3ece443e/20161001/hash_5.png)

### 实现（PHP）
```php
<?php
class FlexiHash{
  private $serverList = array();
  private $isSorted = false;

  /**
   * [addServer 通过hash函数计算除服务器的Hash值，通过此hash值定位服务器列表的某个位置，然后将服务器排序标识置为false]
   * @param {[type]} $server [description]
   */
  function addServer($server){
    $hash = $this->mhash($server);

    if(!isset($this->serverList[$hash])){
      $this->serverList[$hash] = $server;
    }

    $this->isSorted = false;
    return true;
  }

  /**
   * [removeServer 跟addServer实现差不多，将server删除，并将排序标识置为false]
   * @param  {[type]} $server [description]
   * @return {[type]}         [description]
   */
  function removeServer($server){
    $hash = $this->mhash($server);

    if(!isset($this->serverList[$hash])){
      unset($this->serverList[$hash]);
    }

    $this->isSorted = false;
    return true;
  }

  /**
   * [lookup 先计算除key的hash值，然后判断serverList是排序过，如果没有，就先对服务器列表进行倒序排序操作。倒序排序的作用是把服务器列表转换成一个逆时针的圆环。然后遍历服务器列表，找到一个合适的服务器返回]
   * @param  {[type]} $key [description]
   * @return {[type]}      [description]
   */
  function lookup($key){
    $hash = $this->mhash($key);

    if(!$this->isSorted){
      ksort($this->serverList, SORT_NUMERIC);
      $this->isSorted = true;
    }

    foreach($this->serverList as $pos => $server){
      if($hash >= $pos){
        return $server;
      }
    }

    return $this->serverList[count($this->serverList) - 1];
  }

  /**
   * [mhash 首先通过md5把key处理成一个32位的字符串，取其前8个字符。在经过hash算法处理成一个整数并返回。]
   * @param  {[type]} $key [description]
   * @return {[type]}      [description]
   */
  function mhash($key){
    $md5 = substr(md5($key), 0, 8);
    $seed = 31;
    $hash = 0;
    for($i=0;$i<8;$i++){
      $hash = $hash*$seed + ord($md5{$i});
    }
    return $hash & 0x7FFFFFFF;
  }

}

```
测试
```php
<?php
  $hserver = new FlexiHash();

  $hserver->addServer("192.168.1.1");
  $hserver->addServer("192.168.1.2");
  $hserver->addServer("192.168.1.3");
  $hserver->addServer("192.168.1.4");
  $hserver->addServer("192.168.1.5");

  echo "save key1 in server:".$hserver->lookup("key1");
  echo "save key2 in server:".$hserver->lookup("key2");
  echo "==============================================;

  $hserver->removeServer("192.168.1.4");
  echo "save key1 in server:".$hserver->lookup("key1");
  echo "save key2 in server:".$hserver->lookup("key2");
  echo "==============================================;

  $hserver->addServer("192.168.1.6");
  echo "save key1 in server:".$hserver->lookup("key1");
  echo "save key2 in server:".$hserver->lookup("key2");
  echo "==============================================;  
?>
```
通过测试结果可以看出，在增加或者减少memcached服务器的时候，一致性hash算法只会改变很少的一部分数据的存储服务器，从而能够实现最大限度的减少数据丢失的情况。

## 实际应用（PHP）
在实际应用当中，PHP提供了两种的关于Memcached的扩展，memcache和memcached。两种扩展中都可以设置采用哪种分布方式，我们只需要选择我们要采用的策略并修改配置即可，而不需要我们自己来实现具体的分布算法。

**memcache扩展配置**

控制key到服务器的映射（分布式）策略。 php.ini 配置
```ini
[Memcache]
Memcache.allow_failover = 1
memcache.max_failover_attempts = 2
Memcache.hash_strategy =consistent
Memcache.hash_function =crc32

```
Memcache.allow_failover
> 是否在发生错误时（对用户）透明的转移到其他服务器。

memcache.max_failover_attempts
> 定义在写入和获取数据时最多尝试的服务器次数（即：故障转移最大尝试数），仅和 memcache.allow_failover结合使用。

Memcache.hash_strategy
> 控制key到服务器的映射（分布式）策略。

- consistent，采用一致性hash分布策略实现映射
- standard，采用普通hash分布策略实现映射

memcache.hash_function
> 控制在key-server映射时使用哪个hash函数crc32 标明使用标准CRC32进行hash，fnv则说明使用FNV-1a。


**memcached扩展配置**
```
<?php
  $mem = new memcached();
  $mem->setOption(Memcached::OPT_DISTRIBUTION,Memcached::DISTRIBUTION_CONSISTENT);
  $mem->setOption(Memcached::OPT_LIBKETAMA_COMPATIBLE,true);
```

Memcached::OPT_DISTRIBUTION
> 指定元素key分布到各个服务器的方法。当前支持的方法有余数分步法合一致性hash算法两种。一致性hash算法提供 了更好的分配策略并且在添加服务器到集群时可以最小化缓存丢失。
>类型: integer, 默认: Memcached::DISTRIBUTION_MODULA.

Memcached::DISTRIBUTION_MODULA
> 余数分布算法。

Memcached::DISTRIBUTION_CONSISTENT
> 一致性分布算法(基于libketama).

Memcached::OPT_LIBKETAMA_COMPATIBLE
> 开启或关闭兼容的libketama类行为。当开启此选项后，元素key的hash算法将会被设置为md5并且分布算法将会 采用带有权重的一致性hash分布。这一点非常有用因为其他基于libketama的客户端（比如python，urby）在同样 的服务端配置下可以透明的访问key。
>
> **Note:**
> 如果你要使用一致性hash算法强烈建议开启此选项，并且这个选项可能在未来的发布版中被设置为默认开启。
类型: boolean, 默认: FALSE.
