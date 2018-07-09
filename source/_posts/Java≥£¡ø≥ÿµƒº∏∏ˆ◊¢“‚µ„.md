title: Java常量池的几个注意点
date: 2015-09-13 18:08:29
comments: true
tag: [CachePool]
categories: Java
---
最近搞离职跟租房的事情比较烦躁，博客也没更。之前面试的时候，面到一个关于装箱拆箱的问题不是特别清楚，遂敲了几行代码看看结果。代码如下：

```
 public static void main(String[] args){
        int var1=10;
        Integer var2=10;
        Integer var3=new Integer(10); 
        System.out.println(var1==var2);  //True
        System.out.println(var1==var3);  //True
        System.out.println(var2==var3);  //False
    }
```
可以猜测一下，对装拆箱了解的都知道，在操作符`==`时，会进行自动的拆箱，即Integer拆箱成int基础类型。到这是不是觉得没有看下去的必要，接下来有更好玩的。<!-- more -->

看这段代码：

```
public static void main(String[] args){
        Integer var1=128;
        Integer var2=128;
        Integer var3=new Integer(128);
        Integer var4=1;
        Integer var5=1;
        Integer var6=new Integer(1);

        System.out.println(var1==var2);
        System.out.println(var1==var3);
        System.out.println(var5==var4);
        System.out.println(var6==var4);
    }
```
猜猜结果，如果没有了解过常量池的可能看到结果会很吃惊。

```
		System.out.println(var1==var2);		//False
        System.out.println(var1==var3);		//False
        System.out.println(var5==var4);		//True
        System.out.println(var6==var4);		//Flase
```
按照装拆箱的理解，在`==`的时候会进行装箱，将基础类型的int包装成Integer，具体怎么包装的，不妨来看看Integer.valueOf()方法。

```
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }


 public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
注意看IntegerCache这个内部类的static块，说明他的cache[]只加载一次，里面内容就是【-128~127】(low=-128,high=127),这时再来看valueOf()这个方法，不难看出如果要包装的int值在cache[]数组里，就直接从数组取，否则重新new。

这下不难理解为什么只有`var4==var5`是True了。

现在才扯到这个传说中的常量池，var1跟var2指向常量池中的同一块内存，而var4跟var5则是指向堆中不同的内存。

这样可以总结出：Java中的常量池技术，是为了方便快捷地创建某些对象而出现的，当需要一个对象时，就可以从池中取一个出来（如果池中没有则创建一个），则在需要重复重复创建相等变量时节省了很多时间。常量池其实也就是一个内存空间，不同于使用new关键字创建的对象所在的堆空间。

除了Integer有常量池,Java提供的八种基础类型中，除Float跟Double外，其他都实现了常量池技术。  

