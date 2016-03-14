---
layout: post
title: Java Web的过滤器和监听器
category: 技术
keywords: Java
date: 2016-03-14
author: lisijie
---

## 过滤器

过滤器用于对web资源请求和响应进行检查和处理，一个请求会先经过过滤器，然后再到servlet，多个过滤器在一起形成一条过滤链。使用过滤器可以对请求进行登录验证、请求转发等操作。

### 创建过滤器

创建类并实现javax.servlet.Filter接口，该接口定义了以下几个方法：

	import javax.servlet.*;
	import java.io.IOException;
	
	public class TestFilter implements Filter {
	
	    @Override
	    public void init(FilterConfig config) throws ServletException {
	        // TODO 用于完成Filter的初始化
	    }
	
	    @Override
	    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
	        // TODO 对请求进行处理
	        chain.doFilter(request, response); // 如果允许继续执行的话, 将请求传递给下一个过滤器
	        // TODO 对响应进行处理
	    }
	
	    @Override
	    public void destroy() {
	        // TODO 对某些资源进行回收
	    }
	}	
	

在web.xml里的<filter>和<filter-mapping>标签配置过滤器的名称、类名、URL匹配规则、参数等
	
	<filter>
		<filter-name>testFilter</filter-name>
		<filter-class>filter.TestFilter</filter-class>
		<init-param>
			<param-name>参数1</param-name>
			<param-value>参数值1</param-value>
		</init-param>
		<init-param>
			<param-name>参数2</param-name>
			<param-value>参数值2</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<!-- 上面定义的过滤器名称 -->
		<filter-name>testFilter</filter-name>
		<!-- 定义过滤器的URL匹配规则 -->
		<url-pattern>/*</url-pattern>
		<!-- 声明过滤器类别，默认是REQUEST -->
		<dispatcher>REQUEST</dispatcher>
	</filter-mapping>

### 过滤器执行顺序

- web应用启动时执行 `init` 方法。
- 有请求进来时执行`doFilter()`方法。
- doFilter() 方法里调用 `chain.doFilter()` 把请求传递给下一个过滤器，所有过滤器执行完成则执行servlet。
- servlet执行完后，依次执行各个过滤器chain.doFilter()后的代码。
- web应用关闭时调用 `destroy()` 方法。

### 过滤器分类：

- REQUEST （默认），直接访问页面时，web容器将会调用过滤器。
- FORWARD，目标资源是通过RequestDispathcer的forward访问时，过滤器将被调用。
- INCLUDE，目标资源是通过RequestDispathcer的include访问时，过滤器将被调用。
- ERROR，目标资源是通过声明式异常处理机制调用时，过滤器将被调用。
- ASYNC (Servlet 3.0) 支持异步处理。

### Servlet 3.0 新特性：

- 增加ASYNC异步处理过滤器类型。
- 支持使用@WebFilter注解配置过滤器。


## 监听器

用于在对象事件的发生前、发生后做一些处理操作。

在Java Web中，监听的事件源为ServletContext、HttpSession、ServletRequest 3大对象。其中ServletContext和HttpSession对象在应用启动时创建，只有一个实例；ServletRequest，每次有请求过来时创建，请求结束后销毁。

### 监听器类型

监听器的类型，按监听对象划分，有以下几种：

- ServletContextListener，监听应用启动和关闭。
- HttpSessionListener，监听会话产生和销毁。
- ServletRequestListener，监听每个请求初始化、结束。

按监听事件划分，有以下几种：

- ServletContextAttributeListener
- HttpSessionAttributeListener
- ServletRequestAttributeListener

用于监听对象中属性的增加、删除等事件，当调用相应对象的setAttribute()、removeAttribute()方法设置属性时，监听器可进行捕获。

### 监听器使用步骤

- 创建实现Listener接口的类。
- 在web.xml中配置监听器，按照配置文件的先后顺序执行。

Servlet 3.0 新特性：

- 使用 @WebListener 注解标注类。
- 无需配置web.xml（无法配置先后顺序）。