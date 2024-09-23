# xv6 源码阅读

xv6 有两个 cpu（hart）

## 1. 启动第一个进程

首先我们来看一下如何启动 xv6 以及开始第一个进程吧。（课程教材的 2.6 节）

机器启动后，xv6 从 `kernel/entry.S` 开始运行。启动时硬件分页机制未打开，物理地址等于虚拟地址。qemu 将 xv6 加载到地址 `0x80000000`,然后`_entry` 中设置栈， 栈的地址在 `start.c` 中确定，大小为 4096 乘以 CPU 个数，将 sp 的位置设置为栈顶（stack0+4096\*id）。设置好栈后，跳转到 `start()` 函数。


`start()` 函数的工作首先是在 machine 模式下进行一些必要的配置，然后切换到 supervisor 模式。这些配置包括，将 `main` 函数的地址写到 mepc 寄存器中，将 0 写入 satp 寄存器中使得在 supervisor 模式下，关闭页表地址的转换。然后打开时钟中断。执行完这些之后，使用 `mret` 进入 s 模式，并跳转到 `main()` 函数。

在 `main()` 函数中进行一系列必要的初始化后，调用 `user_init()`（在`proc.c`）来创建第一个进程（initcode.S）。这个进程执行 `exec` 系统调用，打开 `init` 程序，`init` 程序(init.c)打开shell，就这样完成了第一个进程，启动了系统。


## 2. 创建新进程

在 `proc.c` 中。主要是 `allocproc()` 函数，首先给新进程创建一个新的 `pid`，使用 `allocpid()` 函数；再给进程分配 `trapframe` （PCB），这里面储存进程中断需要的变量；最后为用户进程分配页表，用于虚拟地址转化为物理地址。`vm.c` 中的 `mappages` 函数创建新的 PTE（页表项），而硬件 mmu 根据这些页表物理内存地址转化为虚拟内存地址（qemu）的工作。

系统分配空间用的都是 `kalloc.c`  中定义的 `kalloc` 函数，一次分配 4096 Byte 的内存，是一个页表的大小。`kalloc` 函数返回的是物理地址。


## 3. 系统调用的过程


xv6 内核地址映射：

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/oslab4.1.png"/>

用户地址映射：

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/oslab4.2.png"/>

举个例子，用户的 `echo` 指令调用了 `write` 函数。`write` 函数是一个系统调用，定义在 `usys.h` 中（用 perl 生成）。它调用 `ecall` 指令，进入内核态。`stvec` 寄存器储存的地址是 `trampoline.s` 中的 `uservec` 函数。储存好上下文后，跳转到 `trap.c` 中的 `usertrap` 函数。这里面我们运行一个 `syscall` 函数，根据传入的参数搜索系统调用，此时即 `sys_write`。调用完毕后，调用 `usertrapret` ，回到 `trampoline.s` 的 `userret`，恢复上下文返回。

trampoline 本意是蹦床。其代码在用户态和内核态之间切换，状态像蹦床一样跳来跳去，所以取这个名字。

### 问题 1：页表切换

内核态和用户态所用的页表是不同的，那么该怎么解决这个问题呢。如 pa 中所学的：我们把初始代码存储在所有用户程序都可见的地方。`ecall` 指令不会切换页表，所以我们在 `trampoline.s` 中，使用的还是用户页表。

我们在创建用户进程时，将用户进程的 `TRAMPOLINE` 地址映射到 `trampoline.s` 的地址。

```c
mappages(pagetable, va:TRAMPOLINE, PGSIZE, pa:(uint64)trampoline, PTE_R | PTE_X)
```

`_trampoline.S` 对应的物理地址在内核程序链接时就已经确定了：

```
. = 0x80000000;

.text : {
(.text .text.)
. = ALIGN(0x1000);
_trampoline = .;
*(trampsec)
. = ALIGN(0x1000);
ASSERT(. - _trampoline == 0x1000, "error: trampoline larger than one page");
PROVIDE(etext = .);
}
```

对每个进程，虚拟地址都是 TRAMPOLINE（用户最高地址），物理地址就是 `trampoline.S` 这段代码对应的地址。

而内核程序，在初始化时，`vm.c` 中的 `kvmmake` 函数，将内核的最高地址映射到 `trampoline.S` 对应的地址：

```c
kvmmap(kpgtbl, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
```

除此以外，内核空间一般是直接映射：虚拟地址和物理地址相同：

```c
kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);
```

用户态和内核态，`trampoline.S` 对应的虚拟地址一致，这样即使切换页表，`trampoline.S` 中的代码也能正常工作。那么上面，`trampoline.S` 在用户态的页表上保存好上下文到用户地址的 `trapframe` 中后，`csrw satp, t1` 就将页表切换到内核页表（内核页表的地址也储存在进程结构体中：`p->trapframe->kernel_satp`），跳转到内核代码。

（而 PA 实验中，由于内核页表地址是恒等映射，这些映射直接复制到用户页表中，因此不必切换）


## 4. 缺页异常

page fault 对应的代码

由硬件检测，若读取到无效地址，the `scause` register indicates the type of the page fault and the `stval` register contains the address that couldn’t be translated. 别忘了地址翻译由硬件执行，所以异常由硬件抛出（不管硬件还是软件异常，都先跳转到 epc 设置的中断向量）。

## 5. exec

用户程序执行 `exec` 系统调用后进入 `kernel/exec` 中的 `exec` 函数，可以看到，它会按照指定的路径读取 ELF 文件。给新进程分配新的页表和内存（`proc_pagetable()` 和 `uvmalloc()`），再通过 `readi()` 函数把文件读取到内存中。然后用 `uvmalloc()` 分配用户栈的内存，并存储 `argv` 的参数。这些准备好以后，就可以通过下面代码更新当前进程的内容了，并释放旧有的内容：

```c
  // Commit to the user image.
  oldpagetable = p->pagetable;
  p->pagetable = pagetable;
  p->sz = sz;
  p->trapframe->epc = elf.entry;  // initial program counter = main
  p->trapframe->sp = sp; // initial stack pointer
  proc_freepagetable(oldpagetable, oldsz);
```

这样执行完毕后，就会跳转到新程序的起始位置 `elf.entry`。


## 6. 多线程切换

