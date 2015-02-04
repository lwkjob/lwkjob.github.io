---
layout: post
title:  "linux Cron定时执行"
keywords: "linux, shell,linux cron"
description: "给jsp统一异常处理"
category: linux
tags: [linux]
---

 

## linux Cron
> 用linux Cron 做定时任务，比如:

- 定时生成对账文件，
- 定时给商户发送通知，
- 错误订单定时清理
- 可以定时执行shell脚本，脚本里面就可以做任何我们想做的事。。。

1 、认识Cron
> cron是一个linux下的定时执行工具，可以在无需人工干预的情况下运行作
> 由于Cron 是Linux的内置服务，但它不自动起来，可以用以下的 
> 方法启动关闭这个服务：

- ` /sbin/service crond start  //启动服务 `
- ` /sbin/service crond stop //关闭服务 `
- ` /sbin/service crond restart //重启服务 `
- ` /sbin/service crond reload //重新载入配置`

> 你也可以将这个服务在系统启动的时候自动启动：

> 在`/etc/rc.d/rc.local` 这个脚本的末尾加上：
` /sbin/service crond start`

2、Cron服务

1)直接用crontab命令编辑
> cron服务提供crontab命令来设定cron服务的，参数与说明：

`crontab -u `//设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数

`crontab -l `//列出某个用户cron服务的详细内容

`crontab -r` //删除某个用户的cron服务

`crontab -e `//编辑某个用户的cron服务

> 比如说root查看自己的cron设置：crontab -u root -l

> 再例如，root想删除fred的cron设置：crontab -u fred -r

> 在编辑cron服务时，编辑的内容有一些格式和约定，

输入：
``` shell 
crontab -u root -e 
```
 进入vi编辑模式，编辑的内容一定要符合下面的格式：
 ``` shell
 */1 * * * * ls >> /tmp/ls.txt
 ```

> 这个格式的前一部分是对时间的设定，后面一部分是要执行的命令，如果要执行的命令太多，
> 可以把这些命令写到一个脚本里面，然后在这里直接调用这个脚本就可以了
> 调用的时候记得写出命令的完整路径。时间的设定我们有一定的约定，
> 前面五个*号代表五个数字，数字的取值范围和含义如下：

分钟（0-59） 小時（0-23） 日期（1-31） 月份（1-12） 星期（0-6）//0代表星期天
 
除了数字还有几个个特殊的符号就是`  * / - , `

-  `*`代表所有的取值范围内的数字，
-  `/`代表每的意思,"*/5"表示每5个单位，
-  `-`代表从某个数字到某个数字,
-  `,`分开几个离散的数字。

## 2 举几个例子说明问题：

每天早上6点

`0 6 * * * echo "Good morning." >> /tmp/test.txt //注意单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。`

每两个小时

`0 */2 * * * echo "Have a break now." >> /tmp/test.txt`

晚上11点到早上8点之间每两个小时，早上八点

`0 23-7/2,8 * * * echo "Have a good dream：" >> /tmp/test.txt`

每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点

`0 11 4 * 1-3 command line`

1月1日早上4点

`0 4 1 1 * command line`

> 每次编辑完某个用户的cron设置后，cron自动在/var/spool/cron下生成一
> 个与此用户同名的文件，此用户的cron信息都记录在这个文件中，
> 这个文件是不可以直接编辑的，只可以用`crontab -e `来编辑。
> cron启动后每过一份钟读一次这个文件，检查是否要执行里面的命令。因此
> 此文件修改后不需要重新启动cron服务
