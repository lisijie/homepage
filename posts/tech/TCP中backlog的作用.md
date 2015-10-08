---
layout: post
title: TCP中backlog的作用
category: 技术
keywords: tcp,backlog
date: 2015-09-30
author: lisijie
---

Linux内核为每个TCP服务器程序维护两条backlog队列，一条是TCP层的未连接队列，一条是应用层的已连接队列，分别对应net.ipv4.tcp_max_syn_backlog和net.core.somaxconn两个内核参数。

一个客户端连接在完成TCP 3次握手之前首先进入到未连接队列，完成握手之后正式建立连接，进入已连接队列，交付给应用程序处理。应用程序调用accept()函数从已连接队列取出连接进行处理。应用层在调用listen()函数时指定的backlog是已连接队列大小，如果大于somaxconn将被设为somaxconn。

如果应用层不调用accept()函数处理一个连接，或者处理不及时的话，将会导致已连接队列堆满。已连接队列已满的话会导致未连接队列在处理完3次握手之后无法进入已连接队列，最终也导致未连接队列堆满，在服务器看到处于未连接队列中的连接状态为SYN\_RECV。 新进来的客户端连接将会一直处于SYN\_SENT状态等待服务器的ACK应答，最终导致连接超时。

**查看队列大小：**

查看未连接队列默认值：

	cat /proc/sys/net/ipv4/tcp_max_syn_backlog

查看已连接队列默认值：

	cat /proc/sys/net/core/somaxconn

**修改队列大小：**

可以直接改写这两个文件的值。要永久修改这两个内核参数的话可以写到/etc/sysctl.conf：

	net.ipv4.tcp_max_syn_backlog = 1024
	net.core.somaxconn = 1024

改完后执行sysctl -p 让修改立即生效。