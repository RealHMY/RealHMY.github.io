---
title: 通过jstack找CPU高占用的线程（Windows）
author: RealHMY
top: false
cover: false
toc: true
mathjax: false
date: 2022-05-25 16:27:27
img:
coverImg:
password:
summary:
tags:
categories:
---

之前有一天开会，开了两小时，发现满电的笔记本剩下的电量不多，觉得有些不对劲，哪怕电池老化，耗电也不至于那么多。

快下班时，本地重启项目后发现JVM有10%的占用，按道理说不应该有超过2%的占用，因为项目里并没有高占用的代码在运行。

于是查找教程，看看Windows下怎么查找高占用的代码。

不同于Linux，Windows的任务管理器不能查看线程的状态，只能看进程，所以要先下载能查看线程状态的工具ProcessExplorer。巨硬官网下载ProcessExplorer：https://download.sysinternals.com/files/ProcessExplorer.zip

解压后打开procexp.exe

按CPU排序，找到高占用的java.exe，找到PID
![](/images/2022-5-26.png)

在cmd拷贝当前虚拟机运行状态
```shell
jstack -l 18688 > c:/temp/19688stack.txt
```

回到ProcessExplorer，右键java.exe，选择Properties（可能有提示版本不对，需要下载新版本的调试工具，直接点×就能跳过提示，巨硬的骚操作

点击Thread 选项，找到高CPU的TID（PID是进程id，TID是线程id，我们需要知道TID）

在计算器中，选择程序员模式，输入TID，复制HEX的值（也就是将十进制转为十六进制）

在刚刚的txt文件搜索十六进制的TID


    "XXXX-timer-28-1" #50 prio=5 os_prio=0 tid=0x0000000029be0800 **nid=0x89a0** runnable [0x000000002e4ee000]    java.lang.Thread.State: RUNNABLE
    at java.lang.Thread.sleep(Native Method)
    at io.netty.util.HashedWheelTimer$Worker.waitForNextTick(HashedWheelTimer.java:567)
    at io.netty.util.HashedWheelTimer$Worker.run(HashedWheelTimer.java:466)
    at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.lang.Thread.run(Thread.java:748)


可以看到nid=0x89a0就是高占用的地方，这个线程是XXXX-timer-28-1，它是一个MQ（这里以XXXX表示）启动的线程，这个MQ依赖于netty，所以还能看到netty的类。

MQ在项目启动后一直处于运行状态，占用一个线程，而我的处理器是八核十六线程，MQ约占10%。至此，项目启动后造成10%占用的元凶就找到了。