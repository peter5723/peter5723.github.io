# 数字滤波器设计

我们简单讲解一下数字滤波器设计。

最开始要说明我们如何实现滤波。我们不断采样获得一个时域上的信号函数 $x[n]$。然后我们设计好的滤波器是 $h[n]$。那么利用卷积：

$$
y[n] = x[n] * h[n]
$$

也就完成了滤波。

## 窗函数法设计 FIR 滤波器

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/sinc.png)

如上图所示：从理想低通滤波器开始。这在频域上是一个完美的矩形。然而其对应的时域函数$h_1[n]$是一个震荡的无限延伸的 $\text{sinc}$ 函数。我们不可能在内存里面存无限个数据。（并且 IIR 响应还要用到未来的数据，影响因果性）。所以我们就要对这个 $\text{sinc}$ 函数进行加窗 $w[n]$。

$$
h'[n] = h_1[n]w[n]
$$

加窗了以后，$\text{sinc}$ 函数在时域上就有限长了，我们就得到了一个 FIR 滤波器。

最朴素的方法是加矩形窗，也就是对 $\text{sinc}$ 函数进行直接截断。

$$
w_{\mathrm{R}}[n]
=
\begin{cases}
1, & 0 \leq n \leq N,\\
0, & \text{otherwise}.
\end{cases}
$$

下图展示了矩形窗的形状以及被矩形窗加窗以后，$\text{sinc}$ 函数对应的时域和频域，看第五张图，是有明显的波动的，即所谓的振铃现象。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/rectangle.png)

要想平滑这样的波动，可以用更加平滑的窗函数，例如hamming 窗。

$$
w_{\mathrm{H}}[n]
=
\begin{cases}
0.54-0.46\cos\left(\dfrac{2\pi n}{N}\right),
& 0 \leq n \leq N,\\
0, & \text{otherwise}.
\end{cases}
$$

下图展示了 hamming 窗的形状以及被 hamming 窗加窗以后 ${sinc}$ 函数对应的时域和频域。很明显，频域平滑了不少。因此在实际运用中更加常用。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/hamming.png)


总结一下，窗函数的设计流程如下：

```
设计要求
   ↓
理想频率响应 Hd
   ↓ 逆傅里叶变换
无限长冲激响应 hd
   ↓ 乘窗函数 w
有限长系数 h
   ↓ 与输入信号卷积
滤波输出 y
```

在工程上，给出采样频率 fs、截止频率 fc、FIR 阶数 N（系数数量等于 N+1）和窗函数也就能直接得到 FIR 滤波器的参数了。


## IIR 滤波器设计——以巴特沃斯滤波器为例