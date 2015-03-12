---
layout: post
title: "memcached linux 安装"
description: " "
keywords: "memcached, 缓存"
category: life
tags: [memcached, 缓存]
---
 
## Memcached安装

1  准备安装文件

下载memcached与libevent的安装文件
[http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz（memcached下载地址）](http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz（memcached下载地址）)
[https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz（libevent下载地址）](https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz（libevent下载地址）)

2 具体安装步骤
 
2.1 由于`memcached`依赖于`libevent`，因此需要安装`libevent`。由于linux系统可能默认已经安装libevent，执行命令：
`rpm -qa|grep libevent`
查看系统是否带有该安装软件，如果有执行命令:
rpm -e libevent-1.4.13-4.el6.x86_64 --nodeps（由于系统自带的版本旧，忽略依赖删除）

2.2 安装`libevent`命令：

``` 
  tar zxvf libevent-2.0.21-stable.tar.gz
  cd libevent-2.0.21-stable
  ./configure --prefix=/usr/local/libevent
  make
  make install
```
至此libevent安装完毕；

2.3 安装`memcached` 执行命令：

```
  tar zxvf memcached-1.4.2.tar.gz
  cd memcached-memcached-1.4.2
        ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local/libevent/
make
make install

```
  至此memcached安装完毕；
3. 可能存在的错误以及解决方案
如果出现客户端连接不上memcached的情况，请将防火墙关闭或将防火墙中的memcached端口（11211端口）打开。

4 启动memcached
4.1 打开一个终端，输入以下命令：
`/usr/local/memcached/bin/memcached -d -m 256 -u root -p 11211 -c 1024 –P /tmp/memcached.pid`


> 启动参数说明：

`-d ` 选项是启动一个守护进程。
`-u root` 表示启动memcached的用户为root。
`-m ` 是分配给Memcache使用的内存数量，单位是MB，默认64MB。
`-M `return error on memory exhausted (rather than removing items)。
`-u `是运行Memcache的用户，如果当前为root 的话，需要使用此参数指定用户。
`-l `是监听的服务器IP地址，默认为所有网卡。
`-p `是设置Memcache的TCP监听的端口，最好是1024以上的端口。
`-c `选项是最大运行的并发连接数，默认是1024。
`-P `是设置保存Memcache的pid文件。
`-f <factor> chunk size growth factor (default: 1.25)。`
`-I Override the size of each slab page. Adjusts max item size(1.4.2版本新增)。`
  也可以启动多个守护进程，但是端口不能重复
4.2 查看`memcached`启动命令
`ps aux|grep memcached`
 
5 停止`memcached`
打开一个终端，输入以下命令：
`ps -ef | grep memcached`
在服务开启的状态下，会出现如下图所示的提示：

```
lwk     2539     1  0 Mar12 ?        00:00:00 /usr/local/memcached/bin/memcached -d -m 256 -u lwk -p 11211 -c 1024 ?P /tmp/memcached.pid
lwk     4283  3264  0 00:57 pts/3    00:00:00 grep memcached
```

其中2539为memcached服务的pid
输入一下命令终止memcached服务
`kill -9 2539`

参考[http://blog.csdn.net/clarkcc1988/article/details/8509822](http://blog.csdn.net/clarkcc1988/article/details/8509822)