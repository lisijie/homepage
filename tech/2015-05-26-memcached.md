---
layout: post
title: memcached 应用笔记
category: 技术
keywords: memcached
---

## memcached 需要注意的地方

- 值上限为1M
- 内存利用率不高
- 数据结构单一，只能保存字符串值
- key 最大250字节
- 设置过期时间需要注意的是：如果小于2592000（30天），则表示当前时间加上该值，大于30天则当成unix时间戳。因此要设置大于30天的过期时间必须用绝对时间戳。
- php的memcached扩展默认会开启压缩功能，当值大小达到阈值（默认2000字节，可在php.ini设置）会启用压缩功能，实际存进memcached的是压缩后的值，所以有时会发现值大于1M还能set成功，在获取该值时进行解压缩然后返回，使得压缩对应用层透明。


## memcached 的内存分配

1. 启动时指定memcached内存上限，例如 2G。
2. memcached 将内存分成多个板（slab），每块1M。
3. 每个slab又划分为多个固定大小的块（chunk），chunk是实际存放值的地方，chunk大小最小为96byte，最大为1M，因此memcached的值大小上限为1M，超过会写入失败。
4. 多个chunk大小相同的slab组成一个组，称为slab class。
5. 启动时可以通过指定 -f 参数设置chunk的增长因子，默认是1.25，合理设置可以减少内存浪费。

		[vagrant@bogon ~]$ /usr/local/memcached/bin/memcached -f 5 -vv
		slab class 1: chunk size 96 perslab 10922
		slab class 2: chunk size 480 perslab 2184
		slab class 3: chunk size 2400 perslab 436
		slab class 4: chunk size 12000 perslab 87
		slab class 5: chunk size 60000 perslab 17
		slab class 6: chunk size 1048576 perslab 1


6. 写入时根据值大小找到一块最合适的块进行写入。缺点是内存利用率不高。例如要将100字节的值存到128字节的chunk中，剩下28字节就浪费了。

![附图](http://blog.lisijie.org/images/memcached-01.png)

## memcached 状态查看

1. telnet 到memcached，然后使用stats、stats slabs、stats items命令查看
2. 使用libmemcaced自带的memstat工具
3. 使用memcached-tool脚本可以方便查看slab使用情况

## memcached 适不适合用来存储session?

memcached会将内存划分为不同大小的slab，96byte到1m，而session的数据大小一般都是差不多大，mc在存储数据时只会选择最合适大小的slab，而不管slab是否用满，所以只会用到一部分的slab，其他就浪费了。当slab用满后，LRU算法就会进行踢除了，会造成表面上看上去内存没用多少，但是用户却频繁掉线的问题。所以mc不适合用来存储session，应该考虑用redis或mysql等。

参考：
[http://www.infoq.com/cn/news/2015/01/memcached-store-session/](http://www.infoq.com/cn/news/2015/01/memcached-store-session/)