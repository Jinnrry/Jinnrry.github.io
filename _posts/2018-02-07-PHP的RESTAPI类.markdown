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
<?php
class Ko_App_RestApi
{
    protected $aPostArg=null;    //存储提交过来的数据
    function __construct($method)
    {
        if($_SERVER['REQUEST_METHOD'] == $method )
        {
            switch ($method) {
                case 'POST':
                    $this->aPostArg = $_POST;
                    if ( !isset($this->aPostArg['action'])  || $this->aPostArg['action']==''   )   //检查action是否存在
                    {
                        exit($this->buildJson(false,'action参数不存在',[]));
                    }
                    break;
                case 'GET':
                    $this->aPostArg = $_GET;
                    if ( !isset($this->aPostArg['action'])  || $this->aPostArg['action']==''   )   //检查action是否存在
                    {
                        exit($this->buildJson(false,'action参数不存在',[]));
                    }
                    break;
                default:
                    break;
            }
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
        if ( !isset($this->aPostArg['action']))   //检查action是否存在
        {
            return;
        }
        $functionName=$this->aPostArg['action'];  //获取action对应的函数名
        $childClassName=get_class($this);         //获取子类的类名
        $re=$this->getFucntionParameterName( $childClassName ,$functionName  );    //获取对应函数的参数名称
        $functionParameter=array();
        foreach ($re as $index=>$ReflectionParameterObject)     //遍历获得函数参数
        {
            if(isset($this->aPostArg[$ReflectionParameterObject->name]))
                $functionParameter[$index]=$this->aPostArg[$ReflectionParameterObject->name];
            else
                $functionParameter[$index]=null;
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

<?php

require_once "RestApi.php";

class postapitest extends Ko_App_RestApi{

    //会自动将POST中的name和id和news 字段注入到 $name $id  $news变量中
    //当POST中的action值为  ttc的时候会调用这个方法
    function  ttc($name,$id,$news)
    {
        return $this->buildJson(true,'POST接口',["name"=>$name,"id"=>$id,"n"=>$news]);
    }


    //会自动将POST中的name和id字段注入到 $name $id 变量中
    //当POST中的action值为  tt的时候会调用这个方法
    function  tt($name,$id)
    {
        return $this->buildJson(true,'POST接口',["name"=>$name,"id"=>$id]);
    }

}

class getapitest extends Ko_App_RestApi{
    //会自动将GET中的name和id和news 字段注入到 $name $id  $news变量中
    //当GET中的action值为  ttc的时候会调用这个方法
    function  ttc($name,$id,$news)
    {
        return $this->buildJson(true,'GET接口',["name"=>$name,"id"=>$id,"n"=>$news]);
    }

    
    //同上
    function  tt($name,$id)
    {
        return $this->buildJson(true,'GET接口',["name"=>$name,"id"=>$id]);
    }

}

$api=new postapitest('POST');
$api->run();

$getapi=new getapitest('GET');
$getapi->run();




{% endhighlight %}

此时对应的接口地址为 http://xxxxx/testapi.php
