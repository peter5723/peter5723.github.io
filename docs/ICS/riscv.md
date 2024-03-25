# RISC-V

这篇用来介绍 risc-v 的学习记录。

## 0. 介绍

什么是 risc-v？我觉得很可悲的一点是，我去年学了小白老师上的汇编语言，在本专业又学了嵌入式系统这门课，在这个月之前，我连什么是 ISA 都没有搞清楚。所以，一门介绍计算机系统的课是很值得上的吧。

我们写出来的高级语言，被编译器翻译成汇编语言，汇编语言则和机器代码一一对应，机器代码可以直接执行。
程序就是这么跑起来的。
而实际上，对于不同的机器（处理器），它能支持的汇编语言也是不一样的。
我们把一套汇编构成的指令集体系结构称为 ISA (Instruction-set Architecture)。
一种处理器只能支持它支持的 ISA，
毕竟做同样的事情的汇编指令，不同的 ISA 对应的机器码是不同的。
现在市面上已经有了很多不同的处理器，它们所使用的 ISA 都是不一样的。
比如，我们的计算机通常使用 Intel 处理器，它使用的 ISA 就是 x86 了。
我们的手机等很多终端设备使用 Arm 处理器，用的 ISA 就是 Arm 和 Thumb。
除此之外，非常常用的 ISA 还有 misp 和 risc-v。
这篇博客主要就是介绍 risc-v 的。

???+ note "Some notes of x86"

    开始 riscv 之前，简单讲几句 x86。

    在小白的课上学习的 8086 汇编，正是 x86 的一种，只不过是 16 位的汇编，非常古老了。值不值得用它来学习 x86，见仁见智。但不管 16 位，32 位还是 64 位，本质上是一样的，在 windows 下反汇编 （可以用 IDA、vs 等），汇编都是 x86 的。所以学习了 16 位以后向外扩展还是很容易的。

    然后我记得在 csapp 中，要在 Linux 系统下，调试著名的 bomb lab，用 objdump 进行反汇编，得到的汇编我还以为又是一种新的汇编语言。但是实际上得到的汇编仍然是属于 x86 ISA 的。

    两者实际上只是语法风格的不同。windows 下更常用的 x86 风格称作 Intel 风格；而 Linux 下更常用的风格称作 AT&T 风格。
    
    举个例子同样的语句，将寄存器 rdx 中的值复制到寄存器 rbx 中，在 Intel 风格下写作：
    ```
    mov rbx, rdx
    ```
    目标操作数在前，源操作数在后。

    而在 AT&T 风格下写作：
    ```
    movq %rdx, %rbx
    ```
    源操作数（source）在前，目标操作数（destination）在后。

    x86 是典型的 CISC ISA，就是说它的指令数量非常多，很繁杂。与此相对应的就是 RISC，精简化指令集，指令数量更少。
    ARM 和 risc-v 都属于 RISC 指令集。

下面就介绍 riscv 了。开始之前，先放一些可能需要的链接和资料吧：

优秀笔记：<https://note.tonycrane.cc/cs/pl/riscv/>

官网：<https://riscv.org/>

官方资料：<https://riscv.org/technical/specifications/>

教材：《计算机组成与设计：硬件软件接口（原书第5版·RISC-V版）》

实际上我很想转载别人的文章，因为链接可能会失效。但是可能会有些不必要的麻烦，那就算了吧。

## 1. 例子
直接讲指令有点抽象，我们来看一个实例吧。下面的代码段打印了 hello world。
```asm
    .text
    .align 2
    .globl main
main:
    addi sp, sp, -16
    sw ra, 12(sp)
    lui a0, %hi(string1)
    addi a0, a0, %lo(string1)
    lui a1, %hi(string2)
    addi a1, a1, %lo(string2)
    call printf
    lw ra, 12(sp)
    addi sp, sp, 16
    li a0, 0
    ret

    .section .rodata
    .balign 4
string1:
    .string "Hello, %s!\n"
string2:
    .string "world"
```

可以看到，程序的结构和 x86 还是有很多相似之处的。

## 2. RV32I 指令集

RV32I 是 RISCV 的基础整数指令集。指令集中的所有指令都是 32 位的，在一个机器周期内就能完成。

RV32I 的指令可以分为 6 种：

- R 型，寄存器-寄存器操作
- I 型，端立即数和访存 load 操作
- S 型，访存 store 操作
- B 型，条件跳转操作
- U 型，长立即数操作
- J 型，无条件跳转

我们来看一看这 6 种指令所对应的机器码的格式。

