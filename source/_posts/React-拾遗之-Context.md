title: React 拾遗之 Context
author: Billy Yin
tags: []
categories: []
date: 2018-12-09 16:46:00
---
### React 拾遗之 Context

>React.js 的 context 其实像就是组件树上某颗子树的全局变量

* 使用方法 

new ( React 16.3 之后的版本适用 )
```javascript

// context.js
export const ThemeContext = React.createContext({
  background: 'red',
  color: 'white'
});

// App.js
import {ThemeContext} from './context.js'

class App extends React.Component {
  render () {
    return (
      <ThemeContext.Provider value={{background: 'green', color: 'white'}}>
        {/*some extra components*/}
       </ThemeContext.Provider>
    )
  }
}

// some child Component
import {ThemeContext} from './context.js'

class ChildComponent extends React.Component {
  render () {
    return (
      <ThemeContext.Consumer>
        {context => (
          <h1 style={{background: context.background, color: context.color}}>
            {this.props.children}
          </h1>
        )}
       </ThemeContext.Consumer>
    )
  }
}
```

old ( React 17 以后将废除 )

```javascript

// App.js

import React, { Component } from 'react';
import PropTypes from 'prop-types'

class App extends Component {
  constructor() {
    super()
    this.state = {
      number: 1,
      text: ''
    }
  }

  static childContextTypes = {
    globalNumber: PropTypes.number,
    globalText: PropTypes.string,
  }

  getChildContext() {
    return {
      globalNumber: this.state.number,
      globalText: this.state.text
    }
  }

  render() {
    return (
        <div>
            {/*some extra components*/}
        </div>
    );
  }
}

// some child Component

class ChildComponent extends Component {
    constructor() {
        super()
    }
    static contextTypes = {
        globalNumber: PropTypes.number
    }
    render() {
        <span>{this.context.globalNumber}</span>
    }
}

```