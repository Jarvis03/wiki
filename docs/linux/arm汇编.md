# ARM汇编

## ARM处理器的工作模式

模式说明:

- User	: 非特权模式，大部分任务执行在这种模式
- FIQ		: 当一个高优先级（fast) 中断产生时将会进入这种模式
- IRQ		: 当一个低优先级（normal) 中断产生时将会进入这种模式
- Supervisor	: 当复位或软中断指令执行时将会进入这种模式
- Abort	: 当存取异常时将会进入这种模式
- Undef	: 当执行未定义指令时会进入这种模式
- System	: 使用和User模式相同寄存器集的特权模式
- Monitor	: 是为了安全而扩展出的用于执行安全监控代码的模式(Cortex多出来此模式)

## 寄存器

1.  ` R13 (the stack pointer register)`  常作为栈指针寄存器，用来保存栈存储空间首地址。每种 异常模式拥有自己的`R13`作为栈指针

2. `R14 (the linker register) `链接寄存器，保存函数的返回地址值。在`ARM`体系中具有下面两种特殊作用。

- 每一种处理器模式自己的物理`R14`中存放当前子程序的返回地址。当通过`BL`或者`BLX`指令调用子程序是，`R14`被设置为子程序的返回地址。在子程序中，当把`R14`的值赋值到程序计数器`PC`中时，子程序返回。可以通过以下两种方式实现

  * 执行下马任何一条指令：

    ```assembly
 MOV PC,LR
 BX  LR
    ```

  * 在子程序入口使用下面指令将PC保存到栈中：

     ```assembly
  STMFD SP! , {<register>, LR}
  /*下面指令可以实现子程序返回*/
  LDMFD SP!, {<REGISTER, PC>}
     ```
  
- 当异常中断发生时，该异常模式特定的物理`R14` 被设置成该异常模式将要返回的地址，对于有些异常模式， `R14`的值可能与将返回的地址有一个常数的偏移量

​      **NOTE：R14寄存器也可以作为通用寄存器使用 **

3. `R15 (the current program counter register)  ` 当前程序计数寄存器，保存当前取指指令的地址值
   
4.  `cpsr (the current program status register)` 当前程序状态寄存器，
     	
```
N[31]: 如果程序运算结果为负数，N[31] 置1否则 清0
Z[30]: 如果结果为0 的话, Z位 被置1 否则 清0
C[29]: 如果运行加法指令 加完的结果用32位没办法表示（溢出了）C 置1  否则  清0
        如果是减法运算产生借位，C位 清0   没有产生借位， C位 置1
V[28]: 符号位发生变化， 那么 V位置1，否则清零
I[7]: 是否禁止 IRQ 中断源
F[6]: 是否禁止 FIQ 中断源
T[5]: 置1， CPU处于 Thumb 状态 清0， CPU处于 ARM 状态
M[4:0]: 模式位， 2^5 , 只用了8个
    10000  User mode;
    10001  FIQ mode; 
    10011  SVC mode;
    10111  Abort mode; 
    11011  Undfined mode; 
    11111  System mode; 
    10110  Monitor mode; 
    10010  IRQ;
```
5. SPSR： 保存程序状态寄存器  (the save progrem status register) 用来保存 CPSR 值

## ARM指令

### 分类

ARM指令集分为跳转指令，数据处理指令，程序状态寄存器传输指令、Load/Store指令、协处理器指令和异常中断指令

### 指令的一般编码格式

语法格式如 `<opcode>{<cond>}{S} <Rd>, <Rn>,<shifter_operand>`

| 31 - 28 | 27 - 25 | 24- 21 | 20   | 19-16 | 15-12 | 11  -  0        |
| :------ | ------- | ------ | ---- | ----- | ----- | --------------- |
| cond    | 001     | opcode | S    | Rn    | Rd    | shifter_operand |



- `opcode`: 指令操作符编码
- `cond`:  指令执行的条件编码
- `S` : 决定指令的操作是否影响CPSR的值
- `Rd`: 目标寄存器编码
- `Rn`: 包含第一个操作数的寄存器编码
- `shifter_operand`:表示第二个操作数

#### ARM指令的条件码域

