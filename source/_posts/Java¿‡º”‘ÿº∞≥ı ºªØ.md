title: Java类加载及初始化
date: 2015-12-04 15:25:56
tags: [ClassLoader]
---
最近觉得java基础很差，便找了基本的java书来看，跟当时自己看Java感觉完全不一样。  

进入正题，关于类加载的问题，有java基础的都知道，首先加载静态域，再进行实例化。如下例：

```
public class Demo01 {
    static{
        System.out.println("加载demo01");
    }
    public static void main(String[] args){
        B a =new B();
    }
}
class B{
    public static int num=100;
    static{
        System.out.println("类加载");
    }
    public B(){
        System.out.println("实例化B");
    }
}
```
运行结果很显然。如果有继承关系，首先加载父类，再对子类加载。<!-- more -->

再来看一道阿里的java面试题，要求写出运行结果。

```
public class Demo02 {
    public static Demo02 test_a=new Demo02("a");
    public static Demo02 test_b=new Demo02("b");
    public static int i=print("i");
    public static int k=99;
    public int j=99;
    {
        System.out.println("Init");
    }
    static{
        System.out.println("static");
    }
    public static int print(String s){
        System.out.println("print()"+i+":"+s+":"+k);
        k++;i++;
        return i;
    }
    public Demo02(String s){
        System.out.println("constructor()"+i+":"+s+":"+k);
        k++;i++;
    }
    public static void main(String[] args){
        Demo02 test_c=new Demo02("c");
    }
}


```
Demo02在加载过程中，首先加载静态域，那静态域的执行过程又是怎样的呢？

`new Demo02("a")`执行了构造块及构造方法，而此时i已加载，但是还未执行，也就是说类在加载的时候，静态域如：test_a、test_b、i、k、j都会加载，但是他们只有默认值(0,false,null)。  

再来看一下Java类的初始化顺序

规则1：在类第一次加载的时候，将会进行静态域的初始化： 
1. 将所有的静态数据域初始化为默认值（0、false 和 null） 
2. 按照在类中定义的顺序依次执行静态初始化语句和静态初始化块 

规则2：调用构造器的具体处理步骤： 
1. 将所有的数据域初始化为默认值（0、false 和 null） 
2. 按照在类中定义的顺序依次执行初始化语句和初始化块 
3. 如果构造器调用的其他的构造器，则转而执行另一构造器 
4. 执行构造器主体


所以这道题的顺序是test_a,test_b,i,static块,test_c。

```
Init
constructor()0:a:0
Init
constructor()1:b:1
print()2:i:2
static
Init
constructor()3:c:99

```

基础还是很重要的，切勿在浅滩建高楼，打牢基础。



