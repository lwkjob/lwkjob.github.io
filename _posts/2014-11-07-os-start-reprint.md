---
layout: post
title: "操作系统启动流程"
description: "转载操作系统的启动流程"
keywords: "reprint, os"
category: os
tags: [转载, os]
---

	转载记录几篇关于操作系统的启动流程,系统启动科普
##### 首先是阮一峰的2篇通俗易懂
- - -
1. 阮一峰[计算机是如何启动的？](http://www.ruanyifeng.com/blog/2013/02/booting.html)
2. 阮一峰[Linux 的启动流程](http://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html)
3. [Linux启动过程详解-《别怕Linux编程》之八](http://roclinux.cn/?p=1301)
4. [Linux 操作系统启动过程 【以Ubuntu 12.04 为例】](http://blog.csdn.net/jandunlab/article/details/11899379)

##### init程序的介绍
- - -
1. [systemd (简体中文)](https://wiki.archlinux.org/index.php/systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
2. [浅析 Linux 初始化 init 系统，第 3 部分: Systemd](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/index.html#ibm-pcon)
3. [设计思路比Upstart更加超前的init系统--systemd](https://linuxtoy.org/archives/more-than-upstart-systemd.html)
4. [systemd wiki百科介绍](http://zh.wikipedia.org/wiki/Systemd)

##### windows启动过程
- - -
1. [windows系统启动过程原理全面分析](http://blog.csdn.net/trypsin/article/details/4466373)

##### 引用阮一峰第二篇文章后面网友的留言
- - -
	
>	Anonymous 说：很多东西说的不足。

> 1 普适性缺乏

>现在debian自己都在讨论要不要引入systemd，RHEL7 alpha1～3(别问为啥我能拿到alpha)/Fedora/Archlinux
都已经完全迁移到systemd，ubuntu用自家的upstart，大概只有debian和gentoo还在用SystemV的这一套了，
因该先将通用的内容，比如init程序，然后再说各种init程序的实现，如systemV/systemd/upstart等等
由于init有不同的实现，所以那些进程脚本位置都有差别。

> 2 缺乏细节

> 即便这篇文章不是针对Linux内核启动的代码分析级别的细致分析，也应该更多的提到bootloader。
最误导人的是读取内核那边，明明是内核+initrd一起读取的。
并且几乎所有非嵌入式发行版通用的initrd这个细节完全没有提到。
其中提到的init程序实际上只要用了initrd都是从initrd里面起起来的。还有initrd的双rootfs等重要的排错机制没提到。
同样第六步提到的各种配置文件，这个一样少了很多东西。
比如pam_env的pam模块就会读取/etc/environment的配置文件，对于这种可以灵活配置的东西因该介绍的是如何自己去寻找读取顺序，而不是只说有哪些
