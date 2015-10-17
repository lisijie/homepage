---
layout: post
title: nginx增加fastcgi参数
category: 技术
keywords: nginx
date: 2015-10-17
author: lisijie
---

在nginx配置的location段使用 **fastcgi\_param var value;** 可对每个请求增加环境变量，常用来指定一些特殊的配置信息，如运行环境参数、数据库配置等。在PHP中使用$_SERVER['var'] 获取。

这样做的好处是：

1. 开发人员可以根据自己的需要修改本地nginx的fastcgi_param，而无需改动项目代码，防止误提交代码。
2. 运维人员可以把mysql之类的配置信息放在nginx配置中，代码中根据环境变量获取，对开发透明，线上需要更改配置信息时直接由运维更改后重启nginx，无需开发人员进行修改发版。

实例：

假设我们的php项目有开发环境（development）、测试环境（testing）、生产环境（production） 3种环境，分别对开发人员、测试人员和线上服务区分配置，常量 RUN_MODE 为当前的运行环境。

nginx配置：

	server {
	     listen 80;
	     server_name local.example.com;
	     root /data/htdocs/example.com;
	     index index.php;
	     location ~ \.php$ {
	          fastcgi_param RUN_MODE "development";
	          fastcgi_pass 127.0.0.1:9000;
	          include fastcgi.conf;
	     }
	}

php示例代码：
	
	// 默认是生产环境
	define("RUN_MODE", isset($_SERVER['RUN_MODE']) ? $_SERVER['RUN_MODE'] : 'production');
	
	/**
	* 加载配置信息
	*
	* 如果当前环境下有自己的配置，则加载环境配置，否则加载默认配置。
	*
	* @param string $file
	* @return array
	*/
	function load_config($file) {
	     $filename = CONFIG_PATH . RUN_MODE . "/{$file}.php";
	     if (!is_file($filename)) {
	          $filename = CONFIG_PATH . "/{$file}.php";
	     }
	     if (is_file($filename)) {
	          return include $filename;
	     }
	     throw new Exception("配置不存在: {$file}");
	}


