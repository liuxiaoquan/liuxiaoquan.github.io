# Linux扩容home目录


机器装了一块新硬盘, 先使用sudo fdisk -l看看新添加的硬盘叫什么, 我的叫sdb接下来按照这篇文章操作即可, 该文章新挂载的硬盘叫sdb1, 注意替换成自己的硬盘名

注释：先使用```sudo fdisk -l```看看新添加的硬盘叫什么,我的叫sdb1

```shell
lsblk -f #列出块设备列表，-f用于输出文件系统的详细信息
```

0.格式化分区

```shell
mkfs.ext4 /dev/sdb1
```

1.创建目录

```shell
mkdir /media/home
```

2.把/dev/sdb1挂载到/media/home

```shell
mount /dev/sdb1 /media/home
```

3.执行几次sync命令,确保文件系统数据都落盘

```shell
sync  #执行三次或更多次sync命令

#在Linux系统中，为了加快数据的读取速度，所以在默认的情况中， 某些已经加载内存中的数据将不会直接被写回硬盘，而是先缓存在内存当中，如此一来， 如果一个数据被你重复的改写，那么由于他尚未被写入硬盘中，因此可以直接由内存当中读取出来， 在速度上一定是快上相当多的！

#不过，如此一来也造成些许的困扰，那就是万一你的系统因为某些特殊情况造成不正常关机 (例如停电或者是不小心踢到power)时，由于数据尚未被写入硬盘当中，哇！所以就会造成数据的升级不正常啦！ 那要怎么办呢？这个时候就需要sync这个命令来进行数据的写入动作啦！ 直接在文字接口下输入sync，那么在内存中尚未被升级的数据，就会被写入硬盘中！所以，这个命令在系统关机或重新启动之前， 很重要喔！最好多运行几次(2-4次)！
```

4.同步/home到/media/home

```shell
rsync -aXS /home/. /media/home/.
#或者加上-v详细模式输出 rsync -avXS /home/. /media/home/.
```

```
# Options
-v, --verbose 			# 详细模式输出。(打印一些信息，比如文件列表、文件数量等)
-q, --quiet 			# 精简输出模式。
-c, --checksum 			# 打开校验开关，强制对文件传输进行校验。
-a, --archive 			# 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD。
-r, --recursive 		# 对子目录以递归模式处理。
-R, --relative 			# 使用相对路径信息。
-b, --backup 			# 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir 			# 将备份文件(如~filename)存放在在目录下。
-suffix=SUFFIX 			# 定义备份文件前缀。
-u, --update 			# 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件，不覆盖更新的文件。
-l, --links 			# 保留软链结。
-L, --copy-links 		# 想对待常规文件一样处理软链结。
--copy-unsafe-links		# 仅仅拷贝指向SRC路径目录树以外的链结。
--safe-links 			# 忽略指向SRC路径目录树以外的链结。
-H, --hard-links 		# 保留硬链结。
-p, --perms 			# 保持文件权限。
-o, --owner 			# 保持文件属主信息。
-g, --group 			# 保持文件属组信息。
-D, --devices 			# 保持设备文件信息。
-t, --times 			# 保持文件时间信息。
-S, --sparse 			# 对稀疏文件进行特殊处理以节省DST的空间。
-n, --dry-run			# 现实哪些文件将被传输。
-w, --whole-file 		# 拷贝文件，不进行增量检测。
-x, --one-file-system	# 不要跨越文件系统边界。
-X, --xattrs			# 保留扩展属性
-B, --block-size=SIZE	# 检验算法使用的块尺寸，默认是700字节。
-e, --rsh=command 		# 指定使用rsh、ssh方式进行数据同步。
--rsync-path=PATH 		# 指定远程服务器上的rsync命令所在路径信息。
-C, --cvs-exclude 		# 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件。
--existing				# 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件。
--delete				# 删除那些DST中SRC没有的文件。
--delete-excluded 		# 同样删除接收端那些被该选项指定排除的文件。
--delete-after 			# 传输结束以后再删除。
--ignore-errors 		# 及时出现IO错误也进行删除。
--max-delete=NUM 		# 最多删除NUM个文件。
--partial				# 保留那些因故没有完全传输的文件，以是加快随后的再次传输。
--force					# 强制删除目录，即使不为空。
--numeric-ids 			# 不将数字的用户和组id匹配为用户名和组名。
--timeout=time 			# ip超时时间，单位为秒。
-I, --ignore-times 		# 不跳过那些有同样的时间和长度的文件。
--size-only 			# 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间。
--modify-window=NUM		# 决定文件是否时间相同时使用的时间戳窗口，默认为0。
-T --temp-dir=DIR 		# 在DIR中创建临时文件。
--compare-dest=DIR 		# 同样比较DIR中的文件来决定是否需要备份。
-P						# 等同于 --partial。
--progress				# 显示备份过程。
-z, --compress 			# 对备份的文件在传输时进行压缩处理。
--exclude=PATTERN 		# 指定排除不需要传输的文件模式。
--include=PATTERN 		# 指定不排除而需要传输的文件模式。
--exclude-from=FILE		# 排除FILE中指定模式的文件。
--include-from=FILE		# 不排除FILE指定模式匹配的文件。
--version				# 打印版本信息。
--address				# 绑定到特定的地址。
--config=FILE 			# 指定其他的配置文件，不使用默认的rsyncd.conf文件。
--port=PORT 			# 指定其他的rsync服务端口。
--blocking-io 			# 对远程shell使用阻塞IO。
-stats					# 给出某些文件的传输状态。
--progress				# 在传输时现实传输过程。
--log-format=formAT		# 指定日志文件格式。
--password-file=FILE	# 从FILE中得到密码。
--bwlimit=KBPS 			# 限制I/O带宽，KBytes per second。
-h, --help				# 显示帮助信息。
```

5.同步完成后重命名/home

```shell
mv /home /home_old
```

6.新建/home

```shell
mkdir /home
```

7.取消/dev/sdb1挂载

```shell
umount /dev/sdb1
```

8.重新挂载/dev/sdb1到home

```shell
mount /dev/sdb1 /home
```

9.查看/dev/sdb1的UUID

```shell
blkid
```

10.把UUID复制下来，修改/etc/fstab文件，实现开机自动挂载

```shell
vim /etc/fstab
mount -a #将/etc/fstab的所有内容重新加载。
```

在文件最后添加如下内容：
UUID=8da46012-ab9c-434f-a855-2484112fd1a7 /home ext4 nodev,nosuid 0 2

11.保存之后重启系统，查看分区的挂载情况

```shell
df –Th
```

12.确认一切正常后删除/home_old

```shell
rm -rf /home_old
```

至此，给/home增加空间的工作就完成了。

