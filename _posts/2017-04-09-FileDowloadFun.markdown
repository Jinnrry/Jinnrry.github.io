---
layout:     post
title:      "php文件下载函数"
subtitle:   "php"
date:       2017-04-09 04:15:33
author:     "蒋为"
header-img: "img/5.jpg"
catalog: true
tags:
    - php
---
>记录

{% highlight php %}

<?php 
function downfile()
{
 $filename=realpath("resume.html"); //文件名
 $date=date("Ymd-H:i:m");
 Header( "Content-type:  application/octet-stream "); 
 Header( "Accept-Ranges:  bytes "); 
Header( "Accept-Length: " .filesize($filename));
 header( "Content-Disposition:  attachment;  filename= {$date}.doc"); 
 echo file_get_contents($filename);
 readfile($filename); 
}
downfile();
?>


{% endhighlight %}



这样在php代码中能够实现推送文件给客户端
