# MakeFile

## 介绍
#### 基本规则

```makefile
目标 ： 依赖
[tab] 命令
```

#### 命名

一般为`GNUmakefile`， `makefile`， `Makefile`，推荐使用`Makefile`

#### 更新

依赖文件不存在或者文件有更新时，执行make会重新编译


## Makefile的内容


在一个完整的 `Makefile `中,包含了 5 个东西:

> 显式规则、隐含规则、变量定义、指示符和注释


1.  **显式规则** ：显式规则说明了，如何生成一个或多的的目标文件。这是由Makefile的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。
2.  **隐晦规则**：由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写Makefile，这是由make所支持的。
3.  **变量的定义**：在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
4. **指示符**：其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。
5.  **注释**：Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用“#”字符，这个就像C/C++中的“//”一样。如果你要在你的Makefile中使用“#”字符，可以用反斜框进行转义，如：“\#”。

### 1. 显式规则

#### 1.1 基本规则

```makefile
目标 ： 依赖
[tab] 命令
```



#### 1.2 清空目标文件的规则

```makefile
.PHONY clean
clean:
	rm $(OBJS)
```



`.PHONY`表示一个伪目标，若不使用`.PHONY`，再次执行 `clean`会发现`clean`是最新的，无法实现清除的目的。`.PHONY`表示一个伪目标，并不是真是存在的，所以每次都可以执行`clean` 。

#### 1.3 文件名使用通配符

Maekfile 中表示文件名时可使用通配符。可使用的通配符有:`*`、`?`和`[...]`。
在 Makefile 中通配符的用法和含义和 Linux(unix)的 Bourne shell 完全相同。例如,`*.c”`代表了当前工作目录下所有的以`.c`结尾的文件等。但是在 Makefile 中这些统配符并不是可以用在任何地方,Makefile 中统配符可以出现在以下两种场合:

1.  可以用在规则的目标、依赖中,make 在读取 Makefile 时会自动对其进行匹配处理(通配符展开);

2. 可出现在规则的命令中,通配符的通配处理是在 shell 在执行此命令时完成的。

除这两种情况之外的其它上下文中,不能直接使用通配符。而是需要通过函数`wildcard`来实现。

#### 1.4 目录搜索

- 一般搜索`VPATH`

`VPATH = src:../headers`

- 选择性搜索`vpath`
- 目录搜索机制
- 命令行和搜索目录
- 隐含规则和搜索目录
- 库文件和搜索路径

#### 1.5 静态模式
```makefile
TARGETS ...: TARGET-PATTERN: PREREQ-PATTERNS ...
	COMMANDS
	
objects = foo.o bar.o
all: $(objects)


$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
```
规则描述了所有的.o文件的依赖文件为对应的.c文件,对于目标“foo.o”,取其茎“foo”替代对应的依赖模式“%.c”中的模式字符“%”之后可得到目标的依赖文件“foo.c”

### 2. 隐式规则

#### 2.1 自动推导规则

`make`本身存在一个默认的规则,能够自动完成对`.c`文件的编译并生成对应的`.o`文件。我们就可以省略掉描述`.c `文件和`.o `依赖关系的规则,而只需要给出那些特定的规则描述

```makefile
# sample Makefile
objects = main.o kbd.o command.o 

edit : $(objects)
	cc -o edit $(objects)
main.o : defs.h         #自动推导
kbd.o : defs.h command.h
command.o : defs.h command.h

.PHONY : clean
clean :
	rm edit $(objects)
```

####  2.2 缺省规则

#### 2.3 后缀规则

### 3.变量

#### 3.1 变量的引用

变量引用的展开过程是严格的文本替换过程,就是说变量值的字符串被精确的展开在变量被引用的地方。因此规则:
```makefile
foo = c
prog.o : prog.$(foo)
	$(foo) $(foo) -$(foo) prog.$(foo)
```
被展开后就是:
```makefile
prog.c : prog.c
	cc -c prog.c
```

#### 3.2 设置变量

