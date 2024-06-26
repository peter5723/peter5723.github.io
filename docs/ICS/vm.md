# 虚拟内存

终于看到了虚拟内存这块了, 感觉自己的笔记速度有点拖时间了.

CSAPP 第九章. 虚拟内存(virtual memory, VM).

## 1. 基本介绍

计算机的主存(即内存)可以看做一个有 M 个连续字节的大数组, NEMU 就是这么实现的. 每一字节都有对应的物理地址, 即数组中的索引. 现代计算机中, CPU 很少直接访问物理地址, 而是通过虚拟寻址的形式. CPU 会先生成一个虚拟地址(virtual address, VA). VA 会被内存管理单元(Memory Management Unit, MMU)翻译成物理地址(physical address, PA). 

虚拟内存是存放在**磁盘**上的有 N 个连续字节组成的数组. 和缓存一样, VM 也将虚拟内存分割成虚拟页(virtual page, VP)这样的大小固定的块来处理磁盘和主存间的传输. 假设每个虚拟页的大小为 P 字节. 物理内存也被分割成物理页(PP), 每页大小与虚拟页一致.

虚拟页面的集合可以分成三个不相交的子集 : VM 系统未分配的页, 缓存在物理内存的已分配页和未占用物理内存的已分配页. 如下图所示:

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm1.png"/>

和缓存一样, VM 也需要判断某个虚拟内存是否被缓存在了主存中. 这个功能是由软硬件联合提供的, 包括操作系统, MMU 和存放在物理内存中的**页表**数据结构. 页表将虚拟页映射到物理页, MMU 中的地址翻译硬件利用页表将虚拟地址转化为物理地址, 操作系统维护页表的内容并在磁盘和主存之间来回传送页.

## 2. 页表

下面介绍页表. 页表是页表条目(page table entry, PTE)的集合, 储存在物理内存中. 虚拟地址空间的每一页都在页表中有对应的 PTE. 我们假定一下每个条目的内容是由一个有效位和 n 位地址位组成. 有效位表示这个虚拟页是否缓存在内存中. 如果是, 地址字段就是缓存该虚拟页的物理页的起始位置. 反之, 设置一个空地址表示它未被分配, 或者这个地址指向虚拟页储存在磁盘的位置. 下面这张图来看看页表是怎么维护的:

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm.png"/>

注意, 任意物理页都可以包含任意虚拟页.

然后, 我们来看看 CPU 想要读取 VP2 的虚拟内存的一个字会发生什么. VP2 被缓存在 DRAM 内存中. MMU 将虚拟地址作为一个索引来定位 PTE2, 并从内存中读取 PTE2. PTE2 设置了有效位, 于是 MMU 知道 VP2 缓存在内存中, 直接使用 PTE 中储存的物理地址, 即可得到这个字的物理地址. 直接从内存中读取到虚拟内存, 叫**页命中**.

反过来, DRAM 缓存不命中就叫**缺页**. 仍是上面的图片, CPU 读取 VP3 中的一个字. VP3 没有缓存在 DRAM 中. MMU 读取 PTE3, 并发现有效位为 0, 于是推断出 VP3 未被缓存, 触发缺页异常程序. 缺页异常程序选择一个牺牲页, 比如 PP3 中的 VP4, 将之复制回磁盘后, 从磁盘复制 VP3 到 PP3, 并更新 PTE3, 再从异常返回. 如下图所示:

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm3.png"/>

可以看到虚拟内存的想法有很多是和缓存类似的. 虚拟内存中的页实际上就是对应缓存中的块. 向上面这样直到不命中才换入页面的策略称为按需页面调度.

下面分配了一个新的虚拟页 VP5, 分配过程就是在磁盘上创建空间并让 PTE5 指向磁盘上这个新创建的页面.

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm4.png"/>

我们把程序的工作集页面调度到内存中之后, 由于程序的局部性, 命中率很高, 并没有那么容易缺页, 所以虚拟内存是可以保证效率的.

## 3. 内存管理

下面介绍一些虚拟内存是如何内存管理的.

操作系统为每一个进程都提供独立的页表, 所以每一个进程都有自己独立的虚拟地址空间, 而多个虚拟页面也可以映射到同个物理页面上. 如下图:

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm5.png"/>

1. 独立的地址空间允许每个进程的映像都使用相同的格式, 不必关心实际物理地址是怎样的. 
2. 如果不同进程使用相同的内核函数, 操作系统可以将它们映射到同一个物理页面, 从而节省内存.
3. 连续虚拟页面, 映射的物理页面上不一定连续, 而是可以随意地分散在内存中.

虚拟内存也很好做内存保护的工作: 为每个 PTE 添加几个控制位即可. 几个控制位包括读写权限等. 如果有一条指令违反了控制位所给的权限, 那么 CPU 就抛出故障, 控制转给异常处理程序, 即 "段错误".

## 4. 地址翻译

### 4.1 基本介绍

整体的硬件的过程就放张图概括了:

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm6.png"/>


而在 MMU 中实现的地址翻译, 经过了哪些过程呢? 首先介绍一下将要用的符号:

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

其中, PTBR 提供页表的地址. 虚拟地址被分为 VPN 和 VPO 两部分. VPN 作为索引用于在页表中找到当前页的 PLE, 在这个 PLE 中存储了对应的 PPN. 将这个 PPN 与虚拟地址的 VPO 结合起来, 就得到了最终的地址. 注意, 物理页面和虚拟页面的大小相同, 故 PPO 和 VPO 总是相同的. 

可以看下面图片: 

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm7.png"/>

### 4.2 TLB 地址翻译加速

CPU 每产生一个虚拟地址, MMU 就要查阅一次 PTE, 将虚拟地址转成物理地址. 但是 PTE 储存在内存中, 我们仍然嫌读取得不够快. 于是在 MMU 中, 添加了一个叫快表(TLB, translation lookaside buffer)的缓存, 来储存最近使用的 PTE. 如果命中, 那么所有的地址翻译工作都在 MMU 中完成, 将会非常快.

### 4.3 多级页表

实际的操作中, 用的更多的还是多级页表. 多级页表可以显著地减少页表的大小, 具体不赘述留给读者思考了. 具体的实现就是将 VPN 分成 k 份, 每个 VPN i 都是第 i 级页表的索引. 每一级页表的 PTE 都是到下一级页表的基址, 最后一级的 PTE 是 PPN. PPO 和 VPO 还是相同的.

<img src="https://cdn.jsdelivr.net/gh/peter5723/imagehost/vm8.png"/>


