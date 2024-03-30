# 链接与程序结构

本章主要是对 CSAPP 这本书第七章的整理.

这一章会简单介绍链接和程序的结构。事实上到做毕设我才体会到链接的力量：将一个大工程分解为各个可以独立修改、编译的模块。

## 1. ELF 文件结构

可以 `man 5 elf` 来获得关于 ELF 文件的信息。

要查看程序 riscv 文件信息是这样的：

```bash
riscv64-linux-gnu-readelf -a <filename>
riscv64-linux-gnu-objdump -d <filename>
```

elf 文件的魔数：0x7F,0x45,0x4c,0x46,

别人也有一些介绍，比如：

<https://zhuanlan.zhihu.com/p/286088470>

<https://blog.csdn.net/npy_lp/article/details/102604380>

ELF 是 linux 系统下的可执行目标文件格式。理解它就理解了程序的结构。

先来一张图来介绍 ELF 的文件结构。我们可以从 section（节）和 segment（段）两种视角来看 ELF 文件是怎么组织的。

<div style="text-align:center;">
    <img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/links1.jpg" style="width:50%; height:50%;">
</div>

### Section 视角

Section 视角提供了用于链接与重定位的信息。从节的角度，ELF header（ELF 头）介绍了这个 ELF 文件的基本信息，比如大小、机器类型、程序入口地址、Section header table （节头部表）的文件偏移等等。ELF 头之后是 Segment header table（程序头表），包括了段的信息，段在后面会介绍。文件的末尾是节头部表，描述了不同节的位置和大小。夹在中间的都是节。

下面介绍一些典型的节。

- `.text`：程序的机器代码
- `.rodata`：只读数据（read-only data）。比如说常量和常量字符串，switch 语句的跳转表都储存在这里。对只读数据段内存尝试修改将会导致段错误。
- `.data`：已初始化的全局和静态 C 变量。注意，局部变量不在 `.data` 或者 `.bss` 节中，而是储存在栈里。
- `.bss`：未初始化的或者被初始化为 0 的全局或者静态变量（反正就是 0）。在文件中，这个节不占据任何的磁盘空间，只有在运行时，才在内存中分配给这些变量空间，初始值为 0。这样做大大提高了空间效率，也是 `.bss` 节为什么总是全都为 0 的原因。
- `.symtab`：符号表，存放函数和全局变量的信息。
- `.rel.text`: `.text` 节中位置的列表, 这些位置可能需要被重定位.
- `.rel.data`: 所有全局变量的重定位信息. 如果全局变量的值是一个地址, 那么就可能被修改.  
- `.debug`：调试符号表，只有以 `-g` 选项编译才能得到这张表，包络必要的调试信息。
- `.strtab`：一个字符串表，包括所有节的节名。

### Segment 视角

从段的视角，我们可以把 ELF 主要分为三段：代码段（`.text`，`.rodata`）、数据段（`.data`，`.bss`）和不加载到内存的符号表。其中，代码段只读，而数据段有读写权限。


Linux 程序调用 `execve` 函数来调用加载器，加载器将可执行目标文件的代码和数据复制到内存中，然后跳转到程序的入口点来运行程序。

下面我们来看一下程序运行时的内存映像。

<div style="text-align:center;">
    <img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/links2.jpg" style="width:50%; height:50%;">
</div>

在 Linux 系统中，程序的代码段总是从地址 `0x400000` 处开始，后面是数据段。堆在数据段之后，调用 `malloc` 库向上增长。用户栈从最大的合法用户地址开始（2**48-1），向下增长。用户栈之上的区域是内核的代码和数据，对用户不可见。


???+ note "练习"

    1. 为什么下面的语句会导致段错误？
    ```c
    char *p = "abcde";
    char *q = "edcba";
    strcpy(p, q);
    ```

    2. 使用 readelf 查看一个 ELF 文件的信息, 你会看到一个 segment 包含两个大小的属性, 分别是 FileSiz 和 MemSiz, 这是为什么? 再仔细观察一下, 你会发现 FileSiz 通常不会大于相应的 MemSiz, 这又是为什么?
    提示：MemSiz 是文件在磁盘中的大小，FileSiz 是文件运行时在内存中的大小。

## 2. 链接

