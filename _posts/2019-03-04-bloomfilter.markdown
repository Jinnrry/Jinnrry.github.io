---
layout:     post
title:      "大数据去重之布隆过滤器"
subtitle:   " 布隆过滤器"
date:       2019-03-04 12:00:00
author:     "木木的木头"
header-img: "img/11.jpg"
catalog: true
tags:
    - 数据结构
---
## 什么是布隆过滤器
本质上布隆过滤器是一种数据结构，比较巧妙的概率型数据结构（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 “某样东西一定不存在或者可能存在”。

相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

## 实现原理
HashMap 的问题

讲述布隆过滤器的原理之前，我们先思考一下，通常你判断某个元素是否存在用的是什么？应该蛮多人回答 HashMap 吧，确实可以将值映射到 HashMap 的 Key，然后可以在 O(1) 的时间复杂度内返回结果，效率奇高。但是 HashMap 的实现也有缺点，例如存储容量占比高，考虑到负载因子的存在，通常空间是不能被用满的，而一旦你的值很多例如上亿的时候，那 HashMap 占据的内存大小就变得很可观了。

还比如说你的数据集存储在远程服务器上，本地服务接受输入，而数据集非常大不可能一次性读进内存构建 HashMap 的时候，也会存在问题。

### 布隆过滤器数据结构
布隆过滤器是一个 bit 向量或者说 bit 数组，长这样：


![](http://www.xjiangwei.cn/img/articleImg/bloomfilter1.webp)   


如果我们要映射一个值到布隆过滤器中，我们需要使用多个不同的哈希函数生成多个哈希值，并对每个生成的哈希值指向的 bit 位置 1，例如针对值 “baidu” 和三个不同的哈希函数分别生成了哈希值 1、4、7，则上图转变为：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter2.webp)   

Ok，我们现在再存一个值 “tencent”，如果哈希函数返回 3、4、8 的话，图继续变为：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter3.webp)   

值得注意的是，4 这个 bit 位由于两个值的哈希函数都返回了这个 bit 位，因此它被覆盖了。现在我们如果想查询 “dianping” 这个值是否存在，哈希函数返回了 1、5、8三个值，结果我们发现 5 这个 bit 位上的值为 0，说明没有任何一个值映射到这个 bit 位上，因此我们可以很确定地说 “dianping” 这个值不存在。而当我们需要查询 “baidu” 这个值是否存在的话，那么哈希函数必然会返回 1、4、7，然后我们检查发现这三个 bit 位上的值均为 1，那么我们可以说 “baidu” 存在了么？答案是不可以，只能是 “baidu” 这个值可能存在。

这是为什么呢？答案跟简单，因为随着增加的值越来越多，被置为 1 的 bit 位也会越来越多，这样某个值 “taobao” 即使没有被存储过，但是万一哈希函数返回的三个 bit 位都被其他值置位了 1 ，那么程序还是会判断 “taobao” 这个值存在。

### 支持删除么
目前我们知道布隆过滤器可以支持 add 和 isExist 操作，那么 delete 操作可以么，答案是不可以，例如上图中的 bit 位 4 被两个值共同覆盖的话，一旦你删除其中一个值例如 “tencent” 而将其置位 0，那么下次判断另一个值例如 “baidu” 是否存在的话，会直接返回 false，而实际上你并没有删除它。



## 概率推导

误判率
误判率就是在插入n个元素后，某元素被判断为“可能在集合里”，但实际不在集合里的概率，此时这个元素哈希之后的k个比特位置都被置为1。

假设哈希函数等概率地选择每个数组位置，即哈希后的值符合均匀分布，那么每个元素等概率地哈希到位数组的m个比特位上，与其他元素被哈希到哪些位置无关(独立事件)。设定数组总共有m个比特位，有k个哈希函数。在插入一个元素时，一个特定比特没有被某个哈希函数置为1的概率是：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter4.jpg)   

插入一个元素后，这个比特没有被任意哈希函数置为1的概率是：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter5.jpg)   

在插入了n个元素后，这个特定比特仍然为0的概率是：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter6.jpg)   

所以这个比特被置为1的概率是：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter7.jpg)   

现在检测一个不在集合里的元素。经过哈希之后的这k个数组位置任意一个位置都是1的概率如上。这k个位置都为1的概率是：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter8.jpg)   

哈希函数个数的最优解
对于给定的m和n，让“误报率”最小的k值为：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter9.jpg)   

此时“误报率”为：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter10.jpg)   

可以简化为：

![](http://www.xjiangwei.cn/img/articleImg/bloomfilter11.jpg)   

