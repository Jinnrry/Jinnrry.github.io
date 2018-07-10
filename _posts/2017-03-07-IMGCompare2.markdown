---
layout:     post
title:      "python实现图像相识度比较2"
subtitle:   "python"
date:       2017-03-07 11:00:00
author:     "蒋为"
header-img: "img/3.jpg"
catalog: true
tags:
    - Python
---
>记录



{% highlight py %}


'''
本类用于对比图像相似度
只需要调用comple方法
'''
class VectorCompare:
    # 计算矢量大小
    def magnitude(self, concordance):
        total = 0
        for word, count in concordance.items():
            total += count ** 2
        return math.sqrt(total)

    # 计算矢量之间的 cos 值
    def relation(self, concordance1, concordance2):
        relevance = 0
        topvalue = 0
        for word, count in concordance1.items():
            if word in concordance2:
                topvalue += count * concordance2[word]
        return topvalue / (self.magnitude(concordance1) * self.magnitude(concordance2))


    def buildvector(self,im):
        d1 = {}
        count = 0
        for i in im.getdata():
            d1[count] = i
            count += 1
        return d1

#对比两张图像的相似度
    def comple(self,img1,img2):
        img1=img1.convert("P")
        img2=img2.convert("P")
        return self.relation(self.buildvector(img1),self.buildvector(img2))


{% endhighlight %}
