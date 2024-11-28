title: js之防抖与节流
author: Billy Yin
tags: []
categories: []
date: 2018-09-25 10:17:00
---
防抖和节流是: 优化高频率执行代码的手段
目的: 节约浏览器(服务器)性能
方式: 减少函数执行次数

### 防抖
上电梯的例子: 电梯总是在进出人(监听事件)之后的5秒关门(执行方法), 如果5秒钟之内又有人进出, 则重新开始计时, 直到最后一个进出电梯的事件发生5秒钟之后执行关门.

思路:  设置定时器延迟执行事件 , 一旦在延迟的过程中触发了新的监听事件 , 则销毁并重新设置定时器 .
```
function Debounce(func, wait) {
  var timeout;
  return function () {
    clearTimeout(timeout)
    timeout = setTimeout(function () { func() }, wait)
  }
}
```
改善后的版本(绑定监听事件时的this,传递arguments)
```
function Debounce(func, wait) {
  var timeout;
  var arg = [];
  for (var i = 2; i < arguments.length; i++) {
    arg.push(arguments[i]);//传递arguments
  }
  return function () {
    var context = this;//传递this
    clearTimeout(timeout)
    timeout = setTimeout(function () { func.apply(context, arg) }, wait)
  }
}
```

### 节流  
播放电影的例子: 人眼的识别效率是最高每秒24次, 所以电影播放的速度是每秒24帧, 大于这个帧数其实也是一种浪费. 如果片源(监听事件)是每秒100帧, 可以进行一些处理, 让它以每秒24帧的最大执行速率播放(执行方法). 

思路 : 设置一个标识符表示当前时间是否可以执行回调函数 , 触发监听事件时根据该变量判断是否执行方法 , 执行方法时更新此变量 . 

```
//时间戳版本
function Throttle(fn, wait) {
  var lastTime = null;//设置标识符
  var arg = [];
  for (var i = 2; i < arguments.length; i++) {
    arg.push(arguments[i]);//传递arguments
  }
  return function () {
    var execTime = + new Date()
    if (execTime - lastTime > wait || !lastTime) {
      fn.apply(this, arg);
      lastTime = execTime
    }
  }
}
```

```
//定时器版本
function Throttle(fn, wait) {
  var canRun = true;//设置标识符
  var arg = [];
  for (var i = 2; i < arguments.length; i++) {
    arg.push(arguments[i]);//传递arguments
  }
  return function () {
    if (!canRun) {
      return;// 判断是否已空闲，如果在执行中，则直接return
    }
    var context = this;//传递this
    canRun = false;
    setTimeout(function () {
      fn.apply(context, arg);
      canRun = true;
    }, wait);
  }
}

```

使用时 , 可以自己封装简单的防抖与节流函数 , 也可以使用第三方方法库 . 
[underscore.js](https://underscorejs.org/)
[lodash.js](https://lodash.com/docs/4.17.10/)

### requestAnimationFrame
>requestAnimationFrame是浏览器用于定时循环操作的一个接口，类似于setTimeout，主要用途是按帧对网页进行重绘。(告诉浏览器在下一个动画帧对页面进行重绘).

```
function execWithRequestAnimationFrame(fn) {
  var scheduledAnimationFrame = false;
  var arg = [];
  for (var i = 1; i < arguments.length; i++) {
    arg.push(arguments[i]);//传递arguments
  }
  return function () {
    if (scheduledAnimationFrame) {
       return;
    }
    scheduledAnimationFrame = true;
    var context = this;//传递this
    window.requestAnimationFrame(() => {
      scheduledAnimationFrame = false;
      fn.apply(context, arg);
    })
  }
}
```

不支持IE9及以下
可配合```cancelAnimationFrame```使用