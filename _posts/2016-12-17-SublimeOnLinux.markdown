---
layout:     post
title:      "解决Linux下Sublime无法输入中文"
subtitle:   ""
date:       2016-12-17 16:00:00
author:     "蒋为"
header-img: "img/14.jpg"
catalog: true
tags:
    - Linux
---
>转载于网络
<h2>1.首先保证你的电脑有c++编译环境</h2>
<p id="one">如果没有，通过以下命令安装
{% highlight perl %}
sudo apt-get install build-essential
sudo apt-get install libgtk2.0-dev
{% endhighlight %}
</p>
<h2>2.在～目录新建一个名为sublime-imfix.c的文件</h2>
<p id="two">
{% highlight c %}
#include <gtk/gtkimcontext.h>

void gtk_im_context_set_client_window (GtkIMContext *context,

         GdkWindow    *window)

{

 GtkIMContextClass *klass;

 g_return_if_fail (GTK_IS_IM_CONTEXT (context));

 klass = GTK_IM_CONTEXT_GET_CLASS (context);

 if (klass->set_client_window)

   klass->set_client_window (context, window);

 g_object_set_data(G_OBJECT(context),"window",window);

 if(!GDK_IS_WINDOW (window))

   return;

 int width = gdk_window_get_width(window);

 int height = gdk_window_get_height(window);

 if(width != 0 && height !=0)

   gtk_im_context_focus_in(context);

}

{% endhighlight %}
</p>



<h2>3.将上述文件编译成共享库libsublime-imfix.so</h2>
<p id="three">
{% highlight perl %}
gcc -shared -o libsublime-imfix.so sublime-imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC
{% endhighlight %}
</p>

<h2>4.将libsublime-imfix.so拷贝到sublime_text所在文件夹</h2>
<p id='four'>
{% highlight perl %}
sudo mv libsublime-imfix.so /opt/sublime_text/
{% endhighlight %}
</p>

<h2>5.修改文件/usr/bin/subl的内容</h2>
<p id='five'>
{% highlight perl %}
sudo vi /usr/bin/subl
{% endhighlight %}
将
{% highlight perl %}
#!/bin/sh
exec /opt/sublime_text/sublime_text "$@"
{% endhighlight %}
修改为
{% highlight perl %}
#!/bin/sh
LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
{% endhighlight %}
</p>



<h2>6.修改启动器</h2>
<p id="six">
为了使用鼠标右键打开文件时能够使用中文输入，还需要修改文件sublime_text.desktop的内容。命令：

{% highlight perl %}
sudo vi /usr/share/applications/sublime_text.desktop
{% endhighlight %}


将[Desktop Entry]中的字符串



{% highlight perl %}
Exec=/opt/sublime_text/sublime_text %F
{% endhighlight %}

修改为



{% highlight perl %}
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"
{% endhighlight %}

将[Desktop Action Window]中的字符串

{% highlight perl %}
Exec=/opt/sublime_text/sublime_text -n
{% endhighlight %}

修改为

{% highlight perl %}
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"
{% endhighlight %}

将[Desktop Action Document]中的字符串


{% highlight perl %}
Exec=/opt/sublime_text/sublime_text --command new_file
{% endhighlight %}

修改为

{% highlight perl %}
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
{% endhighlight %}

</p>





