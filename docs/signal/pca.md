# PCA

将 PCA 方法放在信号处理里面，是想说，信号处理和数据处理本质是相通的，也算是某种复习。


统一说明，下面涉及的数据向量均为列向量。


## SVD 分解

首先讲解一下 SVD 分解：对某一个矩阵 $A$ 为 $m * n$ 的实矩阵，那么他的奇异值分解存在：

$$
A  = USV^T
$$

其中 $U$ 是 $m * m$ 正交矩阵，$V$ 是 $n * n$ 正交矩阵，$S$ 是 $m*n$ 矩形对角矩阵，其对角线元素非负且按降序排列。

注意，若 $A$ 的 rank 是 $k$，那么 $U$ 的前 $k$ 个列向量构成 $A$ 的列空间的标准正交基。$V$ 的前 $k$ 个列向量构成 $A$ 的行空间的标准正交基，后 $n-k$ 个向量构成 $A$ 的零空间的标准正交基。

奇异值分解可以写成下面的单向量分解式：

<!-- $$
A = USV^T = [u_1, u_2 \dots u_m]diag(\sigma_1,\sigma_2,\dots,\sigma_r,0,\dots,0)[v_1,v_2,\dots,v_n]^T = [\sigma_1 u_1, \sigma_2 u_2, \dots, \sigma_r u_r, 0,\dots,0][v_1,v_2,\dots,v_n]^T
= \sum_{i=1}^{r}\sigma_i u_i v_i^T
$$ -->

$$
A = USV^T = \sum_{i=1}^{r}\sigma_i u_i v_i^T
$$

由单向量分解式，我们很容易理解紧奇异值分解。如果丢掉后面某些不重要的项，就成了截断奇异值分解，用于矩阵近似。

## PCA

这个博客的解释相当不错：<https://alberthg.github.io/2018/12/05/%E4%B8%BB%E6%88%90%E5%88%86%E5%88%86%E6%9E%90-PCA/>。可以看看。

主成分分析，本质上就是对数据进行线性变换。首先有这样的一个随机变量向量 $\mathbf{x}=(x_1,x_2,\dots,x_m)^T$。然后进行线性变换后，要求，对第 $i$ 个新坐标 $y_i=\alpha_{1i} x_1 + \alpha_{2i} x_2 + \dots + \alpha_{mi} x_m$，$y_i$ 是方差第 $i$ 大的坐标，称作 $x$ 的第 $i$ 个主成分。不妨将第 $i$ 个坐标所需线性变换的向量记作 $\mathbf{\alpha}_i = [\alpha_{1i}, \alpha_{2i}, \dots, \alpha_{mi}]^T$。那么数据变换矩阵 $\mathbf{A} = [\mathbf{\alpha}_1, \mathbf{\alpha}_2, \dots, \mathbf{\alpha}_m]$。变换后的向量 $\mathbf{y}$ 就是：

$$
\mathbf{y} = \mathbf{A}^T\mathbf{x}
$$

关键要求 $A$ 的各个向量。数学课上学过，第 k 个向量，$\alpha_k$，只需要对 x 的协方差矩阵 $\Sigma$（大小$m*m$） 进行特征值分解，得到第 k 个特征值对应的特征向量即可。换句话说，$A$ 的 m 个向量就是 $\Sigma$ 的 m 个特征向量。

然后对 $y$ 的值，丢弃掉一些维度，就达到了降维的效果。

上面说的是随机变量。实际操作时，都是给你样本矩阵。那只需要

$$
\mathbf{Y} = \mathbf{A}^T\mathbf{X}
$$

即可。

matlab 中，PCA 的函数原型是：

```matlab
[coeff, score, latent, tsquared, explained, mu] = pca(X)
```

其中，`coeff` 就是上面的矩阵 A，包含了样本矩阵 X 的协方差矩阵的各个特征向量。`score` 就是是数据投影到主成分空间之后的坐标，即每个样本在新坐标系（主成分坐标系）中的表示，其矩阵大小应该与 X 相同（实际上就是 Y）。`latent` 是一个列向量，储存协方差矩阵的特征值，实际上表示每个主成分的方差大小，意义就是贡献度。

然后，由于 A 是正交矩阵，所以有 $A^TA=1$ 的性质，这个性质让我们从 Y 中重构 X 非常简单：$X = AY$。所以如果我们要只取前几个主分量对数据进行降噪操作，只需对 A 和 Y 进行截断即可，只取前几个列向量，即 $X_{r} = A_tY_t$。


用 Matlab 进行 PCA 滤波的代码很简单。假如 `x` 是一个随机信号的样本矩阵, `reducedData` 就是滤波后的样本矩阵。

```matlab
[coeff, score, latent, tsquared, explained, mu] = pca(x);

% 选择主要成分（例如，选择前2个主要成分）
numComponents = 2;
reducedData = score(:, 1:numComponents) * coeff(:, 1:numComponents)' + mu;
plot(n, reducedData(:,3));
title("use PCAtools")
```

也可以查看各个主成分分量的大小：

```matlab
stem(latent/sum(latent));
title("各分量")
```

插入图片的意义不大，懒的插了。