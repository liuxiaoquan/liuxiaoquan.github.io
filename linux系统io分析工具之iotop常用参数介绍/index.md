# Linux系统IO分析工具之iotop常用参数介绍


# Linux系统IO分析工具之iotop常用参数介绍

在一般运维工作中经常会遇到这么一个场景，服务器的IO负载很高（iostat中的util），但是无法快速的定位到IO负载的来源进程和来源文件导致无法进行相应的策略来解决问题。

　　Windows操作系统可以通过鲁大师等硬盘检测工具来查看硬盘读写速度，那么linux下测试硬盘IO读写情况怎么看?iotop是linux系统下测试硬盘IO读写的工具，简单的说,iotop是一个用来监视磁盘I/O使用状况的 top 类工具，可监测到哪一个程序使用的磁盘IO的信息（requires 2.6.20 or later)



**1>.安装iotop**

```sh
[root@node105 ~]# yum -y install iotop
```

![](/img/455717929.png)



**2>.查看iotop的帮助信息**

```sh
[root@node105 ~]# iotop -help
```

![](/img/429233705.png)

```
各个参数说明：

　　-o, 		--only只显示正在产生I/O的进程或线程。除了传参，可以在运行过程中按o生效。
　　-b, 		--batch非交互模式，一般用来记录日志。
　　-n NUM, 	--iter=NUM设置监测的次数，默认无限。在非交互模式下很有用。
　　-d SEC, 	--delay=SEC设置每次监测的间隔，默认1秒，接受非整形数据例如1.1。
　　-p PID, 	--pid=PID指定监测的进程/线程。
　　-u USER, 	--user=USER指定监测某个用户产生的I/O。
　　-P, 		--processes仅显示进程，默认iotop显示所有线程。
　　-a, 		--accumulated显示累积的I/O，而不是带宽。
　　-k, 		--kilobytes使用kB单位，而不是对人友好的单位。在非交互模式下，脚本编程有用。
　　-t, 		--time 加上时间戳，非交互非模式。
　　-q, 		--quiet 禁止头几行，非交互模式。有三种指定方式。
　　-q 		只在第一次监测时显示列名
　　-qq 		永远不显示列名。
　　-qqq 		永远不显示I/O汇总。
　　
交互按键：
　　和top命令类似，iotop也支持以下几个交互按键。
　　left和right方向键：改变排序。　　
　　r：反向排序。
　　o：切换至选项--only。
　　p：切换至--processes选项。
　　a：切换至--accumulated选项。
　　q：退出。
　　i：改变线程的优先级。
```



**3>.** **只显示正在产生I/O的进程或线程。除了传参，可以在运行过程中按o生效。**

```sh
[root@node105 ~]# iotop  -o
```

![](/img/1206579995.png)



**4>.时间刷新间隔2秒，输出5次**

```sh
[root@node105 ~]# iotop  -d 2 -n 5
```

![](/img/616593521.png)



**5>.非交互式，输出5次，间隔2秒，输出到屏幕，也可输出到日志文本，用于监控某时间段的io信息**

```sh
[root@node105 ~]# iotop -botq -n 5 -d 2 
```

![](/img/795254-20181109131052927-394915228.png)



**6>.非交互式，输出pid为8382的进程信息**

```sh
[root@kafka118 ~]# iotop -botq -p 8382
```

![](/img/795254-20181109131514749-951139135.png)