| 条件码 | 助记符 | 含义             | CPSR中条件标志           |
| ------ | ------ | ---------------- | ------------------------ |
| 0000   | EQ     | 相等             | Z=1                      |
| 0001   | NE     | 不相等           | Z=0                      |
| 0010   | CS/HS  | 无符号数大于等于 | C=1                      |
| 0011   | CC/LO  | 无符号数小于     | C=0                      |
| 0100   | MI     | 负数             | N=1                      |
| 0101   | PL     | 非负数           | N=0                      |
| 0110   | VS     | 上溢出           | V=1                      |
| 0111   | VC     | 没有上溢出       | V=0                      |
| 1000   | HI     | 无符号数大于     | C=1且Z=0                 |
| 1001   | LS     | 无符号数小于等于 | C=0或Z=1                 |
| 1010   | GE     | 带符号数大于等于 | N=1且V=1或<br />N=0且V=0 |
| 1011   | LT     | 带符号数小于     | N=1且V=0或<br />N=0且V=1 |
| 1100   | GT     | 带符号数大于     | Z=0且N=V                 |
| 1101   | LE     | 带符号数小于等于 | Z=1且N!=V                |
| 1110   | AL     | 无条件执行       |                          |
| 1111   | NV     |                  |                          |



### ARM指令的寻址方式

- 数据处理指令的操作数寻址方式
- 字及无符号字节的`Load/Store`指令的寻址方式
- 杂类`Load/Store`指令的寻址方式
- 批量`Load/Store`指令的寻址方式
- 协处理器`Load/Store`指令的寻址方式

#### 1. 数据处理指令的操作数寻址方式

`<shifter_operand>`通常有下面3中格式：

1. 立即数方式。

   每个立即数由一个8位常数循环右移偶数位得到，其中循环友谊的位数有一个4位二进制的两倍表示，如果立即数记做`<immediate>`，8位常数记做`immed_8`，4位循环右移值记作 `rotate_imm`

   `<immediate> = immed 循环右移（2 * rotate_imm）`

2. 寄存器方式 。

   在寄存器寻址方式下，操作数即为寄存器的数值，如下：

   ```assembly
   MOV R3, R2
   ADD R0, R1, R2
   ```

3. 寄存器移位方式

   寄存器移位方式的操作数为寄存器的数值做相应的移位而得到，具体的移位方式有下面几种：

   - `ASR`：算数右移
   - `LSL`：逻辑左移
   - `LSR`：逻辑右移
   - `ROR`：循环右移
   - `RRX`：扩展的循环右移

#### 2. 字及无符号字节的`Load/Store`指令的寻址方式

#### 3. 杂类`Load/Store`指令的寻址方式

这里说的杂类`Load/Store`指令包括：操作数为半字（无符号或者带符号数）数据；操作数为带符号的字节数据；双字；

#### 4.  批量`Load/Store`指令的寻址方式

#### 5. 协处理器`Load/Store`指令的寻址方式

### 指令集

#### 1. 跳转指令

ARM中有两种方式可以实现程序跳转：一种是跳转指令；另一种是直接向PC寄存器写入目标值

##### B和BL:跳转指令

两者均可以跳转到指令中的目标地址，不同之处是`BL`同时还将`PC`寄存器的值保存到`LR`寄存器中

语法格式：

> `B{L}{<cond>}   <targer_address>`

指令的使用：

BL指令用于实现子程序的调用（如`BL  lable `）。子程序的返回可以通过将LR寄存器的值复制到PC寄存器

- `BX R14`
- `MOV PC , R14`
- 当子程序入口调用了 `STMFD R13! , {<register>, R14}`, 返回可以用`LDMFD R13!, {<register>,PC}`

##### BLX

跳转到目标地址，并将程序状态切换为`Thumb`状态

> BLX  <target_address> 



#### 2. 数据处理指令

大致分3类：数据传送，算数逻辑运算，比较指令

**数据传送指令**用于向寄存器传入一个常数，该指令包括一个目标寄存器和一个源操作数

**算数逻辑运算**包括一个目标寄存器和两个源操作数，该指令将运算结果存入目标寄存器，同时更新`CPSR`中相应的条件标志位

**比较指令**不保存运算结果，只更新`CPSR`中相应的条件位

##### MOV ：数据传送

将`<shifter_opperand>`表示的数据传送到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

| 31 - 28 | 27 - 25 | 24- 21 | 20   | 19-16 | 15-12 | 11  -  0 |
| :------ | ------- | ------ | ---- | ----- | ----- | --------------- |
| cond    | 001     | 1101 | S    | 应为0 | Rd    | shifter_operand |

  > MOV {<cond>} {S} <Rd>, <shifter_operand>

##### MVN

将`<shifter_opperand>`表示的数据的反码传送到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> 格式同MOV

##### ADD : 加法指令

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值相加，把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位



| 31 - 28 | 27 - 25 | 24- 21 | 20   | 19-16 | 15-12 | 11  -  0        |
| :------ | ------- | ------ | ---- | ----- | ----- | --------------- |
| cond    | 001     | 0100   | S    | Rn    | Rd    | shifter_operand |

