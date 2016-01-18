---
layout: post
title: 使用 Supervisor 管理后台进程
category: 技术
keywords: Supervisor
date: 2016-01-16
author: lisijie
---

supervisor 是一个linux下的后台进程监控管理工具，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启。官方网站是 [http://supervisord.org](http://supervisord.org)。

## 安装

centos下可以使用yum进行安装：

     yum install supervisor

安装完后有两个可执行程序：

* supervisord  后台主程序，负责进程监控和重启。安装完后自动启动。
* supervisorctl supervisord的控制台程序，用来向supervisord发出命令，如查看运行状态、刷新配置、启动某个程序、停止某个程序等。


## 配置

安装后在/etc目录下会生成一个名为supervisord.conf的配置文件和一个配置目录supervisord.d。我们可以把要监控的进程直接配置到supervisord.conf或者写成一个单独的文件放到supervisord.d目录，supervisord.conf会把supervisord.d目录下所有扩展名为.ini的配置文件包含进来。

一个基本的程序配置如下：

	[program:myapp]	
	command=/bin/cat
	process_name=%(program_name)s
	numprocs=1
	autostart=true
	autorestart=true
	
* program: 后面的 myapp 为你的程序名。
* command 是配置启动程序的命令。
* process_name 是显示在 supervisorctl 中的进程名称，默认值 %(program_name)s 表示使用跟程序一样的名字，如果名字不一样，那么在 supervisorctl 中显示的名字为 `程序名:进程名` 。如果程序名和进程名一样，你可以使用 supervisorctl start [程序名] 进行启动，否则需要使用 supervisorctl start [程序名:进程名] 启动。
* numprocs 启动多少个进程，默认是1个，如果大于1个，上面的进程名称必须加上%(process_num)s变量。
* autostart 是否在supervisord启动时自动启动（默认为true）。
* autorestart 是否在进程异常退出时自动重启（默认为true）。


除此之外，supervisord 还有很多其他的配置，详细的配置说明请看官方文档：[http://supervisord.org/configuration.html](http://supervisord.org/configuration.html)


## 使用

启动supervisord：

	# supervisord -c /etc/supervisord.conf
	
进入 supervisorctl 交互界面：

	# supervisorctl
	supervisor> help

	default commands (type help <topic>):
	=====================================
	add    clear  fg        open  quit    remove  restart   start   stop  update 
	avail  exit   maintail  pid   reload  reread  shutdown  status  tail  version

进入 supervisorctl 后会显示出所有进程的状态，输入 help 可以看到有哪些命令。

#### 示例

这里以一个简单的php脚本为例介绍如何把它变成后台进程。

	<?php
	// 这个脚本每个3秒输出当前时间到日志文件 app.out
	while (true) {
        sleep(3);
        file_put_contents(__DIR__.'/app.out', date("Y-m-d H:i:s")."\n", FILE_APPEND);
	}

把以上脚本保存到 myapp.php ，然后到 /etc/supervisord.d 目录创建配置文件 myapp.ini ，内容如下：

	[program:myapp]
	command=/usr/local/php/bin/php /root/myapp.php
	process_name=%(program_name)s 
	numprocs=1 
	autostart=true 
	autorestart=true

使用 supervisorctl 启动进程：

	# supervisorctl start myapp
	
查看进程的运行状态：

	# supervisorctl status
	myapp                            RUNNING   pid 1567, uptime 0:00:14
	
可以看到已经运行起来了，进程ID是 `1567`，已经运行了14秒。再使用ps查看系统进程：

	# ps -ef | grep myapp
	root      1567  1523  0 11:02 ?        00:00:00 /usr/local/php/bin/php /root/myapp.php
	
可以看到进程确实存在，进程ID是`1567`，父进程ID是`1523`，而在我的系统中ID为1523的进程是 supervisord 的主进程，说明 myapp 是使用 supervisord 启动的。我们可以使用 kill 命令将myapp的进程杀掉，看看是否会自动重启。

	# kill 1567
	# ps -ef | grep myapp
	root      1581  1523  0 11:12 ?        00:00:00 /usr/local/php/bin/php /root/myapp.php

可以看到进程ID变成1581了，说明原来的进程被杀掉后，supervisord 检测到进程退出，又启动了一个。