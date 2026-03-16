# lc100

## 哈希表

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());
        int ans = 0;
        for(int x:st){
            if(st.contains(x-1)){
                continue;
            }

            int y =x+1;
            while(st.contains(y)) {
                y++;
            }

            ans = max(ans, y-x);
        }
        return ans;
    }
};
```
这个解法里面每个元素最多被访问两次（遍历一次结算一次），所以时间复杂度为 O(n)。

## 双指针
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n = nums.size(), left = 0, right = 0;
        while(right<n){
            if(nums[right]) {
                swap(nums[left], nums[right]);
                left++;// 保证left的结尾是一个0。
            }
            right++;
        }
    }
};
```

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ans = 0, left =0 ,right=height.size()-1;
        while(left<right){
            int area = (right-left)*min(height[left], height[right]);
            ans = max(ans, area);
            if(height[left]<height[right]){
                left++;
            }else{
                right--;            
            }
        }
        return ans;
    }
};
```

## 滑动窗口


```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int n = s.length(), ans = 0,left = 0;
        unordered_set<char> window;
        for(int right=0;right<n;right++){
            char c=s[right];
            while(window.count(c)){
                window.erase(s[left]);
                left++;
            }
            window.insert(c);
            ans = max(ans, right-left+1);
        }
        return ans;
    }
};
```
此题已采取公式做法。`[left,right]`. right 遍历，left 更新边界。

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        int sLen = s.size(), pLen = p.size();
        if(sLen < pLen) {
            return {};
        }
        vector<int> ans;
        vector<int> sCount(26);
        vector<int> pCount(26);

        for(int i=0;i<pLen;i++) {
            sCount[s[i]-'a']++;
            pCount[p[i]-'a']++;
        }
        if(sCount==pCount){
            ans.push_back(0);
        }
        for(int i=0;i<sLen-pLen;i++) {
            sCount[s[i]-'a']--;
            sCount[s[i+pLen]-'a']++;
            //这两句话就模拟了滑动窗口向前移动
            if(sCount == pCount) {
                ans.push_back(i+1);
            }
        }
        return ans;
    }
};
```

定长滑动窗口


## 数组
最大子数组和
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        //f(i) = max(f(i-1)+nums[i], nums[i])
        int pre = 0, maxAns = nums[0];
        for(auto &x:nums){
            pre = max(pre+x,x);
            maxAns = max(maxAns, pre);
        }
        return maxAns;
    }
};
```

轮转数组：三次翻转

```cpp
class Solution {
public:

    void reverse(vector<int>& nums, int start, int end) {
        while(start<end){
            swap(nums[start], nums[end]);
            start++;
            end--;
        }
    }

    void rotate(vector<int>& nums, int k) {
        k%=nums.size();
        reverse(nums, 0, nums.size()-1);
        reverse(nums, 0, k-1);
        reverse(nums, k, nums.size()-1);
    }
};
```


## 链表

翻转链表，请背诵

中间节点，快慢指针经典应用


随机链表的复制：这道题最简单的思路就是把随机链表看成图的复制。

然后就是 hashtable 加回溯就可以了。

```cpp
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        if(head==nullptr){
            return nullptr;
        }
        ListNode* slow = head;
        ListNode* fast = head;
        while(fast!=nullptr && fast->next!=nullptr){
            slow = slow->next;
            fast = fast->next->next;
        } 
        return slow;
    }

    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr, *cur = head;
        while(cur){
            ListNode* next = cur->next;
            cur->next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
    bool isPalindrome(ListNode* head) {
        ListNode* mid = middleNode(head);
        ListNode* head2 = reverseList(mid);
        while(head2){
            if(head->val != head2->val) {
                return false;
            }

            head = head->next;
            head2 = head2->next;
        }
        return true;
    }
};
```

注意，这种情况下解决问题最清楚，但是在翻转链表的时候会破坏链表结构

典型错误：
```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* head = new ListNode();
        ListNode* ans = head;
        int  now_ten = 0;
        while(l1 || l2) {
            int n1 = l1 ? l1->val:0;
            int n2 = l2 ? l2->val:0;
            ans->val = (n1 + n2+now_ten)%10;
            now_ten = (n1+n2+now_ten >=10) ? 1 : 0;
            if(l1) l1 = l1->next;
            if(l2) l2 = l2->next;
            ListNode* temp = new ListNode();
            ans->next = temp;
            ans = ans->next;
        }
        if(now_ten>0){
            ans->val = now_ten;
            ans->next = nullptr;
        }else{delete(ans);}
        return head;;
        
    }
};
```
后创建链表后free，会导致前一个next成为野指针，从而导致错误。

推荐使用哨兵

删除倒数第 n 个节点，快指针先走 n 步即可。
    unordered_map<Node*, Node*> cachedNode;
    Node* copyRandomList(Node* head) {
        if(head == NULL) {
            return NULL;
        }
        if(!cachedNode.count(head)) {
            Node* headNew = new Node(head->val);
            cachedNode[head] = headNew;
            headNew->next = copyRandomList(head->next);
            headNew->random = copyRandomList(head->random);
        }

        return cachedNode[head];
    }
};
```