> ADD {<cond>} {S} <Rd>, <Rn>, <shifter_operand>

**典型用法：**

```assembly
add rx, rx, #1
add rd, rx,rx ,lsl #n
add rs, pc, #offset
```

##### ADC ： 带进位的加法指令

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值相加，在加上`CPSR`中的C条件标志为的值。把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> 格式同 ADD

实现64位数相加：

```assembly
adds r4, r0, r2 @r0,r1分别为高32位和低32位
ADC  r5, r1, r3
```



##### SUB ：减法指令

从`<Rn>`中减去`<shifter_opperand>`表示的数据，把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> SUB {<cond>} {S} <Rd>, <Rn>, <shifter_operand>

```assembly
SUB Rx， Rx， #1 @Rx = Rx - 1 
```



##### SBC ：带位减法指令

从`<Rn>`中减去`<shifter_opperand>`表示的数据，再减去`CPRS`中C条件标志位的反码，把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> 格式同SUB

##### RSB ： 逆向减法

从`<shifter_opperand>`表示的数据中减去`<Rn>`，把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

##### RCS ： 带位逆向减法

从`<shifter_opperand>`表示的数据中减去`<Rn>`，再减去`CPRS`中C条件标志位的反码,把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

##### AND ： 逻辑与

  将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值按位作逻辑与操作，,把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> AND {<cond>} {S} <Rd>， <Rn> , <shifter_operand>

##### ORR ：逻辑或

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值按位作逻辑或操作，,把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> ORR {<cond>} {S} <Rd>， <Rn> , <shifter_operand>

```assembly
ORR R3, R0, R3, LSL #8  @ R3中的数据左移8位，ORR操作将R0的低8位数据获取到
```

##### EOR ：逻辑异或

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值按位作逻辑异或操作，,把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> EOR {<cond>} {S} <Rd>， <Rn> , <shifter_operand>

##### BIC ： 位清除指令

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值的反码，按位作逻辑与操作，,把结果保存到目标寄存器`<Rd>`中，并根据操作数的结果更新`CPSR`中的条件标志位

> BIC {<cond>} {S} <Rd>， <Rn> , <shifter_operand>

##### CMP ：比较指令

将寄存器`<Rn>`的值减去`<shifter_opperand>`表示的数值，根据操作数的结果更新`CPSR`中的条件标志位，后面的指令就可以根据`CPSR`中相应的条件标志位来判断是否执行

> CMP {<cond>} <Rn>, <shifter_oprand>

`CMP`与`SUBS`的区别在于`CMP`指令不保存操作结果

##### CMN ： 基于相反数的比较指令

将寄存器`<Rn>`的值加上`<shifter_opperand>`表示的数值，根据操作数的结果更新`CPSR`中的条件标志位，后面的指令就可以根据`CPSR`中相应的条件标志位来判断是否执行

> CMN {<cond>} <Rn>, <shifter_oprand>

##### TST ： 测试指令

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值按位作逻辑与操作，并根据操作数的结果更新`CPSR`中的条件标志位

> TST {<cond>} <Rn>, <shifter_operand>

##### TSQ ： 相等测试指令

将`<shifter_opperand>`表示的数据与寄存器`<Rn>`的值按位作逻辑异或操作，并根据操作数的结果更新`CPSR`中的条件标志位

> TEQ {<cond>} <Rn>, <shifter_operand>

#### 3. 乘法指令

##### MUL : 32位乘法指令

实现两个32位数（有或无符号）的乘积，并将结果存放到一个32位寄存器中，同时根据运算结果设置`CPSR`寄存器中相应的条件标志位。**考虑指令执行的效率，指令中所有的操作数都放在寄存器中**

> MUL {<cond>}, {S}, <Rd>, <Rm>, <Rs>

```assembly
MUL R0, R1, R2  @ R0 = R1 * R2
```



##### MLA ：32位带加数的乘法

实现两个32位数（有或无符号）的乘积，再将乘积加上第三个数，并将结果存放到一个32位寄存器中，同时根据运算结果设置`CPSR`寄存器中相应的条件标志位。**考虑指令执行的效率，指令中所有的操作数都放在寄存器中**

> MLA  {<cond>}, {S}, <Rd>, <Rm>, <Rs>，<Rn>

```assembly
MUL R0, R1, R2, R3  @ R0 = R1 * R2 + R3
```



##### SMULL： 64位有符号数乘法指令

##### SMLAL ：64位带加数的有符号乘法指令

##### UMULL ：64位无符号乘法指令

##### UMLAL ：64位带加数的无符号乘法指令

#### 4. 杂类的算数指令

- CLZ : 前导0个数计数指令

