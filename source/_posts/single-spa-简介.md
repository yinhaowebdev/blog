title: single-spa 简介
author: Billy Yin
tags: []
categories: []
date: 2020-11-09 23:26:00
---
### 微前端

SPA提升了开发效率。然而，一些维护周期长的项目（如后台管理），随着人员的流动和业务的堆积，易变成“巨石应用。这样的应用很难整体迁移到最新技术栈，也不方便进行架构上的拆分和重组，成为了很多企业和开发者遇到的难题。

微前端是将微服务的理念延伸到浏览器端，即将 Web 应用由单一的单体应用转变为多个小型前端应用聚合为一的应用。
### single-spa
[官网](https://single-spa.js.org/)
简单的说，single-spa可以将多个不同的SPA“粘合”到一个整体。
single-spa 主要包括以下部分
1 html模板

2 微应用配置代码
其中为每个微应用至少注册三个参数：
* 名称
* 加载微应用的函数
* 判断微应用是否激活的函数
```
// single-spa-config.js
import { registerApplication, start } from 'single-spa';
// Simple usage
registerApplication(
  'app2',
  () => import('src/app2/main.js'),
  (location) => location.pathname.startsWith('/app2'),
  { some: 'value' }
);
// Config with more expressive API
registerApplication({
  name: 'app1',
  app: () => import('src/app1/main.js'),
  activeWhen: '/app1',
  customProps: {
    some: 'value',
  }
);
start();
```
3 微应用本身的代码

### 官方实例（老版本API了，仅供参考）
https://github.com/single-spa/single-spa-examples

### 从头搭一个single-spa示例
本例仅使用react项目，为开发示例，不涉及部署
1 项目初始化
```
npm init
```
2 需要安装babel相关/ single-spa相关/ react相关 / webpack相关
```
// package.json
{
  "name": "single-spa-demo",
  "version": "1.0.0",
  "description": "",
  "scripts": {
    "start": "webpack-dev-server"
  },
  "dependencies": {
    "@babel/cli": "^7.0.0-beta.40",
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.8.3",
    "@babel/preset-env": "^7.7.6",
    "@babel/preset-react": "^7.12.5",
    "@babel/runtime": "^7.12.5",
    "babel-core": "^6.26.3",
    "babel-eslint": "^11.0.0-beta.2",
    "babel-loader": "^8.1.0",
    "babel-plugin-inferno": "^3.3.0",
    "babel-plugin-syntax-dynamic-import": "^6.18.0",
    "babel-plugin-transform-class-properties": "^6.24.1",
    "babel-plugin-transform-function-bind": "^6.22.0",
    "babel-plugin-transform-object-rest-spread": "^6.26.0",
    "babel-plugin-transform-react-jsx": "^6.24.1",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-polyfill": "^6.26.0",
    "babel-preset-env": "^1.6.1",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-react": "^6.24.1",
    "cross-env": "^7.0.2",
    "css-loader": "^0.28.7",
    "html-loader": "^0.5.1",
    "react": "^16.12.0",
    "react-dom": "^16.12.0",
    "react-router": "^5.2.0",
    "react-router-dom": "^5.2.0",
    "single-spa": "^5.8.1",
    "single-spa-react": "^2.14.0",
    "style-loader": "^0.19.0",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10",
    "webpack-config-single-spa-react": "^1.0.3",
    "webpack-dev-server": "^3.9.0",
    "webpack-merge": "^4.2.2"
  },
  "author": "",
  "license": "ISC"
}
```

```
yarn
```
3 webpack基本配置
```
// webpack.config.js
module.exports={
    entry: __dirname + '/index.js',
    output: {
        path: process.cwd() + '/build',
        filename: '[name].js',
        publicPath: '/build/',
    },
    devtool: 'inline-source-map',
    devServer: {
        port: 8080,
        publicPath: '/build/',
        contentBase: './',
        index: 'index.html',
        historyApiFallback: true
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader']
            },
            {
                test: /\.html$/,
                exclude: /node_modules/,
                loader: 'html-loader',
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
            },
        ],
      }
}
```
4 入口文件中注册微前端应用
```
// index.js
import {registerApplication, start} from 'single-spa';

registerApplication('nav', () => import('./nav/index.js'), () => location.pathname.startsWith('/'));
registerApplication('react1', () => import('./react1/index.js'), () => location.pathname.startsWith('/react1'));
registerApplication('react2', () => import('./react2/index.js'), () => location.pathname.startsWith('/react2'));

start()
```
5 HTML模版
```
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="navbar"></div>
  <div id="react-app1"></div>
  <div id="react-app2"></div>
  <script src="/build/main.js"></script>

</body>
</html>
```
6 导航栏
```
// nav/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import singleSpaReact from 'single-spa-react';
import rootComponent from './rootComponent.js';

const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent,
  domElementGetter: () => document.getElementById('navbar')
});

export const bootstrap = [
  reactLifecycles.bootstrap,
];

export const mount = [
  reactLifecycles.mount,
];

export const unmount = [
  reactLifecycles.unmount,
];
```
```
// nav/rootComponent.js
import React from 'react';
import { Link, BrowserRouter as Router, } from 'react-router-dom';

export default class Root extends React.Component {
  constructor() {
    super();
  }

  render() {
    return (
        <div>
            <Router>
                <Link to='/react1'>
                    <span>react1</span>
                </Link>
                <Link to='/react2'>
                    <span>react2</span>
                </Link>
            </Router>
        </div>
    );
  }
}

```
7 react1应用
```
// react1/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import singleSpaReact from 'single-spa-react';
import rootComponent from './rootComponent.js';

const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent,
  domElementGetter: () => document.getElementById('react-app1')
});

export const bootstrap = [
  reactLifecycles.bootstrap,
];

export const mount = [
  reactLifecycles.mount,
];

export const unmount = [
  reactLifecycles.unmount,
];
```
```
// react1/rootComponent.js
import React from 'react';
import { BrowserRouter as Router, Route } from "react-router-dom";

export default class Root extends React.Component {
  constructor() {
    super();
  }

  render() {
    return (
        <div>
            <Router>
                <Route path="/react1/page1" component={()=><span>1</span>}/>
                <Route path="/react1/page2" component={()=><span>2</span>}/>
                <Route path="/react1" exact component={()=><span>this is react1</span>}/>
            </Router>
        </div>
    );
  }
}
```
8 react2应用
```
// react1/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import singleSpaReact from 'single-spa-react';
import rootComponent from './rootComponent.js';

const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent,
  domElementGetter: () => document.getElementById('react-app2')
});

export const bootstrap = [
  reactLifecycles.bootstrap,
];

export const mount = [
  reactLifecycles.mount,
];

export const unmount = [
  reactLifecycles.unmount,
];
```
```
// react2/rootComponent.js
import React from 'react';
import { BrowserRouter as Router, Route } from "react-router-dom";

export default class Root extends React.Component {
  constructor() {
    super();
  }

  render() {
    return (
        <div>
            <Router>
                <Route path="/react2/page1" component={()=><span>1</span>}/>
                <Route path="/react2/page2" component={()=><span>2</span>}/>
                <Route path="/react2" exact component={()=><span>this is react2</span>}/>
            </Router>
        </div>
    );
  }
}
```
9 本地运行
```
yarn start
```