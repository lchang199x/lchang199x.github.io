---
layout:     post
title:      An Introduction to GCC
subtitle:   GCC使用指南
date:       2017-02-14
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - GCC
    - C & C++
---

>准确地说，用作`C`编译器的命令是`gcc`，而`GCC`指的是`GNU Compiler Collection`。

# 准备工作

1.在`Linux`环境下，检查`gcc`配置
```Bash
#查看gcc是否安装
which gcc

#查看gcc版本
gcc --version

#查看gcc帮助
gcc --help
```

2.`HelloWorld`，其他程序不一一列出
```C
#include <stdio.h>

int
main (void)
{
  printf ("Hello, world!\n");
  return 0;
}
```

# 简单用法

```Bash
#简单用法
gcc hello.c

#-v选项显示编译过程：预处理、编译、汇编、链接
gcc -v hello.c

#gcc有多级代码优化级别-OLEVEL可选
gcc -O2 hello.c
```

1.`gcc`提供代码优化功能

对于优化选项`-OLEVEL`，`-O0`和不用`-O`选项效果相同，不优化。`-O1~-O3`优化级别越来越高（就生成的可执行文件执行速度来说），代价是牺牲可执行文件的大小。

在磁盘/内存受限的平台上，`-Os`选项可以保证生成的可执行文件尺寸最小。

用于调试一般用`-O0`（即不用），开发和发布一般用`-O2`。

2.编译`C++`程序应该用`g++`

`gcc`可以编译`C++`源代码，但是不能正确地链接到`C++`库，因而不能完成`C++`程序的构建(提示`undefined reference`)。

# 通常用法

默认情况下，`gcc`：1）只提示出错信息；2）生成的可执行文件为`a.out`；3）中间过程生成的文件不会保存。

```Bash
#-o选项指定输出文件名，通常作为最后一个选项
gcc hello.c -o hello

#-Wall选项打开大多数常见编译器警告
gcc -Wall hello.c

#--save-temps选项保存所有中间文件（.i, .s, .o）
gcc --save-temps hello.c

#通常用法
gcc -Wall hello.c -o hello

#执行程序，回车之后文件加载到内存中，CPU开始执行其中地指令
./hello
```

# 编译多源文件程序

将一个程序分成多个源文件，更利于编辑和理解，也使得各部分的单独编译成为可能。

```Bash
#头文件hello.h不用包含在命令行中
#include预处理指令会指导编译器在合适的时候自动包含头文件
#include "" 在查找系统头文件目录之前，先搜索当前文件夹
#include <> 只搜索系统头文件
gcc -Wall main.c hello_fn.c -o hello

#如果main1.c和main2.c都含有main函数则报错
gcc main1.c main2.c

#可成功生成main1.o和main2.o，使用通配符*.c更简单
gcc -c main1.c main2.c
```

1.头文件一般用来保存函数声明，编译器通过检查函数声明，来保证函数的参数类型和返回类型在函数调用和函数定义之间正确匹配

2.不可同时直接用`gcc`生成多个可执行程序/多个含有`main`函数的源文件，但是可以同时为多个文件生成中间结果

# 两步编译

通常，编译过程比链接更耗时。采用先编译后链接的方式，多源文件程序修改了部分源文件时只用重新编译这些改动的文件（如果函数原型改了，那么所有用到该函数的文件都要重新编译），并重新链接即可。

`GUN Make`是一个实用工具，可自动化地决定程序中的哪些部分需要重新编译。

```Bash
#默认产生目标文件hello.o，可用-o选项指定其他文件名
gcc -Wall -c hello.c

#目标文件已经是二进制机器码
#但是它对其他源文件中的函数/变量的内存地址的引用为未定义状态
#链接器负责填充这些缺少的内存地址
gcc -Wall -c hello1.c

#这一步就不用用-Wall选项了
#因为在经过了上几步之后这一步只可能链接错误，不会再有警告
gcc hello.o hello1.o -o hello
```

# 四步编译

`gcc`内部使用的预处理器为`cpp`，汇编器为`as`，链接器为`ld`，我们既可以通过`gcc`的选项，也可以直接来使用它们。