计算寄存器中操作数最高端0个数

> CLZ {<cond>} <Rd>, <Rm>

#### 5. 状态寄存器的访问指令

##### MRS ：状态寄存器到通用寄存器的传送指令

> MRS {<cond>}, <Rd>, CPSR
>
> MRS {<cond>}, <Rd>, SPSR

**指令的使用：**

主要用于以下3种场合：

1. 通常通过`读取 - 修改 - 写回`操作序列修改状态寄存器的内容，`MRS`用于读取

2. 当异常中断允许嵌套时，需要进入异常中断之后，嵌套中断发生之前保存当前处理器模式对应的`SPSR`

3.  在进程切换时也需要保存当前状态寄存器值

   

##### MSR ：通用寄存器到状态寄存器的传送指令

> MRS {<cond>},  CPSR_<fileds>, #<immediate>
>
> MRS {<cond>},  CPSR_<fileds>, <Rm>

```assembly
MSR CPSR_c, R0 @仅仅修改控制位域
MSR CPSR，R1
```



#### 6. Load/Store内存访问指令

##### LDR ： 字数据读取指令

从内存将一个32位的字读取到指令中的目标寄存器中，如果指令中的寻址方式确定的地址不是字对齐，则从内存独处的数据要进行循环右移操作，移位的位数为寻址方式确定地址的bit[1:0]的8倍

> LDR {<cond>}, <Rd> , <adressing_mode>

**指令的使用**

- 用于从内存读取32位字数据到通用寄存器，然后在该寄存器对数据进行一定操作
- 当PC作为指令中的目标寄存器时，可以实现程序跳转

```assembly
LDR R0, [R1, #4] @内存单元R1+4中的字读取到R0寄存器
LDR R0, [R1, R2] @内存单元R1+R2中的字读取到R0寄存器
LDR R0, [R1], #4 @内存单元R1中的字读取到R0寄存器,然后R1=R1+4
LDR R0, [R1], R2 @内内存单元R1中的字读取到R0寄存器,然后R1=R1+R2
```



##### LDRB ：字节数据读取指令

##### LDRH ： 半字数据读取指令

##### LDRSB ： 有符号的字节数据读取指令

##### LDRSH ： 有符号的半字数据读取指令

##### LDRT ： 用户模式的字数据读取指令

##### LDRBT ： 用户模式的字节数据读取指令

##### STR ： 字数据写入指令

从内存将一个32位的字写入到指定的内存单元

> STR {<cond>}, <Rd> , <adressing_mode>

```assembly
STR R0, [R1, #0X100] @将R0中的字数据写入到内存单元（R1 + 0X100)中
STR R0, [R1], #8   @将R0中的字数据写入到内存单元（R1）中，R1=R1+8
```



##### STRB ： 字节数据写入指令

##### STRH ：半字数据写入指令

##### STRBT ： 用户模式字数据写入指令

##### STRT ： 用户模式字数据写入指令



#### 7. 批量Load/Store内存访问指令

##### LDM(1)：批量内存字数据读取

> LDM {<cond>} <addressing_mode> <Rn>{!} , <register>

```assembly
ldr r0, =0x40000000 @伪指令
LDM r0, {r1,r2,r3}
```

##### LDM(2) ：带状态寄存器的

> LDM {<cond>} <addressing_mode> <Rn>{!} , <register>^

##### STM：批量内存字数据批量写入指令

> STM {<cond>} <addressing_mode> <Rn>! , <register>

#### 8. 信号量操作指令

##### SWP ：交换指令

> SWP {<cond>} <Rd>, <Rm>, [<Rn>]

```assembly
swp r1, r2, [r3] @r3读到r1中，同时r2写入r3中
swp r1, r1, [r2] @r1与r2互换 
```



##### SWPB：字节交换指令

#### 9. 异常中断产生指令

##### SWI：软中断指令

用于产生SWI异常中断，ARM通过这种机制实现用户模式对系统中特权模式的程序调用

> SWI {<cond>}  <immed_24>

**指令的使用：**

- 指令中24位立即数制定了用户请求的服务类型，参数是通过通用寄存器传递
- 指令的24位立即数被忽略，用户请求的服务类型由寄存器R0数值决定，参数通过其他通用寄存器传递

SWI指令由系统完成以下工作

```assembly
R14_svc = SWI指令的下面一套指令的地址
SPSR_svc = CPSR     //保存当前的CPSR
CPSR[4:0] = 0b10011 //使处理器切换到特权模式
CPSR[5] = 0         // 使用程序进入ARM状态
CPSR[7] = 1         // 禁止正常中断响应
                    //程序跳转到相应的中断向量处
if high vectors configured then 
PC = 0xffff0008 
else
pc= 0x00000008
```



