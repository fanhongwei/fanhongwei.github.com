---
layout: post
title: "MentalModel开发心得"
date: 2013-10-28 08:39
comments: true
categories: Android
---
学校工业设计系一个实验室的老师在研究Android的交互设计，研究到人的心智模型这一块时，需要做一个实验，然后呢，就要用到一个Android的App，最后我帮他做了这个App，这期间有很多的体会和经验，写下来留给自己看。

这个应用大概是这样子的：

   <img src="/images/实验2.jpg" width="250" height="400"/> <img src="/images/testtwo.png" width="250" height="400" />

<!--more-->

   <img src="/images/testthree.png" width="250" height="400" /> <img src="/images/image.png" width="250" height="400" />

由于是实验型的App，这里边有将近45个页面，刚接触到这个应用时由于跟那老师的交流出了点问题，所以对应用的理解有偏差，没意识到会有这么多的界面，于是乎我就把没一个页面当作一个activity，开始一个新的页面activity是结束掉上一个页面的activity，当工程进行到后边时，深刻体会到了这种思路的缺陷。这应该也是第一次做到会有如此多的布局的App，深刻体会到了命名规范的好处。比如说，总共有四个实验，而每个实验有12中模式，那么对布局文件的命名很值得考量一番，要不会很乱的。

这个App实验三和实验四是图形模式或者flash模式，并且每种模式大概会有20张左右的图片，如果不用listview去分批加载的话，就会出现内存溢出的问题，由于我刚开始是在genymotion上的调试的一直没遇到这问题，也没去考虑，当准备开始测试的时候，在真机上调试之后很容易就出现了这个问题，一下子加载20张图片，这。。。最终，是用listview解决的这个问题。

既然是实验App，那么就肯定需要记录测试者的实验数据，这个实验记录的是测试者找到目标物的时间,7顺理成章的就写了一个专门记载时间的类，项目具体东西放github了。[MentalModel](https://github.com/fanhongwei/MentalModel)

最后，在做这个App的过程中还学到的一个东西，就是真正结束掉一个App，具体是思路是就是利用堆栈的原理，在intent实现activity跳转的时候加`intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);`然后就可以轻松结束应用了，关于结束应用的方法网上还有很多，我就不说了。
