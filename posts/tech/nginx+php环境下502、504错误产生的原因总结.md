---
layout: post
title: nginx+php环境下502、504错误产生的原因总结
category: 技术
keywords: php
date: 2016-05-11
author: lisijie
---
## 502 错误

502 表示网关错误，当nginx服务器作为代理或网关时，如果从请求响应链的上游（PHP）得到一个无效的响应时，就会产生这个错误。

在nginx+php-fpm环境下，当用户访问一个php页面时，nginx使用fastcgi协议将请求转发至php-fpm进程，php-fpm进程处理完后，返回结果给nginx，nginx再返回给用户。

以下几种情况会引起502错误：

- php-fpm.conf 设置了request_terminate_timeout，这个参数用来设置php-fpm进程终止处理的超时时间，设为0表示不启用，当设置为大于0值时，如果php脚本的执行时间超出该值，php-fpm主进程将kill掉该工作进程，并写一条execution timed out的error log。此时nginx连接被fpm进程主动断开，输出502错误。
- php-fpm 进程没有启动或者nginx的fastcgi_pass设置错误。此时nginx因为连接不上PHP，因此输出502错误。
- php-fpm 进程处理不过来，进程数不足或backlog太小。可能是某个php脚本存在性能问题，或者访问量突然增加超出了php的处理能力。前者可以通过fpm的slow log进行排查。

## 504 错误

504表示网关超时，当上游服务（PHP）没有在nginx指定的时间内响应时，nginx将输出504错误。在nginx+php-fpm环境下，有3个参数可以设置fastcgi的超时时间，分别是`fastcgi_connect_timeout`、`fastcgi_send_timeout`、`fastcgi_read_timeout`。为了避免超时产生504错误，可以在nginx配置中将这3个参数设置一个比较大的值，例如 `300`。


## 关于php.ini 中的 max\_execution\_time
max_execution_time，是设置脚本执行的超时时间，包括 sleep、curl 等操作消耗的时间不计算在内，当超过max_execution_time时，php将抛出一条 Fatal error: Maximum execution time of xxx seconds exceeded in xxx 的致命错误。不会导致502、504等错误。
