## C语言遗漏点

### define

宏定义中有几个符号需要注意，如`#`，`##`，`\`，`#@`,下面定义了三个宏

```c
#define Conn(x,y) x##y
#define ToChar(x) #@x
#define ToString(x) #x
```

#####  1. `##` 链接操作符

```c
int n = Conn(123,456);
     ==> int n=123456;
char* str = Conn("asdf", "adf");
     ==> char* str = "asdfadf";
```

需要注意的是 `##` 左右符号必须能够组成一个有意义的符号，否则会报错。

##### 2. `#@`字符化操作

> `#@x`只能用于有传入参数的宏定义中，且必须置于宏定义体中的参数名前。作用是将传的单字符参数名转换成字符，以一对单引用括起来其实就是给`x`加上单引号，结果返回是一个`const char`。

举例说：

```c
char a = ToChar(1);
     ==> char a='1';
```

做个越界试验

```c
char a = ToChar(123);
     ==> char a='3';
```

但是如果你的参数超过四个字符，编译器就给给你报错了

```
！error C2015: too many characters in constant ：P
```

##### 3. `#` 字符串化操作符
> #表示字符串化操作符（stringification）。其作用是：将宏定义中的传入参数名转换成用一对双引号括起来参数名字符串。其只能用于有传入参数的宏定义中，且必须置于宏定义体中的参数名前。说白了，他是给x加双引号：

> 

```c
 char* str = ToString(123132);
 ==> char* str="123132";
```

如果你想要对展开后的宏参数进行字符串化，则需要使用两层宏。

```c
#define xstr(s) str(s)
#define str(s) #s
#define foo 4
str (foo)
     ==> "foo"
xstr (foo)
     ==> xstr (4)
     ==> str (4)
     ==> "4"
```

`s`参数在`str`宏中被字符串化，所以它不是优先被宏展开。然而`s`参数是`xstr`宏的一个普通参数，在被传递到`str`宏之前已经被宏展开。

##### 4. `\` 行继续操作

> `\` 行继续操作当定义的宏不能用一行表达完整时，可以用`\`（反斜线）表示下一行继续此宏的定义。

##### 5. VA_ARGS

###  

`__VA_ARGS__`宏用来接受不定数量的参数。例如：

```c
#define eprintf(...) fprintf (stderr, __VA_ARGS__)

eprintf ("%s:%d: ", input_file, lineno)
==>  fprintf (stderr, "%s:%d: ", input_file, lineno)
```

当`__VA_ARGS__`宏前面`##`时，可以省略参数输入。
 例如：

```c
#define eprintf(format, ...) fprintf (stderr, format, ##__VA_ARGS__)

eprintf ("success!\n")
==> fprintf(stderr, "success!\n");
```

### typedef 

用户自定义类型来替代系统基本类型。

> **typedef中声明的类型在变量名的位置出现**

##### 1.  为数组定义类型

  ```c
  typedef int INT_ARRAY_100[100];
  INT_ARRAY_100 arr;
  ```

##### 2. 为指针定义类型


  ```c
  typedef char* PCHAR;
  PCHAR pa;
  ```

  **与宏定义的区别 **：

  ```c
  #define MYCHAR char*
  MYCHAR a, b;// a是char*,b是char
  PCHAR c, d; // c和d都是 char*
  ```

##### 3. 陷阱



- ```c
  typedef char* PCHAR;
  int strcmp(const PCHAR,const PCHAR);
  ```

在上面的代码中，`const PCHAR` 是否相当于`const char*` 呢？

答案是否定的，原因很简单，`typedef` 是用来定义一种类型的新别名的，它不同于宏，不是简单的字符串替换。因此，`const PCHAR`中的  `const ` 给予了整个指针本身常量性，也就是形成了常量指针`char*const`（一个指向`char`的常量指针）。即它实际上相当于`char*const`，而不是`const  char*`（指向常量 char 的指针）。当然，要想让 `const PCHAR` 相当于 `const char*` 也很容易，如下面的代码所示： 

```c
typedef const char* PCHAR;
int strcmp(PCHAR， PCHAR);
```

 其实，无论什么时候，只要为指针声明 typedef，那么就应该在最终的 typedef 名称中加一个 const，以使得该指针本身是常量。

- 
   还需要特别注意的是，虽然 `typedef` 并不真正影响对象的存储特性，但在语法上它还是一个存储类的关键字，就像 `auto`、`extern`、`static` 和 `register` 等关键字一样。因此，像下面这种声明方式是不可行的：

  ```c
  typedef static int INT_STATIC;
  ```

### 数组

##### 1.数组名为常量

```c
int array[4];
array++; // 错误，不可使用  ++
int *p = array;
p++;     //正确
```

##### 2. 数组首地址

数组名字即为数组首地址

```c
int array[5];
array <=> &array[0]; //都是首元素地址 类型为 int*
&array      //类型指向 int [5]的指针类型，数组指针
```



### switch

switch 类型不能是 `float`,`double`,`bool`

### static

局部变量：延长生命中期

全局变量：限制作用域

函数 ： 限制作用域

### 逗号运算符

### {}作用域

