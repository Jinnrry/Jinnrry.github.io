---
layout:     post
title:      "React项目基本功能使用"
subtitle:   "React从入门到放弃"
date:       2017-07-28 19:00:00
author:     "蒋为"
header-img: "img/7.jpg"
catalog: true
tags:
    - React
---
>记录

目录结构

<img src="/img/articleImg/React.png">

效果
<img src="/img/articleImg/React2.png">

点击上面3个按钮可以实现无刷新切换，列表内容可以通过ajax加载。

Clock.js文件

{% highlight  javascript  %}

import React from "react";


export default class Clock extends React.Component {
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
{%  endhighlight  %}

Header.js文件

{% highlight  javascript  %}

import React from "react"
import {Link} from "react-router-dom"
export default  class  Header extends React.Component{
    render(){
        return(
            <div>
                <h1>location:</h1>
                <Link className="btn btn-default"  to="/welcome">welcome</Link>
                <Link className="btn btn-default" to="/clock">clock</Link>
                <Link className="btn btn-default" to="/list">list</Link>
            </div>
        );
    }
}

{%  endhighlight  %}

Layout.js文件

{% highlight  javascript  %}

import React from "react"
import {Switch,Route} from "react-router-dom"
import Clock from "./Clock"
import Welcome from  "./Welcome"
import Header from "./Header";
import List from "./List"
export default class Layout extends React.Component{
    render(){
        return (
            <div>
                <Header/>
                <Switch>
                    <Switch>
                        <Route component={Clock} path={"/clock"}/>
                        <Route component={Welcome} path={"/welcome"}/>
                        <Route component={List} path={"/list"}/>
                    </Switch>
                </Switch>
            </div>
        );
    }
}

{%  endhighlight  %}

List.js文件

{% highlight  javascript  %}

import React from "react"


export default class List extends React.Component {

    constructor(props) {
        super(props);
        this.state = {
            array: [1, 2, 3, 4, 5, 6],
            students: [
                {
                    "id": 1,
                    "mobile": "13890199242",
                    "password": "e10adc3949ba59abbe56e057f20f883e",
                    "name": "无名氏",
                    "photoImageId": 2,
                    "genderId": 2,
                    "birthday": "2017-07-20T16:00:00.000+0000",
                    "email": "xjiangwei@xjiangwei.cn",
                    "provinceId": 2,
                    "cityId": 2,
                    "signature": "2",
                    "isLocked": 1,
                    "registerDate": "2017-07-24T10:40:20.000+0000",
                    "lastUpdateDate": "2017-07-24T10:40:23.000+0000"
                },
                {
                    "id": 22,
                    "mobile": "18577332907",
                    "password": "e10adc3949ba59abbe56e057f20f883e",
                    "name": "小明",
                    "photoImageId": 1,
                    "genderId": 1,
                    "birthday": "2017-07-21T00:00:00.000+0000",
                    "email": "xjiangwei@xjiangwei.cn",
                    "provinceId": 1,
                    "cityId": 1,
                    "signature": null,
                    "isLocked": 0,
                    "registerDate": null,
                    "lastUpdateDate": null
                },
                {
                    "id": 61,
                    "mobile": "13890199241",
                    "password": "ff98ea5c27046203fd3bc8ffbf52e0ef",
                    "name": "新用户",
                    "photoImageId": 0,
                    "genderId": 0,
                    "birthday": null,
                    "email": null,
                    "provinceId": 0,
                    "cityId": 0,
                    "signature": null,
                    "isLocked": 0,
                    "registerDate": null,
                    "lastUpdateDate": null
                },
                {
                    "id": 62,
                    "mobile": "13890199243",
                    "password": "ff98ea5c27046203fd3bc8ffbf52e0ef",
                    "name": "新用户",
                    "photoImageId": 0,
                    "genderId": 0,
                    "birthday": null,
                    "email": null,
                    "provinceId": 0,
                    "cityId": 0,
                    "signature": null,
                    "isLocked": 0,
                    "registerDate": null,
                    "lastUpdateDate": null
                },
                {
                    "id": 63,
                    "mobile": "13890199245",
                    "password": "ff98ea5c27046203fd3bc8ffbf52e0ef",
                    "name": "新用户",
                    "photoImageId": 0,
                    "genderId": 0,
                    "birthday": null,
                    "email": null,
                    "provinceId": 0,
                    "cityId": 0,
                    "signature": null,
                    "isLocked": 0,
                    "registerDate": null,
                    "lastUpdateDate": null
                },
                {
                    "id": 64,
                    "mobile": "13890199250",
                    "password": "ff98ea5c27046203fd3bc8ffbf52e0ef",
                    "name": "新用户",
                    "photoImageId": 0,
                    "genderId": 0,
                    "birthday": null,
                    "email": null,
                    "provinceId": 0,
                    "cityId": 0,
                    "signature": null,
                    "isLocked": 0,
                    "registerDate": "2017-07-25T12:15:40.000+0000",
                    "lastUpdateDate": null
                }
            ]
        };
    }
    render() {
        return (
            <div>
                {/*<ul>*/}
                    {/*{this.state.array.map((value) => <li key={value}>{value}</li>)}*/}
                {/*</ul>*/}

                <table key="table" className="table table-bordered" >
                    <tbody>
                    <tr key="title"><td key="title-id">id</td><td  key="title-name">name</td><td  key="title-gender">gender</td></tr>
                    {this.state.students.map((val)=> <tr key={val.id+10}><td key={val.id}>{val.id}</td><td key={val.name}>{val.name}</td><td key={val.genderId}>{val.genderId}</td></tr>)}
                    </tbody>
                </table>
            </div>
        );
    }
}

{%  endhighlight  %}

Welcome.js文件

{% highlight  javascript  %}

import React from "react"
export default class Welcome extends React.Component{
    render(){
        return <h1>Hello World! </h1>;
    }
}

{%  endhighlight  %}


Form.js文件

{% highlight  javascript  %}

/**
 * Created by jiangwei on 2017/7/29.
 */
import React from "react"


export default class Form extends React.Component{

    constructor(){
        super();
        this.state={text:"",pick:"1"};

    }

    Click=()=>{
        console.log(this.state);
    }


    render(){
        return(
          <form>
              <table className="table table-bordered">
                  <tbody>
                  {/*双向绑定输入框中的值*/}
                  <tr><td>输入框：</td><td><input type="text" value={this.state.text} onChange={(e)=>{this.setState({text:e.target.value});}}  className="input-sm" /></td></tr>
                  <tr><td> 单选框：</td><td><select  onChange={(e)=>{this.setState({pick:e.target.value});}} className="input-sm" value={this.state.pick}><option value="1">1</option><option value="2">2</option><option value="3">3</option><option value="4">4</option><option value="5">5</option></select><br/></td></tr>
                  <tr><td>确定键：</td><td><button onClick={this.Click} className="btn btn-default">click!</button></td></tr>
                  </tbody>
              </table>
          </form>
        );
    }

}

{%  endhighlight  %}


index.js文件

{% highlight  javascript  %}

import React from "react";
import ReactDom from "react-dom";
import { Switch,HashRouter,Route  } from "react-router-dom"

import Layout from "./components/Layout"
ReactDom.render(
    <HashRouter>
        <Layout/>
    </HashRouter>
    , document.getElementById("root"));

{%  endhighlight  %}

这个项目本身很简单，没有什么意义，但是包括了React的基本使用方法，所以记录下来，方便以后使用React的时候查询