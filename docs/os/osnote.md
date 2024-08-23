# 操作系统

开始学操作系统. 学习的课程是 MIT 的 [6.1810](https://pdos.csail.mit.edu/6.828/2023/schedule.html)(即之前的 6.S081). 使用的教材是 [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/) 和 [xv6-book](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)

由于是每节课一个实验, 那么我就把学习的笔记和实验记录在一起好了. 格外重要的需要详细讲的知识点, 再单独列出.

在实验之前, 最好根据这个链接来配置你的 git: <https://xv6.dgs.zone/labs/use_git/git1.html>, 因为有些坑和之前的不同。

整理一下，就是先 `git remote add github <your-url>` 添加自己仓库的链接为 github 分支。

然后 MIT 那边的远程链接叫 origin，自己的仓库的远程链接叫 github，那么要做新实验 xxx 时：
```
git fetch origin
git checkout xxx
git push github xxx
```
就可以在自己的仓库新建立一个 xxx 分支。

每次想要上传作业到远程仓库时，`git push github xxx` 即可。`git checkout yyy` 可以去到之前完成的作业的分支。

## Lab1: Utilities

这个实验主要是要熟悉一些操作系统的接口. xv6 的接口是模仿 unix 的接口的.

一些重要的接口有 fork，exec，pipe 等等。值得一提的是注意管道机制是阻塞的：只有读取端读取了数据以后，写入端才能写入新的数据。这是做 prime 实验的关键。

`make qemu` 打开 xv6 操作系统

`<C-a> x` 退出

`./grade-lab-xxx abc` 对 abc 进行测试。没参数批改整个实验

## Lab2: Syscall

很简单，就是 GDB 的使用以及一些系统调用的添加。值得一提的是知道了 xv6 中的内存分配（kalloc 和 free）是通过链表实现的。


本课程使用 GDB 的方法：  To use gdb with xv6, run make `make qemu-gdb` in one window, run `gdb-multiarch` (or riscv64-linux-gnu-gdb) in another window. 


## Lab3: Pgtbl

这是一个关于虚拟内存的实验。糟糕的是，目前我没有什么头绪。主要是不知道代码的结构，得仔细看。

首先搞清楚操作系统给进程分配页的过程。在 `proc.c` 中。有三道题，第一题的工作是在进程的页表中添加一个新的页映射到虚拟地址 `USYSCALL` 对应的内存，直接储存 pid 等数据，这样用户端就可以从虚拟地址 `USYSCALL` 直接读取信息，就可以减少系统调用的次数，增加运行效率。这道题的代码都是在内核代码中修改的，所以必须仔细地阅读内核代码才能做出来。（做法是仿照 `proc.c` 中的做法分配和释放一个新的页，如 trapframe。一开始忘记了页的释放过程，而导致 bug）。

第二道题是打印页表。关键是如何遍历整个 PTE？

首先要知道 xv6 使用的页表结构是 riscv 的 sv39 三级页表。原理和前面学过的 Sv32 是一样的。这里我们用 64 位数据表示地址，结构如下图所示：

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/oslab3.1.png"/>

三级页表的转换过程如下图所示：

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/oslab3.2.png"/>

虚拟地址都是 9 位，因此每一级页表的页表项个数为 2 的 9 次方 512 个。

利用 dfs 算法，对无效的 pte 剪枝，就可以轻松地对页表项进行遍历了。

最后一题显示 hard 难度，看看什么来头。其实一点都不难。这个问题是添加一个系统调用，监测读写过的页。我觉得最难的地方是：“如何将读写过的页的页表项的对应位进行置位”。这是操作系统里做不到的，只有在 qemu 硬件上才能直接做到，因为硬件直接将虚拟地址转化为物理地址，并不经过操作系统。最后发现硬件当然是帮我们置位好了，就好像 ISA 手册描述的一样。所以这个纠结了我一天的问题的答案是，我什么也不用做。

这倒是给了我一个经验：硬件的行为是和 ISA 手册描述一致的。

还有就是位运算时可能容易犯的错了。清零用与运算，置位用或运算。

最后有一个 usertest 的测试似乎通不过，但是本次的任务基本和页表的结构等没什么关系，我就不管了。

哎，自己好没耐心啊，失去了分析的能力。问题是：“FAILED -- lost some free pages 25902 (out of 32457)” 那么显然是有一些页没有释放了，其实就是第一题里面定义的页表在结束没有释放内存：

```c
	if(p->u)
		kfree((void*)p->u);
	p->u = 0;
```

另外，PA 的实践收获还是挺大的，因为想不起来过程了可以看自己的代码回忆。