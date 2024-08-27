# kibana-6.8.4 arm环境下编辑安装


- 环境：centos7    aarch64架构

## 安装依赖

```shell
yum install net-tools passwd  java java-devel vim wget -y
```

## 解决方法如下

wget的链接需要魔法才能下载，在这提供一个下载好的部署包  

 [node-v10.15.2-linux-arm64.tar.gz](/package/node-v10.15.2-linux-arm64.tar.gz)

```shell
#下载kibana安装包
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.8.4-linux-x86_64.tar.gz

#解压
tar -xf kibana-6.8.4-linux-x86_64.tar.gz
 
#查看node
[root@b156873121b1 node]# cd /opt/kibana-6.8.4-linux-x86_64/node
[root@b156873121b1 node]# file ./bin/node
./bin/node: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.18, BuildID[sha1]=f135ef59856912584f7b9668f5ff750135716af3, with debug_info, not stripped

#删除node
[root@b156873121b1 kibana-6.8.4-linux-x86_64]# rm -rf node

#下载arm架构的node
[root@b156873121b1 kibana-6.8.4-linux-x86_64]$ wget https://nodejs.org/dist/v10.15.2/node-v10.15.2-linux-arm64.tar.gz
[root@b156873121b1 kibana-6.8.4-linux-x86_64]$ tar -xf node-v10.15.2-linux-arm64.tar.gz 
[root@b156873121b1 kibana-6.8.4-linux-x86_64]$ mv node-v10.15.2-linux-arm64 node

-----------------------------------------------------------------------
#修改kibana配置文件,vim config/kibana.yml

server.port: 5601
server.host: "0.0.0.0"
# 这里要配置kibana的基础路径
server.basePath: "/kibana"
elasticsearch.hosts: ["http://10.21.232.131:9200"]
kibana.index: ".kibana"
elasticsearch.username: "user"
elasticsearch.password: "passworod"
i18n.locale: "zh-CN"
-----------------------------------------------------------------------

#创建账号
#kibana默认禁止root启动，需要创建账号
useradd test
passwd test
usermod -G test:test
chown -R test:test /opt/kibana-6.8.4-linux-x86_64

[root@b156873121b1 kibana-6.8.4-linux-x86_64]# su test
[test@b156873121b1 kibana-6.8.4-linux-x86_64]$ 
[test@b156873121b1 kibana-6.8.4-linux-x86_64]$ nohub ./bin/kibana &
[root@b156873121b1 /]# netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN      398/./bin/../node/b 
tcp        0      0 127.0.0.1:9200          0.0.0.0:*               LISTEN      155/java            
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      155/java  

```