### R 型指令

我们可以将一条 R 型指令分成下面的字段：

<!-- ps, 要加上对应的 css 样式才能成功显示下面的表格 -->
<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">25</td>
    <td class="riscv-table-numnodel">24</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">20</td>
    <td class="riscv-table-numnodel">19</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">15</td>
    <td class="riscv-table-numnodel">14</td>
    <td class="riscv-table-numnode" colspan="1"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="7" class="riscv-table-node">funct7</td>
    <td colspan="5" class="riscv-table-node">rs2</td>
    <td colspan="5" class="riscv-table-node">rs1</td>
    <td colspan="3" class="riscv-table-node">funct3</td>
    <td colspan="5" class="riscv-table-node">rd</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

下面是每个字段名称的含义：

- opcode：操作码，指示指令的基本操作，确定这条指令的类型
- rd：目的操作数寄存器，存放操作结果（register destination）
- funct3：一个另外的操作码字段
- rs1：第一个源操作数寄存器（register source）
- rs2：第二个源操作数寄存器
- funct7：一个另外的操作码字段。

很明显的，R 型里面都是寄存器之间的操作，比如说 add 指令就是一个 R 型操作。看下面的一个指令例子：

```
add x9, x20, x21
```
它执行 C 语言中 `x9 = x20 + x21;` 这个操作。

我们也可以把这条指令以二进制的形式表示：
<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">25</td>
    <td class="riscv-table-numnodel">24</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">20</td>
    <td class="riscv-table-numnodel">19</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">15</td>
    <td class="riscv-table-numnodel">14</td>
    <td class="riscv-table-numnode" colspan="1"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="7" class="riscv-table-node">0000000</td>
    <td colspan="5" class="riscv-table-node">10101</td>
    <td colspan="5" class="riscv-table-node">10100</td>
    <td colspan="3" class="riscv-table-node">000</td>
    <td colspan="5" class="riscv-table-node">01001</td>
    <td colspan="7" class="riscv-table-node">0110011</td>
</tr>
</table>

func7、func3 和 opcode 字段组合起来确定是 add 操作。其中 opcode 字段确定是这条指令的类型是 R 型；然后再由 funct3 和 funct7 确定是 add。

剩下的字段就是用来表示寄存器操作数了。

### I 型指令

<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="10"></td>
    <td class="riscv-table-numnoder">20</td>
    <td class="riscv-table-numnodel">19</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">15</td>
    <td class="riscv-table-numnodel">14</td>
    <td class="riscv-table-numnode" colspan="1"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="12" class="riscv-table-node">imm[11:0]</td>
    <td colspan="5" class="riscv-table-node">rs1</td>
    <td colspan="3" class="riscv-table-node">funct3</td>
    <td colspan="5" class="riscv-table-node">rd</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

新的字段 imm 表示立即数。

I 型使用两个寄存器和和一个立即数进行运算。如果像 R 型那样，只留 5 位给立即数，那能表示的立即数太少了。所以如此设计，可以表示更大的立即数（12位可以表示 -2048 ~ 2048）。 

除此之外，load 类型的操作也属于 I 型的，比如 `ld x9, 240(x10)`。立即数用来储存变址寻址的偏移量。

I 型的运算类型由 opcode 和 funct3 决定。

立即数是 {{20{inst[31]}}, inst[31:20]}。

### S 型指令

<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">25</td>
    <td class="riscv-table-numnodel">24</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">20</td>
    <td class="riscv-table-numnodel">19</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">15</td>
    <td class="riscv-table-numnodel">14</td>
    <td class="riscv-table-numnode" colspan="1"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="7" class="riscv-table-node">imm[11:5]</td>
    <td colspan="5" class="riscv-table-node">rs2</td>
    <td colspan="5" class="riscv-table-node">rs1</td>
    <td colspan="3" class="riscv-table-node">funct3</td>
    <td colspan="5" class="riscv-table-node">imm[4:0]</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

S 型主要用来进行 store 操作，它可以变址寻址。比如 sd 指令就是 S 型的。
举个例子：
```
sd x9, 240(x22) // x9 中的数据储存回 x22 中的地址后移 240 位的地址中。
```
为了保持寄存器 rs1 和 rs2 的位置一致，所以将一个立即数分成两部分表示。比如说，这里 (240)~10~ = (0000111 10000)~2~，那么 0000111 将会保存到 imm[11:5]，10000 将会保存到 imm[4:0]。

S 型的立即数是 {{20{inst[31]}}, inst[31:25], inst[11:7]}。
### B 型指令