对链表的排序用归并排序。记住就可以了。

LRU 缓存看似玄乎，其实很简单，o(1) 的get 和 set 限制了必须用链表，又要 o(1) 的查找，就用unoredered_map

```cpp
struct Node{
    int key;
    int value;
    Node* prev;
    Node* next;

    Node(int k=0, int v=0):key(k), value(v) {}
};


class LRUCache {
private:
    int capacity;
    Node* dummy;
    unordered_map<int, Node*> key_to_node;

    void remove(Node* x) {
        x->prev->next = x->next;
        x->next->prev = x->prev;
    }
    void push_front(Node *x) {
        x->prev = dummy;
        x->next = dummy->next;
        x->prev->next = x;
        x->next->prev = x;
    }

    Node* get_node(int key) {
        auto it = key_to_node.find(key);
        if(it == key_to_node.end()) {
            // or !key_to_node.count(key)
            return nullptr;
        }
        Node* node = it->second;
        remove(node);
        push_front(node);
        return node;
    }
public:
    LRUCache(int capacity) {
        this->capacity = capacity;
        dummy = new Node();
        dummy->prev = dummy;
        dummy->next = dummy;
    }
    
    int get(int key) {
        Node* node = get_node(key);
        return node ? node->value : -1;
    }
    
    void put(int key, int value) {
        Node* node = get_node(key);
        if(node) {
            node->value = value;
            return;
        }
        key_to_node[key] = node = new Node(key, value);
        push_front(node);
        if(key_to_node.size() > capacity) {
            Node* back_node = dummy->prev;
            key_to_node.erase(back_node->key);
            remove(back_node);
            delete back_node;
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

合并 k 个升序列表这道题，最好用的是最小堆：每次都可以弹出堆中的最小节点 x，那么思路就和两个升序列表完全一致
并且，也用哨兵。

```cpp
class Solution {
public:
    struct Status {
        int val;
        ListNode* ptr;
        bool operator<(const Status& rhs) const { return val > rhs.val; }
    };
    priority_queue<Status> q;

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        for (auto node : lists) {
            if (node)
                q.push({node->val, node});
        }
        ListNode head, *tail = &head;
        while (!q.empty()) {
            auto f = q.top();
            q.pop();
            tail->next = f.ptr;
            tail = tail->next;
            if (f.ptr->next)
                q.push({f.ptr->next->val, f.ptr->next});
        }
        return head.next;
    }
};
```


## 树

先从层序遍历开始，层序遍历用的队列，非常公式化
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ret;
        if(!root){
            return ret;
        }

        queue <TreeNode*> q;
        q.push(root);
        while(!q.empty()) {
            int currentLevelSize = q.size();
            ret.push_back(vector<int>());
            for(int i=1;i<=currentLevelSize;i++){
                auto node = q.front(); q.pop();
                ret.back().push_back(node->val);
                if(node->left) q.push(node->left);
                if(node->right) q.push(node->right);
            } 
        }

        return ret;
    }
};
```
我这样可以保证每一次 while 循环里面我都只对一层进行遍历。
c++ list.back() 返回数组的最后一个元素。

### 二叉搜索树：

对所有树节点，都满足，左子树上所有节点的值都小于根节点的值，
右子树上所有节点的值都大于根节点的值，
左右子树分别是二叉搜索树

平衡二叉搜索树保持树的高度尽可能小

AVL树任意节点的左右子树高度差不超过 1

首先碰到的第一题是将有序数组转换为二叉搜索树，可以考虑用分治法。（因为二叉搜索树是递归定义的，所以它的方法也往往是递归的）

```cpp
class Solution {
public:
    TreeNode* dfs(vector<int>& nums, int left, int right) {
        if (left == right) {
            return nullptr;
        }
        int m = left + (right - left) / 2;
        return new TreeNode(nums[m], dfs(nums, left, m),
                            dfs(nums, m + 1, right));
    }
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return dfs(nums, 0, nums.size());
    }
};
```

