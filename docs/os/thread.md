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

### 3.1. 控制中断

```c
void lock() {
    DisableInterrupts();
}

void unlock() {
    EnableInterrupts();
}
```

很简单，把中断关了就没人能进来了。当然关中断缺点怎样就不多说了。

### 3.2. 没有硬件

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

### 3.3. 自旋锁

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


### 3.4. compare-and-swap 

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

### 3.5. 自旋太多

一个线程可能在等待锁的时候一直自旋，这样效率就会非常低下。那么该怎么操作？

最简单的思路是，若发现锁被占用，就直接让出 CPU 给其他线程。这对两线程很有用，但是如果线程更多，就会有很多线程来执行检测——让出，这仍然效率低下。我们必须决定锁释放时，谁能抢到锁。我们可以使用队列来保存等待锁的线程。看下面的代码。

```c
typedef struct lock_t {
    int flag;
    int guard;
    queue_t *q;
} lock_t;

void lock_init (lock_t *m) {
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}

void lock(lock_t *m) {
    while(TestAndSet(&m->guard, 1)==1);
    if(m->flag == 0) { // 持有锁
        m->flag = 1;
        m->guard = 0;
    } else { // 未持有，让出锁
        queue_add(m->q, gettid());
        m->guard = 0;
        park();
    }
}

void unlock(lock_t *m) {
    while(TestAndSet(&m->guard, 1)==1);
    if(queue_empty(m->q)) m->flag = 0;
    else unpark(queue_remove(m->q));
    m->guard = 0;
}
```

注释，我们上面是利用了 Solaris 的支持，`park()` 能够让调用线程休眠，`unpark(ID)` 则唤醒线程 ID。

guard 起到自旋锁的作用。在 lock 函数中，如果线程不能获取锁，则将自身加入队列，将 guard 设置成 0，让出 CPU。unlock 则唤醒最早加入队列的线程，别让它等待太久。


## 4. 条件变量


### 4.1 定义

需求是：我们希望我们的线程等待某个条件变成真时，再继续运行。

所以我们引入条件变量，条件变量是一个显式队列，某个“条件”不满足时，线程可以将自己加入队列，等待这个条件。其他线程若是改变了“条件”，就可以唤醒这个条件队列上的线程。

看下面的代码例子，父线程等待子线程运行完，再继续运行。

```c
#include <pthread.h>
#include <stdio.h>

int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void thr_exit()
{
    pthread_mutex_lock(&m);
    done = 1;
    pthread_cond_signal(&c);
    pthread_mutex_unlock(&m);
}

void *child(void *arg) 
{
    printf("child\n");
    thr_exit();
    return NULL;
}

void thr_join()
{
    pthread_mutex_lock(&m);
    while (done==0) {
        pthread_cond_wait(&c, &m);
    }
    pthread_mutex_unlock(&m);
}

int main()
{
    printf("parent begin\n");
    pthread_t p;
    pthread_create(&p, NULL, child, NULL);
    thr_join();
    printf("parent end\n");
    return 0;
}
```

条件变量有两种相关操作：wait() 和 signal()。线程要睡眠的时候，调用 wait()。想唤醒某个条件变量上的睡眠线程时，调用 signal()。

wait() 的其中一个参数是锁（互斥量），它假定 wait() 参数调用时，这个互斥量是已上锁的状态。wait() 的职责是释放锁，并让调用线程休眠。一直到线程唤醒时（另外某一个线程发信号），它再获取锁，返回调用者。

可能会出现两种情况：若父线程先运行的话，它就直接调用 `thr_join()` 函数等待子线程。它先获取锁，然后调用 `wait()` 函数，让自己休眠。子线程开始运行，并运行完毕后，调用 `signal` 且设置状态变量 `done` 为 1。最后 `wait()` 唤醒父线程继续运行，释放锁，结束。

若子线程先运行，其设置状态变量为 `done` 为 1， 调用 `signal` 函数唤醒其他线程（当然这里没有），就结束了。父线程调用 `thr_join`，发现 `done` 为 1，就可以直接返回了。

然后我们可以看看，真的有必要如此复杂吗？可以省掉某些部分吗？如果舍弃 `done` 会怎样。代码变成下面的部分：

```c
void thr_exit()
{
    pthread_mutex_lock(&m);
    pthread_cond_signal(&c);
    pthread_mutex_unlock(&m);
}

void thr_join()
{
    pthread_mutex_lock(&m);
    pthread_cond_wait(&c, &m);
    pthread_mutex_unlock(&m);
}
```

如果子线程先运行的话，父线程就会卡在 `wait()` 这里，再也没有线程唤醒它了。所以，加上一个变量 `done` 是有必要的。

那么如果舍弃锁会怎样？

```c
void thr_exit()
{
    done = 1;
    pthread_cond_signal(&c);  
}

void thr_join()
{
    while (done==0) {
        pthread_cond_wait(&c);
    }
}  
```

没有锁，意味着 `done` 的值可能随时会变化，考虑这样的情况，父线程刚刚检查完 `done` 的值，然后中断到子线程，子线程修改了 `done` 的值，发出信号，运行完了，然后回到父线程就永远卡在 `wait` 上了。

