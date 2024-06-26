# 数字与算数

原本这一章只是整理移位什么的。做 PA 做着发现我对数字的认识仍然非常浅薄，于是就整理这个。PA2 的实验记录尽快整理，（虽然）才完成一丢丢。

## 字节和字

一般的计算机都以字节为基本单位，1 字节 = 8 位。 PA 的存储模拟就是一个 char 型数组。

字是指针长度。比如 32 位就是 “字”。PA 的模拟器就是 32 位

## 字节序

小端法和大端法。

小端法有利于加法，乘法，比大小。

<https://www.ruanyifeng.com/blog/2022/06/endianness-analysis.html#comment-433763>



## 整数表示

### 1. 无符号数

假设一个整数数据类型有 w 位。无符号数的每一位都用于构成这个数字。于是很显然，w 位可以表示的整数为 $[0, 2^{w}-1]$。

### 2. 有符号数

为了避免混淆，我们这里只说补码表示数。实际上在大多数情况下，计算机中存储的有符号数都是以补码的形式来存储的。（顺带，补码这个名字取得很烂，完全看不出它用的普遍性）

假设一个整数数据类型有 w 位。有符号数的最高位可以看成符号位，之后的 w - 1 位参与构成这个数字。于是，在非负数的情况下，最高位是 0，表示的整数可以在 $[0, 2^{w-1}-1]$ 之间。二进制来看是 $[(00..00)_2, (01..11)_2]$。

在负数的情况下，最高位是 1，表示 $-2^{w-1}$。于是负数的范围是 $[-2^{w-1}, -1]$。二进制来看是 $[(10..00)_2, (11..11)_2]$。

综上，w 位有符号数表示的范围是 $[-2^{w-1}, 2^{w-1}-1]$。

怎么知道一个数的相反数对应的补码表示呢？太简单了，**取反加一**即可。举个例子：111010，取反加一得到 000110，很快就知道原来是 -6，得到的相反数是 6。

### 3. 有符号数与无符号数的转换
碰到这种问题时，不要想太多，从位的角度去考虑。

最简单的方法就是写出二进制表示，再转化成另一种类型，不展开说了。

数据在计算机中存储时，不管是以有符号数还是无符号数的形式读取，都不会改变数据本身。

但是，参与了计算以后，数据的类型会改变运算的结果。这里我们等会再说。

???+ "C 语言中的数"
    先说立即数吧。C 语言的立即数都是默认有符号的，比如 `1234`，声明一个无符号整数请带上后缀 "u"， 比如 `1234u`。

    变量的情况：C 语言中有符号数和无符号数相互转化时，保持底层的位不变。于是下面的代码：
    
    ```c
    int32_t x = -1;
    printf("x = %u = %d\n", x, x);
    ```

    输出 `x = 4294967295 = -1`

    运算的情况下，若二元运算符的一个运算数有符号，还有一个运算数无符号，C 语言会将所有数字转化为无符号数。

    练习题：
    
    最经典的就是 `-1>0u` 这个表达式结果为真，你知道为什么吗。

    `2147483647>(int)2147483648u`，为什么为真？

## 数字扩展和截断

扩展实在是太简单了，对无符号数，左边加 0 即可。有符号数则是左边添符号位。

截断：再说。


## 位运算


其实 C 语言中的位运算还挺麻烦的, 因为没办法直接置位清零.

### 移位运算

左移右移。注意算数右移和有符号扩展对应，逻辑右移和无符号扩展对应。


下面讲一些应用.

将 value 的第 position 位置成 bitValue 的值. 方法是先清零, 再进行或运算.
```c
value = (value & ~(1 << position)) | (bitValue << position);
```

## 整数运算


### 无符号加法

CSAPP 讲的有些抽象。两个无符号数进行加法或者减法，直接进行加法或者减法，然后在把结果转化成对应的无符号数即可。

比如对 32 位 1-2=-1=4294967295

大数相加可能会溢出，溢出去掉多出来的 1 即可，比如对 4 位，9+12=21=5.


### 有符号加法

对于有符号数，只考虑加法就可以了，因为减去一个数相当于加上一个负数。两个数的补码相加，就和无符号数加法一模一样，加起来，舍去进位即可。

正数加负数不会溢出，溢出的情况可能是正数加正数或负数加负数。

## 浮点数

很重要，但永远在鸽。