- `=`：递归方式
- `：=`：静态方式
- `+=`：追加方式
- `？=`：条件赋值

#### 3.3 自动化变量

- `$@`：表示规则的目标文件
- `$<`：规则的第一依赖文件
- `$^`：规则的所有依赖文件表
- `$?`：所有比目标文件更新的依赖文件表

#### 3.4 变量`MAKEFILES`

如果在当前环境定义了一个`MAKEFILES`环境变量,make执行时首先将此变量的值作为需要读入的Makefile文件,多个文件之间使用空格分开。类似使用指示符`include`包含其它Makefile文件一样,如果文件名非绝对路径而且当前目录也不存在此文件,make会在一些默认的目录去寻找。
它和使用`include`的区别:

1. 环境变量指定的 makefile 文件中的`目标`不会被作为` make` 执行的`终极目标`。就是说,这些文件中所定义规则的目标,`make` 不会将其作为`终极目标`来看待。`make` 执行时的`终极目标`就是当前目录下这个文件中所定义的`终极目标`。
2. 环境变量所定义的文件列表,在执行 make 时,如果不能找到其中某一个文件(不存在或者无法创建)。make 不会提示错误,也不退出。就是说环境变量`MAKEFILES`定义的包含文件是否存在不会导致 make 错误(这是比较隐蔽的地方)。
3. make 在执行时,首先读取的是环境变量`MAKEFILES`所指定的文件列表,之后才是工作目录下的 makefile 文件,`include`所指定的文件是在 make 发现此关键字的时、暂停正在读取的文件而转去读取`include`所指定的文件。
4. 变量`MAKEFILES`主要用在`make`的递归调用过程中的的通信，实际应用中很少设置此变量。因为一旦设置了此变量,在多级make
    调用时;由于每一级make都会读取“MAKEFILES”变量所指定的文件,将导致执行出,现混乱。

## GNU make工作方式

#### 1. GNU的make工作时的执行步骤

1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐晦规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

1-5步为第一个阶段，6-7为第二个阶段。第一个阶段中，如果定义的变量被使用了，那么，make会把其展开在使用的位置。但make并不会完全马上展开，make使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开

#### 2.嵌套Make

 在一些大的工程中，我们会把我们不同模块或是不同功能的源文件放在不同的目录中，我们可以在每个目录中都书写一个该目录的Makefile，这有利于让我们的Makefile变得更加地简洁，而不至于把所有的东西全部写在一个Makefile中，这样会很难维护我们的Makefile，这个技术对于我们模块编译和分段编译有着非常大的好处。
      例如，我们有一个子目录叫subdir，这个目录下有个Makefile文件，来指明了这个目录下文件的编译规则。那么我们总控的Makefile可以这样书写：
```makefile
subsystem:
	cd subdir && $(MAKE)
```
其等价于：
```makefile
subsystem:
	$(MAKE) -C subdir
```
定义`$(MAKE)`宏变量的意思是，也许我们的`make`需要一些参数，所以定义成一个变量比较利于维护。这两个例子的意思都是先进入`subdir`目录，然后执行`make`命令。

#### Makefile示例
``` makefile

# $(notdir $(CURDIR)) 获取目录名

TARGET = $(notdir $(CURDIR))

CROSS_COMPILE = gcc

SOURCES = $(wildcard *.c)
HEADERS = $(wildcard *.h)

#静态模式规则 .c 替换成 .o
OBJFILES = $(patsubst %.c,%.o,$(SOURCES))
#OBJFILES = $(SOURCES:%.c=%.o)

all:$(TARGET)
	@echo builded target:$^
$(TARGET): $(OBJFILES)
	@echo linking $@ from $^
	$(CROSS_COMPILE) -o  $@ $^
	@echo obj $(OBJFILES)
$(OBJFILES): %.o:%.c
	@echo compilinng $@ from $< ...
	$(CROSS_COMPILE) -c $< -o $@
	@echo Compile finished
.PHONY :clean
clean:
	@echo Removing generated files
	rm -rf $(TARGET) *.d *.o
```






