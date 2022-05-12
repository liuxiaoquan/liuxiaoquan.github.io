# 普罗米修斯Prometheus


## 1、prometheus 主服务的下载与部署

### 1.1创建安装目录和下载部署包并解压  [下载x86安装包](/package/prometheus-2.27.1.linux-amd64.tar.gz)

```shell
mkdir -p /data/monitor/prometheus
#下载prometheus包，pakage文件夹中有下载好的包
#wget https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-amd64.tar.gz

#解压prometheus包，并且解压后/data/monitor/prometheus里去除prometheus-2.27.1.linux-amd64文件目录
tar -zxvf prometheus-2.27.1.linux-amd64.tar.gz -C /data/monitor/prometheus --strip-components 1
```

### 1.2配置Prometheus启动文件

```shell
cat > /etc/systemd/system/prometheus.service << EOF
[Unit]
Description=prometheus Service
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=simple
WorkingDirectory=/data/monitor/prometheus
ExecStart=/data/monitor/prometheus/prometheus --config.file=prometheus.yml --web.enable-lifecycle --web.external-url=/prometheus
ExecStop=/usr/bin/curl -X POST http://localhost:9090/prometheus/-/quit
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

**注：可以把启动文件直接放到/etc/systemd/system/目录下**    [下载prometheus启动文件](/启动文件/prometheus.service)

![image-20210721145922404](/image/20210721150157.png)

### 1.3 设置Prometheus开机自启动

```shell
systemctl daemon-reload
systemctl enable prometheus
systemctl start prometheus
```

### 1.4 测试，验证Prometheus是否启动成功：访问 IP:9090/prometheus/

![](/image/image-20210721145210131.png)





## 2、部署 exporter采集节点信息

### 2.1创建安装目录和下载部署包并解压  [下载x86安装包](/package/node_exporter-1.1.2.linux-amd64.tar.gz) 使用 uname -a查询架构，[下载arm安装包](/package/node_exporter-1.1.2.linux-arm64.tar.gz)

**(其他版本下载地址：https://github.com/prometheus/node_exporter/releases)**

```shell
mkdir -p /data/monitor/node_exporter
#wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
tar -zxvf node_exporter-1.1.2.linux-amd64.tar.gz -C /data/monitor/node_exporter --strip-components 1
```

### 2.2配置node_exporter 服务

```shell
cat > /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=node-exporter Service
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=simple
WorkingDirectory=/data/monitor/node_exporter
ExecStart=/data/monitor/node_exporter/node_exporter --web.listen-address=0.0.0.0:9100
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

**注：可以把启动文件直接放到/etc/systemd/system/目录下**     [下载node-exporter启动文件](/启动文件/node-exporter.service)

![image-20210721145922404](/image/20210721150157.png)

### 2.3 设置node_exporter开机自启动

```shell
systemctl daemon-reload
systemctl enable node-exporter
systemctl start node-exporter
```

### 2.4 测试，验证node_exporter是否启动成功：访问 IP:9100

![](/image/20210721150439.png)

### 2.5 把 node_exporter 这个采集程序接⼊配置到prometheus.yml

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
    
  - job_name: 'node-exporter'
    
    scrape_interval: 5s
    static_configs:
    - targets: ['localhost:9100','10.9.0.1:9100']
      labels: 
        group: 'web'
        annotation: '节点信息采集'
```

**导⼊node_exporter 后，重新载⼊配置 :**

```shell
curl -X POST http://localhost:9090/prometheus/-/reload
```





## 3、安装grafana展示

### 3.1 创建安装目录与下载部署包  [下载x86安装包](/package/grafana-7.5.7.linux-amd64.tar.gz)

```shell
mkdir -p /data/monitor/grafana
#wget https://dl.grafana.com/oss/release/grafana-7.5.7.linux-amd64.tar.gz
tar -zxvf grafana-7.5.7.linux-amd64.tar.gz -C /data/monitor/grafana --strip-components 1
```

### 3.2 配置grafana 服务  [下载grafana启动文件](/启动文件/grafana.service)

```shell
cat > /etc/systemd/system/grafana.service <<EOF
[Unit]
Description=grafana Service
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=simple
WorkingDirectory=/data/monitor/grafana
ExecStart=/data/monitor/grafana/bin/grafana-server web
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

### 3.3 设置grafana 开机自启动

```shell
systemctl daemon-reload
systemctl enable grafanasy
stemctl start grafana
```

### 3.4 测试，访问：ip:3000（默认登录账户和密码都是admin，进⼊后界面如下）

![](/image/20210721151249.png)





## 4、监控配置 

### 4.1 查看Prometheus管理后台是否有采集到服务器的信息

![](/image/20210721151516.png)

### 4.2 配置grafana管理后台

![](/image/20210721151856.png)

![](/image/20210721152103.png)

### 4.3 导⼊监控模板，保存。[基础监控大盘官网下载地址](https://grafana.com/grafana/dashboards/8919) 注释：[与官网下载的基础监控大盘一样，选择一个下载就行](/大盘配置/基础监控大盘.json)

