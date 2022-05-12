# Docker版本升级


### 升级背景

```
漏洞描述：
Docker是一款开源的应用容器引擎。该产品支持在Linux系统上创建一个容器（轻量级虚拟机）并部署和运行应用程序，以及通过配置文件实现应用程序的自动化安装、部署和升级。 runc是一个可以用于创建和运行容器的CLI(command-line interface)工具。 Docker 18.09.2之前版本存在安全漏洞，运行容器时程序没有正确地处理文件描述符。攻击者可利用该漏洞通过以下情况覆盖主机runc二进制文件并以root权限执行任意命令：（1）使用攻击者控制的镜像创建新容器；（2）攻击者使用docker exec方式连接容器。
漏洞类型：
系统组件漏洞
威胁等级：
高危
修复方案：
请到官网升级Docker到18.09.2以上最新版本修复该漏洞，漏洞修复后需要服务重启。建议业务不繁忙时修复。
下载更新：https://docs.docker.com/install/
检测到服务器存在漏洞风险，建议立即对相关主机进行快照备份，避免遭受损失。
```

### 升级步骤

> 1、以下步骤为单个节点的升级，由于mongodb es redis等服务为集群，为保证数据及服务高可用，建议逐个节点升级，切勿同时升级。
>
> 2、如果某个容器启动失败且无法重启时，可以通过重建容器恢复，init.sh脚本路径如下：
>
> ```shell
> [root@localhost data]# find /data/ -name init.sh
> /data/mongo/init.sh
> /data/redis_sentinel/init.sh
> /data/redis_master/init.sh
> /data/elasticsearch/init.sh
> ```

```shell
# 下载新版本并解压 解压会在当前目录生成一个docker的文件夹
cd /root
wget https://download.docker.com/linux/static/stable/x86_64/docker-18.09.9.tgz
tar zxvf docker-18.09.9.tgz

# 停止docker服务
systemctl stop docker
# 备份原二进制文件
mkdir /root/docker-bak-$(date +%F) && mv /usr/bin/docker* /root/docker-bak-$(date +%F)

# 替换新二进制文件
cp -a docker/* /usr/bin/

# 启动docker服务
systemctl start docker
# 检查docker版本
docker version
# 容器状态
dokcer ps
docker ps -a
```

### 验证

> 1、2步骤在单个节点升级完成时验证，步骤3在所有节点升级完成后验证

```shell
# 1、检查所有里约容器组件端口可用
netstat -lntp |grep ${port}


# 2、检查相关服务状态集群状态
# es集群装填为green、节点数正常,使用实际账号、端口、密码等信息
curl -u rio:ee06167b10a177f60766d35baa81955d localhost:9200/_cluster/health?pretty
# mongodb rs状态,使用实际账号、端口、密码等信息
docker exec -it mongo mongo --username rio --password ee06167b10a177f60766d35baa81955d --authenticationDatabase admin 127.0.0.1:27017/admin
rs.status()
# redis 状态,使用实际账号、端口、密码等信息
docker exec -ti redis_master redis-cli -h 127.0.0.1 -p 6379 -a ee06167b10a177f60766d35baa81955d info

# 3、控制台验证 登录、服务发布、站点发布等功能
```

### 回退步骤

> 如升级后异常容器状态异常切无法重建容器进行回退，回退验证后同上。

```shell
cd /root/
# 停止docker服务
systemctl stop docker
# 删除二进制文件
for i in $(ls ./docker/); do rm -f /usr/bin/$i; done

# 还原原二进制文件
cp -a /root/docker-bak-$(date +%F)/docker*  /usr/bin/

# 启动docker服务
systemctl start docker
# 检查docker版本
docker version
# 容器状态
dokcer ps
docker ps -a
```


