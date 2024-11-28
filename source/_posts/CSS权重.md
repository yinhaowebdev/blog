title: CSS权重
author: Billy Yin
tags: []
categories: []
date: 2021-02-04 23:54:00
---
### 权重计算
[w3c 官网介绍](https://www.w3.org/TR/CSS2/cascade.html#specificity)

> A selector's specificity is calculated as follows:
> * count 1 if the declaration is from is a 'style' attribute rather than a rule with a selector, 0 otherwise (= a) (In HTML, values of an element's "style" attribute are style sheet rules. These rules have no selectors, so a=1, b=0, c=0, and d=0.)
> * count the number of ID attributes in the selector (= b)
> * count the number of other attributes and pseudo-classes in the selector (= c)
> * count the number of element names and pseudo-elements in the selector (= d)

* a: 内嵌样式
* b: id选择器数量
* c: 其它属性选择器(包括class)和伪类选择器的数量
* d: 标签选择器和伪元素选择器的数量

具体计算时遵循a>b>c>d
理论上上一级的比较结果会完全覆盖下一级的
实际上由于内存溢出，在下一级达到256或更高(根据不同的浏览器版本)的计数时会向前“进位”，影响上一级的比较结果

### 后来居上
当以上计算结果相同时，会使用较为靠后的样式

### !important
不属于规范权重计算范围，会覆盖其它任何权重
官方不建议使用
>  Using `!important,` however, is **bad practice** and should be avoided because it makes debugging more difficult by breaking the natural [cascading](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade) in your stylesheets.

