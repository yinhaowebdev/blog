title: '学习Spring Cloud(一): Java'
author: Billy Yin
tags: []
categories: []
date: 2020-11-27 14:32:00
---
### Java安装

> 因为Java程序必须运行在JVM之上，所以，我们第一件事情就是安装JDK。

[Oracle官网JDK下载页面](https://www.oracle.com/java/technologies/javase-downloads.html)

### Hello Java

> touch Hello.java
> vim Hello.java

```
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```
编译为字节码
>javac Hello.java

在JVM上运行
>java Hello

### IDE
主流的有[Eclipse](https://www.eclipse.org/) 和 [IntelliJ IDEA](https://www.jetbrains.com/idea/)

### Maven
* 提供了一套标准化的项目结构；
* 提供了一套标准化的构建流程（编译，测试，打包，发布……）；
* 提供了一套依赖管理机制。
[官网下载](https://maven.apache.org/)
配置文件pom.xml 示例：
```
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
	</dependencies>
</project>
```
