title: redux-persist
author: Billy Yin
tags: []
categories: []
date: 2020-05-08 14:25:00
---
前端应用，状态信息可以存储在：
* 1 应用内部（eg:state 、redux）
* 2 URL 
* 3 缓存（localStorage/seesionStorage）

在使用SPA框架开发时，通常尽量采用上述第一种方式以符合框架设计者的意图，方便状态管理。
然而，有时产品会有这样的需求：刷新页面时不改变某些状态（如id），此时可能考虑使用URL进行辅助的存储。
然而，产品说刷新时不止id不变，整个页面的状态都不可以改变，用URL存这么复杂的状态，会遇到长度限制和管理混乱的问题。
于是，考虑使用上述第三种方式。
自己实现的话思路大概是：
* 修改redux中的监听器，当有状态更新时同步到缓存
* 修改应用最外层组件，在挂载时读取缓存，如果有，读取后手动触发redux状态更新

[redux-persist](https://github.com/rt2zz/redux-persist)是一个实现redux状态持久化的库。下面在介绍它在umi.js中的使用。

首先安装依赖，由于umi和redux-persist在各个大版本区别较大，读者尝试时请注意对应版本：
```
//package.json
    "umi": "2.13.1",
    "umi-plugin-react": "1.15.1",
    "redux-persist": "^5.10.0"
```
添加src/app.js：
```
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/es/storage/session';

const persistConfig = {
    timeout: 1000, // you can define your time. But is required.
    key: 'root',
    storage,
};

const persistEnhancer = () => createStore => (reducer, initialState, enhancer) => {
    const store = createStore(persistReducer(persistConfig, reducer), initialState, enhancer);
    const persist = persistStore(store, null);
    return {
        persist,
        ...store,
    };
};
export const dva =
    process.env.APP_TYPE === 'build'
        ? {
              config: {
                  extraEnhancers: [persistEnhancer()],
              },
          }
        : {
              config: {
                  extraEnhancers: [persistEnhancer()],
              },
              plugins: [
                  {
                      // onAction: createLogger(),
                  },
              ],
          };

```

修改最外层组件，本例中是BasicLayout.js：
```
//...
import { PersistGate } from 'redux-persist/integration/react';
//...
return (
            <React.Fragment>
                <PersistGate persistor={window.g_app._store.persist} loading={null}>
                    <DocumentTitle title="">
                        <ContainerQuery query={query}>
                            {params => (
                                <Context.Provider value={this.getContext()}>
                                    <ConfigProvider locale={zhCN}>
                                        <div className={classNames(params)}>{layout}</div>
                                    </ConfigProvider>
                                </Context.Provider>
                            )}
                        </ContainerQuery>
                    </DocumentTitle>
                    <Suspense fallback={<PageLoading />} />
                </PersistGate>
            </React.Fragment>
        );
//...
```
现在刷新浏览器试试吧？