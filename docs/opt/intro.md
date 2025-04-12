# 优化理论介绍

最优化问题非常常见。有高中学历的读者，求过函数的最大值和最小值，或者线性规划的最优解，这些就是最优化问题。简单说，最优化问题就是求出一个自变量的取值，使得函数取得最值。优化问题的数学定义是：


$$
    p^*=\min_{x} \ f(x) \quad  \\
    \text{subject to:} 
    \quad g_i(x) \leq 0, \quad i = 1, 2, \dots, m, \\
$$

就是，找到一个 $n$ 维向量 $x^*$，使其满足约束函数 $g(x)$ 的限制，使目标函数 $f(x)$ 取得最小值。我们将 $x^*$ 称为原问题的解或最优解，$p^*$ 称为最优值。




人们已经总结出了许多问题的范式以及对应的求解最优值方法。在求解实际问题时，往往希望将问题转化成已知的问题，方便我们求解。以前在读教材的时候，一直不太清楚为何教材大费周章地介绍各类优化问题，却不给解决的算法。现在才知道，教材认为，求解的算法是简单的（不是简单的，而是已经有成熟的算法供给读者直接使用，读者不必知道求解细节），但是难点在于，从实际问题中，抽象出问题的范式，将之转化为有高效求解方法的问题。

我们从最小二乘法开始。

最小二乘法仍然是关于求解矩阵方程 $Ax=y$ 的方法。若 $y$ 不在 $A$ 的列空间中，则 $Ax=b$ 无解！但是，希望能找到一个最好的 $x$，令残差 $r=Ax-y$ 尽可能小。我们用二范数度量 $r$ 的大小。那么最优化问题可以描述成：

$$
\min_x ||Ax - y||_2^2
$$

这实际上是一个二次函数，我们展开求导，可以得到导数为 $0$ 时，$x$ 需要满足方程 $A^T A x = A^T y$。则我们需要的最优解就是 $x = (A^TA)^{-1}A^Ty$。

这是个很好的问题，我们可以直接求出最优解的表达式。实际上很多问题是没有办法这样直接求出来的，但是如果其有最优解，我们可以使用一些迭代的算法来获得最优解。

何时会用到最小二乘问题呢？比如，多项式函数的拟合问题，就可以归结到最小二乘问题。拟合问题，就是说，已知 $n$ 个点的坐标，找到一个最好的函数，使函数与数据点的误差最小。我们以二次函数的拟合为例子：

$$
f(x) = ax^2 + bx + c
$$

我们需要求解最好的$[a, b, c]$，令误差最小。那其实就是求解矩阵方程 $Ax=y$，其中

$$
A = \begin{bmatrix}
x_1^2 & x_1 & 1 \\
x_2^2 & x_2 & 1 \\
\vdots & \vdots & \vdots \\
x_n^2 & x_n & 1 \\
\end{bmatrix}, \quad
y = \begin{bmatrix}
y_1 \\
y_2 \\
\vdots \\
y_n \\
\end{bmatrix}
$$

$$
x = \begin{bmatrix}
a\\
b\\
c
\end{bmatrix}
$$

这样我们就把问题转化成了最小二乘问题，利用通解公式即可求出最优解。

感谢大模型的帮助，我们可以轻松地写出例程：
```python
import numpy as np
import matplotlib.pyplot as plt

def quadratic_fit(x, y):
    """
    使用最小二乘法拟合二次函数 f(x) = ax^2 + bx + c

    参数：
        x: 数据点的 x 坐标 (n维数组)
        y: 数据点的 y 坐标 (n维数组)

    返回：
       x_n, norm_r: 最优系数与残差
    """
    # 构造矩阵 A 和向量 B
    A = np.vstack([x**2, x, np.ones_like(x)]).T
    B = y
 
    # 求解最优解
    x_n =  np.linalg.inv(A.T @ A) @ A.T @ y
    r = y - A @ x_n
    norm_r = np.sqrt(r.T @ r)
    return x_n, norm_r

# 示例数据
x = np.array([1, 2, 3, 4, 5])
y = np.array([1, 2, 2.3, 2 ,1])

# 拟合二次函数
[a, b, c] ,norm_r= quadratic_fit(x, y)
print(f"拟合的二次函数为: f(x) = {a:.2f}x^2 + {b:.2f}x + {c:.2f}, residue = {norm_r}")

# 绘制结果
x_fit = np.linspace(min(x), max(x), 100)
y_fit = a * x_fit**2 + b * x_fit + c

plt.scatter(x, y, label="Data points")
plt.plot(x_fit, y_fit, color='red', label=f"Curve: f(x) = {a:.2f}x^2 + {b:.2f}x + {c:.2f}")
plt.xlabel("x")
plt.ylabel("y")
plt.legend()
plt.title("Quadratic Function")
plt.show()
```