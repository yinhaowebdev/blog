title: 实现一个Redux
author: Billy Yin
tags: []
categories: []
date: 2019-03-11 13:31:00
---
https://github.com/kianyin/hello-redux
## Step 1 构建create-react-app项目

> create-react-app hello-redux

> cd hello-redux

> cnpm i

> cnpm start

## Step 2 调整项目结构

视图组件放到```components```目录下

## Step 3 用react的方式多层传参

![传统的props传参](http://upload-images.jianshu.io/upload_images/9594241-90929daf2bf0ddbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Step 4 使用context

![context](http://upload-images.jianshu.io/upload_images/9594241-0959a84d97e98b63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们已经不用一层一层用```props```传参啦

问题: ```context```的更新还是较为复杂, 且不可控

## Step 5 使用store

状态存到store里, store接收一个参数```reducer``` 并向外暴露三个方法 ```getState```, ```subscribe```, ```dispatch```

```reducer``` 可以视为改变状态的方法集

```getState``` 用于对外暴露 ```state```

```dispatch``` 用于改变 ```state```

```subscribe``` 用于订阅 ```dispatch``` 后的回调

现在, 我们把 ```store``` 直接传入 ```context```

## Step 6 使用dispatch

这里就有点像 ```redux``` 了, 但是似乎执行 ```dispatch``` 之后没什么作用

## Step 7 添加subscribe

这下就知道视图是怎么跟随 ```store``` 的改变而更新的啦

问题: 每个组件里都有大量重复逻辑

## Step 8 构建高阶组件Connect

高阶函数是对函数的封装 => 函数返回一个函数

高阶组件就是对组件的封装 => 函数返回一个组件

```Connect``` 接收视图组件, 返回经过加工之后的视图组件

这样帮助我们节省了很多重复代码

而且视图组件可以从 ```this.props``` 里面获得 ```state``` , 十分的像 ```redux```

问题: 所有的组件都可以获得所有的```state```值, 有些浪费

## Step 9 mapStateToProps

相当于给 ```Connect``` 多加了一个参数, 用来给组件传递指定的 ```state```