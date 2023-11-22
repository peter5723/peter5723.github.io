# DFS

## 介绍

dfs 应该是我用的次数最多的搜索方法了。只要是和遍历相关的，经常就会用到 dfs。个人认为它和回溯法基本上是一样的，所以我就不加区分了。

虽然问题的形式可能很不一样，但是 dfs 的写法基本上就是同一个模版：
```
dfs:
    1. 如果已经完成赋值（到递归出口了）
        判断是否是解，并操作
    
    2. 确定一个当前未赋值的 var
    3. 对这个 var 所有可能的值进行循环
        1. 将这个 var 赋值 value
        2. 判断 value 是否满足约束条件
            1. 若满足约束，则进行下一层递归
                dfs(next)
            2. 不满足约束，就选择下一个 value 赋值
    4. 如果到这里，说明已经遍历完了所有 var 值。说明没有更多的解了，往回退吧。
```
基本上就是这个结构，可能会有略微的改动。比如说，只找一个解的时候，可以把 dfs 的返回值设成 bool，找到解就直接 return 即可，不用在循环里面继续遍历了。

另外，判断约束条件这步可能简单，也可能很难。在人工智能课上还专门花一章讨论这个问题，就是 CSP(constraint satisfaction problems)，约束满足问题。有多个约束条件时，用 AC-3 弧相容算法来处理约束条件。

下面看几个例子吧：

## 数独问题
<https://leetcode.cn/problems/sudoku-solver/description/>

给你一个数独，解这个数独。

没啥好注意的，直接贴解法了：
```cpp
class Solution {
public:
    void solveSudoku(vector<vector<char>>& board) {
        dfs(board);
    }
    bool dfs(vector<vector<char>>& board){
        for(int i=0;i<9;i++){
            for(int j=0;j<9;j++){
                // 确定下一个待赋值的var
                if(board[i][j]!='.'){
                    continue;
                }else{
                    // 遍历所有可能的赋值
                    for(char k=1+'0';k<=9+'0';k++){
                        board[i][j] = k;
                        // 判断是否满足约束条件
                        if(!isValidSudoku(board)){
                            board[i][j] = '.';
                            continue;
                        }else{
                            //进行下一层递归。这里只需找一个解，dfs就设成bool变量了
                            if(dfs(board)){
                                return true;
                            }else{
                                board[i][j] = '.';
                            }
                        }
                    }
                    return false;
                }

            }
        }
        return true;
    }
    bool isValidSudoku(vector<vector<char>>& board) {
        bool shu[10][10]={false};
        bool heng[10][10]= {false};
        bool fang[4][4][10]= {false};

        for(int i=0;i<9;i++){
            for(int j=0;j<9;j++){
                if(board[i][j]=='.'){
                    continue;
                }else if(heng[i][board[i][j]-'0']==true || shu[j][board[i][j]-'0'] == true 
|| fang[(i)/3][(j)/3][board[i][j]-'0']==true){
                    return false;
                }else{
                    heng[i][board[i][j]-'0'] = true;
                    shu[j][board[i][j]-'0'] = true;
                    fang[(i)/3][(j)/3][board[i][j]-'0'] = true;
                }
            }
        }
        return true;

    }
     
};
```

## n皇后问题
<https://leetcode.cn/problems/n-queens/>

dfs 和栈关系密切，我们可以用栈来保存先前的结果。

```cpp
int count = 0; //保存解的个数
int main() {
    int N; //N 是皇后个数
    int stack[]; //回溯问题，我们可以用栈保存先前的结果
    // 比方说这里，我们存储每一列放皇后的行数。
    seek(stack, 0, N); 
    return 0;
}
void seek(int st[],int k,int N){
    //k 表示当前是第 k 列，相当于模版里确定未赋值的 var 的作用
    for(int i=0;i<N;i++){
        if(!(conflict(st,k,i))){ 
            st[k]=i;//加入
            if(k==N-1){
				printres(st,N);
				count++;
    		}
    		else seek(st,k+1,N); //向下dfs
			st[k]=-1; //取出      
        }  //典型的dfs模板
		   
    } 
}

bool conflict(int st[], int k, int i){
    // 判断有无冲突
    for(int j=0;j<=k;j++){
        if(abs(j-k)==abs(st[j]-i) || st[j]==i){
            return true;
        }
    }
    return false;
}
```

## PAT Top 1006
<https://pintia.cn/problem-sets/994805148990160896/exam/problems/994805154321121280?type=7&page=0>

这道题的大意是，告诉你树的元素个数，和残缺的前序中序后序遍历，判断是否能还原唯一的遍历，若可以的话，给出层序遍历。
比如输入是
```
9
3 - 2 1 7 9 - 4 6
9 - 5 3 2 1 - 6 4
3 1 - - 7 - 6 8 -
```
要输出
```
3 5 2 1 7 9 8 4 6
9 7 5 3 2 1 8 6 4
3 1 2 5 7 4 6 8 9
9 7 8 5 6 3 2 4 1
```
那么这道题的思路也是一样的。用 dfs 对每一个空所有可能的值进行遍历，约束条件是判断能否构成一个树。展开讲讲约束条件：如果还有空的数字，则不做判断，直接返回真值。普通情况就是利用二叉树的性质，树根等于先序遍历之首等于后序遍历之尾，在中序遍历中找到树根，找到树根就可以在中序遍历中分成左右两棵子树，利用分治法来判断了。在其中如果碰到矛盾的地方（如元素个数不等，找不到树根，先序首不等于后序尾，等等），就意味着约束条件不满足，赋值不正确。

得到一个解后，存储它；得到第二个解，就直接退出递归。只有解的个数等于 1 才说明有解。

代码比较冗长，就不放了，因为得到解后，还要写层序遍历的代码。有趣的是，网上能找到的代码虽然能 AC，但是并不能通过所有测试，说明这道题确实是有点儿难度的。

我自认为上面说的思路比较清晰了，感觉就好像是搭好房子以后拆掉脚手架的结果，哈哈，实际上之前陷入了一个别的思路，那种方法有些地方很难处理，花了很久的时间。

## 练习

DFS 的题目非常多，练习一下。

全排列：<https://leetcode.cn/problems/permutations/>

组合总和：<https://leetcode.cn/problems/combination-sum/>

24点：<https://leetcode.cn/problems/24-game/description/>