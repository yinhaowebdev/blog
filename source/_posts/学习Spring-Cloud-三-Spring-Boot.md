title: '学习Spring Cloud(三): Spring Boot'
author: Billy Yin
tags: []
categories: []
date: 2020-12-12 23:01:00
---
Spring Boot以“约定大于配置”的思想简化了Spring的开发，使其能够“开箱即用”。

### 项目结构
![项目结构](https://upload-images.jianshu.io/upload_images/9594241-2be4764d5643f7cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* src/main/java中包含主程序入口DemoApplication.java (文件名随项目名可变)
* src/main/resources用于存放配置信息
* src/test为单元测试目录

### 小记 VSCode 开发 Spring Boot 
###### 0 基本环境预备
* 安装JDK
* 安装Maven
* 配置Maven镜像（推荐阿里云）
###### 1 settings.json
```
{
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "java.errors.incompleteClasspath.severity": "ignore",
    "java.home":"/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home",
    "java.configuration.maven.userSettings": "/Library/apache-maven-3.6.3/conf/settings.xml",
    "maven.executable.path": "/Library/apache-maven-3.6.3/bin/mvn",
    "maven.terminal.useJavaHome": true,
    "maven.terminal.customEnv": [
        {
        "environmentVariable": "JAVA_HOME",
        "value": "/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home"
        }
    ],
}
```
###### 2 插件
* Spring Boot Tools
* Spring Boot Extension Pack
* Java Extension Pack
* Debugger for Java

###### 3 新建项目
* control+shift+p create spring boot project
选择 maven
选择 starters (starter POMs 是一系列轻便的包，用于)
生成项目
* 开始运行调试

访问 localhost:8080
![](https://upload-images.jianshu.io/upload_images/9594241-fe230ed668ef3da3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

证明程序启动成功。

###### 4 Hello World
在 DemoApplication.java所在目录下新建  controller/HelloController.java
```
package test.com.example.demo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/")
public class HelloController {
    @Autowired

    @RequestMapping("/hello")
    @ResponseBody
    public String helloWorld(){
        return "Hello World!";
    }
}
```

访问 localhost:8080/hello
![Hello World](https://upload-images.jianshu.io/upload_images/9594241-3c5ed9680710994b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
