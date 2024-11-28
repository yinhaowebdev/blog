title: VS Code C++ 环境配置 (MacOS)
author: Billy Yin
tags: []
categories: []
date: 2023-01-13 17:33:00
---
### 1 安装插件
C/C++
CODELLDB

### 2 配置插件
cmd+shift+p 输入 C/C++: Edit Configurations (UI)

其中
Compiler path 选 /usr/bin/gcc (不选clang)
IntelliSense mode 选 macos-gcc-x64  (不选clang)


### 3 工程目录.vscode文件夹中

task.json
```json

  "tasks": [
    {
      "type": "cppbuild",
      "label": "C/C++: g++ 生成活动文件",
      "command": "/usr/bin/g++",
      "args": [
        "-std=c++17",
        "-stdlib=libc++",
        "-fdiagnostics-color=always",
        "-g",
        "-Wall",
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}"
      ],
      "options": {
        "cwd": "${fileDirname}"
      },
      "problemMatcher": [
        "$gcc"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "detail": "调试器生成的任务。"
    }
  ],
  "version": "2.0.0"
}
```

launch.json
```json
{
  // 使用 IntelliSense 了解相关属性。 
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "C++ debug",
      "program": "${fileDirname}/${fileBasenameNoExtension}",
      "args": [],
      "cwd": "${workspaceFolder}",
      "preLaunchTask": "C/C++: g++ 生成活动文件"
    }
  ]
}
```