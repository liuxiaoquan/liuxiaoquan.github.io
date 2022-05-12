# CentOS升级sudo版本 解决Sudo权限绕过漏洞 修复堆缓冲区溢出漏洞(CVE-2021-3156)


## 漏洞介绍
关于 Sudo 堆缓冲区溢出漏洞(CVE-2021- 3156)的预警通知
    
有关情报显示，sudo 存在缓冲区溢出漏洞。攻击者在取得服 务器基础权限的情况下，可以利用 sudo 基于堆的缓冲区溢出漏 洞，获得 root 权限。在 sudo 解析命令行参数的方式中发现了基 于堆的缓冲区溢出。任何本地用户(普通用户和系统用户，sudo er 和非 sudoers)都可以利用此漏洞，而无需进行身份验证，攻 击者不需要知道用户的密码。
    
漏洞编号:CVE-2021-3156。

一、漏洞情况分析

    Sudo 是一种程序，用于类 Unix 操作系统如 BSD，Mac OS X，以及 GNU/Linux 以允许用户透过安全的方式使用特殊的权限 运行程序。

二、 漏洞影响范围

    影响版本:
    1.8.2 - 1.8.31p2
    1.9.0 - 1.9.5p1

## 升级环境

    系统：CentOS6/CentOS7
    sudo：1.8.23 查看本地sudo版本命令：sudo -V
    最新版本：sudo-1.9.5p2 查看官方最新版本文件：https://www.sudo.ws/dist/

## 准备

* 查看本地sudo版本，查看官方最新版本文件：https://www.sudo.ws/dist/

```shell
sudo -V
```

+ 若已经安装gcc,可以忽略此操作

```shell
yum -y install gcc  
```

+ 若已经安装wget,可以忽略此操作

```shell
yum -y install wget   
```

## 安装

+ 切换到需要下载软件的目录
```shell
cd /opt/jovtec/soft/
```

+ 下载最新版本到服务器解压
```shell
wget https://www.sudo.ws/dist/sudo-1.9.5p2.tar.gz && tar zxf sudo-1.9.5p2.tar.gz
```
+ 执行配置命令
```shell
cd sudo-1.9.5p2 && ./configure --prefix=/usr --libexecdir=/usr/lib --with-secure-path --with-all-insults --with-env-editor --docdir=/usr/share/doc/sudo-1.9.5p2 --with-passprompt="[sudo] password for %p: "
```
+ 编译安装
```shell
make && make install && ln -sfv libsudo_util.so.0.0.0 /usr/lib/sudo/libsudo_util.so.0
```
+ 检验
```shell
sudo -V   
```

```
以非 root 用户登录系统，并使用命令 sudoedit -s /
- 如果响应一个以 sudoedit: 开头的报错，那么表明存在漏洞。
- 如果响应一个以 usage: 开头的报错，那么表明补丁已经生效。
```


