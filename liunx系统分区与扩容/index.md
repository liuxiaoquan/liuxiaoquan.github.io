# Liunx系统分区与扩容


## 1、挂载目录

注释：xfs和ext4是Linux操作系统中常见的两种文件系统

```bash
#格式化文件系统
mkfs.ext4 /dev/sdb

#查看文件系统
blkid /dev/sdb

#挂载分区到目录
vim /etc/fstab 
#把这一段写入fstab文件：/dev/sdb /home ext4 defaults 0 0
mount -a
```



## 2、新磁盘创建卷组与逻辑卷并且挂载/home目录

**只有lvm(逻辑卷才能扩容)，下图意思是：有9块物理硬盘，其中有6块硬盘组成一个卷组，然后这个卷组了20g给逻辑卷用：物理卷（PV）、卷组（VG）和逻辑卷（LV）**

<img src="/img/分区.jpg" style="zoom:50%;" />

### pvcreate 创建物理卷

```bash
#首先是创建PV，上图中的9块硬盘，centos7以上系统直接可以省略这步骤 pvcreate /dev/sdb  /dev/sdc1

[root@localhost ~]# pvcreate  /dev/sdb /dev/sdc 				#直接将磁盘或某个分区转化为物理卷，
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
[root@localhost ~]# pvs											#查看有哪些物理卷
  PV         VG Fmt  Attr PSize  PFree
  /dev/sda2  cl lvm2 a--  11.00g 4.00m
  /dev/sdb      lvm2 ---   8.00g 8.00g
  /dev/sdc      lvm2 ---   6.00g 6.00g
```

### vgcreate 创建卷组

```bash
#创建卷组（虚拟硬盘）:vgcreate 卷组名 物理卷1 物理卷2 ..... 
[root@localhost ~]# vgcreate my_data /dev/sdb /dev/sdc	#创建一个名为my_data的卷组并将sdb和sdc物理卷加入其中，PE的大小默认为4M
[root@localhost ~]# vgs	#查看卷组，下面有一个创建好的my_data卷组为1G
  VG      #PV #LV #SN Attr   VSize    VFree   
  centos    1   2   0 wz--n-   <9.00g    4.00m
  my_data   1   0   0 wz--n- 1020.00m 1020.00m 
```

### lvcreate 创建逻辑卷

```bash
[root@localhost ~]# lvcreate -n lv_data -L 500M  my_data		#在my_data卷组上创建一个名称叫lv_data的500M的逻辑卷
  Logical volume "lv_data" created.
[root@localhost ~]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    centos  -wi-ao----   7.99g                                                    
  swap    centos  -wi-ao----   1.00g                                                    
  lv_data my_data -wi-a----- 500.00m  
```

**重点：**
当我们lvcreate创建一个逻辑卷的时候，其实相当于生成了一个磁盘设备文件，这是由lvm的mapper机制决定，Linux会新建两个软链接文件，如`/dev/vg_name/lv_name、/dev/mapper/vg_name-lv_name`，而这2个文件都是指向`/dev/dm-X` 块设备文件的，所以，当我们使用`df -h`看到的`/dev/mapper/vg_name-lv_name`的时候，这个其实就是和`/dev/vg_name/lv_name`一样的，都是指向`/dev/dm-X` 块设备文件的。

### mkfs 格式化分区并创建文件系统

```bash
[root@localhost ~]# mkfs.ext4 /dev/sdb  #格式化磁盘分区为ext4文件系统，这一步也可以忽略
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdb is entire device, not just one partition!

#对上面刚才创建的lv进行格式化并创建ext4类型的文件系统
[root@localhost ~]# mkfs.ext4 /dev/my_data/lv_data  

#挂载分区到目录
vim /etc/fstab 
#把这一段写入fstab文件：/dev/my_data/lv_data /home ext4 defaults 0 0
mount -a

df -hT
```

## 3、Linux系统扩容Lvm磁盘空间

**扩容步骤为：先扩容卷组 -> 再扩容逻辑卷 ->刷新文件系统**

### vgextend 扩容卷组，即把物理卷加入卷组

```bash
[root@localhost ~]# vgs       #还剩下520M的容量
  VG      #PV #LV #SN Attr   VSize    VFree  
  centos    1   2   0 wz--n-   <9.00g   4.00m
  my_data   1   1   0 wz--n- 1020.00m 520.00m
  
#vgextend 命令把sdd1物理卷加入my_data卷组（sdd1已经是物理卷了，my_data是卷组名称）
[root@localhost /]# vgextend my_data /dev/sdd1	
```

