---
layout:     post
title:      "java文件流编码"
subtitle:   "JAVA"
date:       2017-04-09 04:40:32
author:     "蒋为"
header-img: "img/11.jpg"
catalog: true
tags:
    - java
---
>记录

{% highlight java %}

BufferedReader br = new BufferedReader(new FileReader(fileName));  
String line = null;  
while ((line = br.readLine()) != null) {   
      System.out.println(line);
}  
br.close();  

{% endhighlight %}

原因：Java读取数据流的时候，如果没有指定数据流的编码方式，则将使用本地环境中的默认字符集。

So！

{% highlight java %}

BufferedReader br=new BufferedReader(new InputStreamReader(new FileInputStream(fileName),"UTF-8"));  
String line = null;  
while ((line = br.readLine()) != null) {  
      System.out.println(line);  //乱码消失
}  
br.close();  


{% endhighlight %}
