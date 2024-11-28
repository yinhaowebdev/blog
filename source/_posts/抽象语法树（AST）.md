title: 抽象语法树（AST）
author: Billy Yin
tags: []
categories: []
date: 2020-04-28 20:35:00
---
# 一个极简编译器的实现
一篇文章 ： [https://github.com/jamiebuilds/the-super-tiny-compiler/blob/master/the-super-tiny-compiler.js](https://github.com/jamiebuilds/the-super-tiny-compiler/blob/master/the-super-tiny-compiler.js)
文章里实现了一个简单的编译器，将LISP代码编译为C代码：
| 数学算式|LISP| C |
| ------ | ------ | ------ |
| 2 + (4 - 2)  |  (add 2 (subtract 4 2))|  add(2, subtract(4, 2)) |

代码编译通常经过三个阶段：
*  Parsing
*  Transformation
*  Code Generation

### Parsing

##### 源码
```
(add 2 (subtract 4 2))
```
##### Lexical Analysis Result
```
[
   { type: 'paren',  value: '('        },
   { type: 'name',   value: 'add'      },
   { type: 'number', value: '2'        },
   { type: 'paren',  value: '('        },
   { type: 'name',   value: 'subtract' },
   { type: 'number', value: '4'        },
   { type: 'number', value: '2'        },
   { type: 'paren',  value: ')'        },
   { type: 'paren',  value: ')'        },
]
```
##### Syntactic Analysis Result
```
{
   type: 'Program',
   body: [{
      type: 'CallExpression',
      name: 'add',
      params: [{
        type: 'NumberLiteral',
        value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
         value: '2',
       }]
     }]
    }]
}
```

### Transformation

* 深度优先**遍历**旧AST，作出相应的**修改**，生成新AST

##### 修改
创建Visitor对象，定义遍历经过每一种节点时的行为

##### 遍历
实现深度优先遍历的方法

### Code Generation
将AST重新序列化，组成目标代码

# JavaScript中的AST
[AST Explorer](https://astexplorer.net/)

### Babel
[https://babel.docschina.org/](https://babel.docschina.org/)

##### babel/babylon
[babel/babylon](https://github.com/babel/babylon)
[babel-parser](https://babeljs.io/docs/en/next/babel-parser.html)
```bash
npm install @babel/parser --save-dev
```

```javascript
const code = `const text = 'Hello World';`;

const ast = require("@babel/parser").parse(code);

console.log("ast", ast);

```

##### babel/traverse
[babel-traverse](https://github.com/babel/babel/tree/master/packages/babel-traverse)
[babel-core](https://babel.docschina.org/docs/en/babel-core)
```bash
npm install @babel/traverse --save-dev
```
```javascript
const code = `const text = 'Hello World';`;

const ast = require("@babel/parser").parse(code);

require("@babel/traverse").default(ast, {
  enter(path) {
    console.log("path type is ", path.type);
  },
});

//path type is  Program
//path type is  VariableDeclaration
//path type is  VariableDeclarator
//path type is  Identifier
//path type is  StringLiteral

```

```
const code = `const text = 'Hello World';`;

const ast = require("@babel/parser").parse(code);

require("@babel/traverse").default(ast, {
  enter(path) {
    if (path.node.type === "StringLiteral" && path.node.value) {
      path.node.value = `${path.node.value} new!`;
    }
  },
});

```

##### babel/generator
[babel-generator](https://babel.docschina.org/docs/en/babel-generator)
```bash
npm install @babel/traverse --save-dev
```
```
const code = `const text = 'Hello World';`;

const ast = require("@babel/parser").parse(code);

require("@babel/traverse").default(ast, {
  enter(path) {
    if (path.node.type === "StringLiteral" && path.node.value) {
      path.node.value = `${path.node.value} new!`;
    }
  },
});

const output = require("@babel/generator").default(ast, {});

console.log(output.code);

```


参考资料：
[https://cloud.tencent.com/developer/doc/1260](https://cloud.tencent.com/developer/doc/1260)
[https://www.zcfy.cc/article/347](https://www.zcfy.cc/article/347)
