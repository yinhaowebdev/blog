title: Chrome Extension 简介
author: Billy Yin
tags: []
categories: []
date: 2022-09-26 09:13:00
---
本文基于 manifest_v2 版本
### 简介

https://developer.chrome.com/docs/extensions/

> Extensions are software programs, built on web technologies (such as HTML, CSS, and JavaScript) that enable users to customize the Chrome browsing experience.

### 能力
* 右键菜单
* 调试栏tab
* 插件弹窗
* 监听/拦截网络请求
* 自定义空白页
* 注入脚本
* 书签，浏览记录，下载记录......

### 特点
* 用户控制
* 随时插拔
* 本地运行
* 开源

### 说明文件
manifest.json

示例：
```json
{
    "name": "Hello, World!",
    "version": "1.0",
    "manifest_version": 2, // 现在可用2/3，详情见官方文档
    "permissions": [ // 权限
        "tabs",
        "clipboardWrite",
        "http://*/*",
        "https://*/*",
        "webRequest",
        "bookmarks"
    ],
    "browser_action": { // 插件栏的描述
        "default_title": "这是一个示例Chrome插件",
        "default_popup": "popup.html"
    },
    "content_scripts": [ // 随页面运行的脚本
        {
            "matches": [
                "<all_urls>"
            ],
            "all_frames": false,
            "run_at": "document_end",
            "js": [
                "dev.js"
            ]
        }
    ],
    "background": { // 后台常驻脚本
        "scripts": [
            "background.js" 
        ]
    }
}
```

### 模块
#### background
后台运行脚本，可选常驻
#### popup
插件弹窗
#### content-script
位于当前页面的作用域的脚本，可以和当前页面共享变量
### 通信

#### 页面 & content-scripts
使用postMessage
```js
// 发送方
window.postMessage(
    'content', "*"
);
// 接收方
window.addEventListener("message", function (ev) {
    console.log(ev.data)
})
```
#### content-scripts & popup
chrome.runtime.sendMessage / onMessage
```js
// 发送方
chrome.runtime.sendMessage({
    info: "send"
}, res => {
    console.log(res)
})
// 接收方
chrome.runtime.onMessage.addListener((req, sender, sendResponse) => {
    console.log('send', req)
    sendResponse('callback')
})
```
#### popup & background
```js
var background = chrome.extension.getBackgroundPage(); // 得到background页的windows对象
background.GetMessageFromPopup("给background点东西")

function GetMessageFromBackground(data) {
    console.log("background给我的东西~", data)
}



var popup = chrome.extension.getViews({ type: "popup" })[0]
popup.GetMessageFromBackground("给popup点东西~")

function GetMessageFromPopup(data) {
    var popup = chrome.extension.getViews({ type: "popup" })[0]
    popup.GetMessageFromBackground("给popup点东西~")
    console.log("popup给我的东西~", data)
}

```
### 参考
https://www.jianshu.com/p/3c7fea458245
https://juejin.cn/post/7001037081875054628