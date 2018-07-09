title: JavaScript原型继承
date: 2015-12-21 22:04:17
comments: true
tags: 
        - JavaScript
---

简单总结一下javaScript中原型机制。

js作为一个动态的OOP语言，同样具有继承特性。但是js的继承机制是最为诟病的，是基于原型继承的语言。

在javaScript中，每个对象都保持着一块隐藏的状态——一个对另一个对象的引用，也称作原型。<!-- more -->

```
	var Person = function(){};
	var p=new Person();
```
上述代码执行过程中，new到底做了什么操作？
1、var p={},初始化变量p
2、p.__proto__=Person.protptype
3、p=Person.construct(),调用实例的构造函数。

以上的大意就是讲，一个类的prototype对象中，有一个属性constructor实例化出一个对象，实例化出来的实例中有一个属性__proto__指向该类的prototype对象。

更直观一点可以看以下内存结构：
![](http://jbcdn2.b0.upaiyun.com/2012/05/JavaScript-prototypes-and-inheritance3.jpg)

当然每个对象都是从Object.protptype继承而来，所以我们在使用String、Array等内置对象的时候，都会有toString()方法，所以，原型对象更像是一个共享属性的对象。

顺便提一下，在执行过程中，查找变量是先从本作用域找，有就使用，没有就找父级的原型作用域，一直到Object，Object.prototype=null，这样就形成了链条式的作用域链，也叫原型链。

那如何实现原型继承呢？
例如：

```
	function B(){};
```
此时产生B原型的原型B.prototype 
B.prototype其实是一个Object对象，内容如下:

```
	{
	constructor : B , 
	[[Prototype]] : Object.prototype
}
```
因此，B原型链指向的是Object的原型。
那我想把B的原型链指向A，那这样就可以达到B extends A的目的了，应该如何实现呢？

改变原型链引用地址。
```
B.protptype=A
```
这样就可以实现简单的继承。

在使用原型链实现继承时有一些需要注意的地方：

* 注意继承后`constructor`的变化。
* 通过原型链实现继承时，不能使用字面量定义原型方法，因为这样会重写原型对象

看如下代码：
```
	function SuperClass(){
        this.name = "women";
        this.bra = ["a","b"];
    }
    function SubClass(){
        this.subname = "your sister";
    }
    SubClass.prototype = new SuperClass();
    var sub1 = new SubClass();
    sub1.name = "man";
    sub1.bra.push("c");
    console.log(sub1.name);//man
    console.log(sub1.bra);//["a","b","c"]
    var sub2 = new SubClass();
    console.log(sub1.name);//woman
    console.log(sub2.bra);//["a","b","c"]
```

#### 注意：此处在数组中添加一个元素，所有继承自SuperClass的实例都会受到影响，但是如果修改name属性则不会影响到其他的实例，这是因为数组为引用类型，而name为基本类型。


其实,javaScript中比较灵活的还是原型继承，他没有最好的实现方式，在特定场景下有相对比较好的实现。
