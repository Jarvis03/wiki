### Freetype

#### PC

freetype-2.4.10.tar.bz2 拷贝到linux，然后解压 `tar xjf freetype-2.4.10.tar.bz2` 命名为 `mv freetype-2.4.10 freetype-2.4.10_pc` 进入目录执行`./config -> make -> sudo make install` 安装到 `/usr/local.lib/`


gcc -o example1 example1.c 编译文件
编译错误找不到头文件，需要编译时指定
`gcc -o example1 example1.c -I /usr/local/include/freetype2` 编译提示函数未定义，需要加入库`gcc -o example1 example1.c -I /usr/local/include/freetype2 -lfreetype -lm(math)`

##### 交叉编译

解压`tra xjf freetype-2.4.10.tar.bz2`


./configure --host=arm-linux
make
make DESTDIR=$PWD/tmp install

编译出来的头文件应该放入：
`/usr/local/arm/4.3.2/arm-none-linux-gnueabi/libc/usr/include` 

我的
`/work/tools/gcc-3.4.5-glibc-2.3.6/arm-linux/include`

编译出来的库文件应该放入：
`/usr/local/arm/4.3.2/arm-none-linux-gnueabi/libc/armv4t/lib`

我的 `/work/tools/gcc-3.4.5-glibc-2.3.6/arm-linux/lib`


把tmp/usr/local/include/*  复制到 /usr/local/arm/4.3.2/arm-none-linux-gnueabi/libc/usr/include
cp * /usr/local/arm/4.3.2/arm-none-linux-gnueabi/libc/usr/include -rf
cd /usr/local/arm/4.3.2/arm-none-linux-gnueabi/libc/usr/include
mv freetype2/freetype .

arm-linux-gcc -finput-charset=GBK -o example1 example1.c  -lfreetype -lm
arm-linux-gcc -finput-charset=GBK -o show_font show_font.c  -lfreetype -lm


freetype/config/ftheader.h
freetype2/freetype/config/ftheader.h 



arm-linux-gcc -finput-charset=GBK -fexec-charset=GBK -o show_font show_font.c -lfreetype -lm 
`arm-linux-gcc -o example1 example1.c -I /usr/local/include/freetype2 -lfreetype -lm(math)`