---
layout: post
title: Java中的反射
category: 技术
keywords: Java
date: 2016-02-19
author: lisijie
---

什么是反射：

> JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

反射使用步骤：

* 获取一个类的类类型
* 使用类类型的一系列方法获取类的信息
* 根据返回信息做进一步操作

### 1、类的类类型(Class Type)

Java中每个类都有一个类类型，类使用.class成员变量获取，对象使用getClass()方法获取，同一个类的所有实例获取到类类型都是相等的。

	String s = new String("hello");
	
	Class c1 = s.getClass();
	Class c2 = String.class;
	
	System.out.println(c1 == c2);


### 2、动态加载类

动态加载类可以用来加载一组实现相同接口的类，这样使用接口描述变量对象即可，而不用关心具体是哪个类。

	// 动态获取一个类类型
	Class c = Class.forName("org.lisijie.demo.Foo"); 
	// 使用类类型的 newInstance() 方法实例化
	Foo foo = (Foo) new c.newInstance();

### 3、获取方法信息

类的类类型提供了一系列获取类信息的方法。getMethods() 获取所有public方法信息，包括父类的；getDeclaredMethods()  方法获取的是该类定义的所有方法。

### 4、获取成员变量信息

使用类类型的 getFields() 和getDeclaredFields() 方法获取。

### 5、方法反射的基本操作

使用类类型的getMethod方法获取反射方法，第一个参数为方法名，后面的可变参数是参数类型列表，结果返回符合条件的方法对象。

	Class c = Foo.class;
	
	Method m = c.getMethod("bar", new Class[]{int.class, String.class});
	或
	Method m = c.getMethod("bar", int.class, String.class);
	
	调用方法：
	
	m.invoke(new Foo(), new Object[]{1, "hi"});


### 6、通过反射认识泛型本质

通过反射方法调用可以绕过泛型限制，说明Java中集合的泛型是防止错误输入的，只在编译阶段有效。

验证代码：

	// 定义两个ArrayList，一个不指定类型，一个指定为String类型
	ArrayList list = new ArrayList();
	ArrayList<String> list2 = new ArrayList<String>();
	
	list.add("hello");
	list.add(123);
	
	System.out.println(list);
	
	Class c = list2.getClass();
	Method m = c.getMethod("add", Object.class);
	m.invoke(list2, "hello");
	m.invoke(list2, 123);
	
	System.out.println(list2);