因此有一个小提示：**发信号时必须得持有锁**，并且 `wait` 函数的参数强制要求你这么做，参数的输入是已经持有的锁。




### 4.2 生产者-消费者问题

首先介绍一下背景：生产者将生成的数据项放入缓冲区；消费者从缓冲区取走数据项，以某种方式消费。举个例子，我们常见的管道，就是这样：`grep foo file.txt | wc -l`。


```c
int buffer;
int count = 0;

void put(int value) {
    assert(count == 0);
    count = 1;
    buffer = value;
}

int get() {
    assert(count == 1);
    count = 0;
    return buffer;
}
```

然后我们可以看看最初的一项生产者与消费者方案：

```c
void *producer(void *arg) {
    int i;
    int loops = (int) arg;
    for(i = 0; i < loops; i++) {
        put(i);
    }
}

void *consumer(void *arg) {
    int i;
    while(1) {
        int tmp = get();
        printf("%d\n", tmp);
    }
}
```

问题很简单，我们不加限制地就访问缓冲区了。

下面第二版，就加入条件变量，来进行限制。

```c
cond_t cond;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i < loop; i++) {
        pthread_mutex_lock(&mutex);
        if (count == 1)
            pthread_cond_wait(&cond, &mutex);
        put(i);
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        if (count == 0)
            pthread_cond_wait(&cond, &mutex);
        int tmp = get();
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
}
```

这个对一个生产者和一个消费者有用，但是如果是一个生产者和多个消费者，就会出问题。我们来讨论一下，假设有一个消费者 Tc1 先运行，它获得锁，检查缓冲区，然后等待（等待时会释放锁）。然后生产者 Tp 运行，获得锁，填充缓冲区，释放信号。原本接下来，就是 Tc1 接收信号，执行即可。但是可以出现这样的情况：在 Tc1 执行前，另一个 Tc2 抢先执行，这时候，缓冲区是满的，于是就会跳过 `wait()`，直接进行消费，将缓冲区中的内容挥霍一空。结果到 Tc1 运行时，缓冲区还是空的，触发 `get()` 函数中的 `assert()` 语句导致异常。

然后修改的第三版：记住一个规则，对条件变量的检查，**总是使用 `while` 语句来代替 `if`**


```c
cond_t cond;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i < loop; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 1)
            pthread_cond_wait(&cond, &mutex);
        put(i);
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)
            pthread_cond_wait(&cond, &mutex);
        int tmp = get();
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
}
```

只是这么小的一个变化，就解决了上面的问题。当 Tc1 被唤醒时，不是直接 `get` 消费，而是在 `while` 循环中，再检测一遍 `count` 的值，如果缓冲队列为空，就继续等待。

可惜，这个代码仍然有问题。考虑这样的情况，假设生产者 Tp 先运行，放入了一个值，然后休眠；然后消费者 Tc1 运行，消耗掉一个值。它发送信号，可以唤醒等待队列中的 Tc2 和 Tp。如果唤醒的是 Tp，还好；如果唤醒的是 Tc1，就糟了。Tc1 发现队列未空，就会去睡眠，他并没有发信号。这会导致三个线程都会休眠，于是死机。

所以我们下一步的修改方式是：增加条件变量，让生产者只能唤醒消费者，消费者只能唤醒生产者。

第四版代码：

```c
cond_t empty, fill;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i < loop; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 1)
            pthread_cond_wait(&empty, &mutex);
        put(i);
        pthread_cond_signal(&fill);
        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)
            pthread_cond_wait(&fill, &mutex);
        int tmp = get();
        pthread_cond_signal(&empty);
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
}
```

上面的代码，才是完全正确的代码了。所以要写出好的并发代码，还是有些难度。


## 5. 信号量

信号量是一个整数值，信号量的初始值决定了其不同行为，因此首先我们要初始化。

```c
#include <semaphore.h>
sem_t s;
sem_init(&s, 0, 1);
```

第二个参数表示信号量在同一进程中被所有线程共享，一般设为 0。第三个参数设初值为 1。

下面是对信号量操作的两个函数。

```c
int sem_wait(sem_t *s);
// 将信号量的值减 1，若减后值为负数，则当前线程进入等待
int sem_post(sem_t *s);
// 将信号量的值加 1，若有等待的线程，唤醒其中一个
```


我们可以用信号量来实现锁：

```c
sem_t m;
sem_init(&m, 0, 1);

sem_wait(&m);
// do somthing
sem_post(&m);
```

我们可以用信号量实现条件变量：

```c
sem_t s;
void *
child(void* arg) {
    printf("child");
    sem_post(&s);
    return NULL;
}


int main() {
    sem_init(&s, 0, 0);
    printf("parent begin\n");
    pthread_t c;
    pthread_create(&c, NULL, child, NULL);
    sem_wait(&s);
    printf("parent end\n");
    return 0;
}
```

注意，实现锁时，应当初始化信号量为 1，条件变量则是 0，思考一下即可。
