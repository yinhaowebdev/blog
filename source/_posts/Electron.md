title: Electron
author: Billy Yin
tags: []
categories: []
date: 2020-05-23 00:15:00
---
Electron是一个跨平台桌面应用开发工具
### 安装
```
npm install --save-dev electron
```
### Hello World
```
const { app, BrowserWindow } = require('electron')

function createWindow () {   
  // 创建浏览器窗口
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true // 开启nodejs功能
    }
  })

  // 加载index.html文件
  win.loadFile('index.html')
}

app.whenReady().then(createWindow)
```
[electron/electron-quick-start](https://github.com/electron/electron-quick-start)
[官网模版推荐](https://www.electronjs.org/docs/tutorial/boilerplates-and-clis#electron-react-boilerplate)

### 架构
一个主进程  + 一或多个渲染进程
类似  Chromium 的多进程架构

在普通的浏览器中，web页面通常在沙盒环境中运行，并且无法访问操作系统的原生资源。 然而 Electron 的用户在 Node.js 的 API 支持下可以在页面中和操作系统进行一些底层交互。

主进程使用 BrowserWindow 实例创建页面。 每个 BrowserWindow 实例都在自己的渲染进程里运行页面。 当一个 BrowserWindow 实例被销毁后，相应的渲染进程也会被终止。

### 进程间通信
####  HTML5 APIs

* localStorage
* sessionStorage
* Indexed DB [MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API)  [参考博客](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)


#### Electron remote 模块
```
// main.js
    global.sharedObject = {
      someProperty: 'default value'
    }
```
```
// renderer.js
var electron = require('electron')
var sharedObject = electron.remote.getGlobal('sharedObject')

console.log(sharedObject.someProperty)
```
### Electron ipc
[ipcMain](https://www.electronjs.org/docs/api/ipc-main) / [ipcRenderer](https://www.electronjs.org/docs/api/ipc-renderer)

```
// 在主进程中.
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})


//在渲染器进程 (网页) 中。
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

```
// 在主进程中.
const {ipcMain} = require('electron');
ipcMain.on('synchronous-message', (event, arg) => { 
  console.log(arg) // prints "ping" event.returnValue = 'pong' 
})


//在渲染器进程 (网页) 中。
const { ipcRenderer } = require('electron');
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"
```

### 打包
[electron-builder](https://www.electron.build/)
```
yarn add electron-builder --dev
```
在package.json中添加build配置
```
//摘自 react-electron-template 模版项目
"build": {
    "productName": "ElectronReact",
    "appId": "org.develar.ElectronReact",
    "files": [
      "dist/",
      "node_modules/",
      "app.html",
      "main.prod.js",
      "main.prod.js.map",
      "package.json"
    ],
    "dmg": {
      "contents": [
        {
          "x": 130,
          "y": 220
        },
        {
          "x": 410,
          "y": 220,
          "type": "link",
          "path": "/Applications"
        }
      ]
    },
    "win": {
      "target": [
        "nsis",
        "msi"
      ]
    },
    "linux": {
      "target": [
        "deb",
        "rpm",
        "AppImage"
      ],
      "category": "Development"
    },
    "directories": {
      "buildResources": "resources",
      "output": "release"
    },
    "publish": {
      "provider": "github",
      "owner": "electron-react-boilerplate",
      "repo": "electron-react-boilerplate",
      "private": false
    }
  },
```

参考：

[https://www.electronjs.org/docs](https://www.electronjs.org/docs)

[https://newsn.net/say/electron-ipc.html](https://newsn.net/say/electron-ipc.html)

[https://electron.org.cn/builder/index.html](https://electron.org.cn/builder/index.html)




