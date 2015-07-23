---
layout: post
title: 使用vagrant统一开发环境
category: 技术
keywords: vagrant,linux
date: 2015-07-23
author: lisijie
---

我平常用的电脑有家里的台式机、Mac笔记本、公司的电脑，经常需要在不同的电脑上去写代码。我习惯在Windows下写代码，到Linux下运行。这样我就要在每台机上面都搭建开发环境，非常麻烦，而且操作系统还不一样，所以我希望有个工具可以让我只搭建一次，到处运行。而 Vagrant 就是极佳的选择。

Vagrant是一个基于Ruby的工具，用于创建和部署虚拟化开发环境。Windows、Linux、Mac都可以使用，依赖virtualbox或vmware。

## 安装步骤

#### 1. 安装virtualbox 

首先需要安装虚拟机软件，我选择virtualbox，官方网站是 https://www.virtualbox.org/ 下载后按提示安装即可。

#### 2. 安装Vagrant

官方网站是 https://www.vagrantup.com/ 下载后也是按提示安装。vagrant 是一个命令行工具，没有GUI界面，安装完后打开cmd，输入 vagrant 命令即可显示帮助信息。建议安装 msysgit 并把 "Git Bash Here" 添加到右键菜单，这样就可以随时进入命令行界面了。

#### 3. 下载镜像

镜像就是一个封装好的操作系统，扩展名是.box，[http://www.vagrantbox.es/](http://www.vagrantbox.es/ "http://www.vagrantbox.es/") 这个网站上面有很多，根据自己的喜好选择，我选centos，建议使用下载工具先下回本地。

#### 4. 添加镜像

使用以下命令将镜像添加到vagrant：

	$ vagrant box add centos ./centos-7.0-x86_64.box

其中 *centos* 是别名，可以自己随便取，*./centos-7.0-x86_64.box* 是我刚下回来的镜像文件路径。

#### 5. 初始化环境

现在创建一个目录作为我们的开发环境，例如命名为 develop ，然后打开终端进入develop目录，输入以下命令：

	$ vagrant init centos

*centos* 是我上面添加的box别名，说明是使用centos这个镜像来创建虚拟机。完成后会在当前目录下生成一个名为 **Vagrantfile** 的配置文件。最后启动虚拟机：

	$ vagrant up

此时打开virtualbox可以看到上面多了个由vagrant创建的虚拟机，而虚拟机本身的数据是存放在virtualbox设置的虚拟机目录的，而不是我们刚创建的开发目录 develop。

启动后vagrant会自动把虚拟机的http 80端口和ssh的22端口分别映射到我们本机的8080和2222端口。打开浏览器访问 http://localhost:8080 相当于访问虚拟机的http服务，当然我认为最好的方式还是通过配host的方式直接访问虚拟机。 

#### 6. 登录到虚拟机

打开终端进入到工作目录（develop），执行：

	$ vagrant ssh
	
即可直接登录到虚拟机。你也可以使用xshell或securecrt等终端工具登录，帐号和密码都是**vagrant**，root帐号密码也是**vagrant**。然后就可以为所欲为了。

vagrant默认会把工作目录映射到虚拟的 **/vagrant** 目录下。


#### 7. 其他设置

默认情况下vagrant虚拟机是只能通过映射端口去访问的，这种方式太过局限，我们可以让虚拟机通过桥接方式拥有自己的内网IP，并且可以访问到公网。打开配置文件 **Vagrantfile** ，把

	# config.vm.network "public_network"

前面的 # 号注释掉。再使用 vagrant reload 命令重启虚拟机。然后登录进虚拟机，使用ifconfig命令可看到新增的网卡和IP地址，以后就可以用这个IP地址直接访问虚拟机了。

#### 8. 配置开发环境

接下来就可以根据自己的需要安装配置Nginx、PHP、MySQL、Memcached、Redis等程序了。

#### 9. 打包分发

一切都配置好后，就可以打包带走了，或者分发给其他人使用。 打包前需要先关闭虚拟机，使用命令

	$ vagrant halt  # 关闭虚拟机
	$ vagrant package --output centos-lnmp.box # 打包导出到centos-lnmp.box

打包完后会在当前目录下生成centos-lnmp.box，然后连同 develop 目录一同拷到U盘，换到其他电脑就可以直接用了。

#### 10. 注意事项

使用 Apache/Nginx 时会出现诸如图片修改后但页面刷新仍然是旧文件的情况，是由于静态文件缓存造成的。需要对虚拟机里的 Apache/Nginx 配置文件进行修改：

	# Apache 配置添加:
	EnableSendfile off
	# Nginx 配置添加:
	sendfile off;


参考：[http://segmentfault.com/a/1190000000264347](http://segmentfault.com/a/1190000000264347)