检查二叉搜索树，会碰到一个经典的逻辑错误，[5,4,6,null,null,3,7] 这个样例，因此递归的时候，对每棵树的节点，都要检查它的上下界情况是否满足。


```cpp
class Solution {
public:


    bool helper(TreeNode* root, long long lower, long long upper) {
        if(root == nullptr) {
            return true;
        }
        if(root->val <= lower || root->val >= upper) {
            return false;
        }
        return helper(root->left, lower, root->val) && helper(root->right, root->val, upper);
    }
    bool isValidBST(TreeNode* root) {
        return helper(root, LONG_MIN, LONG_MAX);
    }
};
```

当然更巧妙的方法是直接中序遍历比较大小。中序遍历的迭代法是用栈记录。

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        while(root!=nullptr || stk.empty()==false){
            while(root!=nullptr){
                stk.push(root);
                root=root->left;
            }
            //左中
            root = stk.top();
            stk.pop();
            res.push_back(root->val);
            root=root->right;
            //右
        }
        return res;
    }
};
```

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        // 因为自然地发现每层只有一个可以满足，就是层序遍历的每层最后一个，就可以了。
        // 复习一下各种遍历吧。
        queue<pair<int, TreeNode*>> myQueue;
        // 第一个储存当前层数，第二个储存节点。
        vector<int> ansVec;
        if(!root) return ansVec;
        myQueue.push(make_pair(0, root));
        while (!myQueue.empty())
        {
            pair<int, TreeNode*> tmp = myQueue.front();
            myQueue.pop();
            if(tmp.second->left) {
                myQueue.push(make_pair(tmp.first+1, tmp.second->left));
            }
            if(tmp.second->right) {
                myQueue.push(make_pair(tmp.first+1, tmp.second->right));
            }
            if(myQueue.empty() || tmp.first != myQueue.front().first) {
                ansVec.push_back(tmp.second->val);
            }
        }
        return ansVec;

    }
};
```

二叉树前序中序遍历可以构造树，如下一题所示。

快速由值定位到index构建一个哈希表就可以了

```cpp
class Solution {
private:
    unordered_map<int, int> index;

public:
    TreeNode* mybuildTree(const vector<int>& preorder,
                          const vector<int>& inorder, int preorder_left,
                          int preorder_right, int inorder_left,
                          int inorder_right) {
        if (preorder_left > preorder_right) {
            return nullptr;
        }
        int preorder_root = preorder_left;
        int inorder_root = index[preorder[preorder_left]];

        TreeNode* root = new TreeNode(preorder[preorder_root]);
        int size_left_subtree = inorder_root - inorder_left;
        root->left = mybuildTree(preorder, inorder, preorder_left + 1,
                                 preorder + size_left_subtree, inorder_left,
                                 inorder_root - 1);
        root->right = mybuildTree(
            preorder, inorder, preorder_left + size_left_subtree + 1,
            preorder_right, inorder_root + 1, inorder_right);
        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = preorder.size();
        for(int i=0;i<n;i++) {
            index[inorder[i]] = i;
            // 如何快速的从值创建下标，可以用 哈希表
        }
        return mybuildTree(preorder, inorder, 0, n-1,0, n-1);
    }
};
```

路径总和这道题，思路和和为 k 的子数组做法一模一样。枚举路径的终点，统计有多少个起点


最近公共祖先
```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(!root || root==p || root==q) {
            return root;
        }
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p,q);
        if(left && right) return root;
        return left ? left : right;
    }
};
```

二叉树中的最大路径和，这本质上是一个图的问题，我们这里就用 dfs 算每个节点的最大贡献值，同时更新结果：

```cpp
class Solution {
public:

    int maxSum = INT_MIN;
    int maxGain(TreeNode * node) {
        if(node == nullptr) {
            return 0;
        }

        int leftGain = max(maxGain(node->left), 0);
        int rightGain = max(maxGain(node->right), 0);

        int priceNewpath = node->val + leftGain + rightGain;

        maxSum = max(maxSum, priceNewpath);
        // 一边算出每个节点的最大贡献值一边更新回答的最大值
        return node->val + max(leftGain, rightGain);
    }
    int maxPathSum(TreeNode* root) {
        maxGain(root);
        return maxSum;
    }
};
```

## 图论

岛屿数量问题：很经典地用 dfs 涂色，然后计算岛屿数量。

