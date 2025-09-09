# 贪心算法

算法导论里面说，想学习贪心算法，得先熟悉动态规划。这说明，贪心和动态规划有共通之处。

还是拿一个问题来说吧：活动选择问题。考虑一个活动集合 $S = \{a_1, a_2, \dots,a_n\}$，每个活动有一个开始时间 $s_i$ 和一个结束时间 $f_i$，且 $0 \leq s_i < f_i$。这些活动共享一个场地，同一时间在这个场地只能进行一个活动。如果两个活动时间不重叠，则称它们是兼容的。我们的目标是找到一个最大的兼容活动集合。就是这个场地最多能在这些活动中进行多少个活动？

为方便讨论，将活动按结束时间从早到晚进行排序。

下面是这个问题的一个实例：

| 活动编号 a_i | 开始时间 s_i | 结束时间 f_i|
|---------|---------|---------|
| A₁      | 1       | 4       |
| A₂      | 3       | 5       |
| A₃      | 0       | 6       |
| A₄      | 5       | 7       |
| A₅      | 3       | 9       |
| A₆      | 5       | 9       |
| A₇      | 6       | 10      |
| A₈      | 8       | 11      |
| A₉      | 8       | 12      |
| A₁₀     | 2       | 14      |
| A₁₁     | 12      | 16      |

肉眼观察，可以最多排 4 个，下标为 $\{1,4,8,11\}$。

怎么解决这个问题？首先还是以动态规划的思路看，将问题分解成可以重叠的最优子结构。分解的思路很多，我们复述教科书上的思路好了。

考虑子问题 $S_{ij}$ 的最优解集合为 $A_{ij}$，$S_{ij}$ 表示下标为 i 的事件结束之后开始，下标为 j 的时间开始之前结束的事件集合。这个 $A_{ij}$ 包含一个为 $a_k$ 的事件。容易发现有：

$$
A_{ij} = A_{ik} \cup \{a_k\} \cup A_{kj}
$$

从而有

$$
|A_{ij}| = |A_{ik}| + |A_{kj}| + 1
$$

如果用 dp[i,j] 表示子问题 $S_{ij}$ 的最优解集合大小，则有

$$
dp[i,j] =dp[i,k] +  dp[k,j] + 1
$$

由于不知道具体的 k 是哪一个，所以考虑遍历 $S_{ij}$ 中所有活动：

$$
dp[i,j] = \begin{cases}
    \max_{i<k<j}(dp[i,k]+dp[k,j]+1), S_{ij} \neq \emptyset  \\
    0, S_{ij} = \emptyset
\end{cases}
$$

写出了动态规划的状态方程，就可以写出对应的算法代码解决这个问题了，算出 $dp[0,n+1]$ 的值就可以了。

然后接下来我们考虑简化这个问题。贪心算法的思路就是“懒惰”，动态规划的方程中可能有多个情况，而贪心算法选择其中一个情况，这个情况必定是其中一个最优解，从而达到简化算法的目的。

对这个问题来说，贪心选择是“永远寻找最早结束的活动”。

我们只需要证明这个命题：对任一子问题 $S_{ij}$，$a_m$ 是 $S_{ij}$ 中结束最早的活动，那么这个 $a_m$ 一定在 $S_{ij}$ 的最大兼容活动子集中。

证明：设子问题 $S_{ij}$ 的一个最优解为 $A_{ij}$。$a_q$ 是 $A_{ij}$ 中结束最早的活动。若 $a_q=a_m$，则直接证明。若 $a_q \neq a_m$，则考虑集合 $A_{ij}' = A_{ij}-\{a_q\} \cup \{a_m\}$，由于 $a_m$ 是 $S_{ij}$ 中结束最早的活动，所以有 $f_m \leq f_q$，所以 $A_{ij}'$ 也必定是一个兼容子集，其元素个数与 $A_{ij}$ 相同，所以也是一个最优解。证明完毕。


这样问题就简单很多，一个子问题 $S_{ij}$ 被分解成两部分，第一部分 $a_m$ 是 $a_i$ 结束之后第一个结束的活动，第二部分 $S_{mj}$ 是另一个子问题。

于是状态转移方程简化成：

$$
dp[i,j] = dp[m,j] + 1
$$

m 是 $a_i$ 结束之后第一个结束的活动的下标。

此时状态已经和 j 没有关系，我们进一步简化问题，考虑 $S_i$ 是 $a_i$ 结束之后开始的活动集合，其最优解为 dp[i]，则

$$
dp[i] = \begin{cases}
    dp[m] + 1, \text{m exists} \\
    0, \text{m not exists}
\end{cases}
$$

要求原问题的解，计算 $dp[0]$ 的值即可。

