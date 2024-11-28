title: Dva Subscription
author: Billy Yin
tags:
  - React
categories: []
date: 2018-08-10 13:41:00
---
做项目的时候用到dva, 想实现监听路由传递参数的时候遇到了麻烦,翻文档发现可以用subscription,但官方文档的资料实在不多...
>Subscriptions 是一种从 源 获取数据的方法，它来自于 elm。
>
>Subscription 语义是订阅，用于订阅一个数据源，然后根据条件 dispatch 需要的 action。
>
>数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等。
### 几个例子:

监听history

```javascript
setup({ dispatch, history }) {
    history.listen((location) => {
    const match = pathToRegexp('/map/machineMapInProject/:id').exec(location.pathname);
        if (match) {
            const id = match[1];
            dispatch({
                type: 'updateStateProps',
                payload: {
                    name: 'listParams',
                    value: { project: id },
                },
            });
        }
    });
},
```

监听key事件

```javascript
import key from 'keymaster';
...
app.model({
  namespace: 'count',
  subscriptions: {
    keyEvent(dispatch) {
      key('⌘+up, ctrl+up', () => { dispatch({type:'add'}) });
    },
  }
});
```

后续还会加入监听websocket的例子.
subscriptions的一大好处应该就是可以监测全局的变化, 即使和当前页面不相关model里也可以进行数据改动或者请求接口之类的.