### lvextend扩容逻辑卷，即把卷组空间加入到逻辑卷中

```bash
#/dev/my_data/lv_data这个逻辑卷增加520M
[root@localhost ~]# lvextend -L +520M /dev/my_data/lv_data  
	Size of logical volume my_data/lv_data changed from 500.00 MiB (125 extents) to 1020.00 MiB (255 extents).
  Logical volume my_data/lv_data successfully resized.

#这个时候发现lv_data这个逻辑卷已经扩大到1G
[root@c2 ~]# lvs 
  LV      VG      Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    centos  -wi-ao----    7.99g                                                    
  swap    centos  -wi-ao----    1.00g                                                    
  lv_data my_data -wi-ao---- 1020.00m  

#但是df查看容量没有加载，这个时候需要刷新才行
[root@c2 ~]# df -hT /home/ 
文件系统                    类型  容量  已用  可用 已用% 挂载点
/dev/mapper/my_data-lv_data ext4  477M  2.3M  445M    1% /home

#查看你的逻辑卷是什么文件系统类型，如果是xfs文件系统，使用xfs_growfs命令扩展容量：
xfs_growfs /dev/mysql/lv_data
#查看你的逻辑卷是什么文件系统类型，如果是ext4文件系统，使用resize2fs命令扩展容量：
resize2fs /dev/root_vg/root
```

## 4、扩容根分区

**注释：首先确定根分区是不是lvm卷，如果不是那就不允许扩容**

**扩容步骤为：先扩容卷组 -> 再扩容逻辑卷 ->刷新文件系统**

```shell
#查询卷组，查看卷组名为centos的卷组是否有容量扩容，下面显示vFree为4.00M,显然已经不够扩容
[root@c2 ~]# vgs   
  VG      #PV #LV #SN Attr   VSize    VFree
  centos    1   2   0 wz--n-   <9.00g 4.00m
#查询是否有分区给卷组扩容
[root@c2 ~]# lsblk 

#vgextend 命令把sdd物理卷加入centos卷组（sdd已经是物理卷了，centos是卷组名称）
[root@localhost /]# vgextend centos /dev/sdd

#/dev/centos/root这个逻辑卷增加50G
[root@localhost ~]# lvextend -L +50G /dev/centos/root  

#但是df查看容量没有加载，这个时候需要刷新才行
[root@c2 ~]# df -hT / 
文件系统                类型  容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root ext4  7.8G  6.9G  483M   94% /

#查看你的逻辑卷是什么文件系统类型，如果是xfs文件系统，使用xfs_growfs命令扩展容量：
xfs_growfs /dev/mysql/lv_data
#查看你的逻辑卷是什么文件系统类型，如果是ext4文件系统，使用resize2fs命令扩展容量：
resize2fs /dev/root_vg/root
```



## lv缩容

**缩容lv一般是腾出空间给同vg的其他lv，这并不是一种安全的做法，一般情况下没有人会这么干，一般当lv卷磁盘空间满的时候，会加磁盘来扩容，而不是从同vg下的其他lv腾空间出来，这里仅做出示例：**

