title: em/rem
author: Billy Yin
tags:
  - CSS
categories: []
date: 2021-06-10 23:41:00
---
两者都是相对单位
为什么是m？
因为字母m是字母中最方正的...
### em
相对于父元素的font-size
```
<div>
  <p></p>
</div>
```
```
div {
  font-size: 40px;
  width: 10em; /* 400px */
}
p {
  font-size: 0.5em;/* 20px */
  width: 20em; /* 10em表示和font-size同等大小,以此乘除表示放大或缩小几倍 */
  height: 10em;
  background-color: blueviolet;
}
```

### rem
Root em
相对于根元素的font-size
```
    html {
        font-size: 10px;
        }
    div {
        font-size: 4rem; /* 10px为1rem */
        width: 40rem;  /* 10px为1rem */
    }
```