```Bash
#预处理
gcc -E -o hello.i hello.c

#预处理内部过程
#用>与用-o选项是一样的
#预处理器负责扩展宏和包含头文件
#c++预处理之后的扩展名为.ii文件
cpp hello.c > hello.i

#编译-->汇编
gcc -S -o hello.s hello.i

#汇编-->二进制
gcc -c hello.s -o hello.o

#汇编内部过程
as hello.s -o hello.o

#链接-->可执行文件
gcc hello.o -o hello

#链接内部过程
#较为复杂，链接的目标文件多数来自于系统库和C运行时库
ld -o hello hello.o xx1.o xx2.o ...
```

# 库

库指的是预先编译好、可以被链接到程序中的目标文件集合。

#### 静态库

```Bash
#库的作用通常是提供系统函数，如math库(/usr/lib/libm.a)，标准库libc.a(/usr/lib/libc.a)
#libc.a每次编译都会默认链接，但是很多编译器都不默认链接libm.a
#IDE一般把常用的库都放在linker的search path里
#生成静态链接库
ar cr libhello.a hello_fn1.o hello_fn2.o

#以下两种写法等价
#-lNAME选项会让linker在standard library directories下搜索libNAME.a文件
gcc -Wall calc.c /usr/lib/libm.a -o calc
gcc -Wall calc.c -lm -o calc
```

1.静态库被保存在`Archive Files`归档文件中，后缀为`.a/.lib`，可由`linux ar/windows lib.exe`工具创建。

2.生成自己的静态库时通常要提供一个库头文件，包含该头文件可以让编译器知道所要使用的函数的原型。

3.不包含头文件编译可能不会出错，因为当使用前没有声明函数时，会根据第一次调用添加隐式函数声明（`C`标准特性），其返回值类型会被默认为`int`，如果链接库中恰好有符合这个隐式声明的函数原型，运行结果就会是对的。

4.`GCC`内建函数：`GCC includes built-in versions of many of the functions in the standard C library`，主要用于优化生成的可执行文件，由此也导致不包含头文件编译时常出现“隐式声明与内建函数‘xxx’不兼容”警告。

5.默认情况下，`gcc`先后在`/usr/local/includ`e和`/usr/include`目录下搜索头文件（`include path: standard include file directories`）。

6.默认情况下，`gcc`先后在`/usr/local/lib`和`/usr/lib`目录下搜索库文件 （`link path: standard library directories `）。

7.`-I`，`-L`选项可分别指定包含路径和链接路径（比标准目录优先级高），或者设置`CPATH/LIBRARY_PATH`环境变量来实现相同的效果（优先级介于`-I/-L`选项和标准目录之间，多个目录用`:`隔开 ，当前目录可用`.`或者留空）。

8.`CPATH`也可根据具体语言(`C/C++`)写成`C_INCLUDE_PATH`和`CPLUS_INCLUDE_PATH`。

#### 动态库

```Bash
#生成动态链接库
#-fPIC选项产生适合动态链接库使用的位置无关机器码Position-Independent Code
#-shared选项此处指定生成动态链接库
gcc -fPIC hello_fn1.c hello_fn2.c -shared -o libhello.so
```

1.动态库后缀为`.so/.dll`，链接动态库生成的可执行文件仅包含它所需函数的一个表，而不是将所需函数的二进制代码都从目标文件中拷贝过来。

2.动态链接的可执行程序较小，因本地只要一个备份而节省磁盘空间，运行时因动态库复用而节省内存空间，而且升级库可以不必重新编译使用它的程序（当然接口不能变）。

3.如果动态库可用（在链接路径中且使用`-lNAME`而不是绝对路径指定库名），`gcc`将优先使用库的动态版本（使用`-static`选项/使用静态库的绝对路径可强制静态链接）.

4.动态链接的可执行文件在执行时会首先加载动态库到内存中，如果动态库不在链接路径中将执行失败。这时可以修改`LD_LIBRARY_PATH`环境变量，或者添加到配置文件`/etc/profile`或修改`/etc/ld.so.conf`文件来实现全局配置。
