# 时间同步的问题和解决方法


# 时间同步的问题和解决方法

## 一、虚拟服务器时间同步设置

**使用条件**： 主机已经与授时服务器进行时间同步

配置VMware虚拟服务器的“同步客户机与主机时间”选项，实现虚拟服务器与主机的时间同步。

方法如下：

- 启动VMware vClient 客户端
- 选择虚拟主机，打开编辑设置-->选项---->vmware tools ---->同步客户机时间与主机时间





## 二、物理服务器时间同步设置

### **2.1 ntpdate工具的使用**

**使用条件：** 服务器可访直接问互联网并且支持UDP协议

**在线安装**

```shell
yum install ntpdate -y
```

**离线安装**

```shell
rpm -ivh ntpdate-4.2.6p5-28.el7.centos.x86_64.rpm
```

**注意：** 本文以centos 7.6的安装为基准，在其它版本中安装可能会出现兼容问题，导致不能成功安装。可到http://vault.centos.org/站点选择下载对应版本的安装包。

**手动测试**

```shell
/usr/sbin/ntpdate ntp.ntsc.ac.cn  #后边的域名或IP为授时服务器，ntp.ntsc.ac.cn为国家授时中心域名 
/sbin/hwclock –w 					#写入物理时钟
```

**自动运行，在crontab的最后一行，添加脚本如下：**

```shell
[root@localhost ~]#crontab –e 
#每30分钟执行一次，如果是局域网内，请注意修改ntp.ntsc.ac.cn为对应的内网ntpd服务器IP地址。 
*/30 * * * * /usr/sbin/ntpdate ntp.ntsc.ac.cn ;/sbin/hwclock -w > /dev/null #2>&1
```

**验证crond执行时间同步的情况**

```shell
cat /var/log/cron|grep ntpdate
```



### 2.2 rdate工具的使用

**使用条件：** 服务器在内网区，可访问互联网（可通过接入网关），但只支持TCP协议通信

**在线安装**

```shell
yum install rdate -y
```

**注意：** rdate命令同时支持TCP和UDP通信协议，使用的time协议，但支持time协议的授时服务器不多。



**离线安装**

```shell
rpm -ivh rdate-1.4-25.el7.x86_64.rpm
```

**注意：** 本文以centos 7.6的安装为基准，在其它版本中安装可能会出现兼容问题，导致不能成功安装。可到http://vault.centos.org/站点选择下载对应版本的安装包。

**自动运行**

```shell
[root@localhost ~]#crontab –e 
# 每30分钟执行一次 
*/30 * * * * /usr/bin/rdate -s time.nist.gov ;/sbin/hwclock -w > /dev/null #2>&1
```



## 三、服务器处于隔离网络时方法

**情况1：** 服务器处于隔离网络，但内网有授时服务器，此时可参照第二节的方法

**情况2：** 服务器处于离网络时，无内网授时服务器，这种情况下，我们选择自建ntpd或chrony服务器

### 3.1 ntpd服务端

#### 3.1.1 ntp安装

- 在线安装

```shell
yum install ntp ntpdate -y
```

- 离线安装

```shell
rpm -ivh autogen-libopts-5.18-5.el7.x86_64.rpm ntp-4.2.6p5-28.el7.centos.x86_64.rpm ntpdate-4.2.6p5-28.el7.centos.x86_64.rpm
```

**注意：** 本文以centos 7.6的安装为基准，在其它版本中安装可能会出现兼容问题，导致不能成功安装。可到http://vault.centos.org/站点选择下载对应版本的安装包。

#### 3.1.2 ntpd服务器配置

- 修改配置/etc/ntp.conf文件，（此配置文件请按照实际的来修改，主要修改restrict的配置）

```shell
driftfile /var/lib/ntp/drift 
restrict default kod nomodify notrap nopeer noquery 
restrict 127.0.0.1 
restrict ::1 
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap #对192.168.0.0/16网段授权访问 
restrict 10.0.0.0 mask 255.0.0.0 nomodify notrap #对10.0.0.0/8网段授权访问 
restrict 172.16.1.100 mask 255.255.255.255 nomodify notrap #只对IP为172.16.1.10授权访问 
fudge 127.0.0.1 stratum 10 #设置本地时间级别是10，如果上级时间服务器均失效，对外发布本地时间。
server 127.0.0.1 # 如果公网NTP不可用时，将使用Local时间作为NTP服务提供给NTP Client。 
server ntp.ntsc.ac.cn iburst prefer #国家授时中心 
server 0.centos.pool.ntp.org iburst 
server 1.centos.pool.ntp.org iburst 
server 2.centos.pool.ntp.org iburst 
server 3.centos.pool.ntp.org iburst 

includefile /etc/ntp/crypto/pw 
cdisable monitor
```

- restrict参数

