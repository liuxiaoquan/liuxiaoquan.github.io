# Lvm挂载硬盘步骤


# 备注说明：lvm的硬盘挂载原理，可参考如下链接，有详细解释所有的挂载硬盘原理

https://zhuanlan.zhihu.com/p/296777898

## 步骤：

### 硬盘格式化成pv

```shell
pvcreate /dev/sdb
```

### 创建完PV以后，我们可以使用`pvdisplay`(显示详细信息)、`pvs`命令来查看当前pv的信息

```shell
pvdisplay
pvs
```

### 在创建完PV以后，这时候我们需要创建一个VG，然后将我们的PV都加入到这个卷组当中，在创建卷组时要给该卷组起一个名字

```shell
vgcreate mydata /dev/sdb
```

### 同样，在创建好VG以后，我们也可以使用 vgdisplay 或者 vgs 命来来查看VG的信息

```shell
vgdisplay
vgs	
#新物理卷，将新增卷增加到原有的卷组中            -----说明：新增加硬盘到lvm组里面的，如果没有则不需要执行
#vgextend xiaoluo /dev/sdd1
#基于卷组(VG)创建逻辑卷(LV)　　通过 lvcreate 命令
lvcreate -n mylv -L 2G xiaoluo
lvcreate -n myweb -l 100%free lnweixin #将剩余的所有空间分配给myweb
lvcreate -n myweb -l 100% lnweixin #将所有空间分配给myweb

lvcreate -n mydata -l 100%free mydata

#格式化并使用我们的逻辑卷
mkfs.xfs /dev/mydata/mydata
#格式化我们的逻辑卷以后，就可以使用 mount 命令将其进行挂载，我们将其挂载到 /mnt 目录下
mount /dev/mydata/mydata /data
#便于以后服务器重启自动挂载,需要将创建好的文件系统挂载信息添加到/etc/fstab里面.UUID可以通过 blkid命令查询.
#为了查看/etc/fstab是否设置正确,可以先卸载逻辑卷data1,然后使用mount –a 使内核重新读取/etc/fstab,看是否能够自动挂载.
#自此lvm挂载硬盘已完成
```



## 删除逻辑卷

### 我们在创建好逻辑卷后可以通过创建文件系统，挂载逻辑卷来使用它，如果说我们不想用了也可以将其删除掉。

```
①首先将正在使用的逻辑卷卸载掉　　     通过 umount 命令
②将逻辑卷先删除　　		 通过 lvremove 命令
③删除卷组　　			 通过 vgremove 命令
④最后再来删除我们的物理卷　　	 通过 pvremove 命令
```

```shell
mount /dev/xiaoluo/mylv /mnt/
umount /mnt/
lvremove /dev/xiaoluo/mylv 
vgremove xiaoluo
pvremove /dev/sdb
```

`此时我们的刚创建的逻辑卷 mylv，卷组 xiaoluo以及物理卷 /dev/sdb 已经从我们当前操作系统上删除掉了，通过 lvs、vgs、pvs命令可以查看一下`


