# Iptables实战


注意:配置拒接策略时请配置好SSH连接端口，否则一旦配置失误，将无法再远程连接！！！！！

安装iptables服务

```bash
yum install iptables iptables-services -y
```

启动iptables服务
```bash
service iptables start
```

设置iptables开机自启动
```bash
systemctl enable iptables
```

清除iptables所有规则
```bash
iptables -F
```

添加规则，放开需要放开的端口(这里放开了tcp80端口)
```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

添加规则，允许特定主机访问（这里允许1.1.1.1访问）
```bash
iptables -A INPUT -s 1.1.1.1 -j ACCEPT
```

添加规则，允许特定网段访问（这里允许1.1.1.0/24访问）
```bash
iptables -A INPUT -s 1.1.1.0/24 -j ACCEPT
```

添加规则，拒绝特定网段访问（这里拒绝1.1.1.0/24访问）
```bash
iptables -A INPUT -s 1.1.1.0/24 -j DROP
```

添加规则，允许特定主机访问特定端口（这里允许1.1.1.1访问tcp22端口）
```bash
iptables -A INPUT -s 1.1.1.1 -p tcp --dport 22 -j ACCEPT
```

查看规则及规则号
```bash
iptables -nL --line-number
```

删除规则5
```bash
iptables -D INPUT 5
```

插入规则，插入到规则1之前
```bash
iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
```

添加完规则后保存
```bash
service iptables save
#如果没有service可以用以下命令
sudo iptables-save > /etc/sysconfig/iptables
```

重启iptables服务
```bash
service iptables restart
#或者：systemctl restart iptabels
```

#案例：允许192.2.0.2 和 198.3.22.1的访问，其他ip禁止访问
第一步：放开所有 IP 地址的 SSH 连接，一定要先执行这个，不然远程连接会断开
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT 
```
第二步：允许192.2.0.2 和 198.3.22.1的访问
```bash
iptables -A INPUT -s 192.2.0.2 -j ACCEPT
iptables -A INPUT -s 198.3.22.1 -j ACCEPT
```
第三步：禁止外部访问本机
```bash
#禁止INPUT链处理进入本地主机的数据包
sudo iptables -P INPUT DROP  
#禁止本地转发数据包出去，这个可以不用设在
#sudo iptables -P FORWARD DROP  

#放通本地访问外部数据，这个默认开启也不用设置
#sudo iptables -P OUTPUT ACCEPT
```
第四步：保存
```bash
sudo iptables-save > /etc/sysconfig/iptables
```
