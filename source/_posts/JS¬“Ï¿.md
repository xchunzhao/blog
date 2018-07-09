title: JS乱炖
date: 2016-03-13 22:43:55
---
### call和apply
关于call和apply，是js里用来改变调用函数的调用者。例如：
```
	var a=1;
	function fn(){
		console.log(this.a);
	}
	var obj={
		a:2
	};
	fn();//1
	fn.call(obj);//2
	fn.apply(obj);//2
```
当在浏览器中执行时，fn()调用者为全局对象window,而`fn.call(obj)`跟`fn.apply(obj)`中fn的调用者为obj，从而改变函数内部的this指针。<!-- more -->

call跟apply常见的用法就是当一个对象没有某个函数时，可以用call/apply调用这个函数。又如：
```
	var obj={
		a:1,
		sayA:function(){
			console.log(this.a);
		}	
	};
	obj.sayA();//1
	var obj1={
		a:2
	}
	obj.sayA.call(obj1);
	obj.sayA.apply(obj1);
```
而call跟apply的区别就是传入的参数不同。

`call(context,function,args1,args2...)`
`appply(context,function,[args1,args2...])`
call在传入函数参数是一个一个传，而apply则是一个类数组的形式。所以，当参数不确定时，可以用apply来调用函数。

### callee和caller

`callee`是arguments的一个属性，为指向该函数的一个指针。而js中函数名本身就是指向该函数的指针。因此，当函数中有调用自身的情况下，就可以使用callee了。例如：
```
	//常规写法
	function fn(a){
		return fn(a-1)+a;
	}
	//使用callee
	function fn1(a){
		return arguments.callee(a-1)+a;
	}
```
`callee`使用的好处就是，消除函数内部自身调用跟函数名称的耦合。

而`caller`即调用者，指向调用当前函数的函数的指针。例如：
```
	function fn(){
		console.log(fn.caller);
	}
	function fn1(){
		fn();
	}
	fn1();
```
此时fn1()的结果就是fn1的函数体。


### ES5新增数组方法

1 `forEach(function(item,index,items)`
用于遍历数组，可以简写为forEach(function(item))但是跟for in还是有区别。forEach不支持break。

2 `map(function(item))`
map方法将调用的数组中每个元素传给指定函数，返回一个数组。map返回的是新数组，不会修改原数组的值。

3 `filter(function(item))`
将调用的数组中每个元素传给指定函数，返回当前数组的子集。如:
```
	var a=[1,2,3,4,5];
	console.log(a.filter(function(x){return x%2===0;}));//返回该数组偶数集
```

4 `every/some`
every从字面可以看出，每个都需满足。而some就类似于数学中的存在量词，有一个满足就好。如：
```
	var a=[1,2,3,4,5];
	console.log(a.every(function(x){x<3;}));//false
	console.log(a.some(function(x){x<3;}));//true
```

5 `reduce/reduceRight`
reduce(function,initParam)，第一个参数是执行化简操作的函数，第二个参数是一个传递给函数的初始值。如一个求和的例子：
```
	var a=[1,2,3,4,5];
	var sum=a.reduce(function(x,y){return x+y},0);//15
```
在第一次调用简化函数时，函数参数的第一个值是一个初始值，就是传递给reduce()的第二个参数。在之后的调用中，reduce()的第二个参数就是上次调用简化函数的返回值，并传递给简化函数的第一个参数。

reduceRight用法跟reduce差不多，只是遍历的时候从数组末尾开始遍历。