```cpp
class Solution {
public:
    void dfs(vector<vector<char>>& grid, int i, int j, int m, int n) {
        // 将 (坐标i,j) 所在的岛屿每个位置都做上标记
        if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] != '1') {
            return;
        }
        grid[i][j] = 2;
        dfs(grid, i, j - 1, m, n);
        dfs(grid, i, j + 1, m, n);
        dfs(grid, i - 1, j, m, n);
        dfs(grid, i + 1, j, m, n);
    }

    int numIslands(vector<vector<char>>& grid) {
        int m = grid.size(), n = grid[0].size();
        int ans = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '1') {
                    dfs(grid, i, j, m,
                        n); // 注意，dfs 可以保证，我在循环里碰到的 1
                            // 一定是我没有跑到的新岛屿
                    ans++;
                }
            }
        }
        return ans;
    }
};
```

烂橘子这道题就是对多头 BFS 的模拟：

```cpp
class Solution {
    int DIRECTIONS[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

public:
    int orangesRotting(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        int fresh = 0;
        vector<pair<int, int>> q;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    fresh++;
                } else if (grid[i][j] == 2) {
                    q.push_back({i, j});
                }
            }
        }

        int ans = 0;
        while (fresh && !q.empty()) {
            ans++;
            vector<pair<int, int>> nxt;
            for (auto& [x, y] : q) {
                // 注意这里的C++写法，pair 赋值给两变量
                for (auto d : DIRECTIONS) {
                    int i = x + d[0], j = y + d[1];
                    if (i >= 0 && i < m && j >= 0 && j < n &&
                        grid[i][j] == 1) {
                        fresh--;
                        grid[i][j] = 2;
                        nxt.push_back({i, j});
                    }
                }
            }
            q = move(nxt);
        }
        return fresh ? -1 : ans;
    }
};
```



课程表：拓扑排序的典型例子

用邻接表和入度就可以了。

```cpp
class Solution {
private:
    vector<vector<int>> edges;
    vector<int> indeg;
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        edges.resize(numCourses);
        indeg.resize(numCourses);
        // 首先构造图：用表存储图，并且存储每个节点的入度
        for(const auto& info: prerequisites) {
            edges[info[1]].push_back(info[0]);
            indeg[info[0]]++;
        }

        // 然后接下来就是用广度优先搜索就可以了，不断将入度为0的课程加入队列学习。
        queue<int> q;
        for(int i=0;i<numCourses;i++){
            if(indeg[i]==0){
                q.push(i);
            }
        }
        int visited = 0;
        while(!q.empty()){
            visited++;
            int u = q.front();
            q.pop();
            for(int v:edges[u]) {
                indeg[v]--;
                if(indeg[v]==0){
                    q.push(v);
                }
            }
        }
        // 如果存在环那么入度就不可能变成 0。
        return visited == numCourses;

    }
};
```

前缀树。

按照 0x3f，最简单的理解方式就是理解成 26 叉树，每一个字母对应不同的分叉，结束了。

```cpp
struct Node {
    Node* son[26]{};
    bool end = false; // 标志一个单词是否结束
};

class Trie {
private:
    Node* root = new Node();
    int find(string word) {
        Node* cur = root;
        for (char c : word) {
            c -= 'a';
            if (cur->son[c] == nullptr) {
                return 0;
            }
            cur = cur->son[c];
        }
        return cur->end ? 2 : 1;
    }

public:
    Trie() {}

    void insert(string word) {
        Node* cur = root;
        for (char c : word) {
            c -= 'a';
            if (cur->son[c] == nullptr) {
                cur->son[c] = new Node();
            }
            cur = cur->son[c];
        }
        cur->end = true;
    }

    bool search(string word) { return find(word) == 2; }

    bool startsWith(string prefix) { return find(prefix); }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 */
```


## 回溯

括号生成：阿里常考这道题

全排列：

```cpp
class Solution {
public:
    void dfs(vector<int>& nums, vector<vector<int>>& ans, vector<int>& path,
             unordered_set<int>& has_exist) {
        int n = nums.size();
        if (path.size() == nums.size()) {
            ans.push_back(path);
            return;
        }
        for(int i=0;i<n;i++) {
            if(has_exist.count(nums[i])) {
                continue;
            }
            path.push_back(nums[i]);
            has_exist.insert(nums[i]);
            dfs(nums, ans, path, has_exist);
            path.pop_back();
            has_exist.erase(nums[i]);
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> ans;
        vector<int> path;
        unordered_set<int> has_exist;
        dfs(nums, ans, path, has_exist);
        return ans;
    }
};
```


