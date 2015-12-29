---
layout: post
title: PHP项目配置运行环境的正确姿势
category: 技术
keywords: nginx
date: 2015-10-17
author: lisijie
---

通常一个web项目的开发过程都需要经历 开发->测试->上线 3个过程，为了防止数据互相干扰，需要根据不同环境配置不同的数据库、缓存之类的信息。如果在项目部署的时候手动去修改配置，显得非常不灵活，而且容易被覆盖。更好的方式时把所有环境的配置信息分目录存储，然后根据环境变量自动加载当前环境的配置。

nginx中添加自定义环境变量的方法是：

在配置文件的location段使用 **fastcgi\_param var value;** 设置。

apache 的设置方法是：

在VirtualHost配置中使用 **SetEnv var value** 进行设置。

假设我们的php项目有开发环境（development）、测试环境（testing）、生产环境（production） 3种环境，分别对开发人员、测试人员和线上服务区分配置，常量 RUN_MODE 为当前的运行环境。如果没有配置运行环境的环境变量，默认为生产环境，这样代码直接发布到线上就可以使用，无需再做额外的修改。而开发人员在搭建自己的开发环境时，只需要在本地的nginx/apache配置文件中增加相应的环境变量，无需去修改代码。

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