![](/image/20210721152332.png)

![](/image/20210721152507.png)

### 4.4查看服务器的基础监控

![](/image/20210721152632.png)

## 5、监控里约网关接⼝调用情况

### 1.添加 ES 数据源

![](/image/20210721152826.png)

![](/image/20210721153059.png)

### 2.导⼊ json 模板（Rio-SG-Access日志大盘-1617280890686.json）[下载日志大盘](/大盘配置/Rio-SG-Access日志大盘-1617280890686.json)

![](/image/20210721153228.png)

![](/image/20210721172156.png)



## **6**、部署部 mysqld_exporter采集节点信息

### 6.1在数据库服务器上执行以下操作在

#### **6.1.1**[官网下载包](https://prometheus.io/download/#mysqld_exporter)    注释：[与官网下载的安装包一样，选一个下载就行](/package/mysqld_exporter-0.13.0.linux-amd64.tar.gz)

![](/image/20210721193725.png)

```shell
mkdir -p /data/monitor/mysqld_exporter
tar -zxvf mysqld_exporter-0.13.0.linux-amd64.tar.gz -C /data/monitor/mysqld_exporter/ --strip-components 1
```

#### **6.1.2**配置.my.cnf

.my.cnf默认放置在启动用户的家目录，启动时无需指定；也可以随意放置在任意目录，在启动时通过 `--config.my-cnf={conf_dir}/.my.cnf`指定配置文件。

```shell
#正常这里没有.my.cnf文件，需要创建.my.cnf文件
vim /data/monitor/mysqld_exporter/.my.cnf
```

```
[client]
host=localhost
user=root
port=3306
password=root # 数据库密码
```

#### 6.1.3启动

```shell
cd /data/monitor/mysqld_exporter/
#创建start.sh脚本
vim start.sh
```

**start.sh脚本内容如下：**

```shell
#!/bin/bash
nohup ./mysqld_exporter --config.my-cnf=/data/monitor/mysqld_exporter/.my.cnf &
```

```shell
chmod 744 start.sh
```

**注册服务**

```shell
#创建启动文件
vim /etc/systemd/system/mysqld_exporter.service
```

```shell
[Unit]
Description=mysqld_exporter service
After=network.target
[Service]
Type=forking
WorkingDirectory=/data/monitor/mysqld_exporter
ExecStart=/data/monitor/mysqld_exporter/start.sh
[Install]
WantedBy=multi-user.target
```

```shell
#设置开机自启
systemctl daemon-reload
systemctl enable mysqld_exporter
stemctl start mysqld_exporter
```



#### 6.1.4 启动后的访问端⼝是 启 9104：访问访 http://数据库服务器 数 ip:9104/metrics

![](/image/20210721200801.png)

### 6.2 把 mysql_exporter 这个采集程序接⼊配置到 这 prometheus.yml 参照下图[参考文件](/prometheus.yml)

![](/image/20210722121326.png)

**导⼊node_exporter 后，重新载⼊配置 :**

```shell
curl -X POST http://localhost:9090/prometheus/-/reload
```

### 6.3配置配 grafana管理后台(这一步可以省略)

![](/image/20210721202540.png)

#### 6.3.1 上传上 mysql监控模板  [下载更多面板链接](https://grafana.com/grafana/dashboards?search=mysql)

![](/image/20210722111032.png)

**注：[下载文mysql大盘](/大盘配置/mysql.json)** 

![](/image/20210722111243.png)

![](/image/20210722111433.png)

## 7、 设置邮件告警

**1、 修改修 grafana 告警配置文件 告 SMTP 邮箱配置修改邮箱相关的配置，例如下面。重启 grafan 系统。 vim /data/monitor/grafana/conf/defaults.ini**

![](/image/20210722154300.png)

### 2、配置通知邮箱 配 在 grafana

![](/image/20210722154604.png)

![](/image/20210722155134.png)

![](/image/20210722155401.png)

**关键字： Template variables are not supported in alert queries** 

**分析： 分 由于Prometheus 告警不支持变量，而模板面板使用了⼤量变量，导致不可使用告警。 解决办法：单独配置个告警的视图，用正则匹配出所有的主机 或者每台主机单独⼀个查询 语句**

```xml
(1 - (node_memory_MemAvailable_bytes{instance="101.133.173.223:9100"} / (node_memory_MemTotal_bytes{instance="101.133.173.223:9100"})))*100
```

![](/image/20210724133545.png)

![](/image/20210724133724.png)

## 8、设置其他告警，演示设置企业微信群告警

**注： PrometheusAlert全家桶是开源的运维告警中心消息转发系统,支持主流的监控系统Prometheus,Zabbix,日志系统Graylog和数据可视化系统Grafana发出的预警消息,支持钉钉,微信,华为云短信,腾讯云短信,腾讯云电话,阿里云短信,阿里云电话等**

