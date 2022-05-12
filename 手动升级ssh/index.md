# 手动升级ssh

# openssl、openssh下载地址

https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/  （ssh）
https://ftp.openssl.org/source/                                         (ssl)

# 配置yum源

```shell
mount /dev/sdd /mnt
vim /etc/yum.repos.d/local.repo     #编辑
```

```shell
[my]
name=my
baseurl=file:///mnt
enabled=1
gpgcheck=0
```

```shell
yum list  #检查配置是否成功
```

```shell
[centos7.7]
baseurl = http://10.2.11.11:80/tstack/tstack-repos/repo/centos7.7/
enabled = 1
gpgcheck = 0
name = tstack YUM repo
```



# 安装相关软件包

```shell
yum install -y zlib* pam* krb5* openssl openssl-devel make perl-Test-Simple gcc-c++ libtool

setenforce 0  #关掉seliunx
```



# 安装ssl

```shell
#备份
mv /usr/bin/openssl /usr/bin/openssl_bak
mv /usr/include/openssl /usr/include/openssl_bak

cd /tmp/openssh 					#上传ssl包到/tmp/openssh下（自建建目录）
tar zxvf openssl-1.0.2s.tar.gz      #解压
cd openssl-1.0.2s					#切换目录
./config --prefix=/usr --openssldir=/etc/ssl --shared zlib  #检查环境
make && make test  					
echo $?  (0正常 1不正常)
make install
openssl version -a
```

```
-------------------------------------------
问题：如果升级后发现头文件和库文件匹配不上【OpenSSL 1.1.1k  25 Mar 2021（Library OpenSSL 1.1.1h  25 Mar 2020）】
解决：重新检查环境，并重新执行make & make install
./config --prefix=/usr/local/openssl --openssldir=/etc/ssl --shared zlib
--------------------------------------------
```



# 升级ssh

```shell
cp -rp /etc/ssh /etc/ssh_$(date +%Y%M%d%H)

cd /tmp/openssh

tar zxvf openssh-8.1p1.tar.gz

cd openssh-8.1p1

./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords -with-kerberos5 --with-pam --with-zlib --with-ssl-dir=/usr/

make && make  install

chmod 600 /etc/ssh/ssh_host*_key

sed -i "s/Type=${SSH_START_TYPE}/Type=simple/" /usr/lib/systemd/system/sshd.service

systemctl daemon-reload

systemctl restart sshd.service

ssh -V
```


