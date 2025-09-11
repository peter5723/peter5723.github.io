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

我们在设计时也不需要一定按动态规划考虑。

实际上，贪心算法的难点在于想不到这个贪心选择。如果知道了一个贪心选择就容易实现。有时候很容易看出来贪心法对动态规划的简化，有时候又很不明显。还有贪心选择的证明也是难点。经常会出现直觉能找到贪心选择，但是难以证明的情况。

下面看看几个问题例子。

加油问题：（leetcode 134）

在一条环路上有 n 个加油站，其中第 i 个加油站有汽油 gas[i] 升。你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。给定两个整数数组 gas 和 cost，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1 。如果存在解，测试用例保证它唯一。

有解的定义：存在某一站，从该站出发，油量始终不小于 0。

命题 1：若加油总和小于损耗和，则无解。证明：考虑逆否命题：若有解，即从某一站出发，油量与损耗量之差总大于 0，分别求和，即得油量和大于损耗和，逆否命题成立。则原命题成立。

命题 2：若加油总和不小于损耗和，则必定有解。证明：考虑逆否命题：若无解，即从任何一站出发，总存在一个节点，油量会小于 0。接下来的论证有点拗口：考虑从任何一个节点 a 出发，开到一个节点 e，油量小于 0，那么从途中的 b、c、d 点出发也一定到不了 e 点。因为有两种情况。一是 a 点根本到不了 b 点。如果能到达 b 点，那么此时的油量一定比直接从 b 点出发要多。这样的油量都到不了 e 点，那么直接出发也到不了。对从 a 点到 e 点这段路程，有总油量小于总消耗量。然后从 e 点开始开，由无解的定义，一定到某点之前的油量会小于 0，假设是 h 点。这里也有一个不等式，油量小于消耗量。这样依次循环操作，因为是回路，所以一定会回到有一段路程，会停到 a 点或 a 点之后。这样就可以对所有的不等式进行累加，从而得到对整个回路：加油的总和小于损耗和的结果。逆否命题成立。故原命题成立。

由命题 2 的证明，我们可以写出策略：从任意一站出发，如果到某一站之前没有油了，就从这一站继续走，直到找到一个点走完整个回路或者无解。

代码如下：

```cpp
class Solution
{
public:
    int canCompleteCircuit(vector<int> &gas, vector<int> &cost)
    {
        int n = gas.size();
        // res 是我们希望求的站点解
        int res = 0;
        int now_gas = 0;
        int count_road = 0;
        // 一定不会是解的站点。
        // 从 i+1 开始，到 i+m+1 缺油，则 i+1 到 i+m 都不是解。
        while (res < n)
        {
            now_gas = 0;
            count_road = 0;
            for (int i = res; count_road <= n; i++, count_road++)
            {

                int j = i % n;
                if (now_gas + gas[j] - cost[j] >= 0)
                {
                    now_gas += (gas[j] - cost[j]);

                    continue;
                }
                else
                {
                    res = i + 1;
                    break;
                }
            }
            if (count_road == n + 1)
            {
                return res;
            }
        }
        return -1;
    }
};
```


命题 3：若存在解，相对值的最小值那个点必定是一个解。（相对值，从任何一点出发，其相对值为 0，下一个点的相对值为前一个点加油后损耗得到的值，类似于电压）。证明：从相对值的最小值这个点出发，其初始值为 0。则路上的其他值必定高于 0，即剩下的油量，否则与相对值最小值矛盾。符合有解的定义。从而得证。

由命题 1 和 3 可以写出非常简单的代码。

```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int all_cost = 0;
        int all_gas = 0;
        int num = 0;
        int min_num = num;
        int index = 0;
        for(int i = 0; i < gas.size(); i++) {
            all_cost += cost[i];
            all_gas += gas[i];
            num = num + gas[i] - cost[i];
            if(num < min_num) {
                index = i + 1;
                min_num = num;
            }
        }
        if(all_cost > all_gas){
            return -1;
        }
        return index % gas.size();
    }
};
```

（另外，这个问题是否是贪心问题也是值得商榷的。题解说的方法是贪心法，但是此题似乎并不是最优化问题）

找到一份很好的贪心题单，打算在里面挑几道做做。

[贪心题单](https://leetcode.cn/discuss/post/3091107/fen-xiang-gun-ti-dan-tan-xin-ji-ben-tan-k58yb/)



一个细微的差别就可以导致问题贪心性质的改变。我们来看下面两个问题，装苹果问题和硬币找零问题。

装苹果问题是（leetcode 3074）。描述：有一堆苹果和几个大小不一的箱子。给出所需箱子的最小数量。

硬币找零问题：假设你有若干不同面值的硬币，要找零给顾客，给出所需硬币的最小数量。

装苹果问题就可以用贪心算法，策略就是一直选最大的箱子。但是硬币找零问题不行，因为装苹果箱子可以剩余，但是找零有个额外的条件，找零给顾客的钱必须是精确的不能多出来。

动态规划的方程几乎是一致的：

$$
dp[i] = \min_k\{dp[i-k_j]+1\}
$$

$k_j$ 对应箱子大小、硬币面值。

但是当 $i<\max\{k_j\}$ 时有些许不同。对装苹果问题来说，此时 $dp[i]=\max\{k_j\}$ 即可，但是对硬币找零来说则仍然要进行余下的判断。


不同整数的最少数目（leetcode 1481）

给定一个整数数组和一个整数 k，现在需要从数组中恰好移除 k 个元素，请找出移除后数组中不同整数的最少数目。

直觉想到的策略就是按数组各个元素出现的次数排序，移除出现次数最小的元素即可。

```cpp
class Solution
{
public:
    int findLeastNumOfUniqueInts(vector<int> &arr, int k)
    {
        map<int, int> my_map;
        for (int i = 0; i < arr.size(); i++)
        {
            my_map[arr[i]]++;
        }

        vector<pair<int, int>> vec(my_map.begin(), my_map.end());
        std::sort(vec.begin(), vec.end(), [](const pair<int, int> &a, const pair<int, int> &b)
                  { return a.second < b.second; });

        int j = 0;
        while (k > 0)
        {
            k--;
            vec[j].second--;
            if (vec[j].second == 0)
            {
                j++;
            }
        }
        return vec.size() - j;
    }
};
```
