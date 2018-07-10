---
layout:     post
title:      "MYSQL登录函数"
subtitle:   "MYSQL技巧"
date:       2016-12-23 11:00:00
author:     "蒋为"
header-img: "img/20.jpg"
catalog: true
tags:
    - MySQL
---
>记录


## MYSQL一个简单的登录函数
密码使用md5加密：<br>
{% highlight mysql %}
create function login (username char(10),pw char(10)) 
    	returns int 
    	BEGIN 
	declare tpw char(32); 
	select password   #用户表密码字段 
	into tpw from users   #用户表 
	u where username=user;   #username为用户表用户名字段 
	IF tpw=MD5(pw) THEN 
	return 1; 
	ELSE 
	RETURN 0; 
END IF; 
END
{% endhighlight %}