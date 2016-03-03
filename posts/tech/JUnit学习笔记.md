---
layout: post
title: JUnit学习笔记
category: 技术
keywords: Java
date: 2016-03-03
author: lisijie
---

### JUnit官网：

http://junit.org/

### IDE快捷操作：

* eclipse 可以在类文件打开右键菜单，选择对该类创建测试类，可以一次性创建该类的所有测试方法。

* idea 可以在先在左侧文件列表中对test目录打开右键菜单， 选择“make directiry as ”把该目录设为测试目录。然后在代码编辑器中按 shift + ctrl + t 快速创建测试类。

### 测试方法：

1. 测试方法上必须使用@Test进行修饰。
2. 测试方法必须使用public void 进行修饰，不能带任何的参数。
3. 新建一个源代码目录来存放我们的测试代码。
4. 测试类的包应该和被测试类保持一致。
5. 测试单元中的每个方法必须可以独立测试，测试方法间不能有任何的依赖。
6. 测试类使用Test作为类名的后缀（不是必须）。
7. 测试方法使用test作为方法名的前缀（不是必须）。

### 注解说明：

* @Test:将一个普通的方法修饰成为一个测试方法。
     * @Test(expected=XX.class) 说明预期是抛出一个异常。
     * @Test(timeout=毫秒 )  设置该测试方法的最大运行时间，防止死循环。
* @BeforeClass：它会在所有的方法运行前被执行，static修饰。
* @AfterClass:它会在所有的方法运行结束后被执行，static修饰。
* @Before：会在每一个测试方法被运行前执行一次。
* @After：会在每一个测试方法运行后被执行一次。
* @Ignore:所修饰的测试方法会被测试运行器忽略。
* @RunWith:可以更改测试运行器 org.junit.runner.Runner。
* @BeforeClass修饰的方法会在所有方法被调用前被执行，而且该方法是静态的，所以当测试类被加载后接着就会运行它，而且在内存中它只会存在一份实例，它比较适合加载配置文件。
* @AfterClass所修饰的方法通常用来对资源的清理，如关闭数据库的连接。
* @Before和@After会在每个测试方法的前后各执行一次。

### 测试结果：

* Failure一般由单元测试使用的断言方法判断失败所引起的，这经表示测试点发现了问题，就是说程序输出的结果和我们预期的不一样。
* error是由代码异常引起的，它可以产生于测试代码本身的错误，也可以是被测试代码中的一个隐藏的bug。
* 测试用例不是用来证明你是对的，而是用来证明你没有错。


### 测试套件：

1. 测试套件就是组织测试类一起运行的。
2. 写一个作为测试套件的入口类，这个类里不包含其他的方法。
3. 更改测试运行器Suite.class。
4. 将要测试的类作为数组传入到Suite.SuiteClasses（{}）。

示例代码：
    
    @RunWith(Suite.class)
    @Suite.SuiteClasses({TaskTest1.class,TaskTest2.class,TaskTest3.class})
    public class SuiteTest {

    }


### 参数化测试设置：

1. 更改默认的测试运行器为RunWith(Parameterized.class)。
2. 声明变量来存放预期值和结果值。
3. 声明一个返回值 为Collection的公共静态方法，并使用@Parameters进行修饰。
4. 为测试类声明一个带有参数的公共构造函数，并在其中为之声明变量赋值。

示例代码：
	
	@RunWith(Parameterized.class)
	public class ParameterTest {
	
	     int expected =0;
	     int input1 = 0;
	     int input2 = 0;
	     
	     @Parameters
	     public static Collection<Object[]> t() {
	          return Arrays.asList(new Object[][]{
	                    {3,1,2},
	                    {4,2,2}
	          }) ;
	     }
	     
	     public ParameterTest(int expected,int input1,int input2) {
	          this.expected = expected;
	          this.input1 = input1;
	          this.input2 = input2;
	     }
	     
	     @Test
	     public void testAdd() {
	          assertEquals(expected, new Calculate().add(input1, input2));
	     }
	}


以上总结整理自慕课网课程。