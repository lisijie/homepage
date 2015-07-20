

## 1. 安装MinGW-w64

我的系统是64位win7，早期版本的MinGW我安装后编译go的相关工具会一路报错，换成mingw-w64就可以顺利编译，下载地址：

	http://sourceforge.net/projects/mingw-w64/

下载完进行安装，处理器架构选择x86_64（64位），其他选项使用默认，下一步选择安装路径，假设为C:\MinGW，按下一步开始下载安装。最后把 X:\MinGW\mingw64\bin 加入到系统环境变量。

## 2. 构建Go标准包

进入到go的src目录下，我的是 C:\go\src，执行 all.bat 进行编译，不出意外的话很快就会完成。

接下来就可以在windows下编译linux平台的二进制文件了，方法是：

	:: 设置目标环境处理器架构
	set GOARCH=386
	:: 设置目标操作系统
	set GOOS=linux
	:: 开始编译
	go build