[gitee项目地址](https://gitee.com/feiyu563/PrometheusAlert)  [下载包](/package/linux.zip)

```shell
#如果下载不下来，package文件夹中有liunx.zip包
wget https://github.com/feiyu563/PrometheusAlert/releases/download/v4.4.0/linux.zip

mkdir -p /data/monitor/linux_PrometheusAlert

#确保解压后文件在/data/monitor/linux_PrometheusAlert夹中
mv linux.zip /data/monitor/linux_PrometheusAlert/ && cd /data/monitor/linux_PrometheusAlert/ && unzip linux.zip && cd linux/ && mv * ../ && cd ../ && rm -rf linux/

#运行PrometheusAlert
# ./PrometheusAlert (#后台运行请执行 nohup ./PrometheusAlert &)
```

### 8.1启动

```shell
chmod 777 PrometheusAlert

cd /data/monitor/linux_PrometheusAlert
#创建start.sh脚本
vim start.sh
```

**start.sh脚本内容如下：**

```shell
#!/bin/bash
nohup /data/monitor/linux_PrometheusAlert/PrometheusAlert &
```

```shell
chmod 744 start.sh
```

**注册服务**

```shell
#创建启动文件
vim /etc/systemd/system/PrometheusAlert.service
```

```shell
[Unit]
Description=mysqld_exporter service
After=network.target
[Service]
Type=forking
WorkingDirectory=/data/monitor/linux_PrometheusAlert/
ExecStart=/data/monitor/linux_PrometheusAlert/start.sh
[Install]
WantedBy=multi-user.target
```

```shell
#设置开机自启
systemctl daemon-reload
systemctl enable PrometheusAlert
systemctl start PrometheusAlert

#启动后可使用浏览器打开以下地址查看：http://127.0.0.1:8080
#默认登录帐号和密码在conf/app.conf中有配置
```

### 8.2 企业微信告警配置

打开企业微信,进入企业微信群中,选择群设置-->群机器人-->添加，可参下图：

![](/image/wx1.png)

![](/image/wx2.png)

**复制图中的Webhook地址，并填入PrometheusAlert配置文件app.conf中对应配置项即可。**

![](/image/20210724161438.png)

**企业微信机器人目前支持的markdown语法是如下的子集：**

```markdown
标题 （支持1至6级标题，注意#与文字中间要有空格）
# 标题一
## 标题二
### 标题三
#### 标题四
##### 标题五
###### 标题六

加粗
**bold**

链接
[这是一个链接](http://work.weixin.qq.com/api/doc)

行内代码段（暂不支持跨行）
`code`

引用
> 引用文字

字体颜色(只支持3种内置颜色)
<font color="info">绿色</font>
<font color="comment">灰色</font>
<font color="warning">橙红色</font>
```

![](/image/20210724161707.png)

**测试发送机器人**

![](/image/20210724161851.png)

![](/image/2f4ed6fc5111cd51b0ff94066c17ac1.jpg)

#### grafana（内置固定消息模版）接入配置

**首先使用管理员或者具有告警配置权限的帐号登录进Grafana管理页面，登录后进入notification channels配置。**

![](/image/addchannel.png)

![](/image/20210724162256.png)

**注意这里的url地址填写上自己部署所在的url**

```
/grafana/dingding  处理Grafana告警消息转发到钉钉接口，可选参数(ddurl)
/grafana/weixin    处理Grafana告警消息转发到微信接口，可选参数(wxurl)
/grafana/feishu    处理Grafana告警消息转发到飞书接口，可选参数(fsurl)
/grafana/txdx      处理Grafana告警消息转发到腾讯云短信接口，可选参数(phone)
/grafana/txdh      处理Grafana告警消息转发到腾讯云电话接口，可选参数(phone)
/grafana/hwdx      处理Grafana告警消息转发到华为云短信接口，可选参数(phone)
/grafana/bddx      处理Grafana告警消息转发到百度云短信接口，可选参数(phone)
/grafana/alydx     处理Grafana告警消息转发到阿里云短信接口，可选参数(phone)
/grafana/rlydh     处理Grafana告警消息转发到容联云电话接口，可选参数(phone)
/grafana/email     处理Grafana告警消息转发到email接口，可选参数(email)
/grafana/tg        处理Grafana告警消息转发到telegram接口
/grafana/workwechat处理Grafana告警消息转发到企业微信应用接口
```

![](/image/20210724162608.png)

**效果**

![](/image/83d43352f390a1dabde7f936de6016c.jpg)





## 9.参考文档

```
#Prometheus Alert
https://gitee.com/feiyu563/PrometheusAlert#/feiyu563/PrometheusAlert/blob/master/doc/readme/grafana.md

#grafana创建告警
https://grafana.com/docs/grafana/latest/alerting/old-alerting/create-alerts/
```

注释：大盘json文件  [下载](/大盘配置/政务微信大盘.json)

**Prometheus常用查询函数 产考连接:https://www.cnblogs.com/xjzyy/p/15346954.html**

