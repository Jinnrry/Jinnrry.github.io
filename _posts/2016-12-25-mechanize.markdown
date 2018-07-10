---
layout:     post
title:      "mechanize的简单使用"
subtitle:	“mechanize”
date:       2016-12-25 12:00:00
author:     "蒋为"
header-img: "img/6.jpg"
catalog: true
tags:
    - Python
---
>转载于http://blog.csdn.net/sunmc1204953974





## 安装方法：


用easy_install安装Mechanize，即：

sudo easy_install Mechanize

安装好之后就可以愉快的使用了，首先是

## 模拟一个浏览器的代码：

{% highlight python %}


import mechanize
import cookielib

# Browser
br = mechanize.Browser()

# Cookie Jar
cj = cookielib.LWPCookieJar()
br.set_cookiejar(cj)

# Browser options
br.set_handle_equiv(True)
br.set_handle_gzip(True)
br.set_handle_redirect(True)
br.set_handle_referer(True)
br.set_handle_robots(False)

# Follows refresh 0 but not hangs on refresh > 0

br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

# Want debugging messages?

#br.set_debug_http(True)
#br.set_debug_redirects(True)
#br.set_debug_responses(True)
# User-Agent (this is cheating, ok?)

br.addheaders = [('User-agent', 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1')]



{% endhighlight %}

这样就得到了一个浏览器的实例，br对象。使用这个对象，便可以对

## 网页操作:





{% highlight python %}

# Open some site, let's pick a random one, the first that pops in mind:

r = br.open('http://www.baidu.com')
html = r.read()

# Show the source

print html

# or

print br.response().read()

# Show the html title

print br.title()

# Show the response headers

print r.info()

# or

print br.response().info()

# Show the available forms

for f in br.forms():
    print f

# Select the first (index zero) form

br.select_form(nr=0)

# Let's search

br.form['q']='weekend codes'
br.submit()
print br.response().read()

# Looking at some results in link format

for l in br.links(url_regex='stockrt'):
    print l
	
	
	
{% endhighlight %}

另外如果访问的网站需要验证(http basic auth),那么:


## 网站验证：


{% highlight python %}

# If the protected site didn't receive the authentication data you would
# end up with a 410 error in your face

br.add_password('http://safe-site.domain', 'username', 'password')
br.open('http://safe-site.domain')

{% endhighlight %}

另外利用这个方法，存储和重发这个session cookie已经被Cookie Jar搞定了，并且可以管理浏览器历史:。除此之外还有众多应用，如


## 下载：



{% highlight python %}

# Download

f = br.retrieve('http://www.google.com.br/intl/pt-BR_br/images/logo.gif')[0]
print f
fh = open(f)


{% endhighlight %}


## 为http设置代理 :



{% highlight python %}

# Proxy and user/password
br.set_proxies({"http": "joe:password@myproxy.example.com:3128"})
# Proxy
br.set_proxies({"http": "myproxy.example.com:3128"})
# Proxy password
br.add_proxy_password("joe", "password")



{% endhighlight %}

## 回退（Back）:

打印url即可验证是否回退
{% highlight python %}
    # Back
    br.back()
    print br.geturl()
{% endhighlight %}


## 模拟谷歌和百度查询:

即打印和选择forms，然后填写相应键值，通过post提交完成操作

{% highlight python %}

    for f in br.forms():
        print f

    br.select_form(nr=0)

#谷歌查询football
    br.form['q'] = 'football'
    br.submit()
    print br.response().read()

#百度查询football
    br.form['wd'] = 'football'
    br.submit()
    print br.response().read()
{% endhighlight %}



相应键值名，可以通过打印查出
更多的信息大家可以去官网查看





{% highlight python %}

#!/usr/bin/env python

import mechanize
import cookielib

from time import ctime,sleep

def run():
    print 'start!'
    for i in range(100):
        browse()
        print "run",i,"times ",ctime()
        sleep(1)

def browse():

    br = mechanize.Browser()

    cj = cookielib.LWPCookieJar()
    br.set_cookiejar(cj)

    br.set_handle_equiv(True)
    br.set_handle_gzip(True)
    br.set_handle_redirect(True)
    br.set_handle_referer(True)
    br.set_handle_robots(False)

    br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

    br.addheaders = [('User-agent', 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1')]


    r = br.open('http://www.baidu.com')

    html = r.read()

    #print html

run()

print "!!!!!!!!!!!!!!!!!!all over!!!!!!!!!!!!!!!!!! \n %s" %ctime()

{% endhighlight %}

我还是学生，写的不好的地方还请多多指正，
