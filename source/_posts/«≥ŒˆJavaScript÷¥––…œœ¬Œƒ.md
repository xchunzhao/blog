title: 浅析JavaScript执行上下文
date: 2015-12-09 21:47:35
tag: [Execute Context]
categories: JavaScript
---
之前对js只是会用，简单搭点页面。随着大前端时代的来临，准备对js作稍微深入的了解。最近再看那本`JavaScript学习指南`,对js执行顺序的问题总是有些疑惑。总结一下，以便日后回顾。

首先来看一段简单的js代码：

```
	console.log(x);
	console.log(a);
	var x=20;
	function a(){
		console.log('hello world');
	};
```
对js稍微有点了解的都知道，js在执行时对函数声明、变量声明提前。大概执行流程如下：<!-- more -->

```
	var x=undefined;
	function a(){
		console.log('hello world');
	};
	console.log(x);
	console.log(a);
	x=20;
	//运行结果如下：
	undefined
	[Function: a]
```
那对于命名冲突、函数形参的处理，js的执行引擎是怎么解决的呢？

`ECMAScript3`中对js执行上下文有这样的说明：
执行上下文中是一个栈结构,具体可以查阅[这篇文章](http://segmentfault.com/a/1190000002446390),同时会有一个变量对象`VO`,存储着上下文的以下内容：
* 变量声明
* 函数声明
* 函数形参
具体可以参考[这篇文章](http://www.cnblogs.com/TomXu/archive/2012/01/16/2309728.html)

以下是js的执行顺序：
1、首先按顺序对变量声明提前，并做初始化；
	* 函数参数(若未传入，则初始化该参数undefined);
	* 函数声明(发生命名冲突，覆盖之前的声明);
	* 变量声明(初始化变量为undefined,命名冲突,会忽略本次声明)
2、按照顺序执行赋值语句。

知道上述的一些执行原理后，再来看下面这个问题就简单多了。

```
	console.log(x);
	var x=20;
	console.log(x);
	x=10;
	function x(){};
	console.log(x);
	if(true){
		var a=1;
	}else{
		var b=true;
	}
	console.log(a);
	console.log(b);
```
进入global执行上下文后执行顺序如下：
```
	function x(){};
	var a,b=undefined;
	console.log(x);//[Function:x]
	x=20,console.log(x);//20
	x=10,console.log(x);//10
	a=1,console.log(a);//1
	console.log(b);//undefined
```

对上述需要补充的是，js中没有块级的作用域，js作用域靠函数形成，函数内部定义的变量、函数外部不可以访问(当然，闭包是可以解决以上问题的)。

JavaScript中需要注意的点还是比较多的，但是灵活也显而易见。