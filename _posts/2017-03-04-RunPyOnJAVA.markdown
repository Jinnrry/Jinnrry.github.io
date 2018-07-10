---
layout:     post
title:      "在java中调用python"
subtitle:   "命令行脚本"
date:       2017-03-04 12:00:00
author:     "蒋为"
header-img: "img/1.jpg"
catalog: true
tags:
    - Java
---
>记录


## jython

Jython是一种完整的语言，而不是一个Java翻译器或仅仅是一个Python编译器，它是一个Python语言在Java中的完全实现。Jython也有很多从CPython中继承的模块库。最有趣的事情是Jython不像CPython或其他任何高级语言，它提供了对其实现语言的一切存取。所以Jython不仅给你提供了Python的库，同时也提供了所有的Java类。这使其有一个巨大的资源库。

用他其中的一个库来实现java中调用python

[下载](https://github.com/jiangwei1995910/jiangwei1995910.github.io/raw/master/files/jython-standalone-2.7.0.jar)

## 直接执行python代码


{% highlight java %}

import org.python.util.PythonInterpreter;
  
import java.io.*;
import static java.lang.System.*;
public class FirstJavaScript
{
 public static void main(String args[])
 {
    
  PythonInterpreter interpreter = new PythonInterpreter();
  interpreter.exec("days=('mod','Tue','Wed','Thu','Fri','Sat','Sun'); ");
  interpreter.exec("print days[1];");
    
    
 }
}


   
{% endhighlight %}

## 调用python函数

先建一个python文件 function.py
{% highlight py %}
def add(a, b):
    return a + b
{% endhighlight %}




java调用部分：


{% highlight java %}
import org.python.core.PyFunction;
import org.python.core.PyInteger;
import org.python.core.PyObject;
import org.python.util.PythonInterpreter;
public class FirstJavaScript
{
	/*
	 * 将python方法封装成java方法
	 * */
	public static int add(int x,int y)
	{
		

		 PythonInterpreter interpreter = new PythonInterpreter();
	     interpreter.execfile("/home/jiangwei/桌面/function.py");  //这里是你python文件位置
	     PyFunction func = (PyFunction)interpreter.get("add",PyFunction.class);
	     PyObject pyobj = func.__call__(new PyInteger(x), new PyInteger(y));
	     return Integer.valueOf(pyobj.toString());

	}
	
	
    public static void main(String args[])
    {
          
       System.out.println(add(11,12));
  
  
    }
}

{% endhighlight %}


## 直接运行python文件

先建立一个python文件test.py

{% highlight py %}


print 'hello world!'

{% endhighlight %}




调用：



{% highlight java %}



import org.python.util.PythonInterpreter;
  
public class FirstJavaScript
{
 public static void main(String args[])
 {
    
  PythonInterpreter interpreter = new PythonInterpreter();
  interpreter.execfile("/home/jiangwei/桌面/test.py");
 }
}

{% endhighlight %}