组合总和：

```cpp
class Solution {
public:
    void dfs(vector<int>& candidates, vector<vector<int>>& ans,
             vector<int>& path, int target, int back_index) {
        if (target < 0) {
            return;
        }
        if (target == 0) {
            ans.push_back(path);
            return;
        }
        int n = candidates.size();

        for (int i = back_index; i < n; i++) {
            path.push_back(candidates[i]);
            dfs(candidates, ans, path, target - candidates[i], i); // 这个地方是 i，不是 back_index，想象一下即可
            path.pop_back();
        }
    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> ans;
        vector<int> path;
        dfs(candidates, ans, path, target, 0);
        return ans;
    }
};
```


## 栈

涉及表达式解析的题目，一定用栈
思路都一样，不是右括号就一直入栈
是右括号就出栈处理。

```cpp
class Solution {
public:
    // 获取字符串 s 中从ptr处开始的数字
    string getDigits(string& s, size_t& ptr) {
        string ret = "";
        while (isdigit(s[ptr])) {
            ret.push_back(s[ptr]);
            ptr++;
        }
        return ret;
    }
    // 将字符串拼接成字符串，拼接用+即可。
    string getString(vector<string>& v) {
        string ret;
        for (const auto& s : v) {
            ret += s;
        }
        return ret;
    }

    string decodeString(string s) { 
        vector<string> stk; 
        size_t ptr = 0;
        while(ptr<s.size()) {
            char cur = s[ptr];
            if(isdigit(cur)) {
                string digits = getDigits(s,ptr); // getDigits 会更新ptr
                stk.push_back(digits);
            } else if(isalpha(cur)||cur=='['){
                stk.push_back(string(1, s[ptr]));
                ptr++;
            } else {
                // 是右括号，出stack
                ptr++;
                vector<string> sub;
                while(stk.back()!="[") {
                    // 读取[中的所有字符
                    sub.push_back(stk.back());
                    stk.pop_back();
                }
                reverse(sub.begin(), sub.end());
                stk.pop_back();// 把[也弹出来，此时stk首部是数字
                int repTime = stoi(stk.back());
                stk.pop_back();// 弹数字处理
                string t;
                string o = getString(sub);
                while(repTime) {
                    t+=o;
                    repTime--;
                }
                stk.push_back(t);
            }
        }
        return getString(stk);
    }
};
```


每日温度这道题，是单调栈的典型题目，单调栈就是顾名思义的真的单调栈，从栈底到栈顶温度递减，并每次进入新元素都维护这个关系，将温度比新元素低的元素全部弹出，就可以记录气温的升高了。

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        vector<int> ans(n);
        stack<int> st; // 存储下标
        for (int i = 0; i < n; i++) {
            int t = temperatures[i];
            while (!st.empty() && t > temperatures[st.top()]) {
                int j = st.top();
                st.pop();
                ans[j] = i - j;
            }
            st.push(i);
        }
        return ans;
    }
};
```

另外，虽然这也是二重循环，但是每一个元素入栈一次出栈一次，所以还是 o(n)。


## 排序

数组的第 k 个最大元素，用快速排序可以达到 o(n) 的速度。

我们来学习一下快速排序

```cpp
class Solution {
public:
    int partition(vector<int>& nums, int left, int right) {
        
        int i = left + rand() % (right - left + 1);
        int pivot = nums[i];

        swap(nums[i], nums[left]);

        i = left + 1;
        int j = right;
        // 只需要记住，未处理的区间是 [i,j] 就可以了
        while (true) {
            while (i <= j && nums[i] < pivot) {
                i++;
            }
            while (i <= j && nums[j] > pivot) {
                j--;
            }

            if (i >= j) {
                break;
            }
            swap(nums[i], nums[j]);
            i++;
            j--;
        }
        swap(nums[left], nums[j]);
        return j;
    }
    void quick_sort(vector<int>& nums, int left, int right) {
        if(left>=right) {
            return;
        }
        int i = partition(nums, left, right);
        quick_sort(nums, left, i-1);
        quick_sort(nums, i+1, right);
    }

    vector<int> sortArray(vector<int>& nums) {
        quick_sort(nums, 0, (int)nums.size()-1);
        return nums;
    }
};
```

关键也就是一个找到 pivot，进行划分的过程。只要这个 pivot 比较趋近数组的中间值，就会很快。

## 动态规划
