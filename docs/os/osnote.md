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


## Lab2: Syscall

很简单，就是 GDB 的使用以及一些系统调用的添加。值得一提的是知道了 xv6 中的内存分配（kalloc 和 free）是通过链表实现的。