然后简单地说一个技术问题，可以看到这次的迭代是从后往前计算的，然而 m 是要从前往后计算的，这样不是矛盾了吗？实际上，我们迭代之前可以先把每个下标对应的 m 的值计算出来。“m 是 $a_i$ 结束之后第一个结束的活动的下标。”这是可以做到的。

下面是解决问题的代码：

```cpp
#include <vector>
#include <iostream>
using namespace std;

vector<vector<std::pair<int, int>>> dp_stack(50);
//计算 m
void cal_m(int n, vector<int> &m, vector<int> s, vector<int> f)
{
    for (int i = 0; i < n; i++)
    {
        m[i] = -1;
        // 按结束时间从低到高排序
        //  j<i 不可能兼容，就不用考虑了
        for (int j = i + 1; j < n; j++)
        {
            if (s[j] >= f[i])
            {
                m[i] = j;
                break;
            }
        }
    }
}
// 计算最优解
int cal_dp(int n, vector<int> m, vector<int> s, vector<int> f)
{
    int dp[n] = {0};
    for (int i = n - 1; i >= 0; i--)
    {
        if (m[i] == -1)
        {
            dp[i] = 0;
        }
        else
        {
            dp[i] = dp[m[i]] + 1;
            dp_stack[i] = dp_stack[m[i]];
            dp_stack[i].push_back(std::make_pair(s[m[i]], f[m[i]]));
        }
    }
    dp_stack[0].push_back(std::make_pair(s[0], f[0]));
    return dp[0] + 1;
    // 根据定义，dp[0]计算的是第一个活动结束之后活动集合的最优解，所以整个活动的数量还要加上第一个活动
}

int main()
{
    // vector<int> s = {1,3,0,5,3,5,6,8,8,2,12};
    // vector<int> f = {4,5,6,7,9,9,10,11,12,14,16};

    // vector<int> s = {2};
    // vector<int> f = {5};

    // vector<int> s = {1, 1, 1, 1};
    // vector<int> f = {5, 5, 5, 5};

    // vector<int> s = {1, 2, 3, 4, 5, 6, 7, 8};
    // vector<int> f = {3, 4, 5, 6, 7, 8, 9, 10};

    vector<int> s = {1, 100, 200, 300, 400};
    vector<int> f = {50, 150, 250, 350, 450};

    int n = f.size();
    vector<int> m(n, 0);
    cal_m(n, m, s, f);
    cout << "The result is " << cal_dp(n, m, s, f) << "\n";
    for (int i = dp_stack[0].size() - 1; i >= 0; i--)
    {
        cout << "(" << dp_stack[0][i].first << "," << dp_stack[0][i].second << ")" << ",";
    }
    return 0;
}
```

然后要说的是，由于贪心算法足够简单，实际上，我们在知道了最优策略以后，也不用真的按状态转移方程来写，而是直接模拟最优策略的行为就可以了（比如这个问题就是一直取第一个结束的活动），如下面代码所示：

```cpp
#include <vector>
#include <iostream>
using namespace std;

vector<std::pair<int, int>> dp_stack;
//实际上对应了计算 m 的过程。
int greedy_search(vector<int> s, vector<int> f)
{
    int n = s.size();
    int res = 0;
    int old_f = 0;
    for (int i = 0; i < n; i++)
    {
        if (s[i] >= old_f)
        {
            dp_stack.push_back(std::make_pair(s[i], f[i]));
            res++;
            old_f = f[i];
        }
    }
    return res;
}


int main()
{
    vector<int> s = {1, 3, 0, 5, 3, 5, 6, 8, 8, 2, 12};
    vector<int> f = {4, 5, 6, 7, 9, 9, 10, 11, 12, 14, 16};

    // vector<int> s = {2};
    // vector<int> f = {5};

    // vector<int> s = {1, 1, 1, 1};
    // vector<int> f = {5, 5, 5, 5};

    // vector<int> s = {1, 2, 3, 4, 5, 6, 7, 8};
    // vector<int> f = {3, 4, 5, 6, 7, 8, 9, 10};

    // vector<int> s = {1, 100, 200, 300, 400};
    // vector<int> f = {50, 150, 250, 350, 450};

    int n = f.size();

    cout << "The result is " << greedy_search(s, f) << "\n";
    for (int i = 0; i < dp_stack.size(); i++)
    {
        cout << "(" << dp_stack[i].first << "," << dp_stack[i].second << ")" << ",";
    }

    return 0;
}
```

以上，我们简单介绍了贪心算法。其由动态规划算法演化而来，基本思想仍然是类似的将问题分解为最优子结构，但是其贪心选择性质又大大减少了计算量，因为我们不必考虑子问题的解。

贪心算法的设计步骤：

- 将最优化问题转化为这样的形式：做出一次选择后只剩一个子问题需要求解。
- 证明做出贪心选择后，原问题存在最优解。
- 证明做出贪心选择后，剩下的子问题满足性质：其最优解和贪心选择的组合可以构成原问题的最优解。