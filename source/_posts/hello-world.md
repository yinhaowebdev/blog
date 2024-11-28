title: websocket初探
tags: []
categories: []
date: 2018-08-08 21:31:00
---
### Websocket 是什么?
>WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

![websocket.png](https://upload-images.jianshu.io/upload_images/9594241-fff473e2b2859700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(图片来自阮一峰博客,侵删)

### websocket 的特点(相比ajax轮询的优点)

1,即时通讯
2,节约服务器资源
3,连接持久化
4,不受跨域限制


### 原生websocket的写法(客户端)

```
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  //do something...
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
}; 
```
### socket.io的基础应用(以create-react-app 和 express为例)

##### 最简单的实现
服务端
```
const express = require('express');
const app = express();
const io = require('socket.io');

let server = require('http').createServer(app);
server = app.listen(8080, function () {
    const host = server.address().address;
    const port = server.address().port;
    console.log("访问地址为 http://%s:%s", host, port);
})
let socket = io.listen(server, { origins: '*:*' });

socket.on('connection', function (client) {
    console.log('connection')
    client.on('sendMsg', function (data) {
        client.emit('newMsg', { name: "我", msg: data.msg });
        client.broadcast.emit('newMsg', { name: data.name, msg: data.msg });
    });

    // 当用户断开时执行此指令
    client.on('disconnect', function () {
        console.log('disconnect')
    });
});

```
客户端
```
import React, { Component } from 'react';
import './App.css';

const socket = require('socket.io-client')('http://localhost:8080');

class App extends Component {
    constructor() {
        super()
        this.state = {
            msg: '',
            name: '',
            dialogList: []
        }
    }
    componentDidMount() {
        socket.on('newMsg', (data) => {
            this.setState({
                dialogList: this.state.dialogList.concat({
                    name: data.name,
                    msg: data.msg
                })
            })
        });
    }
    onContentChange(e) {
        this.setState({
            msg: e.target.value
        })
    }
    onNameChange(e) {
        this.setState({
            name: e.target.value
        })
    }
    emitWebsocket() {
        socket.emit("sendMsg", { name: this.state.name, msg: this.state.msg })
    }
    render() {
        return (
            <div className="App">
                姓名:<input onChange={this.onNameChange.bind(this)} type='text' />
                内容:<input onChange={this.onContentChange.bind(this)} type='text' />
                <button onClick={this.emitWebsocket.bind(this)}>发送</button>
                <div>
                    {
                        this.state.dialogList.map((item) => {
                            return (
                                <div>
                                    {item.name}:{item.msg}
                                </div>
                            )
                        })
                    }
                </div>
            </div>
        );
    }
}

export default App;

```

##### room的概念

1. 在不加入或指定room的情况下，socket.io 会默认分配一个default room
2. 同一room下的socket可以广播消息
3. join(房间名) 加入一个房间；leave(房间名) 离开一个房间；broadcast.to(房间名).emit() 给同一个房间的其他socket广播消息

服务端
```
const express = require('express');
const app = express();
const io = require('socket.io');

let server = require('http').createServer(app);
server = app.listen(8080, function () {
    const host = server.address().address;
    const port = server.address().port;
    console.log("访问地址为 http://%s:%s", host, port);
})
let socket = io.listen(server, { origins: '*:*' });

socket.on('connection', function (client) {
    client.on("joinRoom", function (data, fn) {
        client.join(data.room); // join(房间名)加入房间
        fn(`加入房间成功,房间名称:${data.room}`);
    });
    client.on("leaveRoom", function (data, fn) {
        client.leave(data.room); // leave(房间名)退出房间
        fn(`退出房间成功,房间名称:${data.room}`);
    });
    client.on('sendMsg', function (data) {
        client.emit('newMsg', { name: "我", msg: data.msg });
        socket.in(data.room).emit("newMsg", { name: data.name, msg: data.msg });
    });

    // 当用户断开时执行此指令
    client.on('disconnect', function () {
        console.log('disconnect')
    });
});

```

客户端

```
import React, { Component } from 'react';
import './App.css';

const socket = require('socket.io-client')('http://localhost:8080');

class App extends Component {
    constructor() {
        super()
        this.state = {
            msg: '',
            name: '',
            room: '',
            dialogList: []
        }
    }
    componentDidMount() {
        socket.on('newMsg', (data) => {
            this.setState({
                dialogList: this.state.dialogList.concat({
                    name: data.name,
                    msg: data.msg
                })
            })
        });
    }
    onContentChange(e) {
        this.setState({
            msg: e.target.value
        })
    }
    onNameChange(e) {
        this.setState({
            name: e.target.value
        })
    }
    onRoomChange(e) {
        this.setState({
            room: e.target.value
        })
    }
    emitWebsocket() {
        socket.emit("sendMsg", { name: this.state.name, msg: this.state.msg, room: this.state.room })
    }
    onJoinRoom() {
        socket.emit("joinRoom", { room: this.state.room }, (data) => { console.log(data) })
    }
    onLeaveRoom() {
        socket.emit("leaveRoom", { room: this.state.room }, (data) => { console.log(data) })
    }
    render() {
        return (
            <div className="App">
                房间:<input onChange={this.onRoomChange.bind(this)} type='text' />
                <button onClick={this.onJoinRoom.bind(this)}>进入房间</button>
                <button onClick={this.onLeaveRoom.bind(this)}>离开房间</button>
                <br />
                姓名:<input onChange={this.onNameChange.bind(this)} type='text' />
                内容:<input onChange={this.onContentChange.bind(this)} type='text' />
                <button onClick={this.emitWebsocket.bind(this)}>发送</button>
                <div>
                    {
                        this.state.dialogList.map((item) => {
                            return (
                                <div>
                                    {item.name}:{item.msg}
                                </div>
                            )
                        })
                    }
                </div>
            </div>
        );
    }
}

export default App;

```

##### namespace 的概念

1.在不声明新的命名空间情况下，系统会默认使用default namespace。
2.不同命名空间下的socket是不能互相通信了，是处于隔离状态的。
3.服务端使用 io.of(空间名称)声明一个命名空间。
4.客户端修改连接目录连接到一个具体的命名空间。

服务端
```
const express = require('express');
const app = express();
const io = require('socket.io');

let server = require('http').createServer(app);
server = app.listen(8080, function () {
    const host = server.address().address;
    const port = server.address().port;
    console.log("访问地址为 http://%s:%s", host, port);
})

let _io = new io(server)

//创建命名空间
let namespace1 = _io.of("/namespace1");
let namespace2 = _io.of("/namespace2");

// let socket = io.listen(server, { origins: '*:*' });

namespace1.on('connection', function (client) {
    console.log('有连接到了namespace1')
    client.on("joinRoom", function (data, fn) {
        client.join(data.room); // join(房间名)加入房间
        fn(`加入房间成功,房间名称:${data.room}`);
    });
    client.on("leaveRoom", function (data, fn) {
        client.leave(data.room); // leave(房间名)退出房间
        fn(`退出房间成功,房间名称:${data.room}`);
    });
    client.on('sendMsg', function (data) {
        client.emit('newMsg', { name: "我", msg: data.msg });
        socket.in(data.room).emit("newMsg", { name: data.name, msg: data.msg });
    });

    // 当用户断开时执行此指令
    client.on('disconnect', function () {
        console.log('disconnect')
    });
});

namespace2.on('connection', function (client) {
    console.log('有连接到了namespace2')
    client.on("joinRoom", function (data, fn) {
        client.join(data.room); // join(房间名)加入房间
        fn(`加入房间成功,房间名称:${data.room}`);
    });
    client.on("leaveRoom", function (data, fn) {
        client.leave(data.room); // leave(房间名)退出房间
        fn(`退出房间成功,房间名称:${data.room}`);
    });
    client.on('sendMsg', function (data) {
        client.emit('newMsg', { name: "我", msg: data.msg });
        socket.in(data.room).emit("newMsg", { name: data.name, msg: data.msg });
    });

    // 当用户断开时执行此指令
    client.on('disconnect', function () {
        console.log('disconnect')
    });
});

```

客户端(分为两个)
```
import React, { Component } from 'react';
import './App.css';

const socket = require('socket.io-client')('http://localhost:8080/namespace1');//第二个客户端把这里改成namespace2

class App extends Component {
    constructor() {
        super()
        this.state = {
            msg: '',
            name: '',
            room: '',
            dialogList: []
        }
    }
    componentDidMount() {
        socket.on('newMsg', (data) => {
            this.setState({
                dialogList: this.state.dialogList.concat({
                    name: data.name,
                    msg: data.msg
                })
            })
        });
    }
    onContentChange(e) {
        this.setState({
            msg: e.target.value
        })
    }
    onNameChange(e) {
        this.setState({
            name: e.target.value
        })
    }
    onRoomChange(e) {
        this.setState({
            room: e.target.value
        })
    }
    emitWebsocket() {
        socket.emit("sendMsg", { name: this.state.name, msg: this.state.msg, room: this.state.room })
    }
    onJoinRoom() {
        socket.emit("joinRoom", { room: this.state.room }, (data) => { console.log(data) })
    }
    onLeaveRoom() {
        socket.emit("leaveRoom", { room: this.state.room }, (data) => { console.log(data) })
    }
    render() {
        return (
            <div className="App">
                房间:<input onChange={this.onRoomChange.bind(this)} type='text' />
                <button onClick={this.onJoinRoom.bind(this)}>进入房间</button>
                <button onClick={this.onLeaveRoom.bind(this)}>离开房间</button>
                <br />
                姓名:<input onChange={this.onNameChange.bind(this)} type='text' />
                内容:<input onChange={this.onContentChange.bind(this)} type='text' />
                <button onClick={this.emitWebsocket.bind(this)}>发送</button>
                <div>
                    {
                        this.state.dialogList.map((item) => {
                            return (
                                <div>
                                    {item.name}:{item.msg}
                                </div>
                            )
                        })
                    }
                </div>
            </div>
        );
    }
}

export default App;

```

分别启动两个客户端,会发现namespace1和namespace2完全无法进行通讯,它们是完全互相隔离的.

参考资料:

http://www.cnblogs.com/limitcode/p/7900832.html
https://segmentfault.com/a/1190000012411572