##### BKPT ：断点中断指令

#### 10. 协处理指令

协处理器指令包括以下3类：

- 用于ARM处理器初始化ARM协处理器的数据出处操作
- 用于ARM处理器的寄存器和ARM协处理器的寄存器间的数据传送操作
- 用于ARM协处理器的寄存器和内存单元之间传送数据

这些指令包括以下5条：

- CDP  协处理数据操作 
- LDC  协处理数据读取
- STC   协处理器数据写入
- MCR ARM寄存器到协处理器寄存器的数据传送
- MRC 协处理器寄存器到ARM寄存器的数据传送

## 汇编程序设计

### 伪操作

#### 1. 符号定义伪操作

- GBLA, GBLL & GBLS ：声明全局变量

- LCLA，LCLL & LCLS ： 声明局部变量

- SETA，SETL & SETS ：给变量赋值

- RLIST ：为通用寄存器列表定义名称

- CN      ：为协处理器的寄存器定义名称

- CP       ： 为协处理器定义名称

- DN & SN ：为VFP的寄存器定义名称

- FN      ：为FPA的浮点寄存器定义名称

  

#### 2. 数据定义伪操作

- LTORG ：声明一个数据缓冲池的开始
- MAP ：定义一个结构化的内存表的首地址
- FIELD ：定义结构化的内存表中的一个数据域
- SPACE ：分配一块内存单元
- DCB ： 分配一段字节的内存单元，并指定数据初始化
- DCD & DCDU ：分配一段字的内存单元，并用定数据初始化
- DCDO ： 分配一段字的内存单元，并将每个单元的内容初始化成该单元相对于静态基值寄存器的偏移量
- DCFD & DCFDU：分配一段双字内存单元，并用双精度浮点数据初始化
- DCFS & DCFSU ： 分配一段字的内存单元，并用单精度数据初始化
- DCI ：分配一段字节的内存单元，用指定数据初始化，指定内存单元中存放的时代码不是数据
- DCQ & DCQU ： 分配一段半字内存单元，并用64位整数初始化
- DCW & DCWU ：分配一段半字的内存单元 ，并用指令数据初始化
- DATA ：在代码中使用数据，现在已经不再使用

#### 3. 汇编控制伪操作

- IF, ELSE & ENDIF
- WHILE & WEND
- MACRO & MEND
- MEXIT

#### 4. 架构描述伪操作

#### 5. 信息报告伪操作

- ASSERT
- INFO
- OPT
- TTL & SUBT

#### 6. 其他伪操作

- ALIGN
- AREA
- CODE16 & CODE32
- END 
- ENTRY
- EQU
- EXPORT or GLOBAL

### 伪指令

##### 1. ADR(小范围的地址读取伪指令)

该指令基于PC值或者基于及寄存器地址值读取到寄存器中，编译器会用一个ADD或者SUB指令来实现该指令。

> ADR {cond} register , expr

```assembly
start MOV r0 , #10;  // pc值位当前指令地址加8字节
ADR r4, start;       // 被编译器替换为SUB r4, pc , #0xc
```

##### 2. ADRL(中等范围地址读取伪指令)

该指令基于PC值或者基于及寄存器地址值读取到寄存器中，比ADR可以读取更大范围地址。编译器替换成两条指令

> ADRL {cond} register , expr

##### 3. LDR 大范围地址读取伪指令

将一个32位常数或者一个地址读取到寄存器中

> LDR {cond} register, =[expr] [label-expr]

`expr` 为32位常量

- 当`expr`表示的地址没有超过`mov`或`mvn`指令的地址取值范围，编译器会用`mov`或者`mvn`代替
- 当`expr`表示的地址超过`mov`或`mvn`指令的地址取值范围，编辑器将该常数放在数据缓冲区，同时用一条基于`PC`的`LDR`指令读取该常数

```assembly
ldr r1, =0xff5678;
```

##### 4. NOP 空操作伪指令

该伪指令在汇编时被替换成ARM空操作

### 语句格式

### 程序格式

### 汇编实例

##### 1. 子程序跳转





## ARM编译器命令格式

编译器命令格式如下

```
complier [PCS-options] [source-language] [search-paths] [preprocessor-options] [output-format] [target-option] [debug-option] [code-generation-option] [warninf-option] [additional-checks] [error-option] [source]
```

- `complier`可以是`armcc,tcc,armcpp`

- `PCS-options`指定所使用的过程调用标准

- `source-language`指定编译器接受的源程序类型

- `search-path`指定搜索头文件和源文件的路径

  