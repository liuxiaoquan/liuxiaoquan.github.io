# Nslookup命令详解

`nslookup命令用于**查询DNS的记录，查看域名解析是否正常，在网络故障的时候用来诊断网络问题。** nslookup的用法相对来说还是蛮简单的，主要是下面的几个用法。`

## **1、直接查询**

`这个可能大家用到最多，查询一个域名的A记录。`

```shell
nslookup domain [dns-server]
```

`如果没指定dns-server，用系统默认的dns服务器。下面是一个例子：`

```shell
[root@localhost ~]# nslookup baidu.com
Server:     10.30.7.177
Address:    10.30.7.177#53

Non-authoritative answer:
Name:   baidu.com
Address: 123.125.114.144
Name:   baidu.com
Address: 111.13.101.208
Name:   baidu.com
Address: 180.149.132.47
Name:   baidu.com
Address: 220.181.57.217
```

## **2、查询其他记录**

`直接查询返回的是A记录，我们可以指定参数，查询其他记录，比如AAAA、MX等。`

```shell
nslookup -qt=type domain [dns-server]
```

```
其中，type可以是以下这些类型：

A 	地址记录
AAAA 	地址记录
AFSDB 	Andrew文件系统数据库服务器记录
ATMA 	ATM地址记录
CNAME 	别名记录
HINFO 	硬件配置记录，包括CPU、操作系统信息
ISDN 	域名对应的ISDN号码
MB 	存放指定邮箱的服务器
MG 	邮件组记录
MINFO 	邮件组和邮箱的信息记录
MR 	改名的邮箱记录
MX 	邮件服务器记录
NS 	名字服务器记录
PTR 	反向记录
RP 	负责人记录
RT 	路由穿透记录
SRV 	TCP服务器信息记录
TXT 	域名对应的文本信息
X25 	域名对应的X.25地址记录
```

例如：

```shell
[root@localhost ~]# nslookup -qt=mx baidu.com 8.8.8.8
*** Invalid option: qt=mx
Server:     8.8.8.8
Address:    8.8.8.8#53

Non-authoritative answer:
Name:   baidu.com
Address: 111.13.101.208
Name:   baidu.com
Address: 123.125.114.144
Name:   baidu.com
Address: 180.149.132.47
Name:   baidu.com
Address: 220.181.57.217
```

### **3、查询更具体的信息**

`查询语法：`

```shell
nslookup –d [其他参数] domain [dns-server]  #只要在查询的时候，加上-d参数，即可查询域名的缓存。
```


