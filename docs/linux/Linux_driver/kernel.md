

## 内核移植

### 配置与编译

1. 配置编译工具

​    修改顶层`Makefile` 

```Makefile
ARCH		  ?= $(SUBARCH)
CROSS_COMPILE ?= $(CONFIG_CROSS_COMPILE:"%"=%)
# 修改为：
ARCH          ?= arm
CROSS_COMPILE ?= arm-linux-
```

2. 导入配置

`make x6818_defconfig` 此命令会在顶层目录下生成`.config`

3. 图形化界面配置

`make menuconfig` 会启动图形化配置界面，若启动失败则需要安装图形库

```shell
sudo apt-get install libncurses5-dev
sudo apt-get install flex
sudo apt-get install bison
```

4. 内核编译

```shell
make uImage #编译
-------------
make uImage -j4
```

5. 清除命令

与u-boot相同，参见[u-boot](./bootloader)`增加开发板`章节    

### 内核目录

>arch      : 体系结构相关代码
>include : 头文件
>drivers  : 各种驱动代码
>lib	      : 解压压缩代码，红黑树算法，md5, 字符串处理
>fs		   : 文件系统代码
>net 	   : 网络协议代码
>ipc 	    : 进程间通信内核支持代码
>mm 	  : 内存管理
>kernel   : 进程创建，调度
>init 	   : 内核启动之初的初始化代码
>scripts   : 编译脚本
>Documentation  开发文档



### 配置菜单

#### kconfig语法

使用图形化配置会修改`.config`文件，内核通过此文件决定如何编译，下面介绍下如何生成配置选项。

`kconfig` 的语法可以参考`Documentation\kbuild\kconfig-language.txt`。一个简单的配置选项如下，

```
config MODVERSIONS 
    bool "Set version information on all module symbols"
	depends on MODULES
	help  Usually, modules have to be recompiled whenever you switch to a new kernel. 
```

#### 添加驱动示例

1. `driver/char`目录下新建一个`test.c`

2. `driver/char/Makefile`中增加`boj-$(CONFIG_TEST_DEMO) += test.o`

3. 添加菜单选项

   修改`driver/char`目录下的`kconfig`

   ```
   config TEST_DEMO                      
       tristate  "test demo!"
   	default y
       help
   		  This is test code!~
   ```

   执行`make menuconfig` 可以在`Device Drivers->Charater devices`看到新增的选项，因为我们默认值是`y`,所以编译时候可以直接编译新建的驱动到内核。

### 内核启动流程

1. 分析顶层`Makefile`

`vmlinux-lds  := arch/$(SRCARCH)/kernel/vmlinux.lds`指定了链接脚本

2.  分析链接脚本

`ENTRY(stext)`中找到入口符号`stext`，然后从`System.map`找到符号地址为`c0008000`。

```shell
arm-none-linux-gnueabi-addr2line -e vmlinux c0008000 
#返回
kernel-3.4.39/arch/arm/kernel/head.S:94 # stext所在行
```

3. 分析`head.S`

- ` bl  __create_page_tables`
  创建内存页表项， 为使能 mmu 做准备

- `dr r13, =__mmap_switched`
  给r13 赋值一个 符号地址

- `b   __enable_mmu`

  ```assembly
  enable_mmu: 
      b   __turn_mmu_on
  ```

- `__turn_mmu_on`

```assembly
mov r3, r13	
mov pc, r3  @跳转到 __mmap_switched 开始运行
```

- `__mmap_switched` 开始运行

  ```assembly
  @ arch/arm/kernel/head-common.S:81
  
  
  	ldmia	r3!, {r4, r5, r6, r7}
  	cmp	r4, r5				@ Copy data segment if needed
  1:	cmpne	r5, r6
  	ldrne	fp, [r4], #4
  	strne	fp, [r5], #4
  	bne	1b
      ...                    @省略
  	mov	fp, #0				@ Clear BSS (and zero fp)
  	
  	str	r9, [r4]			@ Save processor ID
  	str	r1, [r5]			@ Save machine type
  	str	r2, [r6]			@ Save atags pointer
  	bic	r4, r0, #CR_A			@ Clear 'A' bit
  	stmia	r7, {r0, r4}			@ Save control register values
  	
  	b	start_kernel      @ 执行C代码
  ```

  

- 启动内核

  ```c
  //init/main.c:466
      printk(KERN_NOTICE "%s", linux_banner);
      mm_init(); //内存管理 子系统 初始化
      sched_init(); //进程调度 子系统 初始化
  //  .....
  /* init some links before init_ISA_irqs() */
  	early_irq_init();
  	init_IRQ();
  	prio_tree_init();
  	init_timers();
  	hrtimers_init();
  	softirq_init();
  	timekeeping_init();
  	time_init();
  // .............
  /* Do the rest non-__init'ed, we're now alive */
  	rest_init();
  ```

- rest_init

```c
/*
 * We need to finalize in a non-__init function or else race conditions
 * between the root thread and the init thread may cause start_kernel to
 * be reaped by free_initmem before the root thread has proceeded to
 * cpu_idle.
 *
 * gcc-3.4 accidentally inlines this function, so use noinline.
 */
 static noinline void __init_refok rest_init(void)
 {
      /*
	 * We need to spawn init first so that it obtains pid 1, however
	 * the init task will end up wanting to create kthreads, which, if
	 * we schedule it before we create kthreadd, will OOPS.
	 */
	kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_SIGHAND);
 }
```

- `kernel_init`

  ```c
  static int __init kernel_init(void * unused)
  {
      ....
  	if (sys_access((const char __user *) ramdisk_execute_command, 0) != 0) {
  		ramdisk_execute_command = NULL;
  		prepare_namespace();
  	}
      ....
      init_post(); //执行第一个程序
  }
  
  // do_mounts.c
  void __init prepare_namespace(void)
  { 
      ....
      mount_root();//挂载根文件系统 第一个 文件系统
      ...
  }
  
  
  static noinline int init_post(void)
  {
      ...
      run_init_process("/sbin/init");
  	run_init_process("/etc/init");
  	run_init_process("/bin/init");
  	run_init_process("/bin/sh");
      ...    
  }
  
  ```

  ### 下载内核

1. 拷贝内核
   `$ cp arch/arm/boot/uImage  ~/tftpboot`  

2. 下载内核
   `# tftp 41000000 uImage `

3. 烧写内核

``` shell
# mmc dev 2  //选择设备
# mmc write 41000000 500 3000

```