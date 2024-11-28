title: 记在js中判断css是否加载完成（模拟onload）
author: Billy Yin
tags:
  - FrontEnd
categories: []
date: 2021-12-06 17:19:00
---
### 场景
切换主题时动态获取js和css，获取过程中页面加loading效果，获取结束后取消loading效果。

由于css和js分别请求，自然要用promise.all。思路很简单，js加载完成时和css加载完成时触发resolve。

获取js的相关代码如下
```
new Promise((resolve, reject) => {
    try {
      const themeConst = document.createElement('script');
      themeConst.id = '__theme_const__';
      themeConst.type = 'text/javascript';
      themeConst.src = constSrc;
      const body = document.getElementsByTagName('body');
      if (body && body.length) {
        body[0].insertBefore(themeConst, body[0].childNodes[0]);
        themeConst.onload = themeConst.onreadystatechange = function () {
          if (!this.readyState || this.readyState === 'loaded' || this.readyState === 'complete') {
            resolve();
          }
        };
        themeConst.onerror = function () {
        };
      }
    } catch (err) {
      reject(err);
    }
  });
```
运行顺畅。

### 问题
获取css时，也想用onload复制一波，卒。
https://stackoverflow.com/questions/3078584/link-element-onload
https://stackoverflow.com/questions/2635814/capturing-load-event-on-link
新版chrome等浏览器不支持 link 标签的 unload 回调

### 解决
翻阅资料，可以通过轮询link标签的``sheet. cssRules``来判断是否加载好css资源，于是

```
function superviseCssLinkLoad(link, onload) {
  requestIdleCallback(() => {
    if (link.sheet) {
      if (link.sheet.cssRules?.length) {
        typeof onload === 'function' && onload()
      } else {
        superviseCssLinkLoad(link, onload)
      }
    } else {
      superviseCssLinkLoad(link, onload)
    }
  })
}
new Promise((resolve, reject)=>{
      const themeLink = document.createElement('link');
      themeLink.id = '__theme_link__';
      themeLink.rel = 'stylesheet';
      themeLink.href = linkHref;
      const head = document.getElementsByTagName('head');
      if (head && head.length) {
        head[0].appendChild(themeLink);
        superviseCssLinkLoad(themeLink, () => { resolve() })
      }
})
 
```
此处用了 [requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 设置轮询，充分利用了浏览器效率。