<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">25</td>
    <td class="riscv-table-numnodel">24</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">20</td>
    <td class="riscv-table-numnodel">19</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">15</td>
    <td class="riscv-table-numnodel">14</td>
    <td class="riscv-table-numnode" colspan="1"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="7" class="riscv-table-node">imm[12,10:5]</td>
    <td colspan="5" class="riscv-table-node">rs2</td>
    <td colspan="5" class="riscv-table-node">rs1</td>
    <td colspan="3" class="riscv-table-node">funct3</td>
    <td colspan="5" class="riscv-table-node">imm[4:1,11]</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

B 型指令本质上是 S 型的变种，主要区别就是立即数读取的顺序不同。它用于分支跳转指令，比如 beq，相等时跳转，对应 C 语言的 `if(rs1 == rs2) PC += imm`。

可以看到，B 型和 S 型都需要两个寄存器和一个立即数进行操作。

立即数是 {{19{inst[31]}}, inst[31], inst[7], inst[30:25], inst[11:8], 1'b0}。前面是符号位填充到 32 位，补 1 位 0。
### U 型指令

<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="18"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="20" class="riscv-table-node">imm[31:12]</td>
    <td colspan="5" class="riscv-table-node">rd</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

很显然，就是一个寄存器和一个立即数参与操作；而且没有源操作数。
例子就是 lui，对应的 C 语言描述是 `rd = imm << 12`

U 型操作只能通过 opcode 来区分。

立即数是 {inst[31:12], 12'b0}

### J 型指令

<table class="riscv-table">
<tr>
    <td class="riscv-table-numnodel">31</td>
    <td class="riscv-table-numnode" colspan="18"></td>
    <td class="riscv-table-numnoder">12</td>
    <td class="riscv-table-numnodel">11</td>
    <td class="riscv-table-numnode" colspan="3"></td>
    <td class="riscv-table-numnoder">7</td>
    <td class="riscv-table-numnodel">6</td>
    <td class="riscv-table-numnode" colspan="5"></td>
    <td class="riscv-table-numnoder">0</td>
</tr>
<tr>
    <td colspan="20" class="riscv-table-node">imm[20,10:1,11,19:12]</td>
    <td colspan="5" class="riscv-table-node">rd</td>
    <td colspan="7" class="riscv-table-node">opcode</td>
</tr>
</table>

U 型指令的变种，用于无条件跳转指令，比如 jal。

立即数是 

{{11{inst[31]}}, inst[31], inst[19:12], inst[20], inst[30:21], 1'b0}

前面符号填充，注意最后一位要补上 0。还没有找到这样的原因，但是这个 0 浪费了我很久的时间，先记一下。
## 3. 伪指令

除了上面说的六种指令外，我们也可以用在汇编代码中使用伪指令，在实际编译时，它们会被上面的指令替代。

可以看看下面的几个例子：

| Pseudoinstruction | Base Instruction(s)                                               | Meaning              |
|-------------------|-------------------------------------------------------------------|----------------------|
| nop               | addi x0, x0, 0                                                    | No operation         |
| mv rd, rs        | addi rd, rs, 0                                                    | Copy register        |
| j offset          | jal x0, offset                                                    | Jump                 |
| ret               | jalr x0, x1, 0                                                    | 通过返回地址 x1 返回 |
| call offset       | auipc x1, offset[31 : 12] + offset[11] </br> jalr x1, offset\[11:0\](x1) | 远调用               |

??? "tips"

    1. 怎么插入 markdown 表格？
        可以在这个网站快速生成：<https://www.tablesgenerator.com/markdown_tables>
    2. html 语言，用 `</br>` 表示换行。举个例子：</br>
    3. 可以看到，这些伪代码和有些和 x86 中学过的是很相似的，可以比较容易的迁移学习。
## 4. 指令表

实际上，riscv 的指令是如此的简单，以至于他们很容易就被整理在几张表上了：

cs61c 的表：

<https://github.com/jameslzhu/riscv-card/blob/master/riscv-card.pdf>

官方表：

<https://www.cl.cam.ac.uk/teaching/1617/ECAD+Arch/files/docs/RISCVGreenCardv8-20151013.pdf>

有很多人都做了表，我就不整理指令了。（真的不是因为偷懒吗

## 5. 寄存器

上面代码中出现过寄存器了，下面介绍一下。


寄存器包括 1 个 PC 寄存器和 32 个 32 位特殊寄存器。它们的功能简要介绍如下表：

|寄存器|ABI 名称|用途描述|saver|
|:--:|:--:|:--|:--:|
|x0|zero|硬件 0||
|x1|ra|返回地址（return address）|caller|
|x2|sp|栈指针（stack pointer）|callee|
|x3|gp|全局指针（global pointer）||
|x4|tp|线程指针（thread pointer）||
|x5|t0|临时变量/备用链接寄存器（alternate link reg）|caller|
|x6-7|t1-2|临时变量|caller|
|x8|s0/fp|需要保存的寄存器/帧指针（frame pointer）|callee|
|x9|s1|需要保存的寄存器|callee|
|x10-11|a0-1|函数参数/返回值|caller|
|x12-17|a2-7|函数参数|caller|
|x18-27|s2-11|需要保存的寄存器|callee|
|x28-31|t3-6|临时变量|caller|


## 6. 特权级指令

TODO


## 7. 分页机制

参考:

[鹤翔万里的笔记本 RISC-V 页表相关](https://note.tonycrane.cc/cs/pl/riscv/paging/)

[riscv 的分页机制](https://junimay.github.io/wiki/RISCV/%E5%88%86%E9%A1%B5%E6%9C%BA%E5%88%B6/)

[官方文档](https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf)

分页机制的基本原理在虚拟内存的地址翻译介绍了, riscv 也是大同小异. 这里只介绍最基本的 SV32 分页机制.

首先介绍一下将要用的符号:

| 符号 | 描述                                    |
|------|-----------------------------------------|
| VA   | virtual address 虚拟地址                |
| VPO  | virtual page offset 虚拟页面偏移量      |
| VPN  | virtual page number 虚拟页号            |
| PA   | physical address 物理地址               |
| PPO  | physical page offset 物理页面偏移量     |
| PPN  | physical page number 物理页号           |
| PTBR | page table base register 页表基址寄存器 |
| PTE  | page table entry 页表项                |

riscv 有 PTBR 功能的是 satp 寄存器, 用于设置页表相关项. 其结构如下
```
                            +---+-----------+--------------------------+
    satp register           | M |   ASID    |            PPN           |
                            +---+-----------+--------------------------+
                            (1b)    (9b)               (22b)
```

其中, PPN 是根页表物理页号;  ASID 是地址空间 ID; M 是分页模式, 0 代表不翻译, 1 代表采用 SV32 模式进行翻译. 

SV32 的地址布局如下:

```
                            +----------+----------+------------+
  Virtual Address           |  VPN[1]  |  VPN[0]  |   offset   |  -----> 32 bits
                            +----------+----------+------------+
                             (10 bits)  (10 bits)    (12 bits)

  Physical Address        +------------+----------+------------+
                          |   PPN[1]   |  PPN[0]  |   offset   |  -----> 34 bits
                          +------------+----------+------------+
                             (12 bits)   (10 bits)    (12 bits)

                    +------------+----------+--+-+-+-+-+-+-+-+-+
  Page Table Entry  |   PPN[1]   |  PPN[0]  |  |D|A|G|U|X|W|R|V|  -----> 32 bits
                    +------------+----------+--+-+-+-+-+-+-+-+-+
                       (12 bits)   (10 bits) |
                                             `- RSW (2 bits)
```

上面就是 VA, PA 和 PTE 的结构. 注意 PTE 前面储存的是下一级物理页号/物理地址的高 20 位, 剩下的是 flag, 如最后一位 V 是有效位.

下面是 SV32 地址翻译的流程图, 采用了两级页表.

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/20240326000545.png"/>

最后是地址翻译的具体过程, 在 RISC-V Privileged Spec 中的 4.3.2 节. 

1. 获得页表的基地址 a,  和当前处在级数 i. 可知 a = satp.ppn × PAGESIZE, i = LEVELS - 1. 对 SV32 来说, PAGESIZE = 2^12, 两级页表 LEVELS = 2, 故 a = setp.ppn << 12, i = 1.
2. 读取这一级的 PTE 项, pte = *(a + va.vpn[i] × PTESIZE), 其中 PTESIZE = 4.
3. 如果 i = 0(或者利用 pte.R == 1 || pte.X == 1), 那么说明得到了物理地址, 退出到 4. 否则继续循环, 获得下一级页表的基地址, 即更新 a = pte.ppn × PAGESIZE, 更新 i = i - 1, 跳转到 2.
4. 经过检验, 若都合法, 则翻译完成: pa.pgoff = va.pgoff, pa.PPN[1:i] = pte.PPN[1:i].

