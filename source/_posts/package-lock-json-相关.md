title: package-lock.json 相关
author: Billy Yin
tags: []
categories: []
date: 2020-03-11 15:35:00
---
目前日常开发时，使用的包管理命令是cnpm。
但是cnpm是依赖 [npminstall]([https://www.npmjs.com/package/npminstall](https://www.npmjs.com/package/npminstall)
) 的，特性之一是不会生成package-lock.json文件。

cnpm工具 相对于 npm 的特点：
1 速度快
2 node_modules中会生成更多的文件夹
3 不会生成 package-lock.json文件

### package-lock.json
理想情况下，依赖包的版本是按照  [semver规范](https://semver.org/lang/zh-CN/) 来的，然而实际上有的包可能没有遵循这一原则。
>主版本号：当你做了不兼容的 API 修改，
次版本号：当你做了向下兼容的功能性新增，
修订号：当你做了向下兼容的问题修正。

有一种思路是为了迎合第三方库自己的修正，package.json 中对于第三方库的写法如下：
```json
"express": "^4.17.1",
```
这样可以及时的将第三方库的bugfix等在项目中体现。

另一种思路是为了保持版本稳定不出错，不自动更新第三方库的版本，即使是修订号的更新，这时需要一个文件来存储“这一次”到底安装了哪些依赖。
package-lock.json对依赖的描述如下：
```json
       "@ant-design/colors": {
            "version": "3.2.2",
            "resolved": "http://r.cnpmjs.org/@ant-design/colors/download/@ant-design/colors-3.2.2.tgz",
            "integrity": "sha1-WtQ9YZ6RHzSI66wwPWBuZqhCOQM=",
            "requires": {
                "tinycolor2": "^1.4.1"
            }
        },
```

描述范围包括版本、源地址、校验hash和它自身的依赖。指定它们之后可以保证使用该文件安装的副本是完全一样的。

### NPM 对 package-lock.json 的策略


1、npm 5.0.x 版本，不管package.json怎么变，npm i 时都会根据lock文件下载

[package-lock.json file not updated after package.json file is changed · Issue #16866 · npm/npm](https://github.com/npm/npm/issues/16866)

问题：明明手动改了package.json，却不升级包

2、5.1.0版本后 npm install 会无视lock文件 去下载最新的npm

 [why is package-lock being ignored? · Issue #17979 · npm/npm](https://github.com/npm/npm/issues/17979)

问题：lock文件失去意义，无法进行版本锁定

3、5.4.2版本后
如果修改了package.json，且package.json和lock文件不同，在执行npm install时npm会根据package中的版本号以及语义含义去下载最新的包，并更新至lock。
如果在执行npm install时package.json和lock文件相同，则忽略更新版本。

### NRM ( NPM registry manager)
一种可以使用npm工具安装淘宝源的方法：
```
npm install nrm -g
nrm ls
nrm use cnpm
```
这种情况下速度较快，会生成package-lock.json文件。

### Yarn
特点：
* 引入 yarn.lock 文件来管理依赖版本问题，保证每次安装都是一致的。
* 缓存加并行下载保证了安装速度

yarn.lock 与 package-lock.json 的主要区别在于 yarn.lock 中只有一层，而package-lock.json中是嵌套结构。

### 讨论
1 是否使用lock方案
2 如果要使用lock方案，使用哪种方案？（nrm-taobao source/ yarn / npm / 其它）
3 为保证依赖稳定，是否有其它注意事项？（如统一 node与npm 版本）