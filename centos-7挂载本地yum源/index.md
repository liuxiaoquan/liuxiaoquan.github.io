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
[root@Server ~]# mkdir /media/cdrom 					#创建挂载点的目录
[root@Server ~]#  mount -o  loop /home/iso/CentOS-7-x86_64-DVD-1708.iso   /media/cdrom
[root@Server ~]# df -HT			#查看镜像是否挂载成功
备注： -o是参数，loop是把一个文件当成硬盘分区mount挂着到目录
```

# 4、修改yum源的配置文件

```shell
[root@Server ~]# cd /etc/yum.repos.d/		
[root@Server yum.repos.d]# ls
[root@Server yum.repos.d]# mkdir bak/			
[root@Server yum.repos.d]# mv ./*.repo  ./bak/
[root@Server yum.repos.d]# vim  CentOS-local.repo 		#修改配置文件，内容如下
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
[root@Server yum.repos.d]# yum clean all 
[root@Server yum.repos.d]# yum repolist all #查看yum本地源是否启用
```


