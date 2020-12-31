## 根文件系统

### 根文件系统目录

```
bin : 使用系统的必要的命令，所有用户都可以使用
etc : 软件的配置文件、 内核启动文件
lib : 库文件， 标准C库，数学库等
mnt   : 文件系统挂载点， 用于临时挂载文件系统，SD U盘
root  : 超级用户的 工作目录
sys   : 虚拟文件系统 sysfs 的挂载点，主要用于管理设备驱动
usr   : bin  sbin  用户自己安装的软件，不是系统必要的
dev   : 设备节点文件， 一切设备皆文件
home  : 普通用户的家目录
linuxrc : 内核运行的第一程序
proc  : 虚拟文件系统， 用于管理内核进程，获取内核进程状态等
sbin  : 用于管理内核系统的必要的命令程序，通常需要 sudo 
tmp   : 存放临时文件， 标准建议系统启动时，就会清空此目录
var	  : 软件（系统）的log 日志、软件的缓存

```

### 使用busybox制作根文件系统

#### 生成基本命令

1. 配置交叉编译工具   

   打开顶层目录，修改`Makefile`

   ```makefile
   CROSS_COMPILE ?= arm-linux-
   ARCH ?= arm
   ```

2. 默认配置
   `$ make defconfig `

3. 自定义配置

```shell
$ make menuconfig
###############################
#   添加 insmod rmmod modinfo ...
Linux Module Utilities  ---> 
	[*] modinfo              
	[ ] Simplified modutils  
	[*]   insmod             
	[*]   rmmod              
	[*]   lsmod              
	[*]     Pretty output    
	[*]   modprobe           
	[*]     Blacklist support
	[*]   depmod

######################################
#配置工具生成到 那个目录
Busybox Settings  --->
				Installation Options ("make install" behavior)  --->  
					(./_install) BusyBox installation prefix 
```

4. 编译

```shell
$ make  /  make all -j4  V=1 
$ make install
```

5. 清除

```makefile
make clean
make mrproper
make distclean
```

#### 制作文件系统

1. `$ mkdir  dev  etc  home  lib   mnt  proc  root   sys  tmp   var -p`
2. 添加`inittab`

  ```shell
$ touch  etc/inittab

#添加以下内容
#this is run first except when booting in single-user mode.
::sysinit:/etc/init.d/rcS
# /bin/sh invocations on selected ttys
::respawn:-/bin/sh
# Start an "askfirst" shell on the console (whatever that may be)
::askfirst:-/bin/sh
# Stuff to do when restarting the init process
::restart:/sbin/init
# Stuff to do before rebooting
::ctrlaltdel:/sbin/reboot
::shutdown:/sbin/swapoff -a
  ```

3. 添加`rcS`

```shell
mkdir etc/init.d/ -p
touch etc/init.d/rcS
# 添加以下内容
#!/bin/sh
#This is the first script called by init process
/bin/mount -a
echo /sbin/mdev>/proc/sys/kernel/hotplug
mdev -s
```

4. 添加 `fstab`

```shell
touch etc/fstab 
#添加以下内容
#device     mount-point     type     	options    dump     fsck order
proc       	/proc			proc		defaults	0		0
tmpfs     	/tmp			tmpfs		defaults	0		0
sysfs     	/sys			sysfs		defaults	0		0
tmpfs       /dev			tmpfs		defaults	0
```

5. 添加`profile`

```shell
touch /etc/profile
# 添加以下文件
#!/bin/sh
export HOSTNAME= 自定义
export USER=root
export HOME=root
export PS1="[$USER@$HOSTNAME \W]\# "
#export PS1="[\[\033[01;32m\]$USER@\[\033[00m\]\[\033[01;34m\]$HOSTNAME\[\033[00m\ \W]\$ "
PATH=/bin:/sbin:/usr/bin:/usr/sbin
LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
export PATH LD_LIBRARY_PATH
```

6. 添加动态库

  `which arm-linux-gcc	// 查看命令在哪个目录下面 `

0. 查看 命令 依赖的 库文件 
   `arm-linux-readelf -d /bin/ls`

1. 拷贝动态库
   `cp /opt/gcc-4.9.4/arm-none-linux-gnueabi/sysroot/lib/* _install/lib -ra`
2. 添加权限 
   `chmod 777 _initall/lib/* -Rf`
3. 删除静态库
   `$ rm  lib/*.a`
4. 剥离动态库的调试信息，符号表等等
   `du -h` 查看文件大小  瘦身
   `$ arm-linux-strip *`

### 通过网络挂载根文件系统

> NFS设置格式：
> nfsroot = [<server-ip>:]<root-dir>[,<nfs-options>]   
> ip = <client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>

```shell
修改 uboot 环境变量 	
set bootargs "root=/dev/nfs nfsroot=192.168.1.5:/home/guo/rootfs rw console=/dev/ttySAC0,115200 init=/linuxrc ip=192.168.1.3"
set bootcmd  "tftp 41000000 uImage;bootm 41000000"
```



### 通过 eMMC 挂载根文件系统

1. 删除 `emmc` 所有分区

```
fdisk /dev/mmcblk0  分区命令

/************************************************/
Command (m for help): 
m 帮助信息
n 添加一下分区
p 打印分区表
d 删除分区表
w 写分区表   把分区表 写到 block0
/************************************************/
Command (m for help): d      //  d 代表删除分区
//  多执行几次 d 就可以删除所有分区
```

2. 为 `emmc` 创建分区
   `Command (m for help): n`
3. 输入 `p` 表示创建主分区

```
Command action
    e   extended
    p   primary partition (1-4)
```

4. 输入 `1`  表示分区标号为 1 
   `Partition number (1-4): 1`
5. 输入`13568` 表示从`emmc` 中的那个块开始分区
   `First cylinder (1-236032, default 1): 13568`
6. 直接按回车使用默认值，表示分区到最后一块儿
   `Last cylinder or +size or +sizeM or +sizeK (13568-236032, default 236032): 236032`
7. 将创建的分区表写入到`emmc`的 block0 中
   `Command (m for help): w`

8. 将刚创建的分区 `/dev/mmcblk0p1` 格式化分区为 `ext2` 文件系统类型
   `mkfs.ext2 /dev/mmcblk0p1`

9. 将刚格式化好的文件系统挂载到` /mnt` 下
   `mount -t ext2 /dev/mmcblk0p1 /mnt/`

10. 拷贝 `rootfs` 根文件系统的文件到` /mnt`, 并解压

11. 解除挂载
    `umount /mnt`
12. 重启开发板，进入`uboot`中设置`bootargs`,`bootcmd` 环境变量 

```shell
set bootargs "root=/dev/mmcblk0p1 rw rootfstype=ext2 init=/linuxrc console=ttySAC0,115200 ip=192.168.1.5"
set bootcmd "mmc dev 2;mmc read 0x41000000 500 3000; bootm 0x41000000"
```

13. 启动开发板linux内核
    `boot `

