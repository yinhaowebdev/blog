title: Linux 压缩/解压
author: Billy Yin
tags: []
categories: []
date: 2020-06-14 20:48:00
---
# tar
##### 归档
tar -cvf test.tar file1 file2
##### 更新
tar -tf test.tar --add-file=1.js
##### 新增
tar -rf test.tar 2.js
##### 删除
tar --delete -f test.tar  1.js
##### 压缩 
tar -zcvf test.tar.gz file1 file2  // gzip方式
tar -jcvf test.tar.bz2 file1 file2 //bzip2
[Gzip , BZip2 , Lzo Snappy 四种方式的优缺点 和使用场景](https://blog.csdn.net/weixin_40040107/article/details/87885210)

排除某些文件 --exclude
tar -zcvf test.tar.gz --exclude=file1 ./

删除源文件 --remove-files 
tar -zcvf test.tar.gz 1.html --remove-files 

##### 查看
tar -tvf test.tar.gz

##### 解压
tar -xvf test.tar.gz -C dir
tar -xvf test.tar.gz 1.html -C ./dir  #解压指定文件


# zip
##### 压缩 （指定压缩率1-9）
zip -r8 test.zip ./

##### 向压缩包中添加文件
zip -u test.zip 1.js

##### 删除zip中的文件
zip -d test.zip 1.js 

##### 加密
zip -r test.zip 1.html -P 66666

##### 解压
unzip  test.zip -d .

##### 解压指定
unzip test.zip "1.html" -d .


# 压缩率比较
一般来说
tar.bz2>tar.gz>zip>tar