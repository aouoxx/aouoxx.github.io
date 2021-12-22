---
layout: post
title: linux系统
categories: [底层原理]
description: linux系统
keywords: 底层原理
---

<meta name="referrer" content="no-referrer"/>

### 系统硬件

#### cpu load 介绍

```
我们经常去看linux的平均负载, 通过uptime或者 top命令就可以显示出来, 平均负载的内容如下
gaoshuoshuo@hb16381 ~ % uptime
15:18  up 2 days, 21:29, 2 users, load averages: 2.34 2.74 2.87

三个数字分别代表1分钟, 5分钟 和15分钟三个时间段的CPU负载的平均值, 且数值越低越好。
数字越高表示系统出现了问题或者机器过载。
对单核而言,负载0.7需要注意并排查原因,1.0表示不紧急需要处理,5.0紧急状态,立即处理


在多处理器上,负载相对于可用处理器核心数量的。在单核系统上,"100%利用率"表示负载1.0,双核系统为2.0,四核为4.0
	"核数=最大负载": 在多核系统上, 您的负载不应超过可用核数
	"Core就是core"的经验法则, cpu core的性能与cpu上的分布方式无关
			两个四核==四个双核==八个单核。他们的性能与8个Core的性能等同。

mac查看cpu核数
DannideMacBook-Pro:~ danni$ sysctl -n machdep.cpu.core_count
4
linux查看cpu的核数
cat /proc/cpuinfo 可以获得系统的CPU信息。
若只想得到CPU核数，可以运行： grep 'model name' /proc/cpuinfo | wc -l
```

#### 修改文件句柄数

```
vi /etc/security/limits.conf
在文件末尾添加两行
$ cat /etc/security/limits.conf
* soft core unlimited
* soft nofile 1000000
* hard nofile 1000000
* soft nproc 1024000
* hard nproc 1024000

soft和hard为两种限制方式，其中soft表示警告的限制，hard表示真正限制，nofile表示打开的最大文件数。
整体表示任何用户一个进程能够打开1000000个文件。注意语句签名有 号 表示任何用户

shutdown -r now 重启linux
```
