title: 巧用Hash
date: 2016-09-25 11:13:15
tags: [Hash]
---
> ### 何为Hash

Hash,又称散列，查找数据中存在与关键词K相等的记录，那么通过特定的散列函数f(k),可以计算出关键词所在位置。

常见的散列例如jdk中的hashMap,具体实现可以翻阅hashMap源码，本文重点是如何自己构建简单的Hash函数。<!-- more -->

> ### 如何构建Hash

有这样一道题；
```
给定两个分别由字母组成的字符串A和字符串B，
字符串B的长度比字符串A短。
请问，如何最快地判断字符串B中所有字母是否都在字符串A里？

```

如果不考虑效率，我们只需要两层遍历，如果B的某个字符不在A中，则返回false。时间复杂度O(m*n)

或者将两个字符串先排序，再分别遍历。时间复杂度跟我们使用的排序算法有关，假设是快排，那么这种思路的时间复杂度为：O(mlogm)+O(nlogn)+O(m+n)

进一步思考，能不能提高时间复杂度？

这道题的目的，就是在A中快速找出B的某个字符。可以不遍历A就能直接算出是否包含这个字符呢？

我们可以构建一个256长度的hash表，将A中的字符放入hash表中，key是某字符通过散列函数映射之后的值，value是判断有无这样的字符。

例如A中字符“saTHL”,可以想到用字符的ASCII作为该字符映射之后的值。

那么在映射之后hash表中，第115位为1，第98位为1，依次。。。其余都是0。这样该Hash表就构建完成。

接下来就可以遍历B，直接利用B中该字符的ASCII码作为hash表的下标去查找，如果为1则存在该字符。

这样的思路，时间复杂度明显降低，为O(m)+O(n),但是空间复杂度为O(256)。其实，如果能确定AB中都是字母，那么该hash表的大小就可以缩小。

[示例代码](https://github.com/xchunzhao/LeetCode/blob/master/src/algorithm/TAOPP/Chapter1/StringContains/StringContains02.java)

有了这样的构建经历，那么lintcode上[变位词](http://www.lintcode.com/zh-cn/problem/two-strings-are-anagrams/)这道题也能迎刃而解。


> ### 更优秀的做法

接伤到判断字符串包含的那道题。有没有办法将空间复杂度降低。看到[july](https://github.com/julycoding)大神的一个做法，通过位运算，来构建hash表。
直接上代码：

```
 private static boolean stringContains02_03(char[] strA, char[] strB){
        int lenA = strA.length;
        int lenB = strB.length;
        int hash = 0;
        for (int i = 0; i < lenA; ++i)
        {
            //使用位或操作
            hash |= (1 << (strA[i] - 'A'));
        }
        for (int i = 0; i < lenB; ++i)
        {
            //判断B中的某个字符不在A中，运用&操作
            if ((hash & (1 << (strB[i] - 'A'))) == 0)
            {
                return false;
            }
        }
        return true;
    }
```
可以先看到这行代码`hash |= (1 << (strA[i] - 'A'))`,假设都是字母，右边意思是,hash中从左开始数第strA[i]-'A'位为1，说明存在该字符strA[i]。

而位或运算`|`则是保证如果A中有重复字符，hash中不做特殊处理。

那判断是否存在某个字符的时候，使用`&`操作，如果存在某个字符，那么跟hash相与的话，是1。这样就可以保证快速找到是否存在某个字符。

该做法时间复杂度还是O(m)+O(n),空间复杂度为O(1)。


> ### 最后

算法、数据结构、操作系统、计网等CS基础课程才是核心！！
