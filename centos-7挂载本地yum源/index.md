# Centos 7挂载本地yum源


# 1、检查yum源

```shell
yum list #查看是否能打开yum源
cat /etc/redhat-release #查看系统版本

```

# 2、上传对于的iso镜像包

`使用winscp上传iso文件到/home/iso/ `目录随便

# 3、创建挂载点并挂载镜像文件

```shell
mkdir /media/cdrom 					#创建挂载点的目录

#备注： -o是参数，loop是把一个文件当成硬盘分区mount挂着到目录
mount -o  loop /home/iso/CentOS-7-x86_64-DVD-1708.iso   /media/cdrom

df -HT			#查看镜像是否挂载成功
```

# 4、修改yum源的配置文件

```shell
cd /etc/yum.repos.d/		
ls
mkdir bak/			
mv ./*.repo  ./bak/
vim  CentOS-local.repo 		#修改配置文件，内容如下
```

```shell
[my]
name=my
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```



# 5、清除缓存

```shell
yum clean all 
yum repolist all #查看yum本地源是否启用
```


