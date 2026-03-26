各种类型的八股，都放在这里，python 的，计算机网络的。

1. python 怎么做垃圾回收？

python 的垃圾回收方式是引用计数，每个对象身上都有个计数器，有一个变量指向它，计数器加 1；变量销毁，计数器减 1。减到 0 直接在内存中抹除。

标记-清除（Mark and Sweep）与分代回收（Generational GC）。引用计数有个致命弱点叫“循环引用”（A 指向 B，B 指向 A，计数器永远不为 0）。Python 会定期扫描内存池，把这种互相抱死但实际上已经没用的对象找出来清理掉；同时把存活时间长的对象放到“老年代”，减少扫描频率以提升性能。

2. python 的装饰器。

装饰器本质上是语法糖。看下面的例子：

```python
import time

# 1. 这是你的核心业务函数（不许改它内部代码）
def do_work():
    print("Agent 正在疯狂检索代码...")
    time.sleep(1)

# 2. 这是一个“套娃”工厂（也就是装饰器本质）
def timer_factory(func):
    # 内部定义一个包装器函数
    def wrapper():
        start = time.time()
        func() # 执行真正的核心逻辑
        end = time.time()
        print(f"耗时: {end - start} 秒")
    
    # 把包装好的新函数返回出去
    return wrapper

# 3. 手工替换过程
do_work = timer_factory(do_work) 

do_work() # 现在调用的，其实是 wrapper()
```

装饰器的本质：它是一个接收函数作为参数，并返回一个增强版新函数的函数。

下面的代码用了装饰器，与上面的代码等价，但更好看了。

```python
# 定义装饰器
def timer_factory(func):
    def wrapper(*args, **kwargs):  # 面试重点：必须用 *args, **kwargs 兼容所有参数
        start = time.time()
        result = func(*args, **kwargs) # 获取原函数的返回值
        end = time.time()
        print(f"[{func.__name__}] 耗时: {end - start} 秒")
        return result
    return wrapper

# 使用装饰器
@timer_factory
def do_work_with_args(name):
    print(f"{name} 正在疯狂检索代码...")
    time.sleep(1)
    return "检索成功"

do_work_with_args("Agent_A")
```

当你写下 `@timer_factory` 时，Python 解释器在背后偷偷帮你执行了：`do_work_with_args = timer_factory(do_work_with_args)`。

FastAPI 的装饰器：

```python
@app.post("/chat")
```

fastAPI 底层有一个 hash 路由表。上面的代码
拦截了下面定义的业务函数，并将其内存地址、请求方法（POST）以及 URL 路径（/chat）作为一对 Key-Value，注册到了 FastAPI 内部维护的**路由表（Routing Table）**中。

3. python 的多线程

一、 逃不开的 GIL 与真正的“多线程” (threading)
Python 底层的 threading 模块调用的其实是操作系统的原生线程（比如 Linux 下的 pthreads）。但在 CPython（官方默认实现）中，有一把全局大锁（GIL）。

GIL 的作用： 无论你创建了多少个线程，无论你的 CPU 有多少个物理核心，GIL 强制规定：同一时刻，只能有一个 OS 线程在执行 Python 字节码。

这就意味着： 对于计算密集型（CPU-bound）的任务（比如跑复杂的数学矩阵运算、加密解密、长时间的 while 循环），Python 的多线程是伪并行。多个线程只能在一个 CPU 核心上互相抢夺 GIL 来回切换，不仅无法利用多核，甚至还会因为频繁的上下文切换导致比单线程还慢。

那 Python 多线程还有什么用？
答案是：I/O 密集型任务（I/O-bound）。
当你的线程在做网络请求、读写磁盘、或者等待数据库返回结果时，它是处于阻塞状态的。Python 非常聪明，当一个线程遇到 I/O 阻塞时，它会自动释放 GIL。 此时，其他线程就可以趁机拿到 GIL 去执行。
所以，如果你的任务是“同时向 100 个网站发请求”，用 Python 多线程依然会快到飞起。

二、 突破封锁的“多进程” (multiprocessing)
既然 GIL 锁死了线程，那如果我非要榨干服务器的 32 个 CPU 核心来做高强度的算法计算呢？
Python 提供了 multiprocessing 模块。

底层逻辑： 它不再创建轻量级的线程，而是直接向操作系统申请 fork 出多个独立的子进程。

优势： 每个进程都有自己独立的内存空间，最关键的是，每个进程都有自己独立的一把 GIL。这样，多个进程就可以完美地分散到多个 CPU 核心上，实现真正的物理级并行。

代价： 进程的创建和销毁开销远大于线程，且进程间通信（IPC，比如通过管道或共享内存传递数据）比较复杂。

三、 现代高并发的终极杀器：“协程” (asyncio)
这是目前 Python 后端（比如 FastAPI 框架）处理高并发最主流的方式。

底层逻辑： 它完全抛弃了操作系统层面的线程切换。协程运行在单线程内部，通过一个“事件循环（Event Loop）”在应用层进行任务调度。

优势： 当遇到 I/O 阻塞（比如 await db.query()）时，代码会瞬间挂起当前任务，让出控制权去执行其他任务。因为没有操作系统级的线程上下文切换，它的内存占用极小，一个普通的单核服务器就能轻松维持成千上万个并发连接。