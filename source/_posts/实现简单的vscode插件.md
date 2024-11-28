title: 实现简单的vscode插件
author: Billy Yin
tags: []
categories: []
date: 2019-10-12 00:15:00
---
### 安装工具

脚手架工具 Yeoman 和 Generator-code

> cnpm install -g yo generator-code

插件打包和发布工具 vsce

> cnpm install -g vsce

### 使用脚手架搭建vscode项目

> yo code

选择 "New Extension (JavaScript)"

### 编辑项目
编辑 package.json
```
// ...
"contributes": {
        "commands": [
            {
                "command": "extension.helloWorld",
                "title": "Hello World"
            }
        ],
        "snippets": [
            {
                "language": "javascript",
                "path": "./snippets/javascript.json"
            }
        ]
    },
// ...
```

> mkdir snippets
> cd snippets
> touch javascript.json
> vim javascript.json

```
{
    "forEach": {
        "prefix": "fe",
        "body": [
            "${1:array}.forEach(function(item) {",
            "\t${2:// body}",
            "});"
        ],
        "description": "Code snippet for \"forEach\""
    },
    "setTimeout1000": {
        "prefix": "st1000",
        "body": [
            "setTimeout(function() {",
            "\t${0:// body}",
            "}, ${1:1000});"
        ],
        "description": "Code snippet for 'setTimeout'"
    },
    "HelloWorld": {
        "prefix": "hello",
        "body": [
            "console.log('hello world')"
        ],
        "description": "Code snippet for 'hello world'"
    },
    "dispatch": {
        "prefix": "dispatch",
        "body": [
            "dispatch({",
            "\ttype:'${0://type}',",
            "\tpayload:{",
            "\t\t${2:// payload}",
            "\t}",
            "});"
        ],
        "description": "Code snippet for 'dispatch'"
    }
}
```

### 调试

Ctrl+Shift+D 

Run Extension

Tip:  
可能需要更新VSCode版本

### 打包

> vsce package

Tip: 
1 可能需要在package.json中加入发布者名称
```
"publisher": "yinhao",
```
2 可能需要修改 README.md

### 安装使用

Ctrl+Shift+p

extension : ...

