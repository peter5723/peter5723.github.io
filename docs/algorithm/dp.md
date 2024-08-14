# 动态规划

动态规划到底是什么呢？一般来说，它解决的都是最优解问题。我觉得它和递归有类似之处，因为要构造出递归式（即状态转移方程），只是它是拿空间来换时间。

拿教科书上的概念上说：动态规划要具备两个要素。一是**最优子结构**，一个问题的最优解可以被分解为子问题的最优解，从而形成递归结构，这部分和状态转移方程相对应。二是**子问题重叠**，问题可以反复求解相同的子问题，这使得我们可以存储子问题的解在一个表中，之后查表求解只需常量时间即可了。

## 最长公共子序列
动态规划的问题非常多，随便来几个例子，比如这个最长公共子序列问题：给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长公共子序列的长度。如果不存在公共子序列，返回 0 。

<https://leetcode.cn/problems/longest-common-subsequence/description/?envType=study-plan-v2&envId=dynamic-programming>


这是一个最优解问题。假设 `DP[i][j]` 表示问题 `text[0:i]` 和 `text[0:j]` 的最优解（最长子序列长度）。

那么我们很容易就写出状态转移方程：

```
if text1[i] == text2[j]:
    DP[i][j] = DP[i-1][j-1] + 1
else:
    DP[i][j] = max(DP[i-1][j], DP[i][j-1])
```

然后求出各个状态转移方程的解，有两种做法，自顶向下或者自底向上。

自顶向下就是从后往前，利用递归来求解。自底向上就是从 `DP[0][0]` 开始向后推理到 `DP[i][j]`

自顶向下的伪代码：

```
set DP to -1
f(i, j):
    if i == -1 and j == -1:
        return 0
    if DP[i][j] >= 0:
        return DP[i][j]
    if text1[i] == text2[j]:
        DP[i][j] = f(i-1, j-1) + 1
    else:
        DP[i][j] = max(f(i-1, j), f(i, j-1))
    return DP[i][j]
```

注意，如果没有 `if DP[i][j] > 0 return DP[i][j]`，则其和普通的递归完全相同。正是对解的存储，避免了大量重复的计算。

但是有递归，就还是有多余的开销的。尽管时间复杂度都是相同的 O(MN)，自底向上的方法更快，因为不用递归。

下面是自底向上的伪代码：

```
f(M, N):
    for i = 0 to M:
        for j = 0 to N:
            if (text1[i] == text2[j]):
                DP[i][j] = DP[i - 1][j - 1] + 1;
            else:
                DP[i][j] = max(DP[i][j-1], DP[i-1][j])
    return DP[M][N];
```

自底向上的速度总是更快，所以更推荐使用。


## 接雨水

上面这道题基本已经涉及了动态规划的所有步骤。下面我们再看一个接雨水问题，据称字节的保洁阿姨和保安大叔都会做。

<https://leetcode.cn/problems/trapping-rain-water/description/>

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

好吧做出来怎么感觉和动态规划的关系并没有那么紧密而且感觉也没有很 hard... 是这样，对每一个格子，它能接的雨水的容量 W 大小都是：min(该点左边最高柱子 - 该点右边最高柱子) - 该点格子高度，且大于等于 0。从左到右遍历所有格子相加结果就是题目的解。只是在求该点左右最高柱子的时候可以用上动态规划（不用也行，复杂度高一点）。状态转移方程是：
```
l[i] = max(l[i-1], h[i-1])
r[i] = max(r[i+1], h[i+1])
```
接雨水的总量就是
```
sum(max(min(l[i], r[i]) - h[i] , 0))
```

## 打家劫舍

<https://leetcode.cn/problems/house-robber/description/?envType=study-plan-v2&envId=dynamic-programming>

假设偷窃 n 家，且不能偷窃相邻的房屋，那么最多能偷多少钱。假设每家的钱存在数组 w 中，偷窃前 n 家的前的最优解存储在 dp 中，则状态转移方程是：

```
dp[n] = max(dp[n-2] + w[n], dp[n-1])
```

这种问题熟悉了以后就和求斐波那契数列一样简单的。C++ 代码如下，自底向上，忘记时可作为模版参考。

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
    vector<int> dp(nums.size(), 0);
        for (int i = 0; i < nums.size(); i++) {
            if (i == 0) {
                dp[i] = nums[i];
            } else if ( i == 1) {
                dp[i] = max(nums[0], nums[1]);
            }
             else {
                dp[i] = max(dp[i-1], dp[i-2]+nums[i]);
            }
        }
        return dp[nums.size()-1];
    }
};
```