---
layout: post
title: "Linux内核编译"
date: 2014-03-09 21:05
comments: true
categories: Linux
---
系统发行版ubuntu13.04，内核版本3.12.13
首先，官网下载所需内核。

为了之后的方便，切换到root权限    命令：`sudo -i`

复制内核包到制定目录    命令：`cp ./linux-3.12.13.tar.xz   /usr/src`

解压内核包   命令：`tar -xvf linux-3.12.13.tar.xz`

切换到/usr/src目录下  命令：`cd /usr/src`

![](/images/image-5.png)


<!--more-->


![](/images/image-9.png)

把原来编译产生的垃圾删除   命令：`make mrproper`

配置内核,选用基于图形窗口模式的配置界面   命令：`make xconfig`

正常的话应该可以开始配置内核了，可惜ubuntu没有自带qt环境或者我之前吧qt环境卸载了

![](/images/image-7.png)

缺qt3环境，然后去装qt3环境，发现现在大多数源里边已经没有qt3了，索性安装qt4的环境，重新开始配置内核，发现2.6的内核必须要qt3的环境，于是换到了3.12.13的内核。

重新执行 `make xconfig`,出现配置界面

![](/images/image-11.png)

开始配置内核，看不懂，找到了@金步国的一篇博文  [博文链接](http://www.jinbuguo.com/kernel/longterm-3_10-options.html)

![](/images/image-12.png)

为了确保所有有关文件都处于最新版本状态  命令：`make clean`

开始编译压缩形式的内核  命令：`make zImage`

![](/images/image-14.png)

大概编译了20左右，内核编译完毕。

![](/images/image-16.png)

开始编译模块,这个过程时间比较长。  命令:`make modules`

编译完模块之后开始将编译后的模块转移到系统标准位置  命令：`make module_install`

最后执行 `make install`,这应该算是最后一步了，结果出问题了。

![](/images/image-18.png)

原因：当时装系统的时候boot分区是单挂的，分了80M进去，结果`make install`的时候报错：gzip: stdout: No space left on device.  看样子是boot分区太小gzip解压包的时候没有空间了。

解决办法：删掉boot分区下边当前没用的所有内核，然后update-grub 结果boot还是太小，最后想着把boot umount掉，然后把活动分区标记为根分区，想了想太危险，万一搞坏了，现在ubuntu下的所有配好的环境都完蛋了，最后选择了虚拟机。