```shell
kod 使用kod技术防范“kiss of death”攻击 
ignore 拒绝任何NTP连接 
nomodify 用户端不能使用ntpc,ntpq修改时间服务器参数，可以进行网络校时 
noquery 用户端不能使用ntpc,ntpq查询时间服务器参数，可以进行网络校时 
notrap 不提供远程日志功能 
notrust 拒绝没有认证的客户端 
restrict ip 或者 restrict IP地址 + mask + 子网掩码 + 参数 例如:restrict default nomodify notrap nopeer noquery #默认拒绝所有访问 只可以同步时间 
restrict 211.71.14.254 mask 255.255.255.0 #添加允许211.71.14.254/24网段访问 
restrict 10.111.1.1 mask 255.0.0.0 nomodify #添加10.0.0.0/8网段访问
```

- server 参数

```shell
server    用于设定ntp同步时间的外网时间服务器 
prefer    默认上级时间服务器 
burst     当一个运程NTP服务器可用时，向它发送一系列的并发包进行检测。 
iburst    当一个运程NTP服务器不可用时，向它发送一系列的并发包进行检测
```

#### 3.1.3 ntpd启动和验证

```shell
systemctl enable ntpd 
systemctl start ntpd
```

- 验证ntpd运行情况

```shell
[root@localhost packages]# netstat -nap|grep ntpd 
udp 	0 	0 192.168.56.101:123 	  	0.0.0.0:* 			16037/ntpd 
udp 	0 	0 127.0.0.1:123 		  	0.0.0.0:* 			16037/ntpd 
udp 	0 	0 0.0.0.0:123 			  	0.0.0.0:* 			16037/ntpd 
udp6 	0 	0 fe80::1c7b:80ff:fe2:123 	:::* 				16037/ntpd 
udp6 	0 	0 ::1:123 				  	:::* 				16037/ntpd 
udp6 	0 	0 fe80::a00:27ff:fe46:123 	:::* 				16037/ntpd 
udp6 	0 	0 :::123 				  	:::* 				16037/ntpd
```

#### 3.1.4 ntpdate客户端安装

请参考第二节的ntpdate工具的使用

-------------------------------------分割线-------------------------------------------

### 3.2 chrony服务端

#### 3.2.1 chrony安装

- 在线安装(chrony需要在所有服务端和客户端中执行安装)

```shell
yum install chrony -y
```

- 离线安装

```shell
rpm -ivh chrony-3.2-2.el7.x86_64.rpm
```

**注意：** 本文以centos 7.6的安装为基准，在其它版本中安装可能会出现兼容问题，导致不能成功安装。可到http://vault.centos.org/站点选择下载对应版本的安装包。

#### 3.2.2 chrony服务器配置

- 修改配置/etc/chrony.conf文件 （带注释的为重要配置）

```shell
server ntp.ntsc.ac.cn iburst #国家授时中心 
server 0.centos.pool.ntp.org iburst 
server 1.centos.pool.ntp.org iburst 
server 2.centos.pool.ntp.org iburst 
server 3.centos.pool.ntp.org iburst 

driftfile /var/lib/chrony/drift 
makestep 1.0 3 
rtcsync 

allow 192.168.0.0/16 #允许被访问的IP段，请根据实际情况修改 
allow 192.168.56.101/24 #允许被访问的IP 

local stratum 10 #设置本地时间级别是10，如果上级时间服务器均失效，对外发布本地时间。 

logdir /var/log/chrony
```

#### 3.2.3 chrony客户端配置

- 修改配置文件/etc/chrony.conf（有注释行为重要配置）

```sh
server 192.168.56.101 iburst #根据实际情况，指向本地授时服务器的IP地址 

driftfile /var/lib/chrony/drift 
makestep 1.0 3 
rtcsync 
logdir /var/log/chrony
```

#### 3.2.4 启动服务

```sh
systemctl enable chronyd.service #开机自启动 
systemctl start chronyd.service #启动chrony 
systemctl status chronyd.service #查看chrony服务状态
```

#### 3.2.5 验证时间同步状态

- MS列中包含^*的行，指明NTP服务当前同步的服务器。当前同步的源为114.118.7.163：

```shell
[root@localhost ~]# chronyc sources 
210 Number of sources = 2 
MS Name/IP address 			Stratum Poll Reach LastRx Last sample =============================================================================== 
^- stratum2-1.ntp.sea03.us.> 	2 	6 	377 	6 	+40ms[ +40ms] +/- 160ms 
^* 114.118.7.163 				2 	6 	77 		16 	-908us[-1190us] +/- 51ms
```

- 查看当前时间是否准确，其中NTP synchronized: yes说明同步成功

```sh
[root@localhost ~]# timedatectl 
		Local time: 四 2019-05-30 13:25:29 CST 
	Universal time: 四 2019-05-30 05:25:29 UTC 
		  RTC time: 三 2019-05-29 11:57:46 
		 Time zone: Asia/Shanghai (CST, +0800) 
	   NTP enabled: yes 
  NTP synchronized: yes 
   RTC in local TZ: no 
        DST active: n/a
```

