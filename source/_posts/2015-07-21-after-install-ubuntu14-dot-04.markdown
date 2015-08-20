---
layout: post
title: "After install Ubuntu 14.04 LTS"
date: 2014-05-12 14:34
comments: true
categories: Linux
---

# 前奏
之前从ubuntu 13.10升级到ubuntu 14.04 LTS,由于输入法的问题,在删除iBus的时候不小心删除了Unity的一些东西,结果桌面环境挂掉了,无奈重装系统,装完系统倒是焕然一新了,可是各种缺环境.然后开始了一系列配置环境的艰辛历程,通过这篇博文记录这艰辛历程，以备下次强迫症患了再一次作大死。

# 开工
### *基本系统配置*
**换源:** 果断将源换成我大[联创](http://hustunique.com)自己的源 [mirrors.hustunique.com](http://mirrors.hustunique.com).之后`sudo apt-get update`更新一下.

**安装chrome:** 直接[chrome官网](https://www.google.com/intl/zh-CN/chrome/browser/)下载chrome的deb包,然后通过软件中心安装即可.很庆幸的14.04中没有之前版本中出现的缺少lib的问题.

**终端配置:** 安装Termiantor终端,此终端比自带的终端强悍很多.

**终端环境：** 弃用bash,换上高大上的zsh.通过github上的 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)来管理zsh配置,执行以下shell即可
```
sudo apt-get install zsh
curl -L http://install.ohmyz.sh | sh  //自动安装
chsh -s /bin/zsh

```

<!--more-->

**键盘映射修改:** 由于个人习惯,总是喜欢将CapsLk替换为Ctrl,通过Xmodmap来修改键盘映射,在/home下添加.Xmodmap文件

.Xmodmap
```
remove Lock = Caps_Lock
keysym Caps_Lock = Control_L
add control = Control_L
```

**主题配置:** 首先安装Ubuntu Tweak这个主题管理软件.

然后安装我自己喜欢的Numix主题
```
sudo add-apt-repository ppa:numix/ppa
sudo apt-get update
sudo apt-get install numix-gtk-theme
```
字体选择钟爱的Manaco.
最后通过Ubuntu Tweak工具去管理设置这些配置。

**输入法：** 好不容易搜狗支持Linux就用它吧,前往[官网](http://pinyin.sogou.com/linux/)下载deb包,之后安装即可,前提是得安装fcitx.关于输入法这一块会出不少问题,耐心点,慢慢来.

### *Android 开发配置*

**jdk安装：** 终端通过`sudo apt-get install XXX`安装即可.

**Android Studio:** 前往 [Android开发者网站](http://developer.android.com/sdk/installing/studio.html)下载Android Studio.另外,Android Studio有creat Desktop Entry这个功能,不再需要我们自己去写.desktop文件.

**GIMP:** 这个主要就是平时设计师给图之后用来取色或者切图之类的.
```
sudo add-apt-repository ppa:otto-kesselgulasch/gimp
sudo apt-get update
sudo apt-get install gimp
```

**DropBox:** 方便与设计师进行图片的同步,前往 [DropBox官网](https://www.dropbox.com/install?os=lnx)下载deb包即可。

**Genymotion:** 目前作为一个学生党开发者，没有大量的手持设备用来适配，通过Genymotion这些虚拟设备可以达到同样的效果。为了方便,在/usr/share/applications目录下创建genymotion.desktop文件用于快捷启动

genmotion.desktop
```
[Desktop Entry]
Type=Application
Name=Genymotion
Icon=/home/dsnc/Downloads/genymotion/icons/icon.png
Exec="/home/dsnc/Downloads/genymotion/genymotion"
Terminal=false
Categories=Development;IDE;
```

### *博客管理*

**Ruby环境配置:** 个人博客是用Octopress搭建的,所以得配置ruby环境,因为14.04比较新,配置起来比较纠结,缺少各种依赖库,一下是我的成功配置过程

```
sudo apt-get update
sudo apt-get install curl
\curl -sSL https://get.rvm.io | bash -s stable       //install RVM by using curl
source ~/.rvm/scripts/rvm          //load RVM
type rvm | head -n 1          //if install success,there is "rvm is a function"

//install lib,without these the "rvm requirements will failed"
sudo apt-get install gawk g++ libreadline6-dev zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 autoconf libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
rvm requirements  
rvm install ruby
```
>* RVM:Ruby Version Manager

**ReText:** 用来写markdown,直接软件中心下载即可.

### *其他常用应用*

**Docky:** 应用中心搜索

**WPS:** 前往 [WPS官网](http://linux.wps.cn/)下载deb包即可.

**Synapse:** 快速启动器
```
sudo add-apt-repository ppa:synapse-core/testing
sudo apt-get update
sudo apt-get install synapse
```

**截图工具：** 这个我习惯使用的是Shutter
```
sudo add-apt-repository ppa:shutter/ppa
sudo apt-get update
sudo apt-get install shutter
```


## *越来越发现Ubuntu softerware center还是有很多好玩的应用,并且博文中提到的很多软件也都可以同多软件中心直接安装,不用通过添加PPA源的方式安装*
