## u-boot 分析

### u-boot常用命令

通过`help`可以查看`u-boot`所支持的命令，常用命令如下所示

```shell
printenv           #打印环境变量
saveenv            #保存环境变量 
setenv/set         #修改内存中的环境变量
tftpboot/tftp      #从网络下载文件到开发板的内存中
loadb/loady/loadx  #通过串口下载文件到内存中
go                 #执行一个内存中的应用程序
ping               #测试网络连通性
bootm              #启动内核（linux）
reset              #重启命令
	
```

### 目录结构

```
arch: 	cpu架构相关代码
      arch/arm/cpu/arm11  arm9 armv8 armv7
board:	不同厂商板子相关代码
      board/samsung/:  三星厂商的板子
driver: 设备驱动代码
fs:   文件系统
lib:  解压缩代码，字符串处理，校验代码, 红黑树，快速排序
net:  网络协议
common:	命令相关代码
doc:  	文档， 相比Linux内核 的文档太少了
include:  头文件
       include/configs/   各个开发板的配置头文件
scripts:	 编译时使用的 Makefile shell 脚本
dts: 	以后的uboot版本也将使用设备树
disk:	硬盘分区相关代码
CREDITS:  对uboot的主要贡献者
boards.cfg:  uboot 都支持哪些板子
```

### 增加开发板

1. `boardsl.cfg` 中增加板子`x6818`

   ` Active  arm   armv8   s5p6818  newsamsung  s5p6818      x6818   -`

2. 拷贝`s5p6818`相关配置文件

   ```shell
   mkdir ./board/newsamsung
   cp /board/samsung/s5p6818/ ./board/samsung/common/ ./board/newsamsung/ -r
   ```

3. 设置`x6818.h`

拷贝别人写好的，该头文件如果编写待续

`cp ./include/configs/fs6818.h ./include/configs/x6818.h`

4. 配置

   修改`Makefile`,指定编译工具链

   ```makefile
   # 修改如下
   ifeq(arch,arm)
   CROSS_COMPILE ?= arm-linux-
   endif
   ```

   使用`make x6818_config`命令进行配置，`O=/path`可以指定输出路径。执行此过程，会在`/include`目录下生成`config.h`和`config.mk`两个文件

5. 编译

```makefile
make              #编译
make all -j4      #多核编译
make all V=1      # 显示编译细节
```

6. 清除

```makefile
make clean
make mrproper   #清除 .o .bin 配置文件...
make distclean  #清除.o .bin 配置文件 rm.bak...
```

### 添加新命令

比如添加显示设备信息的命令`device`

1.  创建`cmd_device.c`

`touch /common/cmd_device.c`

命令设置代码框架如下

```c
#include <common.h>
#include <command.h>

static int do_aaaa(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    printf("device informations\n");
    return 0;
}

U_BOOT_CMD(
	device,		// 命令的名字
	2,			// 命令参数个数
	0,  		// 是否支持重复
	do_device,	// 命令对应的函数
	"display device informations",	// 简单的描述
				// 详细描述信息
	"   - print brief description of device\n" 
			);
```

2. 修改`Makefile`

在`common/Makefile`中添加 `obj-y += cmd_device.o`

3. 重新编译 

```makefile
make
 #以下显示编译信息
 ...
 CC common/cmd_device.o  #已经编译
 ....
```



### u-boot烧写

```shell
tftp 41000000 u-boot.bin
mmc dev 0 # 选择设备
#         源地址    目的  大小
mmc write 41000000  0   500

```

### 启动流程分析

