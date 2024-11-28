title: 初步理解React Fiber
author: Billy Yin
tags: []
categories: []
date: 2020-01-19 15:12:00
---
Fiber是React 16中新的协调器（Reconciler）

### 之前的协调器
Stack Reconciler
见[React Diff 初探](https://www.jianshu.com/p/0c50f06de379)

一口气执行所有的Diff，render和真实DOM改变

缺点：js执行是单线程的，它将GUI描绘，时间器处理，事件处理，JS执行，远程资源加载统统放在一起，阻塞其他任务，影响用户体验

### Fiber的目标
一种能够彻底解决主线程长时间占用问题的机制
> The “fiber” reconciler is a new effort aiming to resolve the problems inherent in the stack reconciler and fix a few long-standing issues.

### Fiber的基本思想
把渲染和更新拆分成小任务，拆分的粒度称为Fiber。通过合理的调度机制来达到更强的控制力

![Stack Reconciler ](https://upload-images.jianshu.io/upload_images/9594241-5b7524cfd8f3d184.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Fiber Reconciler](https://upload-images.jianshu.io/upload_images/9594241-7781a8cbb8dc0313.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 两个阶段

###### 1 diff阶段 （reconciliation / render）

找出新老实例的差异和对应的DOM change

可以拆分，默认为低优先级任务，可以暂停或重来
```
[UNSAFE_]componentWillMount
[UNSAFE_]componentWillReceiveProps
getDerivedStateFromProps
shouldComponentUpdate
[UNSAFE_]componentWillUpdate
render
```
###### 2 patch阶段 （commit）

把找出的DOM change应用到DOM上，是一连串的DOM操作

不可拆分
```
getSnapshotBeforeUpdate
componentDidMount
componentDidUpdate
componentWillUnmount
```
#### 粒度
基本单位称为fiber，fiber tree 是通过 virtual dom tree 对应的构造出来的，可以理解为一个fiber对应一个虚拟dom节点

#### fiber tree 结构
每个节点存储return, child, sibling，分别对应父节点， 第一个孩子， 它右边的兄弟。有了它们，我们可以把树结构变成链表结构，以此实现深度优先遍历。
![fiber tree](https://upload-images.jianshu.io/upload_images/9594241-923f284109f901a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### workloop
基本规则是：每个工作单元结束检查是否还有时间做下一个，没时间了就先“挂起”

```
// Flush asynchronous work until there's a higher priority event
while (nextUnitOfWork !== null && !shouldYieldToRenderer()) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
```

优先级规则：

```
module.exports = {
  NoWork: 0, // No work is pending.
  SynchronousPriority: 1, // For controlled text inputs. Synchronous side-effects.
  AnimationPriority: 2, // Needs to complete before the next frame.
  HighPriority: 3, // Interaction that needs to complete pretty soon to feel responsive.
  LowPriority: 4, // Data fetching, or result from updating stores.
  OffscreenPriority: 5, // Won't be visible but do the work in case it becomes visible.
};
```

优先级策略的核心是，在reconciliation阶段，低优先级的操作可以被高优先级的操作打断，并让主线程执行高优先级的更新，用户可感知的响应更快。

#### 断点/恢复
在需要退出主线程占用时保存当前的成果（effectlist），tag标记当前节点，开一个requestIdleCallback，等下一次被唤起继续进行或重新开始

#### requestIdleCallback
[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)

>window.requestIdleCallback()方法将在浏览器的空闲时段内调用的函数排队。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。

对于不支持这个API的浏览器，react会加上pollyfill。

### 总结
* react fiber解决了前的栈协调器之带来的任务阻塞问题
* 从执行上来看，render及之前的生命周期属于diff阶段，可以异步执行，render后的生命周期属于patch阶段，同步执行
* diff阶段的过程实际上就是收集effectList的过程
* 尽量不要在diff阶段的生命周期中编写含有副作用的代码

### 参考
https://juejin.im/post/5c70f044f265da2de4507ab9
https://github.com/MuYunyun/blog/blob/master/React/%E6%B7%B1%E5%85%A5Fiber%E6%9E%B6%E6%9E%84.md
https://zhuanlan.zhihu.com/p/51483167
https://zhuanlan.zhihu.com/p/57346388
https://zhuanlan.zhihu.com/p/58863799