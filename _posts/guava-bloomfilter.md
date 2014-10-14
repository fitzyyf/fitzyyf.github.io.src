title: "Google Guava的布隆过滤器"
date: 2013-01-29 10:04
categories: 
- 编程语言
tags: 
- bloomfilter
- google-guava
---

为啥要写这个：

1. 看到了老外写的[Google Guava BloomFilter](http://codingjunkie.net/guava-bloomfilter/)
2. 第一次接触布隆过滤器是在2010年的时候，当时是为了解决在网络抓取时，对于已经抓取过的URL，如果进行判断已经抓取的问题，在项目中使用过布隆过滤器解决了这个问题，一直都想总结下。
3.  在新版的[Google Guava](http://code.google.com/p/guava-libraries/)中，Google为我们提供了一个布隆过滤器的实现；
4.  一直在使用 [Google Guava](http://code.google.com/p/guava-libraries/)，但是其他同事不是很了解。

## 布隆过滤器介绍
布隆过滤器是巴顿.布隆于1970年提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。

布隆过滤器是一种独特的数据结构，用以表明元素是否被保存在一个集合(`Set`)中。它最大的区别在于，布隆过滤器能够明确的指出元素*绝对不存在* 于一个集合中，或者可能存在于一个集合中。

我们通过以下的例子来说明下：

假定我们存储一亿个电子邮件地址，我们先建立一个十六亿二进制（比特），即两亿字节的向量，然后将这十六亿个二进制全部设置为0.对于每一个电子邮件地址X，我们用八个不同的随机数产生器(F1,F2,…,F8)产生八个信息指纹（f1,f2,…,f8）。在用一个随机数产生器G吧这八个信息指纹映射到1到十六亿中的八个自然数g1,g2,...,g8。现在我们把这八个位置的二进制全部设置为一。当我们对这一亿个email地址都进行这样的处理后。一个针对这些email地址的布隆过滤器就建成了。（见下图）

{% img /images/blog/bloomfilter.jpg %}


现在,让我们看看如何用布隆过滤器来检测一个可疑的电子 邮件地址 Y 是否在黑名单中。我们用相同的八个随机数产生器 (F1, F2, ..., F8)对这个地址产生八个信息指纹 s1,s2,...,s8,然后将这八个指纹对应到布隆过滤器的八个二进 制位,分别是 t1,t2,...,t8。如果 Y 在黑名单中,显然, t1,t2,..,t8 对应的八个二进制一定是一。这样在遇到任何在黑 名单中的电子邮件地址,我们都能准确地发现。

布隆过滤器决不会漏掉任何一个在黑名单中的可疑地址。但是,它有一条不足之处。也就是它有极小的可能将一个不在黑名单中的电子邮件地址判定为在黑名单中,因为有可能某个好的邮 件地址正巧对应个八个都被设置成一的二进制位。好在这种可能性很小。我们把它称为误识概率。在上面的例子中,误识概率在万分之一以下。
布隆过滤器的好处在于快速,省空间。但是有一定的误识别率。常见的补救办法是在建立一个小的白名单,存储那些可能别误判的邮件地址。

从维基百科的介绍来看，Goolge在BigTable中就使用了BloomFilter，以避免在硬盘中寻找不存在的条目。

另外还有个应用场景：[使用布隆过滤器优化SQL查询](http://asemanfar.com/Using-a-bloom-filter-to-optimize-a-SQL-query)
	
## 使用Guava的布隆过滤器

Guava的布隆过滤器通过调用BloomFilter类的静态函数创建，传递一个[Funnel](http://docs.guava-libraries.googlecode.com/git-history/v11.0.2/javadoc/com/google/common/hash/Funnel.html)对象以及一个*代表预期插入数量的整数*。`Funnel`对象的作用，将数据发送给一个接收器（[Slink](http://docs.guava-libraries.googlecode.com/git-history/v11.0.2/javadoc/com/google/common/hash/Sink.html)）。

具体例子请点击 @see code 查看代码

1. Guava的默认实现

    {% gist 4640364 %}

2. 创建一个带有自定义Funnel实现的布隆过滤器

    在这里我们模拟一种对于邮箱规则进行处理的自定义Funnel的实现。
    
    {% gist 4640358 %} 


**最后，我们需要注意：**
> 正确估计预期插入数量是很关键的一个参数。当插入的数量接近或高于预期值的时候，布隆过滤器将会填满，这样的话，它会产生很多无用的误报点。

> 这里有另一个版本的 BloomFilter.create 方法，它额外接收一个参数，一个代表假命中概率水平的双精度数字（必须大于零且小于1）

> 假命中概率等级影响哈希表储存或搜索元素的数量。百分比越低，哈希表的性能越好。

## 其他参考
1. 《数学之美》中对布隆过滤器的介绍
2. [Guava 布隆过滤器的单元测试类](https://github.com/bbejeck/guava-blog/blob/master/src/test/java/bbejeck/guava/hash/BloomFilterTest.java)
3. [维基百科的介绍](http://en.wikipedia.org/wiki/Bloom_filter)

