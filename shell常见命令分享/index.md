# Shell常见命令分享

# centos基础

```shell
ifconfig/ip addr # 查看ip
```

```shell
netstat -tunlp | grep 8080 #查询8080端口是否存在
```

```shell
ping 10.0.0.1 -s 65507 #包大小为65507
ping 10.0.0.1 -l 65500 #window
```

```shell
ifconfig eth0 192.168.60.128 #修改ip
```

```shell
free -m #查看内存
free -h 
```

![img](/img/1.png)

`主要看第一行Mem 总共 15710 M , 使用了 823 M , 剩余空闲 7895 M 。这个shared 223M 也不知道用在哪里。`

```shell
#清除缓存
sudo sync && sudo echo 3 > /proc/sys/vm/drop_caches
```

## Linux之间互传文件：

### 本机->远程服务器

```shell
scp /home/shaoxiaohu/test1.txt shaoxiaohu@172.16.18.1:/home/test2.txt
```

### 远程服务器->本机

```shell
scp shaoxiaohu@172.16.18.2:/home/test2.txt /home/shaoxiaohu/test1.txt
```

### 文件夹复制

```shell
scp -r shaoxiaohu@172.16.18.2:/home/ /home/
```

`如果端口号有更改，需在scp 后输入：-P 端口号 （注意是大写，ssh的命令中 -p是小写）`

```shell
tail -100f access.log #查看100条日志
```

```shell
echo 0 > access.log  #清除日志
```

## dump抓包

`抓包检查，使用tcpdump抓包，tcpdump -vv -i 网卡名称 tcp port 监听端口 -w 文件名.pcap，在服务端和客户端上都抓一下，保存下来分析。（使用wireshark来分析）`

```shell
tcpdump -i eth0 host 10.50.13.121 -vv  -w 001.pcap
```

## 定时任务

`查询crontab服务是否开启 systemctl status crond.service`

```shell
vim /data/monitor.sh
```

```shell
#!/bin/sh
Monitor()
{
for i in 1
do
  curl http://127.0.0.1:8888/hello
done
}
Monitor>>/tmp/tcpbridgeMonitor.log
```

### 运行crontab –e 编写一条定时任务

```shell
crontab -e
```

### 写入

```shell
* * * * * /data/monitor.sh   

* * * * * sleep 20; /data/monitor.sh

* * * * * sleep 40; /data/monitor.sh  ##每隔20秒执行一次监控脚本
```

###  赋权

```shell
chmod -R 777 /data/monitor.sh
```

### 修改拥有者和群组

```shell
chown -R mysql:mysql /data/test #-R处理指定目录以及其子目录下的所有文件
```

### 查看最近的crontab执行情况

```shell
tail -f /var/spool/mail/root
```

###  查询当前用户定时任务

```shell
crontab -l
```

###  删除当前用户定时任务

```shell
crontab -r
```

# 日志拉取与截取

## 拉取指定时间的日志

```shell
sed -n ‘/2010-11-17 09:[0-9][0-9]:[0-9][0-9]/,/2010-11-17 16:[0-9][0-9]:[0-9][0-9]/p’  /hime/log.csv > /data/serverlog/log.csv
```

`注意：如果需要截取的日志太大，达到几个G的话，不能去vi打开文件:
根据之前的日志格式，[使用正则表达式](https://www.baidu.com/s?wd=使用正则表达式&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YkP1c1uHTLn1ndnH9-PH0z0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHRLrHT3nHDd): 截图中日志的格式为：2019-06-13 10:15:27.878 所以正则表达式为：2019-06-13 09:[0-9][0-9]:[0-9][0-9].[0-9][0-9][0-9]`

- **如果截取的时间段是22/Feb/2019:15:57:00，那么可以使用在 / 前使用转移符  \** 

```shell
sed -n  '/22\/Feb\/2019:15:57:00/,/22\/Feb\/2019:15:57:59/'p  /home/wwwlogs/access.log >/root/access0925_0928.log
```

# mysql全量and库备份与数据还原

## 全量备份

```shell
/usr/local/mysql/bin/mysqldump -u用户名 -p密码 --all-databases > /保存路径/文件名.sql
```

![img](/img/quanliang.png) 

## 库备份

```shell
/usr/local/mysql/bin/mysqldump -u用户名 -p密码 库名1 库名2 > /保存路径/文件名.sql
```

![img](/img/kubeif.png) 

## 使用source 命令恢复数据库

![img](/img/huifu.png)

 

# nohup启动

```shell
nohup ./yzweworkplatform-backend >log.txt &
```

`（注释：加了&参数为后台启动，没有加按ctrl会退出）`


# 主机名**

```shell
[root@fangjian ~]# hostnamectl #查看主机名
   Static hostname: brace
   Pretty hostname: Brace
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 20191225111607875619293640639763
           Boot ID: 25ac5021d229471382a26bea3d351de3
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1062.9.1.el7.x86_64
      Architecture: x86-64
```

## 临时修改主机名

```shell
[root@fangjian ~]# hostname yin    #临时修改主机名，关机后失效
[root@fangjian ~]# hostname
yin
```

## 永久修改主机名

1、方法一：使用hostnamectl命令

```shell
[root@fangjian ~]# hostnamectl set-hostname Brace  #永久设置用户名，关机后不失效
[root@fangjian ~]# hostname
brace
```

2、方法二：修改配置文件 /etc/hostname 保存退出 

```shell
[root@fangjian ~]# vi /etc/hostname   # 进入vi，删除旧主机名，输入新主机名，Esc后冒号 wq退出保存
brace　　　　　　　　　　　　　　　　　 # reboot重启生效
```

## 域名的md5值

```shell
echo -n "demo.com" | md5sum 
```



## 释放内存

```shell
free -h #查询内存
sync   #落盘
echo 3 >/proc/sys/vm/drop_caches #释放缓存
```


