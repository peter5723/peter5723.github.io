# 线程

## 0. 线程介绍

线程这个概念和进程很像，都可以看做是程序执行的抽象。单线程的进程只有一个程序计数器，多线程的进程每个线程都有程序计数器用于执行。进程上下文存在 PCB 中，线程的上下文存在 TCB（线程控制块） 中。线程和进程的不同之处在于，

1. 一个进程中的多线程共享一个地址空间。在线程之间的上下文切换时，地址空间保持不变，不需要切换页表；但是进程的切换是需要切换页表的。
2. 传统的进程是单线程，只有一个栈，而多线程的进程中，每个线程独立运行，每个线程都有自己的栈。


从这个概念上说，PA 中对进程和线程的概念是混用的。因为很明显 PA 中只用 PCB 控制，并且 PA 中的两个程序使用的虚拟地址空间不同，因此都是进程。

## 1. 两个 api

关于线程，最先会遇到两个 api：线程创建和等待线程完成。

### 线程创建 api

```c
int pthread_create( pthread_t *thread, 
                    const pthread_attr_t *attr, 
                    void *(*start_routine) (void *), 
                    void *arg);
```

- `thread` 是指向 `pthread_t` 结构类型的指针，我们利用这个结构和该线程交互。
- `attr` 表明线程的属性，默认传入 `NULL` 即可。
- `start_routine` 是一个指向函数的指针，这个函数将在新的线程中运行。
- `arg` 是传递给线程要执行的函数的参数。

当我们用上面的 api 创建了一个线程，它就开始与程序其它的线程在相同的地址空间内同时运行。

### 等待线程完成 api

```c
int pthread_join(  pthread_t thread, 
                    void **retval);
```

- `thread` 是你想要等待的线程的标识符。
- `retval` 是一个指向 `void*` 的指针，它将被设置为线程的返回值。如果你不关心线程的返回值，你可以传递NULL。

调用这个函数来等待子线程的完成，起到阻塞调用它的线程（主线程）的功能。注意只有调用该函数的线程会被阻塞，其它线程仍然正常运行。直到这个被调用的子线程运行完毕，主线程才继续运行。

## 2. 冲突

我们来看看下面一个程序。

```c

#include <stdio.h>
#include <pthread.h>
#include <assert.h>

static volatile int counter = 0;

void *mythread(void *arg) {
    printf("%s: begin\n", (char*)arg);
    int i;
    for(i=0;i<1e7;i++){
        counter++;
    }    
    printf("%s: done", (char*)arg);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p1, p2;
    printf("main:begin(counter=%d)\n",counter);
    pthread_create(&p1, NULL, mythread, "A");
    pthread_create(&p2, NULL, mythread, "B");

    pthread_join(p1, NULL);
    pthread_join(p2, NULL);
    printf("main:end(counter=%d)\n",counter);
    return 0;
}
```

运行这段程序，会发现计数器的值是不同的，而不会是 20000000. 为什么？

假定给计数器加一的汇编代码是：

```asm
mov 0x8049a1c, %eax
add $0x1, %eax
mov %eax, 0x8049a1c
```
那么可以出现下面的情况：

变量 `counter` 在地址 `0x8049a1c` 中。假设线程 1 进入此代码区域，将 `counter` 的值（假设是 50）加载到寄存器 `eax` 中，然后加一。但是如果此时时钟中断发生，操作系统切换线程。那么线程 1 的上下文将被保存到它的 `TCB` 中。之后线程 2 开始运行，进入这段代码，获取计数器的值放入 `eax` 中，加一并放回去，使 `counter` 的值变成 51。 随后发生又发生一次上下文切换到线程 1，执行最后一步 `mov` 操作，将 `counter` 的值又设为 51。因此这样就达到了加了两次但是只加一的效果。

注意有多种可能的情况，随代码的时间执行而定。这种不确定结果的行为称为竞态条件（race condition）。这段代码访问了共享变量，被称为临界区（critical section）。从上面的讨论知道，临界区决不能由多个线程同时执行。这样的代码叫互斥（mutual exclusion），只能有一个线程能够进入临界区而其他线程被阻止进入。

上面的竞态条件是由几条指令之间的中断导致的。要是我们能把几条指令合成一条指令直接执行就好了！这就是“原子性代码”（atomic code）。下面的锁就用到了这个思想。

## 3. 锁

为了做到安全地并发编程，我们引入锁这个概念。锁放在临界区的周围，保证临界区像单原子指令一样运行。

锁（lock），在 POSIX 标准库中也被称为互斥量（mutex），伪代码如下所示：

