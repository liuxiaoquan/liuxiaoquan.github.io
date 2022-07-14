# Centos 7下配置iptables


# 1、安装iptables

```shell
systemctl status iptables #查看是否安装iptables
```

![](/image/20180402093142757.png)

{{< admonition warning >}}
输出结果表示没有iptables的相关服务，我们需要安装
{{< /admonition >}}

```sh
yum install iptables-services #安装
systemctl status iptables 
systemctl start iptables
systemctl enable iptables.service #设置开机启动
systemctl disable iptables.service #禁止开机启动
```

{{< admonition warning >}}
这里需要关闭SELINUX，因为当SELINUX不关闭时，iptables不读取配置文件 `vi /etc/selinux/config`

![](/image/20180402094955791.png)

`重新加载配置 source /etc/selinux/config `

{{< /admonition >}}

{{< admonition danger >}}
第一次创建规则要使用-F 清空，否者会出现很多奇怪的问题！
{{< /admonition >}}

## 永久插入规则

``vim /etc/sysconfig/iptables``

```

# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
#-A INPUT -p icmp -j ACCEPT ---注释这几行
#-A INPUT -i lo -j ACCEPT---注释这几行
#-A INPUT -p tcp --dport 22 -j ACCEPT---注释这几行
#-A INPUT -j REJECT --reject-with icmp-host-prohibited---注释这几行
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited---注释这几行
-A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 10 -j REJECT
-A INPUT -p tcp --dport 8080 -m connlimit --connlimit-above 10 -j REJECT
-A INPUT -p tcp --dport 28080 -m connlimit --connlimit-above 10 -j REJECT
-A INPUT -p tcp --dport 20080 -m connlimit --connlimit-above 10 -j REJECT

-A INPUT -i eth0 -p tcp -s 45.61.185.76 -j DROP
-A INPUT -i eth0 -p tcp -s 185.7.214.104 -j DROP

COMMIT
```

``执行这个命令，友好一点，返回结果第一列，外部IP的连接数，第二列为外部连接IP
netstat -na|grep ESTABLISHED|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -r``

# 2、配置规则

## 禁止访问9200端口

```sh
iptables -I INPUT -p tcp --dport 9200 -j DROP
```

## 允许ip为192.168.1.1的机器访问

```shell
iptables -I INPUT -p tcp -s 192.168.1.1 --dport 9200 -j ACCEPT
#插入到第10行
iptables -I INPUT 10 -p tcp -s 192.168.1.1 --dport 9200 -j ACCEPT
```

## 允许ip在192.168网段的机器访问

```sh
iptables -I INPUT -p tcp -s 192.168.1.1/16 --dport 9200 -j ACCEPT
```

## 允许所有ip访问

```sh
iptables -I INPUT -p tcp --dport 9200 -j ACCEPT
```

## 查看规则列表

```sh
iptables -nvL --line-number
```

## 删除规则

```sh
#删除第10行规则
iptables -D INPUT 10
```

## 清空所有规则

```sh
iptables -F
```

【列子】

```sh
iptables -I INPUT -p tcp -s 192.168.1.1 --dport 9200 -j ACCEPT
iptables -I INPUT -p tcp -s 192.168.1.2 --dport 9200 -j ACCEPT
iptables -I INPUT -p tcp -s 192.168.1.3 --dport 9200 -j ACCEPT
iptables -I INPUT -p tcp --dport 9200 -j DROP
```

说明：首先放开192.168.1.1至192.168.1.3ip段的9200端口，然后再禁用掉本机的9200端口

## [9个常用iptables配置实例]

### **1.删除已有规则**

在新设定iptables规则时，我们一般先确保旧规则被清除，用以下命令清除旧规则：

```shell
iptables -F
(or iptables --flush)
```



### **2.设置chain策略**

对于filter table，默认的chain策略为ACCEPT，我们可以通过以下命令修改chain的策略：

```shell
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP
```

以上命令配置将接收、转发和发出包均丢弃，施行比较严格的包管理。由于接收和发包均被设置为丢弃，当进一步配置其他规则的时候，需要注意针对INPUT和OUTPUT分别配置。当然，如果信任本机器往外发包，以上第三条规则可不必配置。

 

### **3.屏蔽指定ip**

有时候我们发现某个ip不停的往服务器发包，这时我们可以使用以下命令，将指定ip发来的包丢弃：

```shell
BLOCK_THIS_IP="x.x.x.x"
iptables -A INPUT -i eth0 -p tcp -s "$BLOCK_THIS_IP" -j DROP
```

以上命令设置将由x.x.x.x ip发往eth0网口的tcp包丢弃。

 

### **4.配置服务项**

利用iptables，我们可以对日常用到的服务项进行安全管理，比如设定只能通过指定网段、由指定网口通过SSH连接本机：

```shell
iptables -A INPUT -i eth0 -p tcp -s 192.168.100.0/24 --dport 22 -m state --state NEW,ESTABLESHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

若要支持由本机通过SSH连接其他机器，由于在本机端口建立连接，因而还需要设置以下规则：

```shell
iptables -A INPUT -i eth0 -p tcp -s 192.168.100.0/24 --dport 22 -m state --state ESTABLESHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
```

类似的，对于HTTP/HTTPS(80/443)、pop3(110)、rsync(873)、MySQL(3306)等基于tcp连接的服务，也可以参照上述命令配置。

 

对于基于udp的dns服务，使用以下命令开启端口服务：

```shell
iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPT
iptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```

### **5.网口转发配置**

对于用作防火墙或网关的服务器，一个网口连接到公网，其他网口的包转发到该网口实现内网向公网通信，假设eth0连接内网，eth1连接公网，配置规则如下：

```shell
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

 

### **6.端口转发配置**

对于端口，我们也可以运用iptables完成转发配置：

```shell
iptables -t nat -A PREROUTING -p tcp -d 192.168.102.37 --dport 422 -j DNAT --to 192.168.102.37:22
```

以上命令将422端口的包转发到22端口，因而通过422端口也可进行SSH连接，当然对于422端口，我们也需要像以上“4.配置服务项”一节一样，配置其支持连接建立的规则。

 

### **7.DoS攻击防范**

利用扩展模块limit，我们还可以配置iptables规则，实现DoS攻击防范：

```shell
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```

--litmit 25/minute 指示每分钟限制最大连接数为25

--litmit-burst 100 指示当总连接数超过100时，启动 litmit/minute 限制

```shell
#在80端口同一个ip并发量超过10就拒绝
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 10 -j REJECT
```



### **8.配置web流量均衡**

我们可以将一台服务器作为前端服务器，利用iptables进行流量分发，配置方法如下：

```shell
iptables -A PREROUTING -i eth0 -p tcp --dport 80 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.101:80
iptables -A PREROUTING -i eth0 -p tcp --dport 80 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.102:80
iptables -A PREROUTING -i eth0 -p tcp --dport 80 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.103:80
```

以上配置规则用到nth扩展模块，将80端口的流量均衡到三台服务器。

 

### **9.将丢弃包情况记入日志**

使用LOG目标和syslog服务，我们可以记录某协议某端口下的收发包情况。拿记录丢包情况举例，可以通过以下方式实现。

首先自定义一个chain：

```shell
iptables -N LOGGING
```

其次将所有接收包导入LOGGING chain中：

```shell
iptables -A INPUT -j LOGGING
```

然后设置日志前缀、日志级别：

```shell
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables Packet Dropped: " --log-level 7
```

最后将包倒向DROP，将包丢弃：

```shell
iptables -A LOGGING -j DROP
```

另可以配置syslog.conf文件，指定iptables的日志输出。