这个[教程](https://blog.csdn.net/m0_37621078/article/details/88376228) 非常详细地介绍了如何在 linux 系统下使用静态库和动态库, 瑕不掩瑜的是, 有一些理解还不够准确. 下面我就按教材来整理了.

写完一个程序以后, 如果调用 gcc 来编译成可执行文件, 经历这样的步骤: 首先运行 C 预处理器, 将源程序的宏定义等等替换; 然后运行 C 编译器, 将 C 文件翻译成 .s 汇编文件; 接下来运行汇编器, 将 .s 汇编文件翻译成**可重定位目标文件(relocatable object file)**, 以 .o 为后缀名. 最后运行链接器, 将各个 .o 目标文件组合成一个可执行目标文件. 只需调用加载器, 就可以运行这个文件. 我们下面的重点, 就是放在链接这个步骤上面.

提一嘴, 这个 object 翻译成目标, 使我看书看的云里雾里.

ld 是 linux 系统下典型的静态链接器, 它以一组 .o 可重定位目标文件作为输入, 生成一个可执行目标文件作为输出. 输入的可重定位目标文件包含了各种不同的代码和数据节, 这些节的内容已经在前面介绍了. 链接器构造的可执行目标文件就是这些可重定位目标文件的结合.

### 符号解析与重定位

为了构造可执行文件, 链接器必须完成两个主要任务: 符号解析和重定位.

符号就是函数和变量的名字. 符号解析的工作就是把每个对符号的引用和一个确定的符号定义联系起来, 编译器只允许每个模块的局部符号有一个定义. 以函数为例, 符号引用就是函数的声明, 符号的定义就是函数定义, 函数可以被声明多次, 但是只能被定义一次. (我们假定你了解 static 的细节, 这些描述只是概括).

完成了所有的符号解析, 接下来就是重定位了. 重定位由两步组成: 

1. 重定位节和符号定义. 这一步中, 链接器将所有相同类型的节合并成一个新的聚合节, 然后将运行时内存地址赋给新的聚合节, 以及定义的每个符号, 这样程序的每条指令和全局变量都有唯一的地址了.
2. 重定位节中的符号引用. 连接器修改代码和数据节中对每个符号的引用, 使得它们指向正确的运行时地址.


### 静态库

前面我们都是直接连接所有的 .o 文件并把它们链接起来形成一个输出的可执行文件. 但是实际上还有一种静态库机制: 把所有相关的目标模块打包成为一个单独的文件作为静态库用作链接器的输入, 但是链接器在构造输出时, 只复制静态库里被应用程序引用的模块.

Linux 静态库命名规范，必须是`lib[your_library_name].a`：lib 为前缀，中间是静态库名，扩展名为 .a. 静态库是存档文件(archive), 是一组连接起来的 .o 文件的集合.

以标准库的例子为例: C 语言标准库和数学库中的函数被封装在 `libc.a` 中. `libc.a` 的大小有大约 5 MB. 如果每一个调用 `printf` 函数的程序都链接一整个标准库的话, 那么即便是最简单的打印 `hello world` 程序都会有几兆的大小. 但实际当然不是这样, 链接器只选择了标准库中的 `printf.o` 和几个 `printf.o` 调用的模块进行链接, 因此程序的大小只有几 KB. 

假设我们这样的例程:

```c
// addvec.c
int addcnt = 0;
void addvec(int *x, int *y, int *z, int *n)
{
    addcnt++;
    int i;
    for(i<0;i<n;i++)
        z[i] = x[i] +y[i];
}

//multvec.c
int multcnt=0;
void multvec(int *x, int *y, int *z, int *n)
{
    multcnt++;
    int i;
    for(i<0;i<n;i++)
        z[i] = x[i] * y[i];
}

//main2.c
#include <stdio.h>
#include "vector.h"
int x[2] = {1,2};
int y[2] = {3,4};
int z[2];
int main()
{
    addvec(x,y,z,2);
    printf("z = [%d %d]\n", z[0], z[1]);
    return 0;
}
```

首先创建上面两个函数的静态库:
```bash
gcc -c addvec.c multvec.c
ar rcs libvector.a addvec.o multvec.o
```
创建可执行文件:
```bash
gcc -c main2.c
gcc -static -o prog2c main2.o -L. -lvector
```

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/link3.png"/>


### 动态库

静态库的缺点是, 如果静态库更新, 那么程序就要重新和更新的库链接. 除此之外, 如果有多个进程同时使用相同的函数, 每个进程都要复制一份相同的函数代码, 这样浪费空间.

于是就引入了所谓的动态共享库, 也叫共享目标, 在运行和加载时可以加载到任意的内存地址并和内存中的程序连接起来. 共享库的好处是, 多个进程可以共用内存中相同的库代码, 从而大大节省了内存资源.

Linux 动态库命名规范，必须是`lib[your_library_name].so`：lib 为前缀，中间是动态库名，扩展名为 .so.

接上面的例子, 生成位置无关的代码
```bash
gcc -shared -fpic -o libvector.so addvec.c multvec.c
```
链接到示例程序
```bash 
gcc -o prog2l main2.c ./libvector.so
```

这里链接时, 只复制了一些动态库的重定位和符号表信息, 在程序运行时, 才由动态链接器将库中的内容复制到内存中; 而静态库是在编译过程就完成了这些动作.

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/link4.png"/>

### 位置无关代码

共享库中代码可以被加载到内存的任何位置而无需链接器修改, 这种代码叫位置无关代码(PIC). 编译器用延迟绑定(lazy binding)技术来实现 PIC 函数调用.