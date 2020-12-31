#### LCD驱动程序

##### 假设

###### app:  

     open("/dev/fb0", ...)   主设备号: 29, 次设备号: 0

##### kernel:

         fb_open
         	int fbidx = iminor(inode);
         	struct fb_info *info = = registered_fb[0];

###### app:  

    read()

###### kernel:

		fb_read
			int fbidx = iminor(inode);
			struct fb_info *info = registered_fb[fbidx];
			if (info->fbops->fb_read)
				return info->fbops->fb_read(info, buf, count, ppos);
	     	
			src = (u32 __iomem *) (info->screen_base + p);
			dst = buffer;
			*dst++ = fb_readl(src++);
			copy_to_user(buf, buffer, c)         	

##### 问  1. registered_fb在哪里被设置？

    答1. register_framebuffer

##### 怎么写LCD驱动程序？

    1. 分配一个fb_info结构体: framebuffer_alloc
    2. 设置
    3. 注册: register_framebuffer
    4. 硬件相关的操作

##### 测试：

      1. make menuconfig去掉原来的驱动程序
        -> Device Drivers
          -> Graphics support
        <M> S3C2410 LCD framebuffer support
    
    2. make uImage
       cp arch/arm/boot/uImage  /work/nfs_root/uImage_nolcd
       
       make modules 
       cp drives/video/cfb*.ko  /work/nfs_root/first_fs
    
    3. 使用新的uImage启动开发板:
       root
       nfs 30000000 192.168.3.108:/work/nfs_root/uImage_nolcd
       bootm 30000000
    
    4. 
      insmod cfbcopyarea.ko 
      insmod cfbfillrect.ko 
      insmod cfbimgblt.ko 
      insmod lcd.ko

##### echo hello > /dev/tty1  // 可以在LCD上看见hello

##### cat lcd.ko > /dev/fb0   // 花屏

###### 5. 修改 /etc/inittab

    tty1::askfirst:-/bin/sh
    用新内核重启开发板
    
    insmod cfbcopyarea.ko 
    insmod cfbfillrect.ko 
    insmod cfbimgblt.ko 
    insmod lcd.ko
    insmod buttons.ko
    
    echo hello > /dev/tty1
    cat lcd.ko > /dev/fb0

