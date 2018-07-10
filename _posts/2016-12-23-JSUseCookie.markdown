---
layout:     post
title:      "JavaScript操作cookie"
subtitle:   "Javascript"
date:       2016-12-23 13:00:00
author:     "蒋为"
header-img: "img/23.jpg"
catalog: true
tags:
    - JavaScript
---
>记录



## 设置cookie函数

c_name:cookie名 <br>
value:cookie值 <br>
expiredays:cookie过期时间 <br>


{% highlight javascript %}



function setCookie(c_name, value, expiredays)
{
 　　var exdate=new Date();
 　　exdate.setDate(exdate.getMinutes() + expiredays);
 　　document.cookie=c_name+ "=" + escape(value) + ((expiredays==null) ? "" : ";expires="+exdate.toGMTString());
}
 
 
 


{% endhighlight %}




## 读取cookie函数

c_name:cookie名 <br>
返回值：所取cookie的值 <br>


{% highlight javascript %}


 
 function getCookie(c_name){
　　　　if (document.cookie.length>0)
		{　　
　　　　　　c_start=document.cookie.indexOf(c_name + "=")　　
　　　　　　if (c_start!=-1)
			{ 
　　　　　　　　c_start=c_start + c_name.length+1　　
　　　　　　　　c_end=document.cookie.indexOf(";",c_start)　
　　　　　　　　if (c_end==-1) c_end=document.cookie.length　　
　　　　　　　　return unescape(document.cookie.substring(c_start,c_end))　　
			} 
　　　　}
　　　　return 0
　　}
  
  
  
  
{% endhighlight %}  