```shell
umount /dev/mapper/my_data-lv_data				#卸载lv
e2fsck -f /dev/mapper/my_data-lv_data			#先扫描、检查磁盘在执行resize2fs，不然它会提示你先执行e2fsck -f命令的
resize2fs /dev/mapper/my_data-lv_data 100G		#缩容文件系统到100G
lvreduce -L 100G /dev/mapper/my_data-lv_data	#缩容，lv到100G,此时vg就空闲了很多PE出来了，可以通过vgdisplay命令查看
lvreduce -L -100G /dev/mapper/my_data-lv_data	#缩容，lv缩减100G,此时vg就空闲了很多PE出来了，可以通过vgdisplay命令查看
mount /dev/mapper/my_data-lv_data /my_data/		#重新挂载，此时文件系统就是100G大小

#以上缩容发现，一个是需要卸载，这点可以影响业务，其次缩容后重新挂载，原来的文件仍存在，没有丢失。

#下面是一个例子
/dev/mapper/vg--data-lv1T 1008G   77M  957G   1% /gpt1
/dev/mapper/vg--data-lv2   2.0T   81M  1.9T   1% /gpt2
[root@kubernetes ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  centos    1   2   0 wz--n- <19.00g    0 
  vg-data   1   2   0 wz--n-  <3.00t    0 
[root@kubernetes ~]# lvdisplay /dev/vg-data/lv1T /dev/vg-data/lv2
  --- Logical volume ---
  LV Path                /dev/vg-data/lv1T
  LV Name                lv1T
  VG Name                vg-data
  LV UUID                oYtQ0a-hUQA-pVkK-e8iy-NFam-eRs4-PrMsZW
  LV Write Access        read/write
  LV Creation host, time kubernetes, 2023-03-29 10:20:33 +0800
  LV Status              available
  # open                 1
  LV Size                1.00 TiB
  Current LE             262144
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/vg-data/lv2
  LV Name                lv2
  VG Name                vg-data
  LV UUID                APJ81S-hOEC-LCfp-Oamo-5OTk-C8pA-GLhN2L
  LV Write Access        read/write
  LV Creation host, time kubernetes, 2023-03-29 10:21:08 +0800
  LV Status              available
  # open                 1
  LV Size                <2.00 TiB
  Current LE             524287
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
[root@kubernetes ~]# 

#打算对/gpt1缩减100G，腾出空间给/gpt2：
umount  /gpt1
e2fsck -f  /dev/mapper/vg--data-lv1T
#注意这条命令是缩减文件系统到多少G，957-100=857
resize2fs /dev/mapper/vg--data-lv1T  857G

#重新挂载，现在是844G
mount /dev/mapper/vg--data-lv1T /gpt1/
[root@kubernetes ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/vg--data-lv1T  844G   77M  801G   1% /gpt1


# 文件系统缩减了但是lv没有缩减呀，你看看：
[root@kubernetes ~]# lvdisplay /dev/mapper/vg--data-lv1T
  --- Logical volume ---
  LV Path                /dev/vg-data/lv1T
  LV Name                lv1T
  VG Name                vg-data
  LV UUID                oYtQ0a-hUQA-pVkK-e8iy-NFam-eRs4-PrMsZW
  LV Write Access        read/write
  LV Creation host, time kubernetes, 2023-03-29 10:20:33 +0800
  LV Status              available
  # open                 1
  LV Size                1.00 TiB			#和没缩减文件系统时一模一样
  Current LE             262144			#和没缩减文件系统时一模一样
  Segments               1	
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   
# 所以现在开始缩减lv,这里我们写-100G
lvresize -L -100G  /dev/vg-data/lv1T

#缩减成功，但是好像不对，lv显示924G，挂载之后的文件系统怎么才844G
[root@kubernetes ~]# lvdisplay /dev/mapper/vg--data-lv1T
  --- Logical volume ---
  LV Path                /dev/vg-data/lv1T
  LV Name                lv1T
  VG Name                vg-data
  LV UUID                oYtQ0a-hUQA-pVkK-e8iy-NFam-eRs4-PrMsZW
  LV Write Access        read/write
  LV Creation host, time kubernetes, 2023-03-29 10:20:33 +0800
  LV Status              available
  # open                 1
  LV Size                924.00 GiB
  Current LE             236544
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
 
[root@kubernetes ~]# vgs	#vg多了100G
  VG      #PV #LV #SN Attr   VSize   VFree  
  vg-data   1   2   0 wz--n-  <3.00t 100.00g
  
[root@kubernetes ~]# df -h /dev/vg-data/lv1T
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/vg--data-lv1T  844G   77M  801G   1% /gpt1
[root@kubernetes ~]# 重新卸载重新挂载也是一样的，怎么回事？
```

## 5、删除逻辑卷、删除卷组

一般不会这样干，这里只做示例

```shell
[root@localhost ~]# umount  /dev/mapper/my_data-lv_data			#删除一个lv之前必须先卸载文件系统
[root@localhost ~]# lvremove /dev/my_data/lv_data				#删除逻辑卷,如果不知道lv的路径，可以通过lvdisplay命令查看
Do you really want to remove active logical volume my_data/lv_data? [y/n]: y
  Logical volume "lv_data" successfully removed




root@localhost ~]# vgremove  my_data					#删除整个卷组
Do you really want to remove volume group "my_data" containing 1 logical volumes? [y/n]: y
  Volume group "my_data" is removed







#删除物理卷，该物理卷必须从vg中卸载下来，使用vgreduce my_data /dev/sdd1卸载即可
[root@localhost ~]# pvremove /dev/sdd1
  Removed "/dev/sdd1" from volume group "my_data"
```

最后献上这篇博客的[参考文章](https://blog.csdn.net/MssGuo/article/details/120473476)

