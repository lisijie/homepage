---
layout: post
title: windows搭建Go语言交叉编译环境
category: 技术
keywords: Go,Windows,交叉编译
date: 2015-07-21
author: lisijie
---

我的环境：

 - 64位 windows 7 SP1
 - go version go1.4.2 windows/amd64

## 1. 安装MinGW-w64

我的系统是64位win7，早期版本的MinGW我安装后编译go的相关工具会一路报错，换成mingw-w64就可以顺利编译，下载地址：

	http://sourceforge.net/projects/mingw-w64/

下载完进行安装，处理器架构选择x86_64（64位），其他选项使用默认，下一步选择安装路径，假设为C:\MinGW，按下一步开始下载安装。最后把 X:\MinGW\mingw64\bin 加入到系统环境变量。

## 2. 构建Go标准包

进入到go的src目录下，我的是 C:\go\src，执行 all.bat 进行编译，不出意外的话很快就会完成。

接下来就可以在windows下编译linux平台的二进制文件了，进入项目目录，创建一个批处理文件 make-linux.bat，输入以下命令：

	:: 设置目标环境处理器架构
	set GOARCH=amd64
	:: 设置目标操作系统
	set GOOS=linux
	:: 开始编译
	go build
	pause

保存后运行，就可以看到当前目录下已经编译生成了可在64位linux环境下运行的可执行文件。

环境变量 GOARCH 和 GOOS 分别用来指定编译目标环境的处理器架构和操作系统类型，支持以下组合：

	$GOOS		$GOARCH
	darwin		386
	darwin		amd64
	dragonfly	386
	dragonfly	amd64
	freebsd		386
	freebsd		amd64
	freebsd		arm
	linux		386	
	linux		amd64
	linux		arm
	netbsd		386
	netbsd		amd64
	netbsd		arm
	openbsd		386
	openbsd		amd64
	plan9		386
	plan9		amd64
	solaris		amd64
	windows		386
	windows		amd64

注意Go语言对系统是有要求的，版本太低的系统可能不支持，具体可以看这里 [https://golang.org/doc/install](https://golang.org/doc/install "https://golang.org/doc/install")

## 3. 使用交叉编译工具Gox

gox 是一个交叉编译的辅助工具，项目地址在 [https://github.com/mitchellh/gox](https://github.com/mitchellh/gox)

使用 go get github.com/mitchellh/gox 进行安装后会在 $GOPATH/bin 目录下生成 gox.exe，为了方便以后使用，最好拷到 c:\go\bin 目录下。

首先需要先编译出其他平台所需的库

	$ gox -build-toolchain
	The toolchain build can't be parallelized because compiling a single
	Go source directory can only be done for one platform at a time. Therefore,
	the toolchain for each platform will be built one at a time.
	
	--> Toolchain: darwin/386
	--> Toolchain: darwin/amd64
	--> Toolchain: linux/386
	--> Toolchain: linux/amd64
	--> Toolchain: linux/arm
	--> Toolchain: freebsd/386
	--> Toolchain: freebsd/amd64
	--> Toolchain: openbsd/386
	--> Toolchain: openbsd/amd64
	--> Toolchain: windows/386
	--> Toolchain: windows/amd64
	--> Toolchain: freebsd/arm
	--> Toolchain: netbsd/386
	--> Toolchain: netbsd/amd64
	--> Toolchain: netbsd/arm


然后进入到项目目录，执行 gox 即可一次性完成所有平台的编译。

	$ gox
	Number of parallel builds: 4
	
	-->      darwin/386: github.com/lisijie/mdwiki
	-->    darwin/amd64: github.com/lisijie/mdwiki
	-->       linux/386: github.com/lisijie/mdwiki
	-->     linux/amd64: github.com/lisijie/mdwiki
	-->       linux/arm: github.com/lisijie/mdwiki
	-->     freebsd/386: github.com/lisijie/mdwiki
	-->   freebsd/amd64: github.com/lisijie/mdwiki
	-->     openbsd/386: github.com/lisijie/mdwiki
	-->   openbsd/amd64: github.com/lisijie/mdwiki
	-->     windows/386: github.com/lisijie/mdwiki
	-->   windows/amd64: github.com/lisijie/mdwiki
	-->     freebsd/arm: github.com/lisijie/mdwiki
	-->      netbsd/386: github.com/lisijie/mdwiki
	-->    netbsd/amd64: github.com/lisijie/mdwiki
	-->      netbsd/arm: github.com/lisijie/mdwiki


如果只想编译64位linux和windows下的程序，可使用：

	gox -os "windows linux" -arch amd64

更多用法请看帮助信息
	
	gox -h

