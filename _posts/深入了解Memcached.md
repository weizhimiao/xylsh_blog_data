---
title: 深入了解Memcached
date: 2016-11-27 22:30:00
tags:
- memcached
categories:
- Linux
---

![Memcached多线程模型原理](http://n.sinaimg.cn/games/3ece443e/20161128/memcachedDuoXianChengMoXingYuanLiTu.png)

<!-- more -->

### Memcached的高并发支持原理

- 多路复用I/O模型
- 多线程模式


Memcached 使用了多路复用I/O模型（如，epoll、select等）。

传统的阻塞I/O中，系统可能会因为某个用户连接还没有做好I/O准备而一直等待，知道这个连接做好I/O准备。如果在这是有其他的用户连接到服务器，很可能会因为系统阻塞而得不到响应。

而多路复用I/O是一种消息通知模式，用户连接做好I/O准备后，系统会通知我们这个连接可以进行I/O操作了，这样系统就不会阻塞在某个用户连接了。因此就能够支持更高的并发。

此外，Memcached使用了多线程模式，在开启Memcached服务器时，通过使用 `-t` 参数指定要开启的线程数。（但，并不是线程数越多越好，一般设置为CPU核数，这样效率最高。）

### Memcached内存分配算法了解

Memcached 在存储数据时，使用的是Slab内存分配算法。这种算法可以减少生成的内存碎片，提高使用效率。但这种算法也导致Memcached只能够使用不大于1MB的内存（所以，Memcache默认只能存储不大于1MB的数据）。


### Slab分配算法原理

Slab分配算法把每1MB大小的内存块称之为一个slab页，每次向系统申请一个slab页（所以，slab是一次申请内存的最小单位），然后再通过分割算法将这个slab页分割成若干小块的chunk，然后把这些chunk分配给用户使用。

![img](http://n.sinaimg.cn/games/3ece443e/20161128/slabNaCunFenPeiYuanLi.png)


默认情况下，Memcached可分为40种slab页，每种slab页的chunk块大小都不相同。

当保存数据时Memcached向Slab层申请内存时，Slab层找到一个一个合适的slab页，然后分配其中一个空闲的chunk块给Memcached使用。

### Memcached过期数据删除机制

Memcached 为每一个item都设置了一个过期时间，但不是到期了之后就把item从内存中删除，而是访问item时，先进行判断，如果到期有效期，才把item从内存中删掉。

### Memcached淘汰数据算法
> 当内存不足时，Memcached会把访问比较少的或者一段时间没有访问的item淘汰（并不是主动去遍历那些过期的item），以便腾出内存空间存放新的item。

当Memecached使用的内存数大于设置的内存数之后，为了腾出空间来存放新的数据项，Memcached采用的是LRU算法(最近最少使用算法)来淘汰旧的数据项。

数据淘汰流程：

- 1、当Memcached第一次申请内存失败时，就开始算法进行淘汰数据。
- 2、首先从数据项列表（item_list）尾部开始遍历。
- 3、在列表中查找一个引用计数器（refcount）为0的item，把此item占用的内存释放掉。
- 4、再次申请内存，如果失败的话，继续进行淘汰算法。
- 5、查找3小时没有访问过的item，并将这些item释放掉。
- 6、再次申请内存，如果还是失败的话，就返回NULL（申请内存失败）。

**Tips：**

1、为什么从数据项列表尾部开始遍历？

因为，Memcached会把刚刚访问过的item放到item列表头部，所以尾部的item就是没有被访问过的（或者是很少被访问到的），这就是LRU的精髓。

### Memcached多线程模型

Memcached 是一个多线程的缓存服务器程序。在Memcached内部，线程分为：
- 主线程（main thread）：负责接收客户端连接，并把连接分配给工作线程处理；
- 工作线程（worker thread）：处理主线程分配过来的客户端连接请求。

Memcached 多线程模型原理如图所示。


![Memcached多线程模型原理](http://n.sinaimg.cn/games/3ece443e/20161128/memcachedDuoXianChengMoXingYuanLiTu.png)

主线程主要负责侦听客户端连接，在客户端连接到Memcached时，Memcached接收到来的请求，并把连接push到工作线程的CQ队列中，并向工作线程发送一个信号，告诉工作线程有新的客户端连接需要处理。

当工作线程收到主线程的信号后，便会把CQ队列上的客户端连接注册到libevent进行侦听，libevent会侦听客户端连接的读写请求，病调用相关的回调函数进行处理。
