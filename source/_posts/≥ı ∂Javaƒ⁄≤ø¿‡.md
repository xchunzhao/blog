title: 初识Java内部类
date: 2015-09-05 19:50:23
tag: [Inner Class]
categories: Java
---
最近关注了一位大牛的博客，看了挺多，不懂的也挺多。其中一个问题是Java中单例模式的实现。通过控制构造器的访问权限，在内部提供一个实例这种方式还是了解的。大牛最后的优化是使用一个内部类来实现，很是不解，便来了解了一下Java内部类。

那什么是内部类呢？就是在类中嵌套类。


那内部类的作用怎么体现呢？查了挺多资料，观点各不相同，无非分为下面几点：
1 实现操作的隐藏，即封装
2 内部类拥有外层类所有成员的访问权限
3 间接实现多重继承
4 避免修改接口从而实现同类中同名方法的调用 <!-- more -->

#### 如何实现隐藏操作
看下面这段代码
```
public static void main(String[] args){
    InnerClassTest1 test1=new InnerClassTest1();
    InterfaceTest interfaceTest=test1.getWords();
    interfaceTest.sayHello();
}
```

单看这段代码(InterfaceTest是个接口)，可以看出来有个类叫InnerClassTest1，提供一个getWords()方法返回一个InterfaceTest实例，然后InterfaceTest中有个方法叫sayHello()
那么就猜测是不是有个类实现了InterfaceTest的方法呢？

那一般非内部类的访问修饰符是public，我们很容易去了解他的实现。内部类的作用来了，代码如下：

```
public class InnerClassTest1 {
    private class InnnerClass implements InterfaceTest{
        public void sayHello(){
            System.out.println("hello world");
        }
    }
    public InterfaceTest getWords(){
        return new InnnerClass();
    }
    public static void main(String[] args){
        InnerClassTest1 test1=new InnerClassTest1();
        InterfaceTest interfaceTest=test1.getWords();
        interfaceTest.sayHello();
    }
}
```
内部类的访问修饰符可以是private\protect，这时我们就很难去了解sayHello()的具体实现了，从而实现很好的封装。

#### 如何实现多重继承
初学Java的都知道Java中是单继承多实现，开始看的时候，这个多继承就改变了我的三观了。不多说上代码：

父类Father.java

```
public class Father {
    private int strong=10;
    public int getStrong(){
        return this.strong;
    }
}
```
母类Mother.java

```
public class Mother {
    private int kind=8;
    public int getKind(){
        return this.kind;
    }
}
```
实现类Son.java

```
public class Son {
    private class kindNum extends Mother{
        @Override
        public int getKind() {
            return super.getKind();
        }
    }
    private class strongNum extends Father{
        @Override
        public int getStrong() {
            return super.getStrong();
        }
    }
    public int getKind(){
        return new kindNum().getKind();
    }
    public int getStrong(){
        return new strongNum().getStrong();
    }
    public static void main(String[] args){
        Son son=new Son();
        System.out.println(son.getKind());
        System.out.println();
    }
}
```
Mother有个善良指数，Father有个强壮指数，单继承情况下Son只能继承一个指数，使用内部类之后，Son就可以具有kind、strong两个指数，从而间接实现了多重继承。

#### 如何避免修改接口实现同类中同名方法的调用

看上去比较拗口，其实就是一个类继承一个父类，同时要实现一个接口，但是接口跟父类中有同名的方法，如何区分调用？

父类
```
public class InnerClassFather {
    public void sayHello(){
        System.out.println("Hello");
    }
}
```
接口

```
public interface InterfaceTest {
    public void sayHello();
}
```

```
public class InnerClass extends InnerClassFather {
    @Override
    public void sayHello() {
        super.sayHello();
    }
    private class ClassIn implements InterfaceTest{

        public void sayHello() {
            sayHello();
        }
    }
}
```
其实就是简单的方法调用，但是内部类实现了强大的功能，外部的方法重写了父类的方法，而内部类实现了接口中的方法，从而有效区分了同名方法的调用。


### 总结
内部类的使用在程序设计方面用的是比较多的，内部类的主要功能就是实现多重继承，有兴趣的不妨多看看相关项目的源码，深刻认识。