```c
lock_t mutex;
...
lock(&mutex);
banlance++;
unlock(&mutex);
```

首先声明一个锁变量。一个线程调用 `lock()` 尝试获得锁。只有获得锁的线程才能进入临界区。若这个锁已经被另外一个线程持有，那么线程就会等待，而不会从 `lock()` 返回。锁的持有者调用 `unlock()` 以后，释放这个锁。此时这个锁可用，其它等待线程中的其中一个会接收锁状态的变化，获得这个锁，进入临界区。通过上面的机制可以保证每次只有一个线程能够进入临界区。

锁要怎么实现呢？首先介绍评价方案：互斥、公平和性能。下面介绍实现的方案。

### 1. 控制中断

```c
void lock() {
    DisableInterrupts();
}

void unlock() {
    EnableInterrupts();
}
```

很简单，把中断关了就没人能进来了。当然关中断缺点怎样就不多说了。

### 2. 没有硬件

基于一系列原因，系统设计者让硬件支持锁。我们来看代码，没有硬件的支持会怎样呢？

```c
typedef struct lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
    mutex->flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex->flag == 1) 
        ; // spin wait
    mutex->flag = 1;
}

void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```

这段代码很容易理解逻辑。让标志位来表示是否上锁，`lock()` 中如果线程检测到标志位是 1，就在循环中自旋等待（自旋等待，spin wait，循环中检测标志的值）。反之则设标志位为1，进入临界区。`unlock` 则将标志位清零。但是正确吗？

看下面的例子

```
+--------------------------------------+------------------------------+
| Thread1                              | Thread2                      |
+--------------------------------------+------------------------------+
| call lock()                          |                              |
| while(flag==1)                       |                              |
| // now flag is 0,                    |                              |
| // so the expression is false        |                              |
| interrupt: switch to Thread2         |                              |
+--------------------------------------+------------------------------+
|                                      | call lock()                  |
|                                      | while(flag==1) // false      |
|                                      | flag = 1;                    |
|                                      | interrupt: switch to Thread1 |
+--------------------------------------+------------------------------+
| // because the expression is false,  |                              |
| // so we jump out of the loop.       |                              |
| flag = 1;                            |                              |
| return;                              |                              |
+--------------------------------------+------------------------------+
```

如表格所示，只要中断位置合适，两个线程都能够进入临界区。这是为什么呢？我们设置锁是为了保证只有一个线程能进入临界区，线程的切换是由中断控制的。但是现在我们的锁也受到中断的控制，本质上也可以说是“临界区”。要解决这个问题，就是实现上面提到的“原子性”。

### 3. 自旋锁

所以我们需要硬件的帮助来设计自旋锁（在循环中判断就叫自旋）。硬件设计者给我们设计的指令是 test-and-set（测试并设置）。它的伪代码大概是：

```c
int TestAndSet(int *old_ptr, int new)
{
    int old = *old_ptr;
    *old_ptr = new;
    return old;
}
```

它返回 `old_ptr` 指向的旧值（测试）同时更新为 `new` 的新值（设置）。增加这条指令我们就可以设计一个自旋锁（spin lock）了。

```c
typedef struct lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
    mutex->flag = 0;
}

void lock(lock_t *mutex) {
    while (TestAndSet(&mutex->flag, 1) == 1) 
        ; // spin wait
}

void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```

逻辑和上一节的代码基本是一样的。唯一的不同是，上一节中出现的问题代码

```c
while (mutex->flag == 1) 
        ; // spin wait
    mutex->flag = 1;
```

因为有硬件的帮助，直接变成了一条指令：

```c
while (TestAndSet(&mutex->flag, 1) == 1) 
        ; // spin wait
```

把判断和赋值的过程合在了一起，这样就可以保持互斥性了。因为问题出在在指令之间出现了中断，现在我们把几条指令合在一起成为一条原子指令一次性执行，就可以规避中断的问题。

自旋锁总算是正确的了，但是它不公平，会导致饿死，因为一个线程可能永远占用着临界区。


### 4. compare-and-swap 

前面我们使用了 `test-and-set` 指令来实现自旋锁，而有些系统提供的硬件指令是 `compare-and-swap`。它的伪代码大致是：

```c
int CompareAndSwap(int *ptr, int expected, int new) {
    int actual = *ptr;
    if (actual == expected)
        *ptr = new;
    return actual;
}
```
 
可以把它看成是 `test-and-set` 指令的改良版。要实现自旋锁，只要将 `lock()` 函数作下面替换即可：

```c
void lock(lock_t *mutex) {
    while (CompareAndSwap(&mutex->flag, 0, 1) == 1) 
        ; // spin wait
}
```


