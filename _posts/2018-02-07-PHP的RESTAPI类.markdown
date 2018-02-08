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

一个超级轻量级的PHP REST API框架的实现，只需要在项目中引入这个类
{% highlight php %}
class RestApi
{
    protected $aPostArg;    //存储POST过来的数据，大致相当于$_POST
    function __construct()
    {
        $this->aPostArg=$_POST;    //获取POST数组

        if ( !isset($this->aPostArg['action']))   //检查action是否存在
        {
            exit($this->buildJson(false,'action参数不存在',[]));
        }
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

    //获取方法里面的参数名
    function getFucntionParameterName($classname,$func)
    {
        $ReflectionMethod = new \ReflectionMethod($classname,$func);
        return $ReflectionMethod->getParameters();
    }

    function run()
    {
        $ParameterArgs=array();                   //所调用函数的参数名称
        $functionName=$this->aPostArg['action'];  //获取action对应的函数名
        $childClassName=get_class($this);         //获取子类的类名
        $re=$this->getFucntionParameterName( $childClassName ,$functionName  );    //获取对应函数的参数名称
        $functionParameter=array();
        foreach ($re as $index=>$ReflectionParameterObject)     //遍历获得函数参数
        {
            $ParameterArgs[$ReflectionParameterObject->name]=$this->aPostArg[$ReflectionParameterObject->name]   ;
            $functionParameter[$index]=$this->aPostArg[$ReflectionParameterObject->name];

        }

       echo $this->$functionName(...$functionParameter);


    }



    /**构建一个json格式数据
     * boolean status     表示请求是否成功
     * string msg    返回说明
     * array  data    返回数据
     */
    protected function buildJson($status,$msg,$data)
    {
        $args=array("status"=>$status,"data"=>$data,"msg"=>$msg);
        return json_encode($args);
    }

}

{% endhighlight %}


使用方法

testapi.php

{% highlight php %}

class testapi extends RestApi{

    //会自动将POST中的name和id字段注入到 $name $id变量中
    //当POST中的action值为  getName的时候会调用这个方法
    function getName($name,$id)    
    {
        return buildJson(true,'success',[]);
    }


}


$api=new testapi();
$apt->run();
{% endhighlight %}

