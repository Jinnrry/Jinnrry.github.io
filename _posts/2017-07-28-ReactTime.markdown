---
layout:     post
title:      "React开发时钟"
subtitle:   "React从入门到放弃"
date:       2017-07-28 10:00:00
author:     "蒋为"
header-img: "img/17.jpg"
catalog: true
tags:
    - React
---
>记录


通过我上一篇文章配置好开发环境后，在index.js中敲入如下代码

{% highlight  javascript  %}

import React from "react";
import ReactDom from "react-dom";


class Clock extends React.Component {
    constructor() {
        super();
        this.state = {date: new Date()}
    }

    render() {
        return (
            <div>
                <h1>当前时间： </h1>
                <h2>{this.state.date.toLocaleTimeString()}</h2>

            </div>

        );
    }

	//申明一个tick函数，设置时间，注意，这里是使用匿名函数赋值给tick的，这样申明不会造成this错误，但是这样写必须给babel添加transform-class-properties扩展
	//另外也推荐React类中的成员函数都通过这种方式申明
    tick = () => {
        this.setState({date: new Date()});
    }

    //组件显示时调用
    componentDidMount() {
        this.interval = setInterval(this.tick, 1000);  //申明定时器

    }


    //组件消失时调用
    componentWillUnmount() {
        clearInterval(this.interval);
    }

}


ReactDom.render(
    <Clock/>
    , document.getElementById("root"));


{%  endhighlight  %}


说明：

class Clock extends React.Component 这样写表示申明了一个组件，这是class方式申明组件，还有一种方式是function方式申明，但是function方式申明有些功能没法实现。

其中重写了3个方法：

constructor()   //组件的构造函数
 
componentDidMount()    //组件显示时调用

componentWillUnmount()   //组件消失时调用

另外，时间变量是放在组件中的state中的，state是组件的一个变量，这个变量改变的时候组件将会刷新，
注意，只有state变量更新的时候组件才会更新，其他组件变量改变组件都不会更新。另外，注意state是一个对象，而且更新的时候必须使用this.setState()函数更新