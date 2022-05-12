# Crontab的应用


```shell
crontab -l 
crontab -r 
crontab -e 
```

```
-e : 执行文字编辑器来设定时程表，内定的文字编辑器是 VI，如果你想用别的文字编辑器，则请先设定 VISUAL 环境变数来指定使用那个文字编辑器(比如说 setenv VISUAL joe)
-r : 删除目前的时程表
-l : 列出目前的时程表
```

## 列子

```shell
0 */2 * * * /sbin/service httpd restart  意思是每两个小时重启一次apache 

50 7 * * * /sbin/service sshd start  意思是每天7：50开启ssh服务 

50 22 * * * /sbin/service sshd stop  意思是每天22：50关闭ssh服务 

0 0 1,15 * * fsck /home  每月1号和15号检查/home 磁盘 

1 * * * * /home/bruce/backup  每小时的第一分执行 /home/bruce/backup这个文件 

00 03 * * 1-5 find /home "*.xxx" -mtime +4 -exec rm {} \;  每周一至周五3点钟，在目录/home中，查找文件名为*.xxx的文件，并删除4天前的文件。

30 6 */10 * * ls  意思是每月的1、11、21、31日是的6：30执行一次ls命令
```

## 例子2

```shell
vim /root/restart_v2ray.sh
```

```shell
#!/bin/bash

v2ray restart
```

```shell
crontab -e
```

```shell
#每5个小时重启一次
0 */5 * * *  . /etc/profile;/bin/sh /root/restart_v2ray.sh > /dev/null 2>&1

#注意：当程序在你所指定的时间执行后，系统会发一封邮件给当前的用户，显示该程序执行的内容，若是你不希望收到这样的邮件，请在每一行空一格之后加上 > /dev/null 2>&1 即可
#注意一定要添加“ . /etc/profile; ，这句用于将环境变量include进当前脚本的执行环境！
```

## 例子3

```shell
*/3 * * * * if [ $(netstat -tnpl |grep 8683  | wc -l) -ne 1 ];then systemctl start test.service;fi
```

## 查看日志

```shell
tailf /var/log/cron
```


