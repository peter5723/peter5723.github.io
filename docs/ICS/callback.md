# 回调函数

不管是 PA 还是实际工程中，我们已经碰到过很多次 “回调函数（callback）”这个概念了，那么它究竟是什么呢？

可以看一下这个博客：

<https://www.runoob.com/w3cnote/c-callback-function.html>

<https://www.zhihu.com/question/19801131>

用最简单的理解来说，回调函数就是通过函数指针被调用的函数。或者引用别人的话：A "callback" is any function that is called by another function which takes the first function as a parameter.

为什么要用回调函数呢？就应用场景来说，回调函数允许特定事件发生时（鼠标点击、触发异常、接受数据等等）执行特定的操作。就作用来说，可以解耦：调用回调函数的函数（称为起始函数）往往是不可见的或者难以改动的，然而只需要传入不同的函数指针，就可以执行不同的操作了。

虽然是一个较抽象的概念，其实也不难理解。

下面看一个回调函数的例子：（其实就是链接里的例子，懒的自己写了）
```c
#include <stdio.h>

int Callback_1(int x) // Callback Function 1
{
    printf("Hello, this is Callback_1: x = %d \n", x);
    return 0;
}

int Callback_2(int x) // Callback Function 2
{
    printf("Hello, this is Callback_2: x = %d \n", x);
    return 0;
}

int Callback_3(int x) // Callback Function 3
{
    printf("Hello, this is Callback_3: x = %d \n", x);
    return 0;
}

int Handle(int y, int (*Callback)(int))
{
    printf("Entering Handle Function. \n");
    Callback(y);
    printf("Leaving Handle Function. \n");
    return 0;
}

int main()
{
    int a = 2;
    int b = 4;
    int c = 6;
    printf("Entering Main Function. \n");
    Handle(a, Callback_1);
    Handle(b, Callback_2);
    Handle(c, Callback_3);
    printf("Leaving Main Function. \n");
    return 0;
}
```

有时我们也会碰到“注册”回调函数的说法。这个“注册”的说法也是相当抽象了，其实就是初始化的意思。初始化需要调用的回调函数，之后在触发某种事件时，就可以直接调用这个函数了。比如说操作系统中的 `signal` 函数，函数原型是：
```c
void (*signal(int sig, void (*func)(int)))(int)
```
调用 `signal` 函数进行初始化，之后程序接受到 `sig` 信号时，就会调用回调函数 `func`。

例程如下：

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>

void sighandler(int);

int main()
{
   signal(SIGINT, sighandler);

   while(1) 
   {
      printf("sleep 1 second\n");
      sleep(1);
   }

   return(0);
}

void sighandler(int signum)
{
   printf("get signal: %d, jump out...\n", signum);
   exit(1);
}
```

程序首先是每个一秒打印一条语句，按下 <Ctrl-c> ，让程序接收到 `SIGINT` 信号，从而跳出，进入 `sighandler` 函数。

<!-- 关于 `signal` 函数，可以看 <https://www.runoob.com/cprogramming/c-function-signal.html> 这个链接。 -->
