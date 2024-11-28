title: GC in V8 Engine
author: Billy Yin
tags: []
categories: []
date: 2021-06-07 14:14:00
---
# 空间
在node.js中查看内存使用情况
```
process.memoryUsage()
```
堆空间限制原因之一：GC耗时影响体验和程序性能，限制空间总大小可以限制GC时长

# 大致过程

### 1 标记

目前V8采用可达性来判断堆中是否是活动对象(是否需要回收)
即从根节点(GC Roots)进行遍历，遍历到的为活动对象、反之为非活动对象
根节点包括但不限于:
* windows 对象
* DOM树
* 栈中变量
  
### 2 回收
清理内存中未被标记对象
### 3 整理
清理内存碎片，防止出现无大连续空间可分配情况
# 代际假说

>* 大部分对象在内存中存在的时间很短，就是说很多对象一经分配内存，立即被使用后，很快就不在被需要了。比如函数局部变量，或者块级作用域中的变量
>* 不死的对象，会存活得很久。比如全局变量、 window、DOM等对象

基于代际假说，V8 将堆分为新生代和老生代
望文生义：
* 新生代存放生存时间较短的对象
* 老生代存放生存时间较长的对象

分别对应了两个垃圾回收器
### 副垃圾回收器---作用范围新生代
采用Scavenge算法，将新生代内部分为两个区
* From Space 对象区域 新
* To Space 空闲区域

新对象都放在From Space，当From Space快满了，便执行上述的``标记``过程，然后将存活的对象复制到To Space，复制的过程也发挥了上述的``整理``过程的作用。复制完成后，将From Space 和 To Space角色翻转，进入下一个周期

##### 对象晋升
将经过两次回收依然存活的对象移入老生代
### 主垃圾回收器---作用范围老生代
##### 标记-清除 (Mark Sweep)
![标记-清除](https://upload-images.jianshu.io/upload_images/9594241-3d0853b2ec7560da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将未标记的非活动对象清除
##### 标记-整理 （Mark Compact）
![标记-整理](https://upload-images.jianshu.io/upload_images/9594241-1a979b10dd034531.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将标记的活动对象移到内存区域的一端，然后对另一端的内存整体消除

# STW
JavaScript运行在主线程上，GC需要将主线程内的脚本暂停，GC完成之后再恢复执行，称为Stop-The-World。STW会造成程序卡顿，影响体验。为此，人们提出了一些策略来优化GC的执行
# 并行回收
![并行回收](https://upload-images.jianshu.io/upload_images/9594241-501fe7e7cf13cd9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

GC时多线程，但仍然STW

# 增量标记
![Incremental Marking](https://upload-images.jianshu.io/upload_images/9594241-cc7540e9d41a0ec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由于是增量过程，需要记下上一次执行标记的情况

### 三色标记
* 起初，所有对象都是白色
* 发现一个对象，就将它标为灰色
* 当一个灰色对象中所有字段都被访问过，这个灰色对象变为黑色
* 当没有灰色对象时，剩余白色对象可被安全回收

### 写屏障
由于是增量标记，两次标记间程序可能会写入新的对象，导致出现不该被回收的对象被回收，如后插入的对象没来得及标记(为白色)。
写屏障强制规定黑色对象不能直接指向白色对象
![V8中的写屏障](https://upload-images.jianshu.io/upload_images/9594241-9419796d6f44e29f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 并发回收

![并发回收](https://upload-images.jianshu.io/upload_images/9594241-159089b80547b732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

感觉很迷幻啊，这个原理我暂时也不是很清楚，留个坑，慢慢补

