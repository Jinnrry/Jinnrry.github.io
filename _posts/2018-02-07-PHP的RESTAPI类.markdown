---
layout:     post
title:      "PHP的RESTAPI类"
subtitle:   "PHP"
date:       2018-02-07 04:40:32
author:     "蒋为"
header-img: "img/11.jpg"
catalog: true
tags:
    - php
---
>记录

{% highlight php %}

<?php
class RestApi{
    private $aPostArg;
    function __construct()
    {
        $this->aPostArg=$_POST;    //获取POST数组

        if ( !isset($this->aPostArg['action']))   //检查action是否存在
        {
            exit( $this->buildJson(false,'action参数不存在',[]) );
        }
    }

    /**
     * 接口函数
     * 当action的值为这个函数名的时候就会执行这个函数并返回
     *
     */
    function dosomething()
    {
        return $this->buildJson(true,'成功',[]);   //返回一个json对象
    }


    /**
     * 当action对应的函数不存在的时候给出错误返回
     * @param $name
     * @param $arguments
     * @return string
     */
    function __call($name, $arguments)
    {
        return $this->buildJson(false,'该action('.$name.')操作不存在',[]);
    }

    function run()
    {
        $functionName=$this->aPostArg['action'];  //获取action对应的函数名
        echo  $this->$functionName();
    }


    /**构建一个json格式数据
     * boolean status     表示请求是否成功
     * string msg    返回说明
     * array  data    返回数据
     */
    function buildJson($status,$msg,$data)
    {
        $args=array("status"=>$status,"data"=>$data,"msg"=>$msg);
        return json_encode($args);
    }

}


if($_SERVER['REQUEST_METHOD']=='POST')
{
    $restapi=new RestApi();
    $restapi->run();
}


{% endhighlight %}


将通过POST中的action值执行对应的函数，然后返回JSON格式的数据
