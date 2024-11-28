title: 记Ubuntu16.04 下 LibreOffice的安装与使用
author: Billy Yin
tags: []
categories: []
date: 2020-09-27 15:52:00
---
# LibreOffice主包和语言包
```bash
wget http://mirrors.ustc.edu.cn/tdf/libreoffice/stable/6.4.6/deb/x86_64/LibreOffice_6.4.6_Linux_x86-64_deb.tar.gz
wget http://mirrors.ustc.edu.cn/tdf/libreoffice/stable/6.4.6/deb/x86_64/LibreOffice_6.4.6_Linux_x86-64_deb_langpack_zh-CN.tar.gz
tar xf LibreOffice_6.4.6_Linux_x86-64_deb.tar.gz
sudo dpkg -i LibreOffice_6.4.6.2_Linux_x86-64_deb/DEBS/*.deb
tar xf LibreOffice_6.4.6_Linux_x86-64_deb_langpack_zh-CN.tar.gz
sudo dpkg -i LibreOffice_6.4.6.2_Linux_x86-64_deb_langpack_zh-CN/DEBS/*.deb
```

# 字体

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/adobe-fonts/source-han-serif/SubsetOTF/SourceHanSerifCN.zip
wget https://mirrors.tuna.tsinghua.edu.cn/adobe-fonts/source-han-sans/SubsetOTF/SourceHanSansCN.zip
unzip SourceHanSansCN.zip
unzip SourceHanSerifCN.zip

sudo cp -r SourceHanSansCN /usr/share/fonts/
sudo cp -r SourceHanSerifCN /usr/share/fonts/

sudo apt install fontconfig
fc-cache -fv
```

# 其它依赖

```bash
sudo apt install -y openjdk-8-jdk libxinerama1 libcairo2 libcups2 libsm6
```

# 使用（word转pdf）

```bash
libreoffice6.4 --invisible --convert-to pdf input.docx --outdir .
```


附: 使用PHPWord生成docx文件

https://blog.nuozhilin.site/2020/09/18/2020-09-18-laravel-library-phpword/