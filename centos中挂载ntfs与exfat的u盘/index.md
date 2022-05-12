# Centos中挂载NTFS与exFAT的U盘


# 1、认识挂载：

（1）Linux的宗旨是一切皆文件，从以上我们也看到。我们存储的所有文件都在sda3下存放着，sda3也就是我们的根。那我们要在sda5中写入文件时，首先要sda5要与sda3先建立一个联系，这个联系就是一个目录。建立联系的过程我们叫做**挂载**。 （2）当我们访问sda3底下的这个目录的时候，实际上我们访问的才是sda5这个设备文件。这个目录相当于一个访问sda5的入口，可以理解为一个接口，有了这个接口才可以访问这个磁盘。

# 2、磁盘的挂载：

（1）挂载点目录:我们将磁盘切到根目录， **media** 和 **mnt** 这两个目录被叫做挂载点目录。除此之外，我们也可以自己创建一个目录作为一个挂载点目录， 

![](/mount/v2-2f85d8317145c1adbefdb4be4f98c7e1_r.jpg)

（2）**临时挂载**：将指定的一个目录作为挂载点目录时，如果挂载点的目录有文件，那么文件会被隐藏。因此当我们需要挂载目录时，最好新建一个空文件夹来作为挂在点目录。（**重启后失效**）



# **挂载NTFS过程**

1.因为是虚拟机，所以先到系统的服务目录下寻找【VMware USB Arbitration Service】服务，并确保此服务已经启动。

2.进入Centos 7环境，在mnt目录下创建一个子目录：udisk（用来将U盘挂载到此目录），所需命令为：**mkdir -p /mnt/udisk**。注意，此目录名称可以随意命名，按个人爱好创建。

3.插入U盘

4.运行命令：fdisk -l查看U盘是否已经加载到Centos中，方法为：通过检查没有插入U盘与插入U盘情况下，系统的输出是否一致，如果一致，说明系统没有加载到U盘，否则说明系统加载到U盘。系统加载到U盘后的输出如下图所示：

![](/mount/1017421-20181009110827235-215258733.png)

5.运行命令：其中sdb1就是上个步骤查询出的U盘名称。运行此命令时，一般会报错：`mount:unknown filesystem type ntfs-3g`

```shell
mount -t ntfs-3g /dev/sdb1 /mnt/udisk
```

## 报错原因

Linux 和 Mac OS X 本身不支持读写 NTFS 文件系统（windows系统），大多数人平时也不需要与 NTFS 做数据文件的交互，只是有时候 Windows 用户应急状态下需要使用大容量移动硬盘拷贝数据，必须实现 Linux 下挂载 NTFS，而 Tuxera 恰好为 Linux 和 Mac 用户提供了灰常简单的实现方法

## 介绍

+ 我偷个大懒直接引用官方的原话

NTFS-3G is a stable, full-featured, read-write NTFS driver for Linux, Android, Mac OS X, FreeBSD, NetBSD, OpenSolaris, QNX, Haiku, and other  operating systems. It provides safe handling of the Windows XP, Windows  Server 2003, Windows 2000, Windows Vista, Windows Server 2008, Windows  7, Windows 8 and Windows 10 NTFS file systems. A  [high-performance](http://www.tuxera.com/products/tuxera-ntfs-embedded/performance/)  alternative, called Tuxera NTFS is available for  [embedded devices](http://www.tuxera.com/products/tuxera-ntfs-embedded/)  and  [Mac OS X](http://www.tuxera.com/products/tuxera-ntfs-for-mac/).

The release notes and the software changes can be found on the  [Release History](http://www.tuxera.com/community/release-history/)  page. Subscribe  [here](http://lists.sourceforge.net/mailman/listinfo/ntfs-3g-news)  for new release notifications.

## Open Source: NTFS-3G

```shell
#Installation
tar zxvf /tmp/ntfs-3g_ntfsprogs-2016.2.22.tgz
cd ntfs*
#yum -y install gcc gcc-c++ make
./configure
make
make install # or 'sudo make install' if you aren't root

#Usage
fdisk -l
mkdir -p /mnt/windows
mount -t ntfs-3g /dev/sda1 /mnt/windows
#You can also make NTFS to be mounted during boot by adding the following line to the end of the /etc/fstab file:
vi /etc/fstab
/dev/sda1 /mnt/windows ntfs-3g defaults 0 0
```

## 卸载U盘，请使用命令：

```shell
umount /mnt/udisk
```



# **挂载exFAT过程**

下载fuse-exfat模块：https://github.com/relan/exfat/archive/refs/heads/master.zip

```shell
#编译前，请先检查系统，如果系统中没有scons和gcc，请通过yum安装 ，这个软件是fuse模块，编译需要fuse-devel包支持
yum install -y scons gcc autoconf automake pkg-config fuse-devel make

unzip master.zip

cd exfat-master

autoreconf --install

./configure

make && make install 

##使用方法
mount.exfat /dev/spec /mnt/exfat
```


