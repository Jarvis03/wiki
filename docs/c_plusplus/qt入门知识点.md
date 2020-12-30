## QT 入门知识点

## 安装

需要先下载对应平台的安装包,以下两个网站都可以下载[官方镜像](http://download.qt.io/archive/qt), [清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/qt/)。在国内清华大学的镜像网站会更快点。

- windows

  安装到下图步骤时候，应该选择一个编译器，默认是没有勾选的。笔者第一次没有勾选此选项。试图使用自己电脑已经安装的`mingW`,结果在配置

  `qmake`时找不到该工具。

  

> 安装完成后需要配置环境路径

- linux

安装完成后，配置环境路径为`/home/linux/Qt5.12.3/5.12.3/gcc_64/bin`。执行`qmake -v` 查看版本，若能显示则安装成功。

`QT`运行时依赖`libgl`和`libgstreamer`

```shell
sudo apt-get install libgl1-mesa-dev
sudo apt-get install libgstreamer0.10-0
sudo apt-get install libgstreamer-plugins-base0.10-0
```

## 工具

- `assistant`   帮助手册

- `designer`    设计`ui`图形化界面

- `moc`         :元对象编辑器
  将非标准`C++`的语法，转换为标准的`C++`语法

- `qmake`       `QT`工程构建器
  构建`QT`工程

- `rcc`         资源管理工具 
  将资源文件，转换为标准的`C++`语法 

- `uic`        ` UI`转换器 
  将`ui`文件转换为标准的`C++`式头文件

- `qtcreator`   集成`IDE`开发环境 
  将上边所有的工具都进行集成

## 构建编译

1. `qmake -project`

首次构建，生成`***.pro`文件。若需要增加模块，和源代码，需要修改此文件

2. `qmake`

生成`Makefile`文件

3. `make`

生成可执行文件

## 信号与槽机制

信号和槽本质就是函数，QT的通信机制

```c++
//连接函数
//				connect
//	信号函数   ---------> 槽函数
//
// SIGNAL(信号函数名(形参类型)) 
//		将信号函数转换为char *类型 	
//	SLOT(槽函数名(形参类型))  
//		将槽函数转换为char *类型 

	QObject::connect(
	const QObject *sender,    //发送者
	const char *signal,       //信号
	const QObject *receiver,  // 接收者
	const char *method,       //槽函数
	Qt::ConnectionType type = Qt::AutoConnection)
	
//!! 定义信号与槽函数的使用需要使用宏 Q_OBJECT
// 信号函数参的个数 >= 槽函数，一般情况是个数和类型一致       
```

一般情况下槽函数和信号函数的参数个数与类型一致，如果槽函数参数个数多于信号函数，必须提供缺省值


