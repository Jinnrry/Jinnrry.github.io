---
layout:     post
title:      "PHP快速排序实现"
subtitle:   "快速排序"
date:       2018-07-11 12:00:00
author:     "蒋为"
header-img: "img/17.jpg"
catalog: true
tags:
    - 算法
---
>记录

```
<?php
/**
 * Created by PhpStorm.
 * User: jiangwei
 * Date: 2018/7/11
 * Time: 上午10:54
 */


//快速排序法封装函数
function quick_Sort($array){
	//先判断是否需要继续进行，若所要排序数组只有一个元素或没有元素则不需要排序
	$len = count($array);
	if($len <= 1)
	{
		return $array;
	}
	//如果所给数组元素大于1个，需要排序
	//选择数组第一个元素作为标尺
	$key = $array[0];
	//初始化两个数组
	$left_array = array();//小于标尺的
	$right_array = array();//大于标尺的

	//遍历所给数组除了标尺外的所有元素，按照大小关系放入两个数组内
	for($i=1;$i<$len;$i++){
		if($array[$i]<$key){
			//如果数组元素小于标尺则将该元素放入左数组
			$left_array[] = $array[$i];
		}else{
			//如果数组元素大于标尺则将该元素放入右数组
			$right_array[] = $array[$i];
		}
	}
	//再分别对 左数组 和 右数组进行相同的排序处理方式
	//递归调用这个函数,并记录结果
	$left_array = quick_Sort($left_array);
	$right_array = quick_Sort($right_array);
	//合并左数组 标尺 右数组
	//array_merge() 函数把两个或多个数组合并为一个数组。
	//如果键名有重复，后面的键名的值覆盖前面的键名的值。如果数组是数字索引的，则键名会以连续方式重新索引。
	//语法   array_merge(array1,array2,array3...)
	return array_merge($left_array,array($key),$right_array);
}



$sortarray = array(13,89,23,9,19,88,56,78,34,69,10,14,90,89);
print_r(quick_Sort($sortarray));
```