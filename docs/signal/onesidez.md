# z 变换

未来如果需要从 matlab 的 mlx 文件导出 markdown 文件的话，用下面的指令即可。`EmbedImages` 选项关闭可以让我们以超链接的形式插入图片。更详细的内容，`doc export` 即可。

```matlab
mdfile = export("myscript.mlx",Format="markdown",EmbedImages=false)
```


## z 变换的讨论

z 变换的定义是

$$
X(z) = \sum_{n=-\infty}^{\infty}x[n]z^{-n}
$$

我们先不必考虑太多这个式子的意义，只是从数学的角度把它看成一种**对信号的替代表示**，对每一个 n 乘以一个幂指数系数后再进行叠加。这种看法虽然简单，但是是十分有用的。对于一种给定的 z 变换，$z^{-n}$ 的系数是信号在时间 n 的值。换句话说，给定一个信号，它可以跟一个 z 变换及其收敛域一一对应。

然后我们就来看这样一个问题，为什么在非弛豫条件下，我们不能使用双边 z 变换来求解差分方程呢？

就拿这样的一个很多教科书里都有的差分方程为例子（如奥本哈姆《离散时间信号处理》e3.23）：

$$
y[n]-ay[n-1]=x[n], x[n]=Au[n], y[-1] \neq 0
$$

注意弛豫条件，即 IAR(initial at rest) 的意思是当输入 $x[n]=0$ 时，输出 $y[n]=0$。而这里有初始输出，是不满足的。这会导致什么后果呢？直观的后果就是，上面这个差分方程的等号只有在 $n \geq 0$ 时才成立，而若没有 $y[-1]\neq 0$ 这个条件则对 $(-\infty,+\infty)$ 都成立。

好，我们解这个方程，有的时候想也不想就套公式了，对两边做 z 变换，得到

$$
Y(z)-az^{-1}Y(z) = X(z)
$$

上面这一步的问题出在哪里呢？把具体的过程展开是：

$$
\sum_{n=-\infty}^{\infty}y[n]z^{-n}-a\sum_{n=-\infty}^{\infty}y[n-1]z^{-n}=\sum_{n=-\infty}^{\infty}x[n]z^{-n}
$$

想一想，这个等式成立吗？是不成立的！因为当 $n\leq-1$ 时，差分方程的等号就不成立了。比如说，$y[-2]-ay[-3]=x[-2]$ 就不成立。z 变换本质上是对每一个 n 乘以一个幂指数系数后再进行叠加。而原本单个式子的等号就不成立，对式子进行叠加以后，等号当然就不成立了。

解决方法是，改进一下信号的表示方式，即采用单边 z 变换。单边 z 变换的公式是：

$$
X(z) = \sum_{n=0}^{\infty}x[n]z^{-n}
$$

这次我们用单边 z 变换来进行带入：

$$
\sum_{n=0}^{\infty}y[n]z^{-n}-a\sum_{n=0}^{\infty}y[n-1]z^{-n}=\sum_{n=0}^{\infty}x[n]z^{-n}
$$

这次等号就都成立了。简化为：

$$
Y(z) - az^{-1}Y(z) - ay[-1] = X(z)
$$

后面是常规的做法就不多说了。前面说到 z 变换和信号是一一对应的关系，所以这里求解下去得到 $Y(z)$ 的表达式以后自然就会得到 $y[n]$ 的表达式，对单边 z 变换也一样。

最后得到解的表达式是：

$$
y[n]=a^{n+1}y[-1]+\sum_{k=0}^{n}a^kx[n-k]
$$

简单分析一下，由于初始条件的存在，这个线性常系数差分方程的解包括两个部分，$y_{zi}[n]=a^{n+1}y[-1]$ 是系统的零输入响应，只和初始条件有关，并且衰减。$y_{zs}[n]=\sum_{k=0}^{n}a^kx[n-k]$ 是零状态响应，它是线性的输出。在弛豫条件下，系统的输出将只包含后一部分 $y_{zs}[n]$，此时系统就是线性时不变的 IIR 系统。事实上，弛豫条件下，线性常系数差分方程描绘的系统都是线性时不变的。

**这里有个问题，两本教材说的不符合，在有初始条件下，系统是否是 LTI 的呢？**

我的分析是对的，在有初始条件的情况下，系统是非线性而且时变的。我们要证明一个内容不简单，但是否定却很容易，因为举出一个反例就可以了。

就这个问题来说，当我输入为 $\delta[n]$ 时，$y_1[0]=ay[-1]$，当输入为 $\delta[n-1]$ 时，$y_2[1]=1+a^2y[-1]\neq y_1[0]$，所以是时变的。

当我输入是任意 $x[n]$ 时，$n = -1$ 时输出 $y[-1]$；输入是 $2x[n]$ 时，输出仍然是 $y[-1]$，并未扩大到原来两倍，所以显然是非线性的。

总结而言，双边变换必须基于 IAR 条件（对傅里叶、拉普拉斯变换也是如此）。但实际情况下，虽然输入往往是只有有限时间，但在这之前，不可能保证输出都是 0，因此有初始条件的差分方程更加常见，所以单边 z 变换的应用更加广泛。我们也可以把单边 z 变换看成差分方程的一种普遍解决手段。

另外值得一提的是，在 IAR 条件下，双边 z 变换和单边 z 变换是完全一致的。否则，就不能使用双边 z 变换。从表面上看，单边 z 变换的时移性质也是考虑了初始条件，这是双边 z 变换所没有的。

matlab 的 filter 函数可以添加初值：

对于下面这个方程:

$$
y[n]+1.5y[n-1]+0.5y[n-2]=x[n]-x[n-1], y[-1]=2, y[-2]=1
$$

```matlab
num = [1 -1 0];
den = [1 1.5 .5];
n = 0:20;
x = ones(1,length(n));
%zi = [-a1*y[-1] - a2*y[-2], -a2*y[-1]]
zi = [-1.5*2-0.5*1,-0.5*2]; %zi就是y[n]初值向量
y = filter(num,den,x,zi);
```

## 求解斐波那契数列

用 z 变换可以轻松求解出斐波那契数列的通项公式。已知其递推式是：

$$
y[n+2]=y[n+1]+y[n], y[0]=y[1]=1
$$

做单边 z 变换和时移性质，可得

$$
Y(z) = \frac{1}{1-z^{-1}-z^{-2}}
$$

设 $a_1=\frac{1+\sqrt{5}}{2}, a_2=\frac{1-\sqrt{5}}{2}$

容易得到 $Y(z)$ 的分解式可以被表为，

$$
Y(z)=\frac{a_1a_2}{a_2-a_1}(\frac{a_1}{1-a_1z^{-1}}-\frac{a_2}{1-a_2z^{-1}})
$$

做 z 逆变换得到

$$
y[n]=\frac{a_1a_2}{a_2-a_1}(a_1^{n+1}-a_2^{n+1})
$$

也就是：

$$
y[n]=\frac{1}{\sqrt{5}}((\frac{1+\sqrt{5}}{2})^{n+1}-(\frac{1-\sqrt{5}}{2})^{n+1}),n\geq0
$$

## 生成函数

参考链接：
<https://zhuanlan.zhihu.com/p/63003976>

生成函数（母函数）是组合数学中的一个重要工具，它的形式和单边 z 变换是一样的。数学上的定义，就是对一个序列 $a_n$，它的生成函数定义为 

$$
G(x)=\sum_{n=0}^{\infty}a_nx^n
$$

我们仍用和 z 变换类似的理解，不考虑别的，只考虑它是一种对序列表示的简化。