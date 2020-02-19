---
layout:     post
title:      "js深拷贝记录"
subtitle:   " "
date:       2020-02-19 12:00:00
author:     "Jinnrry"
header-img: "img/22.jpg"
catalog: true
tags:
    - JavaScript
---
js经常需要用到深拷贝，深拷贝实现一般有2种：

1、遍历object，复制到新对象中，比如vue底层的实现：
[https://github.com/vuejs/vuex/blob/dev/src/util.js#L22](https://github.com/vuejs/vuex/blob/dev/src/util.js#L22)

做了下整理，合成一个函数了：

```
/**
 * 深拷贝
 * @param {*} obj 拷贝对象(object or array)
 * @param {*} cache 缓存数组
 */
function deepCopy (obj, cache = []) {
  // typeof [] => 'object'
  // typeof {} => 'object'
  if (obj === null || typeof obj !== 'object') {
    return obj
  }
  // 如果传入的对象与缓存的相等, 则递归结束, 这样防止循环
  /**
   * 类似下面这种
   * var a = {b:1}
   * a.c = a
   * 资料: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cyclic_object_value
   */
  const hit = cache.filter(c => c.original === obj)[0]
  if (hit) {
    return hit.copy
  }

  const copy = Array.isArray(obj) ?  [] :   {}
  // 将copy首先放入cache, 因为我们需要在递归deepCopy的时候引用它
  cache.push({
    original: obj,
    copy
  })
  Object.keys(obj).forEach(key => {
    copy[key] = deepCopy(obj[key], cache)
  })

  return copy
}
```



2、将对象json编码，再json解码

```
JSON.parse(JSON.stringify(Object))
```